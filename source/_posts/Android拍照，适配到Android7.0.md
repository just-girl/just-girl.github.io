---
title: Android拍照适配到Android7.0
tags: ["Android"]
categories: Android
date: 2018-04-19
grammar_cjkRuby: true
copyright: ture
---

### 获得相机权限

因为需要用到camera，所以需要在manifest 文件里面声明设备 `需要安装`有camera程序。
<!-- more -->

``` xml
<manifest ... >
    <uses-feature android:name="android.hardware.camera"
                  android:required="true" />
    ...
</manifest>
```

### 获得缩略图
Anroid相机程序将通过`onActivityResult()` 返回 `Bitmap`缩略图

``` java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data){
    if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK)     {
        Bundle extras = data.getExtras();
        Bitmap imageBitmap = (Bitmap) extras.get("data");
        mImageView.setImageBitmap(imageBitmap);
    }
}
```
**注意：** 从“**data**”从获取来的缩略图只适合作为icon，不适合其他用途。

### 保存大尺寸图片
一般用户应该将通过相机拍摄的照片保存在公共外部存储目录，这样可以方便其他应用程序访问。 `getExternalStoragePublicDirectory()`方法和 `DIRECTORY_PICTURES`参数可以将公共照片保存在正确的目录，因为这种方法提供的目录是所有应用程序之间共享，各自需要`READ_EXTERNAL_STORAGE`和`WRITE_EXTERNAL_STORAGE`权限。如果允许了写的权限，那么读的权限也将被隐式允许。所以，如果你需要读写外部存取目录，你只需要声明一个权限。


``` xml
<manifest ...>
    <uses-permission
	android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    ...
</manifest>
```
然而，如果你想将照片保存到私有目录只供你的app使用，那么你可以使用`getExternalFilesDir()`方法。在Android4.3及以下，仍需要`WRITE_EXTERNAL_STORAGE` 权限；但是从Android4.4开始，就不再需要声明此权限了，因为这个目录无法被其他app访问。因此你可以这样声明权限，仅仅只在较低的版本上添加`maxSdkVersion`属性：

``` xml
<manifest ...>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
	android:maxSdkVersion="18" />
    ...
</manifest>
```
**注意：** 用`getExternalFilesDir()`和 `getFilesDir()`方法保存在目录的文件，会随着用户卸载app而一并删除。

 - 以下是谷歌提供的生成唯一文件名的一种方法

``` java
String mCurrentPhotoPath;

private File createImageFile() throws IOException {
    // Create an image file name
    String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
    String imageFileName = "JPEG_" + timeStamp + "_";
    File storageDir = getExternalFilesDir(Environment.DIRECTORY_PICTURES);
    File image = File.createTempFile(
        imageFileName,  /* prefix */
        ".jpg",         /* suffix */
        storageDir      /* directory */
    );

    // Save a file: path for use with ACTION_VIEW intents
    mCurrentPhotoPath = image.getAbsolutePath();
    return image;
}
```

 - 然后调用`Intent` 启动相机

``` java
static final int REQUEST_TAKE_PHOTO = 1;

private void dispatchTakePictureIntent() {
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    // Ensure that there's a camera activity to handle the intent
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        // Create the File where the photo should go
        File photoFile = null;
        try {
            photoFile = createImageFile();
        } catch (IOException ex) {
            // Error occurred while creating the File
            ...
        }
        // Continue only if the File was successfully created
        if (photoFile != null) {
            Uri photoURI = FileProvider.getUriForFile(this,
                                                  "com.example.android.fileprovider",
                                                  photoFile);
            takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, photoURI);
            startActivityForResult(takePictureIntent, REQUEST_TAKE_PHOTO);
        }
    }
}
```
**注意：** 从Android7.0开始，通过`getUriForFile(Context, String, File)` 返回的`content://` URI来访问数据，使用过去的`file://`URI将产生`FileUriExposedException`异常，现在一般使用`FileProvider`这种更加通用的方式来存储文件。

 - 配置`FileProvider`
 在清单文件中：
``` xml
<application>
   ...
   <provider
        android:name="android.support.v4.content.FileProvider"
        android:authorities="com.example.android.fileprovider"
        android:exported="false"
        android:grantUriPermissions="true">
        <meta-data
            android:name="android.support.FILE_PROVIDER_PATHS"
            android:resource="@xml/file_paths"></meta-data>
    </provider>
    ...
</application>
```
必须确保`android:authorities`里面的值和class文件中的 `getUriForFile(Context, String, File)` 方法的值保持一致。`exported:`要求必须为`false`，为`true`则会报安全异常。`grantUriPermissions:true`，表示授予 URI 临时访问权限。
为了指定共享的目录我们需要在资源(res)目录下创建一个xml目录，然后创建一个名为“file_paths”(名字可以随便起，只要和在manifest注册的provider所引用的resource保持一致即可)的资源文件。

``` xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path name="my_images"
	path="Android/data/com.example.package.name/files/Pictures" />
</paths>
```
上述代码中`path=""`，是有特殊意义的，它代码根目录，也就是说你可以向其它的应用共享根目录及其子目录下任何一个文件了，如果你将`path`设为`path="pictures"`， 
那么它代表着根目录下的**pictures**目录(eg:/storage/emulated/0/pictures)，如果你向其它应用分享**pictures**目录范围之外的文件是不行的。
### 将图片保存到相册
**注意：** 如果你是用`getExternalFilesDir()`方法保存的文件，那么媒体将无法扫描到，因为它只对你的app私有化。

``` java
private void galleryAddPic() 
{
    Intent mediaScanIntent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
    File f = new File(mCurrentPhotoPath);
    Uri contentUri = Uri.fromFile(f);
    mediaScanIntent.setData(contentUri);
    this.sendBroadcast(mediaScanIntent);
}
```
### 按比例缩小图片

``` java
private void setPic() 
{
    // Get the dimensions of the View
    int targetW = mImageView.getWidth();
    int targetH = mImageView.getHeight();

    // Get the dimensions of the bitmap
    BitmapFactory.Options bmOptions = new BitmapFactory.Options();
    bmOptions.inJustDecodeBounds = true;
    BitmapFactory.decodeFile(mCurrentPhotoPath, bmOptions);
    int photoW = bmOptions.outWidth;
    int photoH = bmOptions.outHeight;

    // Determine how much to scale down the image
    int scaleFactor = Math.min(photoW/targetW, photoH/targetH);

    // Decode the image file into a Bitmap sized to fill the View
    bmOptions.inJustDecodeBounds = false;
    bmOptions.inSampleSize = scaleFactor;
    bmOptions.inPurgeable = true;

    Bitmap bitmap = BitmapFactory.decodeFile(mCurrentPhotoPath, bmOptions);
    mImageView.setImageBitmap(bitmap);
}
```


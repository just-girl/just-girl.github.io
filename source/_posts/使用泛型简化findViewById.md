---
title: 使用泛型简化findViewById
tags: ["Android"]
categories: Android
date: 2018-09-26
grammar_cjkRuby: true
copyright: ture
---

### 简化findViewById

在Android开发中，通过findViewById来引用资源文件的view，但是资源文件的view过多会导致做很多重复性的工作，代码很冗余，开发效率随之下降。为了减少重复书写findViewById的次数，我们可以这么来写。　　　

 <!-- more -->

``` java
public final class ViewUtils {

    public static  <T extends View> T findViewById(Activity activity,int resId){
        return (T) activity.findViewById(resId);
    }

    public static <T extends View> T findViewById(View view,int resId){
        return (T) view.findViewById(resId);
    }
}
```

>通过使用泛型，可以避免了每次都进行类型转换，很大程度的简化了代码，开发效率也随之提升了。

 
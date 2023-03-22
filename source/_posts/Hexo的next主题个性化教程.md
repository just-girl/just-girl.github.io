---
title: Hexo的next主题个性化教程
tags: ["Hexo"]
categories: Hexo
date: 2019-02-15
grammar_cjkRuby: true
copyright: ture
---

### 设置中文

修改配置文件(_config.yml)

```shell
language: zh-Hans
```

<!-- more -->

### 酷炫的动态背景

打开`\themes\NexT\_config.yml`，修改以下内容：　　　

```shell
# 背景特效以下4项，是NexT主题集成的，只需将 false 改为 true，即可启用
# 不用向网上说的，到模板文件里加引用js的脚本，NexT已经集成了，开启即用

# Canvas-nest 浮动线条
canvas_nest:
  enable: true
  onmobile: true # display on mobile or not
  color: '0,0,255' # RGB values, use ',' to separate
  opacity: 0.5 # the opacity of line: 0~1
  zIndex: -1 # z-index property of the background
  count: 99 # the number of lines

# three_waves 水波粒子
three_waves: false
# canvas_lines 这个是一根线连接两个小点，组成的一个随鼠标放大缩小的东西
canvas_lines: false
# canvas_sphere 这个是一个很多刺组成的一个球
canvas_sphere: false
```

### 主页文章添加阴影效果

打开`themes/next/source/css/_custom/custom.styl`文件添加：

```shell
.post {
   margin-top: 60px;
   margin-bottom: 60px;
   padding: 25px;
   -webkit-box-shadow: 0 0 5px rgba(202, 203, 203, .5);
   -moz-box-shadow: 0 0 5px rgba(202, 203, 204, .5);
  }
```

### 启用：页面加载过程中顶部的进度条

打开`\themes\NexT\_config.yml`，找到字段 `pace`，设为`true`。还可以将`pace_theme:`的值，改成相应的名字,变更不同样式的加载条。

### 修改文章底部的那个带#号的标签

修改模板`/themes/next/layout/_macro/post.swig`，搜索 `rel="tag">#`，将 `#` 换成`<i class="fa fa-tag"></i>`

### 在每篇文章末尾统一添加“本文结束”标记

在路径`\themes\next\layout\_macro`中新建 `passage-end-tag.swig` 文件，并添加以下内容：

```html
<div>
    {% if not is_index %}
        <div style="text-align:center;color: #ccc;font-size:14px;">-------------本文结束<i class="fa fa-paw"></i>感谢您的阅读-------------</div>
    {% endif %}
</div>
```

接着打开`\themes\next\layout\_macro\post.swig`文件，在`post-body` 之后， `post-footer` 之前添加如下画红色部分代码（`post-footer`之前两个DIV），代码如下：
```html
<div>
  {% if not is_index %}
    {% include 'passage-end-tag.swig' %}
  {% endif %}
</div>
```

然后打开主题配置文件（_config.yml)，在末尾添加：

```shell
# 文章末尾添加“本文结束”标记
passage_end_tag:
  enabled: true
```

### 主页文章添加阴影效果

打开`\themes\next\source\css\_custom\custom.styl`，向里面加入：

```shell
// 主页文章添加阴影效果
 .post {
   margin-top: 60px;
   margin-bottom: 60px;
   padding: 25px;
   -webkit-box-shadow: 0 0 5px rgba(202, 203, 203, .5);
   -moz-box-shadow: 0 0 5px rgba(202, 203, 204, .5);
  }
```

### 设置网站的图标Favicon

一张（32*32）的ico图标，或者去别的网站下载或者制作，并将图标名称改为favicon.ico，然后把图标放在`/themes/next/source/images`里，并且修改主题配置文件：

```shell
# Put your favicon.ico into `hexo-site/source/` directory.
favicon: /favicon.ico
```

### 在文章底部增加版权信息

在目录 `next/layout/_macro/`下添加 `my-copyright.swig`：

```html
{% if page.copyright %}
<div class="my_post_copyright">
  <script src="//cdn.bootcss.com/clipboard.js/1.5.10/clipboard.min.js"></script>
  
  <!-- JS库 sweetalert 可修改路径 -->
  <script src="https://cdn.bootcss.com/jquery/2.0.0/jquery.min.js"></script>
  <script src="https://unpkg.com/sweetalert/dist/sweetalert.min.js"></script>
  <p><span>本文标题:</span><a href="{{ url_for(page.path) }}">{{ page.title }}</a></p>
  <p><span>文章作者:</span><a href="/" title="访问 {{ theme.author }} 的个人博客">{{ theme.author }}</a></p>
  <p><span>发布时间:</span>{{ page.date.format("YYYY年MM月DD日 - HH:mm") }}</p>
  <p><span>最后更新:</span>{{ page.updated.format("YYYY年MM月DD日 - HH:mm") }}</p>
  <p><span>原始链接:</span><a href="{{ url_for(page.path) }}" title="{{ page.title }}">{{ page.permalink }}</a>
    <span class="copy-path"  title="点击复制文章链接"><i class="fa fa-clipboard" data-clipboard-text="{{ page.permalink }}"  aria-label="复制成功！"></i></span>
  </p>
  <p><span>许可协议:</span><i class="fa fa-creative-commons"></i> <a rel="license" href="https://creativecommons.org/licenses/by-nc-nd/4.0/" target="_blank" title="Attribution-NonCommercial-NoDerivatives 4.0 International (CC BY-NC-ND 4.0)">署名-非商业性使用-禁止演绎 4.0 国际</a> 转载请保留原文链接及作者。</p>  
</div>
<script> 
    var clipboard = new Clipboard('.fa-clipboard');
    $(".fa-clipboard").click(function(){
      clipboard.on('success', function(){
        swal({   
          title: "",   
          text: '复制成功',
          icon: "success", 
          showConfirmButton: true
          });
	});
    });  
</script>
{% endif %}
```

在目录`next/source/css/_common/components/post/`下添加`my-post-copyright.styl`：

```shell
.my_post_copyright {
  width: 85%;
  max-width: 45em;
  margin: 2.8em auto 0;
  padding: 0.5em 1.0em;
  border: 1px solid #d3d3d3;
  font-size: 0.93rem;
  line-height: 1.6em;
  word-break: break-all;
  background: rgba(255,255,255,0.4);
}
.my_post_copyright p{margin:0;}
.my_post_copyright span {
  display: inline-block;
  width: 5.2em;
  color: #b5b5b5;
  font-weight: bold;
}
.my_post_copyright .raw {
  margin-left: 1em;
  width: 5em;
}
.my_post_copyright a {
  color: #808080;
  border-bottom:0;
}
.my_post_copyright a:hover {
  color: #a3d2a3;
  text-decoration: underline;
}
.my_post_copyright:hover .fa-clipboard {
  color: #000;
}
.my_post_copyright .post-url:hover {
  font-weight: normal;
}
.my_post_copyright .copy-path {
  margin-left: 1em;
  width: 1em;
  +mobile(){display:none;}
}
.my_post_copyright .copy-path:hover {
  color: #808080;
  cursor: pointer;
}
```

修改`next/layout/_macro/post.swig`，在代码

```shell
<div>
      {% if not is_index %}
        {% include 'wechat-subscriber.swig' %}
      {% endif %}
</div>
```

之前添加增加如下代码：

```html
<div>
      {% if not is_index %}
        {% include 'my-copyright.swig' %}
      {% endif %}
</div>
```

修改`next/source/css/_common/components/post/post.styl`文件，在最后一行增加代码：

```shell
@import "my-post-copyright"
```

保存重新生成即可。
如果要在该博文下面增加版权信息的显示，需要在 Markdown 中增加`copyright: true`的设置

如果你觉得每次都要输入`copyright: true`很麻烦的话,那么在`/scaffolds/post.md`文件中添加：

```markdown
---
title: {{ title }}
date: {{ date }}
tags:
categories: 
copyright:
---
```

这样每次hexo new "你的内容"之后，生成的md文件会自动把`copyright:`加到里面去

### 隐藏网页底部powered By Hexo / 强力驱动

打开`themes/next/layout/_partials/footer.swig`，注释或者直接删除以下代码：

```html
{% if theme.footer.powered.enable %}
  <div class="powered-by">{#
  #}{{ __('footer.powered', next_url('https://hexo.io', 'Hexo', {class: 'theme-link'})) }}{#
  #}{% if theme.footer.powered.version %} v{{ hexo_env('version') }}{% endif %}{#
 #}</div>
{% endif %}

{% if theme.footer.powered.enable and theme.footer.theme.enable %}
  <span class="post-meta-divider">|</span>
{% endif %}

{% if theme.footer.theme.enable %}
  <div class="theme-info">{#
  #}{{ __('footer.theme') }} – {{ next_url('https://theme-next.org', 'NexT.' + theme.scheme, {class: 'theme-link'}) }}{#
  #}{% if theme.footer.theme.version %} v{{ version }}{% endif %}{#
#}</div>
{% endif %}
```

### 添加站内搜索

安装`hexo-generator-searchdb`插件

```shell
npm install hexo-generator-searchdb --save
```

编辑`_config.yml`站点配置文件，新增以下内容到任意位置：

```shell
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

编辑`themes/next/_config.yml` 主题配置文件，启用本地搜索功能,将`local_search:`下面的`enable:`的值，改成`true`

```shell
# Local search
local_search:
  enable: true
```

### 首页不显示全文(只显示预览)

打开主题配置文件`/themes/next/_config.yml`，将`auto_excerpt`下面的`enable:`的值，改成`true`

```shell
# Automatically Excerpt. Not recommand.
# Please use <!-- more --> in the post to control excerpt accurately.
auto_excerpt:
enable: true
length: 150
```

除了这个方法，还有一个更灵活的方法，直接在编辑的文章中添加<!--more-->标记。这样只会显示<!--more-->标记之前的那部份。

### 添加打赏功能

打开`themes/next/_config.yml`站点配置文件，找到`# Reward`把`wechatpay:`和`alipay:`前面的`#`号删除。然后将自己的二维码文件`wechatpay.jpg`、`alipay.jpg`图片放入`themes/next/source/images`中即可。

```shell
# Reward
reward_comment: 坚持原创技术分享，您的支持将鼓励我继续创作！
wechatpay: /images/wechatpay.jpg
alipay: /images/alipay.jpg
#bitcoin: /images/bitcoin.png
```

### Hexo载入动画效果

编辑`themes/netx/_config.yml`找到`motion`,将`enable`的值，改成`true`

```shell
# Use velocity to animate everything.
motion:
  enable: true
  async: false
  transition:
    # Transition variants:
    # fadeIn | fadeOut | flipXIn | flipXOut | flipYIn | flipYOut | flipBounceXIn | flipBounceXOut | flipBounceYIn | flipBounceYOut
    # swoopIn | swoopOut | whirlIn | whirlOut | shrinkIn | shrinkOut | expandIn | expandOut
    # bounceIn | bounceOut | bounceUpIn | bounceUpOut | bounceDownIn | bounceDownOut | bounceLeftIn | bounceLeftOut | bounceRightIn | bounceRightOut
    # slideUpIn | slideUpOut | slideDownIn | slideDownOut | slideLeftIn | slideLeftOut | slideRightIn | slideRightOut
    # slideUpBigIn | slideUpBigOut | slideDownBigIn | slideDownBigOut | slideLeftBigIn | slideLeftBigOut | slideRightBigIn | slideRightBigOut
    # perspectiveUpIn | perspectiveUpOut | perspectiveDownIn | perspectiveDownOut | perspectiveLeftIn | perspectiveLeftOut | perspectiveRightIn | perspectiveRightOut
    post_block: fadeIn
    post_header: slideDownIn
    post_body: slideDownIn
    coll_header: slideLeftIn
    # Only for Pisces | Gemini.
    sidebar: slideUpIn
```

`#`号里都是载入效果，有时间可以自己多试一试！

### 美化右侧滚动条

打开`themes\next\source\css\_custom\custom.styl`文件，将下面的代码添加进去。

```shell
//设置滚动条的样式
//参考https://segmentfault.com/a/1190000003708894
::-webkit-scrollbar {
      width: 5px;
      height: 5px;

}
//滚动槽
::-webkit-scrollbar-track {
      background: #eee;

}
//滚动条滑块
::-webkit-scrollbar-thumb {
      border-radius: 5px;
        background-color: #ccc;

}
::-webkit-scrollbar-thumb:hover {
      background-color: rgb(247, 149, 51);

}
```

### 优化鼠标选择文字的样式

打开`themes\next\source\css\_custom\custom.styl`文件，将下面的代码添加进去。

```shell
::selection {
      background-color: rgb(255, 241, 89);
        color: #555;

}
```

### 网页中文乱码解决办法

所有配置文件都可以用记事本打开，如果输入中文后网页显示乱码，则需要将该文件另存为`utf-8`编码文件

即打开后点击左上角 文件—>另存为，然后在弹出的窗口下方有一个 编码 的下拉菜单，点击选择`UTF-8`选项，然后点击保存，会提示 是否替换，选择 是 即可。以后出现中文乱码问题都这样解决

### 设置侧边栏头像

将头像图片放在主题目录的`source/images/`目录下，尺寸是正方形即可，然后将照片改名为avatar.jpg，或者其他后缀名的图片。
编辑 `主题配置文件`，找到关键字avatar，删掉前面的#号，值设置成头像的链接地址，比如

```shell
avatar: /images/avatar.jpg
```

保存后可用hexo s打开本地服务器预览一下效果

### 修改链接样式

在`themes\next\source\css\_common\components\post\post.styl`在末尾添加如下css样式：

```shell
// 文章内链接文本样式
.post-body p a{
  color: #0593d3;
  border-bottom: none;
  border-bottom: 1px solid #0593d3;
  &:hover {
    color: #fc6423;
    border-bottom: none;
    border-bottom: 1px solid #fc6423;
  }
}
```


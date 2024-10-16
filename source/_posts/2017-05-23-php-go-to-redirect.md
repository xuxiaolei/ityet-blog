---
layout: post
title: PHP给网站添加外链go跳转效果
categories: PHP
description: PHP给网站添加外链go跳转效果
index_img: 
date: 2017-05-23 09:09:09
tags: [PHP,WORDPRESS]
---
> 给网站添加外链go跳转效果既美观又可避免权重流失，确实是一个不错的功能。有的主题自带go跳转，但是别的主题不一定有，所以想要移植这个功能，其实也还蛮简单的，这里简单分享下这个实现方法。

#### 代码方法

**①、新建一个go.php文件，放置到[wordpress]的根目录下，在go.php里面输入以下代码**  
这是知更鸟的go.php代码
```
<?php   
 
$t_url = preg_replace('/^url=(.*)$/i','$1',$_SERVER["QUERY_STRING"]);  
if(!emptyempty($t_url)) {  
    preg_match('/(http|https):\/\//',$t_url,$matches);  
    if($matches){  
        $url=$t_url;  
        $title='页面加载中,请稍候...';  
    } else {  
        preg_match('/\./i',$t_url,$matche);  
        if($matche){  
            $url='http://'.$t_url;  
            $title='页面加载中,请稍候...';  
        } else {  
            $url='http://zhangge.net/';  
            $title='参数错误，正在返回首页...';  
        }  
    }  
} else {  
    $title='参数缺失，正在返回首页...';  
    $url='http://ityet.com/';  
}  
?>  
<html>  
<head>  
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">  
<meta http-equiv="refresh" content="1;url='<?php echo $url;?>';">  
<title><?php echo $title;?></title>  
<style>  
body{background:#000}.loading{-webkit-animation:fadein 2s;-moz-animation:fadein 2s;-o-animation:fadein 2s;animation:fadein 2s}@-moz-keyframes fadein{from{opacity:0}to{opacity:1}}@-webkit-keyframes fadein{from{opacity:0}to{opacity:1}}@-o-keyframes fadein{from{opacity:0}to{opacity:1}}@keyframes fadein{from{opacity:0}to{opacity:1}}.spinner-wrapper{position:absolute;top:0;left:0;z-index:300;height:100%;min-width:100%;min-height:100%;background:rgba(255,255,255,0.93)}.spinner-text{position:absolute;top:50%;left:50%;margin-left:-90px;margin-top: 2px;color:#BBB;letter-spacing:1px;font-weight:700;font-size:36px;font-family:Arial}.spinner{position:absolute;top:50%;left:50%;display:block;margin-left:-160px;width:1px;height:1px;border:25px solid rgba(100,100,100,0.2);-webkit-border-radius:50px;-moz-border-radius:50px;border-radius:50px;border-left-color:transparent;border-right-color:transparent;-webkit-animation:spin 1.5s infinite;-moz-animation:spin 1.5s infinite;animation:spin 1.5s infinite}@-webkit-keyframes spin{0%,100%{-webkit-transform:rotate(0deg) scale(1)}50%{-webkit-transform:rotate(720deg) scale(0.6)}}@-moz-keyframes spin{0%,100%{-moz-transform:rotate(0deg) scale(1)}50%{-moz-transform:rotate(720deg) scale(0.6)}}@-o-keyframes spin{0%,100%{-o-transform:rotate(0deg) scale(1)}50%{-o-transform:rotate(720deg) scale(0.6)}}@keyframes spin{0%,100%{transform:rotate(0deg) scale(1)}50%{transform:rotate(720deg) scale(0.6)}}  
</style>  
</head>  
<body>  
<div class="loading">  
  <div class="spinner-wrapper">  
    <span class="spinner-text">页面加载中,请稍候...</span>  
    <span class="spinner"></span>  
  </div>  
</div>  
</body>  
</html>  
```
### 使用方法

**②、保存，则外链跳转形式为：{本站地址}/go.php?url={外链地址}  
使用:在添加外链的时候，只要给外链加上统一的跳转前缀：http://网站地址/go.php? url=**即可。
* * *

外链方法的使用是需要在手动添加外链的，同时跳转样式可以自行修改。

### 效果参考

点击查看:[https://nas.itmax.cn:4435/go/?url=http://www.baidu.com/](https://nas.itmax.cn:4435/go/?url=http://www.baidu.com)

#### 插件实现

有些插件也能实现这个外链跳转
1\. Simple URLs插件  
设置简单，只需要要后台设置好后缀和目标页面即可  
2\. Link-Hopper插件  
跳转链接的基础地址可以随意设置  
3\. Pretty Link Lite插件  
后台功能十分强大的，还有Pro版本，不过要money的。用它完全可以做一个短网址，像t.cn和bit.ly一样。  
4\. Affiliate Link Cloaking插件  
推广链接转换工具，用来推广你的淘宝客等要隐藏的链接，可以使用它，附带统计功能。  
5\. WP No External Links插件  
这个一款可以自动将博客内的外部链接转成内部链接，如http://www.baidu.com，则显示为http://www.ityet.com/goto/http://www.baidu.com，可以尝试使用这个插件防权重丢失的。  
6\. Go Codes插件  
和前面的外链跳转链接插件设置一样简单，带统计功能。

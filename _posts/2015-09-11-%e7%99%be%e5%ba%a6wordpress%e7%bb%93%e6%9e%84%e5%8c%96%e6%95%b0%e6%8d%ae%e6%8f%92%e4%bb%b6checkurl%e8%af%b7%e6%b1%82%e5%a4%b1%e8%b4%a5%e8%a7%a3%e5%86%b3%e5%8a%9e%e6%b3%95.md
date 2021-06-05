---
id: 238
title: 百度WordPress结构化数据插件checkurl请求失败解决办法
date: 2015-09-11T22:05:18+00:00
author: Tango
layout: post
guid: http://www.wtango.com/?p=238
permalink: '/%e7%99%be%e5%ba%a6wordpress%e7%bb%93%e6%9e%84%e5%8c%96%e6%95%b0%e6%8d%ae%e6%8f%92%e4%bb%b6checkurl%e8%af%b7%e6%b1%82%e5%a4%b1%e8%b4%a5%e8%a7%a3%e5%86%b3%e5%8a%9e%e6%b3%95/'
tags:
  - wordpress
---

在安装百度Wordpress结构化数据插件后，需要进行站点校验。在校验的时候会出现<span style="color: #ff0000;">“checkurl请求失败”<span style="color: #000000;">错误，在百度搜索了一番没有找到答案，大多数都不是问题的关键。</span></span>

查看了校验部分的代码之后，发现了有两个地方有比较大的嫌疑：

<!--more--> 在插件目录下的checksign.php文件中：

```php
if (!function_exists('add_action')) {
    require_once dirname(__FILE__) . DIRECTORY_SEPARATOR . './../../../wp-config.php';
```

在插件目录下的sitemap.php文件中：

```php
<if (!function_exists('add_action')) {
    require_once dirname(__FILE__) . DIRECTORY_SEPARATOR . './../../../wp-config.php';
```

我使用的openshift服务器，实际上http根目录是链接到另一个路径的。在baidusubmit插件目录下

```
ls ./../../../
blogs.dir  current  languages  plugins	themes	uploads
[******.rhcloud.com data]\> 
```

发现到了data目录下了，data目录下的current目录才是我们真正存放WordPress程序的地方。要说为什么我们在baidusubmit目录下向上退三层到了这个data目录下，是因为plugin目录又是一个链接，所以向上退3层的时候没有退到正确的地方，所以没有找到wp-config.php文件。

我使用的是openshift的服务器，才会出现这个问题，所以我将上面两行代码改为：

```php
if (!function_exists('add_action')) {
    require_once dirname(__FILE__) . DIRECTORY_SEPARATOR . './../../current/wp-config.php';
```

让百度结构化数据插件能正确找到wp-config.php文件，就不会出现<span style="color: #ff0000;">&#8220;checkurl 请求错误&#8221;</span>这个问题了。

如果你也遇到这个问题，可以检查一下是否因为百度结构化数据插件因为路径不正确找不到wp-config.php文件造成的，修改上述两处代码让插件能正确找到wp-config.php文件就能解决问题。
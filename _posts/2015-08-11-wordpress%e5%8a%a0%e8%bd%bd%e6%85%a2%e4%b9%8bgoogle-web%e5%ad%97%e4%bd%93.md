---
id: 98
title: 'WordPress加载慢(1)&#8211;Google Web字体'
date: 2015-08-11T21:55:18+00:00
author: Tango
layout: post
guid: http://wtango.com:8888/?p=98
permalink: '/wordpress%e5%8a%a0%e8%bd%bd%e6%85%a2%e4%b9%8bgoogle-web%e5%ad%97%e4%bd%93/'
categories:
  - wordpress
tags:
  - WordPress
---
在新版的 WordPress 中，为了后台的美观度，开发者在页面上加入了 Google Web 字体，这本来会让英文显示更加精美。但在国内，由于 googleapis.com 等域名常年抽风(你懂的)，直接导致的结果就是后台经常打开一点就卡住打不开了，加载极为缓慢，其实我们只要移除 Google 在线字体即可恢复原来的速度。

<!--more-->

##### 在你的主题的 function.php 顶部加入以下代码即可：

<pre class="brush: php; title: ; notranslate" title="">add_filter('gettext_with_context', 'disable_open_sans', 888, 4 );
function disable_open_sans( $translations, $text, $context, $domain )
{
          if ( 'Open Sans font: on or off' == $context && 'on' == $text ) {
                     $translations = 'off';
          }
          return $translations;
}
</pre>

文章转载自：http://www.iplaysoft.com/item/2019
---
id: 44
title: 使用shell命令去除文件中的BOM
date: 2015-08-08T11:36:13+00:00
author: Tango
layout: post
guid: http://tango.wicp.net:8888/?p=44
permalink: '/%e4%bd%bf%e7%94%a8shell%e5%91%bd%e4%bb%a4%e5%8e%bb%e9%99%a4%e6%96%87%e4%bb%b6%e4%b8%ad%e7%9a%84bom/'
categories:
  - Linux
---
<span style="color: #ff0000;">使用前请先备份文件</span>,因为会直接修改原文件。

由于使用一些编辑器会导致文件中有BOM，在Linux下使用VIM可以看到<feff>这样的标签，查看其字节组成可以找出其16进制表示为EF BB BF。可以使用bash命令来去除BOM：

<!--more-->

<pre>grep -r -I -l $'^\xEF\xBB\xBF' . | xargs sed -i 's/^\xEF\xBB\xBF//g'</pre>

这个命令将从当前名录开始递归检索，遇到 EF BB BF 会将其删除，并将修改写到原文件中。如果不要使用递归就把 grep -r 选项去掉。还可以修改开始检索的名录，把&#8217;.&#8217;号改成要检索的目录即可。目录和子目录都不能有空格，因为xargs命令默认使用空格来分割参数。
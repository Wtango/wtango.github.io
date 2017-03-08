---
id: 47
title: Ubuntu下nginx,mysql,php与wordpress的安装与配置
date: 2015-08-08T13:46:47+00:00
author: Tango
layout: post
guid: http://tango.wicp.net:8888/?p=47
permalink: '/ubuntu%e4%b8%8bnginxmysql-php-wordpress%e7%9a%84%e5%ae%89%e8%a3%85%e4%b8%8e%e9%85%8d%e7%bd%ae/'
categories:
  - Linux
tags:
  - Linux
---
<div id="toc" class="toc-article">
  <p>
    <strong class="toc-title">文章目录</strong>
  </p>
  
  <ol>
    <li>
      <a class="toc-link" href="#关于Wordpress"><span class="toc-text">关于Wordpress</span></a>
    </li>
    <li>
      <a class="toc-link" href="#前提"><span class="toc-text">前提</span></a>
    </li>
    <li>
      <a class="toc-link" href="#下载Wordpress"><span class="toc-text">下载Wordpress</span></a>
    </li>
    <li>
      <a class="toc-link" href="#创建Wordpress数据库和用户"><span class="toc-text">创建Wordpress数据库和用户</span></a>
    </li>
    <li>
      <a class="toc-link" href="#配置Wordpress"><span class="toc-text">配置Wordpress</span></a>
    </li>
    <li>
      <a class="toc-link" href="#配置nginx"><span class="toc-text">配置nginx</span></a>
    </li>
    <li>
      <a class="toc-link" href="#执行Wordpress安装"><span class="toc-text">执行Wordpress安装</span></a>
    </li>
    <li>
      <a class="toc-link" href="#参考文章"><span class="toc-text">参考文章</span></a>
    </li>
  </ol>
</div>

本文介绍如何在Ubuntu系统下在nginx中部署wordpress。包括的内容有wordpress下载及配置，MySQL数据库及用户的创建，nginx虚拟主机的配置等内容。

<!--more-->

## 关于Wordpress {#关于Wordpress}

> WordPress是一个注重美学、易用性和网络标准的个人信息发布平台。WordPress虽为免费的开源软件，但其价值无法用金钱来衡量。 使用WordPress可以搭建功能强大的网络信息发布平台，但更多的是应用于个性化的博客。针对博客的应用，WordPress能让您省却对后台技术的担心，集中精力做好网站的内容。

&nbsp;

## 前提 {#前提}

在继续本教程之前，需要在服务器上面配置好PHP环境。

<pre class="brush: bash; title: ; notranslate" title="">#安装nginx http服务器
 apt-get install nginx
 #安装php-fpm
 apt-get install php5-fpm
 #安装mysql数据库
 apt-get install mysql-server
 #安装php5-mysql
 apt-get install php5-mysql
</pre>

&nbsp;

## 下载Wordpress {#下载Wordpress}<figure class="highlight shell">

<code lang="shell">wget http://wordpress.org/latest.tar.gz</code></figure> 

解压文件包。假设解压至用户的主目录中。<figure class="highlight shell"> 

<pre class="brush: bash; title: ; notranslate" title="">tar -xzvf latest.tar.gz</pre></figure> 

## 创建Wordpress数据库和用户 {#创建Wordpress数据库和用户}

登录MySQL<figure class="highlight shell"> 

<div class="line">
  <pre class="brush: bash; title: ; notranslate" title="">mysql -u root -p</pre>
</div></figure> 

创建数据库：<figure class="highlight sql"> 

<pre class="brush: bash; title: ; notranslate" title="">CREATE DATABASE wordpress;
</pre></figure> 

创建MySQL用户：<figure class="highlight sql"> 

<div class="line">
  <pre class="brush: bash; title: ; notranslate" title="">CREATE USER wordpressuser@localhost;&lt;/pre&gt;
</pre>
</div></figure> 

设置密码：<figure class="highlight sql">[coe lang=&#8221;shell&#8221;]SET PASSWORD FOR wordpressuser@localhost= PASSWORD(&#8220;password&#8221;);[/code]</figure> 

配置权限：<figure class="highlight sql"> 

<div class="line">
  <pre class="brush: bash; title: ; notranslate" title="">GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser@localhost IDENTIFIED BY 'password';
</pre>
</div>

<pre class="brush: bash; title: ; notranslate" title="">FLUSH PRIVILEGES;</pre></figure> 

## 配置Wordpress {#配置Wordpress}

拷贝配置示例文件。注意，需要根据文件的存放路径来修改命令：<figure class="highlight shell"> 

<div class="line">
  <pre class="brush: bash; title: ; notranslate" title="">cp ~/wordpress/wp-config-sample.php ~/wordpress/wp-config.php</pre>
</div></figure> 

编辑配置文件：<figure class="highlight shell"> 

<pre class="brush: bash; title: ; notranslate" title="">sudo vim ~/wordpress/wp-config.php</pre></figure> 

修改下面的选项：<figure class="highlight php"> 

<pre class="brush: bash; title: ; notranslate" title="">// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'wordpressuser');

/** MySQL database password */
define('DB_PASSWORD', 'password');</pre></figure> 

拷贝文件到网站根目录下 一般将/var/www设置为网站的根目录。<figure class="highlight shell"> 

<div class="line">
  <pre class="brush: bash; title: ; notranslate" title="">sudo mkdir -p /var/www</pre>
</div></figure> 

拷贝文件：<figure class="highlight shell"> 

<div class="line">
  <pre class="brush: bash; title: ; notranslate" title="">sudo cp -r ~/wordpress/* /var/www</pre>
</div></figure> 

修复权限：<figure class="highlight shell"> 

<pre class="brush: bash; title: ; notranslate" title="">sudo chown -R  www-data:www-data /var/www</pre></figure> 

## 配置nginx {#配置nginx}

现在需要设置nginx虚拟主机了。可以使用默认的配置，或者重新拷贝一份。<figure class="highlight shell"> 

<pre class="brush: bash; title: ; notranslate" title="">sudo vi /etc/nginx/sites-available/default</pre></figure> 

将server配置改为如下：

<pre class="brush: bash; title: ; notranslate" title="">server {
        listen   80;
        root /var/www;
        index index.php index.html index.htm;
        server_name 192.34.59.214;
        location / {
                try_files $uri $uri/ /index.php?q=$uri&amp;$args;
        }
        error_page 404 /404.html;
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
              root /usr/share/nginx/www;
        }
        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        location ~ \.php$ {
                try_files $uri =404;
                #fastcgi_pass 127.0.0.1:9000;
                # With php5-fpm:
                fastcgi_pass unix:/var/run/php5-fpm.sock;
                fastcgi_index index.php;
                include fastcgi_params;
                 }
        }</pre>

重启nginx和php-fpm：

<pre class="brush: bash; title: ; notranslate" title="">sudo service nginx restart
sudo service php5-fpm restart</pre>

## 执行Wordpress安装 {#执行Wordpress安装}

在浏览器中访问 `http://127.0.0.1/wp-admin/install.php` 执行安装。

## 参考文章 {#参考文章}

<a title="How To Install WordPress with nginx on Ubuntu 12.04" href="https://www.digitalocean.com/community/articles/how-to-install-wordpress-with-nginx-on-ubuntu-12-04" target="_blank" rel="external">How To Install WordPress with nginx on Ubuntu 12.04</a>
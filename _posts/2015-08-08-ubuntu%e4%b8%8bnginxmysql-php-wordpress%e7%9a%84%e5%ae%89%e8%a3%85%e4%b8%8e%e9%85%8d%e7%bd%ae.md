---
id: 47
title: Ubuntu下nginx,mysql,php与wordpress的安装与配置
date: 2015-08-08T13:46:47+00:00
author: Tango
layout: post
guid: http://tango.wicp.net:8888/?p=47
permalink: '/ubuntu%e4%b8%8bnginxmysql-php-wordpress%e7%9a%84%e5%ae%89%e8%a3%85%e4%b8%8e%e9%85%8d%e7%bd%ae/'
tags:
  - Linux
---

- [关于Wordpress](#关于wordpress)
- [前提](#前提)
- [下载Wordpress](#下载wordpress)
- [创建Wordpress数据库和用户](#创建wordpress数据库和用户)
- [配置Wordpress](#配置wordpress)
- [配置nginx](#配置nginx)
- [执行Wordpress安装](#执行wordpress安装)
- [参考文章 {#参考文章}](#参考文章-参考文章)

本文介绍如何在Ubuntu系统下在nginx中部署wordpress。包括的内容有wordpress下载及配置，MySQL数据库及用户的创建，nginx虚拟主机的配置等内容。

<!--more-->

## 关于Wordpress

> WordPress是一个注重美学、易用性和网络标准的个人信息发布平台。WordPress虽为免费的开源软件，但其价值无法用金钱来衡量。 使用WordPress可以搭建功能强大的网络信息发布平台，但更多的是应用于个性化的博客。针对博客的应用，WordPress能让您省却对后台技术的担心，集中精力做好网站的内容。

## 前提

在继续本教程之前，需要在服务器上面配置好PHP环境。

```shell
 # 安装nginx http服务器
 apt-get install nginx
 # 安装php-fpm
 apt-get install php5-fpm
 # 安装mysql数据库
 apt-get install mysql-server
 # 安装php5-mysql
 apt-get install php5-mysql
```

## 下载Wordpress

下载最新WordPress包  
`wget http://wordpress.org/latest.tar.gz`

解压文件包。假设解压至用户的主目录中。  
`tar -xzvf latest.tar`

## 创建Wordpress数据库和用户

登录MySQL   
`mysql -u root -p`

创建数据库:  
`CREATE DATABASE wordpress;`

创建MySQL用户  
`CREATE USER wordpressuser@localhost;`

设置密码：  
`SET PASSWORD FOR wordpressuser@localhost= PASSWORD;`

配置权限:  
`GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser@localhost IDENTIFIED BY 'password';`  
`FLUSH PRIVILEGES;`

## 配置Wordpress

拷贝配置示例文件。注意，需要根据文件的存放路径来修改命令:  
`cp ~/wordpress/wp-config-sample.php ~/wordpress/wp-config.php`

编辑配置文件:  
`sudo vim ~/wordpress/wp-config.php`

修改下面的选项:

```php
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'wordpressuser');

/** MySQL database password */
define('DB_PASSWORD', 'password');
```

拷贝文件到网站根目录下 一般将/var/www设置为网站的根目录。  
`sudo mkdir -p /var/www`

拷贝文件：  
`sudo cp -r ~/wordpress/* /var/www`

修复权限：  
`sudo chown -R  www-data:www-data /var/www`

## 配置nginx

现在需要设置nginx虚拟主机了。可以使用默认的配置，或者重新拷贝一份。  
`sudo vi /etc/nginx/sites-available/default`

将server配置改为如下：

```nginx
server {
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
        }
```

重启nginx和php-fpm：

```shell
sudo service nginx restart
sudo service php5-fpm restart
```

## 执行Wordpress安装

在浏览器中访问 `http://127.0.0.1/wp-admin/install.php` 执行安装。

## 参考文章 {#参考文章}

<a title="How To Install WordPress with nginx on Ubuntu 12.04" href="https://www.digitalocean.com/community/articles/how-to-install-wordpress-with-nginx-on-ubuntu-12-04" target="_blank" rel="external">How To Install WordPress with nginx on Ubuntu 12.04</a>
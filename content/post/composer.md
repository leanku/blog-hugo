---
title: "composer"
date: 2022-11-15
draft: false
categories: ["composer"]
tags: ["composer"]
keywords: ["composer"]
---

# [composer](https://www.phpcomposer.com/)
> Composer是 PHP 用来管理依赖（dependency）关系的工具。你可以在自己的项目中声明所依赖的外部工具库（libraries），Composer 会帮你安装这些依赖的库文件。

1.[安装composer](https://pkg.xyz/#how-to-install-composer)

2.常用命令

3.文件说明

[packagist仓库](https://packagist.org/)

<br>






## 安装composer
```
#下载安装脚本 － composer-setup.php － 到当前目录。
php -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');"

#执行安装过程。
php composer-setup.php

#删除安装脚本。
php -r "unlink('composer-setup.php');"

#全局安装
sudo mv composer.phar /usr/local/bin/composer

#启用本镜像服务
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```
<br>

## composer常用命令
|命令|描述|
|:--|:--|
|composer list |获取帮助|
|composer init |以交互方式填写composer.json文信息|
|composer install |读取当前目录composer.json文件，处理依赖关系并安装到vendor目录|
|composer update |获取依赖的最新版本，升级composer.lock文件|
|composer require |添加新的依赖到composer.json文件 并执行更新|
|composer search |在当前项目中搜索依赖包|
|composer show |列举所有可用的资源包|
|composer validate |检测composer.json文件是否有效|
|composer lisself-update |将composer工具升级到最新版本|
|composer create-project |基于composer创建一个新的项目|
<br>

## composer.json常用项说明

|键名|说明|
|:--|:--|
|name |包名称,以/分割，斜杠前代表所有者，后代表项目名|
|description|项目介绍|
|license|声明相关、许可证|
|authors|作者，可以是多个|
|require |版本信息的指定，1.标准版本如0.0.1；2.范围版本如>,>=,<,<=,!=；3.通配符 如1.0.*相当于>=1.0且<1.1版本即可；4.下一个重要版本如~1.2.3相当于>=1.2.3且<1.3|
|minimun-stability |告诉composer当前开发的项目依赖要求的包的全局稳定性级别 包括 dev,alpha,bate,PC,stable, stab为默认值|

## composer.lock
composer.lock文件会根据composer.json的内容自动生成，和composer.json在同一目录下。即在安装完所有需要的包之后composer会在composer.lock文件中生成一张标准的包版本的文件，这将锁定所有包的版本。
当使用composer安装包时，会优先从composer.lock文件读取依赖版本，再根据compsoer.json文件去获取依赖，这就确保了该库的每个使用者都能得到相同的依赖版本，这对团队开发来讲非常重要。
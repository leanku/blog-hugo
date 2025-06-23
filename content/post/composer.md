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

## 一、Composer 基础概念
Composer 是 PHP 的依赖管理工具，它通过定义项目所需的依赖包及版本，自动安装和管理这些依赖，确保项目在不同环境下都能稳定运行。其核心概念主要包括以下几个方面：

### 1.1 包（Package）
包是 Composer 管理的基本单元，它可以是一个 PHP 类库、框架组件，或是某个功能模块。比如用于 HTTP 请求处理的 Guzzle，以及知名的 PHP 框架 Laravel、Symfony 等，都可以看作是 Composer 包。这些包被发布在 Packagist 等包仓库中，方便开发者获取使用。

### 1.2 composer.json 与 composer.lock
composer.json是项目的依赖配置文件，开发者在其中定义项目所需的依赖包及其版本约束。例如，若要在项目中使用 PHPUnit 进行单元测试，可在composer.json添加 "phpunit/phpunit": "^9.5" ，表示使用 PHPUnit 9.5 及以上版本，但低于 10.0 版本。​

composer.lock则是锁定依赖包具体版本的文件。当执行composer install或composer update时，Composer 会将实际安装的依赖包版本记录在composer.lock中。下次在其他环境安装依赖时，该文件能确保安装的依赖包版本与之前完全一致，避免因版本差异导致项目出现兼容性问题 。

### 1.3 包仓库（Repository）​
包仓库是存储 Composer 包的地方，默认情况下，Composer 使用 Packagist 作为官方包仓库，它汇聚了海量的 PHP 开源包。除了 Packagist，开发者还可以搭建私有包仓库，用于管理内部开发的私有包，或者使用其他公共仓库，拓展包的来源。
[packagist仓库地址](https://packagist.org/)

## 二、[安装composer](https://pkg.xyz/#how-to-install-composer)
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


## 三、 Composer常用命令与操作
### 3.1 初始化项目
在新建 PHP 项目时，首先需要初始化 Composer。在项目根目录下打开命令行，执行composer init命令，此时会进入交互式配置界面，系统会依次询问项目名称、描述、作者、许可证等信息。完成信息填写后，会在项目根目录生成composer.json文件。如果不想进行交互式配置，也可以使用composer init -n命令，采用默认配置快速生成composer.json文件。

### 3.2 安装依赖包
安装依赖包是 Composer 最常用的操作，主要有以下两种方式：​

安装指定依赖包：若要安装某个具体的依赖包，如安装用于处理 JSON 数据的league/fractal包，执行```composer require league/fractal``` 。Composer 会自动从包仓库下载该包，并将其添加到composer.json的require字段中，同时在composer.lock记录实际安装的版本。如果要安装特定版本的包，可使用composer require league/fractal:^1.0格式指定版本约束。​

安装项目所有依赖：当获取到已有项目代码，且项目根目录下存在composer.json和composer.lock文件时，执行composer install命令，Composer 会根据composer.lock文件中记录的版本，安装项目所需的所有依赖包。如果没有composer.lock文件，则会根据composer.json中的版本约束，安装符合条件的最新版本依赖包。

### 3.3 更新依赖包
随着依赖包的更新迭代，项目中的依赖包也需要适时更新。更新依赖包可使用composer update命令，该命令会根据composer.json中的版本约束，将所有依赖包更新到符合条件的最新版本，并更新composer.lock文件。如果只想更新某个特定的依赖包，可使用```composer update package/name``` ，将package/name替换为具体的包名称。

### 3.4 移除依赖包
当项目不再需要某个依赖包时，可执行```composer remove package/name```命令将其移除。该命令会从composer.json的require字段中删除对应的依赖包配置，并删除项目中已安装的该包文件，同时更新composer.lock 。

### 3.5 命令列表
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

## 四、composer.json
### 4.1 常用项说明
|键名|说明|
|:--|:--|
|name |包名称,以/分割，斜杠前代表所有者，后代表项目名|
|description|项目介绍|
|license|声明相关、许可证|
|authors|作者，可以是多个|
|require |版本信息的指定，1.标准版本如0.0.1；2.范围版本如>,>=,<,<=,!=；3.通配符 如1.0.*相当于>=1.0且<1.1版本即可；4.下一个重要版本如~1.2.3相当于>=1.2.3且<1.3|
|minimun-stability |告诉composer当前开发的项目依赖要求的包的全局稳定性级别 包括 dev,alpha,bate,PC,stable, stab为默认值|

### 4.2 版本通配符
在composer.json中，版本通配符用于灵活指定依赖包的版本范围，确保项目既能获取到更新功能，又能避免因版本变化过大导致的兼容性问题。下面为你详细介绍常见的版本通配符及其含义：
1. ^（ caret ）：该符号表示允许更新到不改变主版本号的最新版本。例如，^1.2.3 表示允许更新到 1.x.x 的最新版本，即可以更新到 1.2.4、1.3.0 等，但不会自动更新到 2.0.0 及以上版本。这是因为主版本号的变化通常意味着有不兼容的重大改动。在使用框架或大型类库时，^ 符号很常用，它能在保证项目稳定性的前提下，获取功能改进和 bug 修复。
2. ~（ tilde ）：它表示允许更新到不改变次版本号的最新版本。比如 ~1.2.3，意味着可以更新到 1.2.4、1.2.5 等，但不会更新到 1.3.0 及更高版本 。相比 ^，~ 的限制更严格，适用于对版本稳定性要求更高的场景，例如在生产环境中，希望依赖包的变动尽可能小，以降低引入新问题的风险。
3. *（ asterisk ）：此符号代表允许使用任何版本，即会安装满足依赖条件的最新版本。例如 * 或 1.*，前者会安装仓库中可用的最新版本，后者会安装 1 系列的最新版本 。虽然这种方式能确保获取到最新功能，但由于版本变动不可控，很容易因版本不兼容导致项目出现问题，因此在实际项目中，特别是生产环境，应谨慎使用 * 通配符。
4. **>、<、>=、<=**：这些比较运算符可以用来指定明确的版本范围。>1.0.0 表示大于 1.0.0 的所有版本；<=2.0.0 表示小于等于 2.0.0 的版本；>=1.2.0 <2.0.0 则表示大于等于 1.2.0 且小于 2.0.0 的版本 。使用比较运算符能够根据项目需求，精确地控制依赖包的版本区间，不过在依赖包更新频繁时，可能需要更频繁地调整版本范围设置。
5. x（ 或 X ）：x 用作通配符时，代表该位置可以是任意版本号。例如 1.x 等同于 1.0.*，表示 1 主版本下的任意次版本和修订版本；1.2.x 则表示 1.2 次版本下的任意修订版本 。它常用于在希望保持一定版本稳定性的同时，允许较小的版本更新。

## 五、 Composer 高级应用与技巧
### 5.1 自定义脚本（Scripts）
Composer 支持定义自定义脚本，在执行特定命令或事件时自动运行。例如，在项目部署时，可能需要自动生成配置文件、清除缓存等操作。可以在composer.json的scripts字段中定义脚本，如下所示：
```json
{
    "scripts": {
        "post-install-cmd": [
            "php artisan key:generate",
            "php artisan cache:clear"
        ]
    }
}
```
上述配置表示在执行composer install命令完成后，会自动执行php artisan key:generate和php artisan cache:clear两条命令。

### 5.2 管理开发依赖（dev-require）
项目中有些依赖包仅用于开发阶段，如单元测试框架 PHPUnit、代码静态分析工具 PHPStan 等，这些依赖包在生产环境中并不需要。可以将这类依赖包添加到composer.json的dev-require字段中，执行composer require --dev package/name命令安装。这样在生产环境部署时，使用composer install --no-dev命令安装依赖，就不会安装dev-require字段中的包，减小项目部署包的体积 。

### 5.3 私有包管理
对于企业内部开发的私有包，可以搭建私有包仓库进行管理。常用的私有包仓库管理工具如 Satis、Artifactory 等。以 Satis 为例，通过配置 Satis 生成静态的包仓库，然后在项目的composer.json中添加私有仓库配置：
```json
{
    "repositories": [
        {
            "type": "composer",
            "url": "https://your-private-repo-url"
        }
    ],
    "require": {
        "your-private-package/name": "^1.0"
    }
}
```
这样就可以像使用公共包一样，在项目中安装和管理私有包。

### 5.4 自定义安装路径
默认情况下，Composer 会将依赖包安装到项目的vendor目录下，但在一些特殊场景中，你可能希望自定义安装路径。例如，要将依赖包安装到libs目录，可以在composer.json中添加如下配置：
```json
{
    "config": {
        "vendor-dir": "libs"
    }
}
```
之后执行composer install或composer update命令时，依赖包就会被安装到指定的libs目录中。

### 5.5 自动发现命名空间
Composer 支持自动发现 PHP 代码的命名空间，这对于大型项目或多包管理非常有用。在composer.json的autoload字段中，可以定义不同类型的自动加载规则，例如
```json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```
上述配置表示，当代码中使用App命名空间时，Composer 会自动在src目录下查找对应的 PHP 文件。定义好自动加载规则后，执行composer dump-autoload命令，Composer 会生成自动加载文件，从而实现命名空间与文件路径的自动映射，提高代码的组织性和可维护性。

### 5.6 使用脚本钩子与助手函数
Composer 的脚本除了能执行系统命令，还可以编写 PHP 脚本，并借助助手函数实现更复杂的逻辑。例如，在项目中可能需要在安装依赖后，自动生成一个配置文件，其中包含项目的一些基本信息。可以在composer.json中定义如下脚本：
```json
{
    "scripts": {
        "post-install-cmd": [
            "PhpScripts\\ComposerScripts::generateConfigFile"
        ]
    },
    "autoload": {
        "psr-4": {
            "PhpScripts\\": "scripts/"
        }
    }
}
```
在scripts/ComposerScripts.php文件中编写具体的 PHP 函数：
```php
<?php

namespace PhpScripts;

class ComposerScripts
{
    public static function generateConfigFile()
    {
        $config = [
            "project_name" => "My Project",
            "version" => "1.0.0",
            "author" => "John Doe"
        ];
        file_put_contents('config/config.php', "<?php return " . var_export($config, true) . ";");
        echo "配置文件已生成！\n";
    }
}
```
这里通过定义generateConfigFile函数，使用 PHP 的文件操作函数生成了一个配置文件。在这个过程中，借助了 PHP 本身的函数作为 “助手函数” 来完成文件写入等操作，使得 Composer 在安装依赖后能够自动执行复杂的任务，增强了项目管理的自动化程度。
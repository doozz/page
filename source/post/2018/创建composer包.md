```toml
title = "创建composer包"
slug = "创建composer包"
desc = "创建composer包"
date = "2018-11-12 21:07:11"
update_date = "2018-11-12 21:07:11"
author = "doozz"
thumb = ""
tags = ["php","composer"]
```

composer作为php的好基友，学习php一定绕不过composer,那么如何创建一个composer呢？

#### Composer 安装

直接[官网](https://docs.phpcomposer.com/00-intro.html)

#### 创建composer包
```shell
- src
--|-- v1
- composer.json 
```

1.在[github](https://github.com/)上创建一个项目,然后把项目clone到本地
2.使用composer init 创建一个composer.json文件

```shell
{
    "name": "doozz/say-hi",
    "description": "test",
    "type": "library",
    "license": "MIT",
    "authors": [
        {
            "name": "doozz",
            "email": ""
        }
    ],
    "minimum-stability": "dev",
    "require": {
		"php": "^7.1"
    },
    "autoload": {
        "psr-4": { "hello\\": "src" }
    }
}
```

3.创建src文件夹，对应上面的autoload。在里面新建test.php,并写入namespace。自动加载会按psr4规范映射文件路径，这样实例化拓展文件里的类时，会自动加载相应文件。

```shell
<?php

namespace hello；

class test
{
    public static function say()
    {
        echo "hello world !<br />";
    }
}
```

4.把写好的文件push到git，并且发行一个tag版

```shell
创建了本地一个版本
git tag -a 1.0 -m 'release'

同步tag到远程仓
git push origin --tags

查看tag
git tag
```

5.远程仓库提交到[packagist](https://packagist.org/packages/submit)，通过检测后，提交即可。
6.然后就可以愉快的在项目下的comoiser.json下包含我们刚刚提交的composr包 - doozz/say-hi了。


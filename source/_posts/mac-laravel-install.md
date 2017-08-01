---
title: MacOS 安裝、使用Laravel
date: 2017-04-20 13:54:43
tags:
  - php
  - MacOS
  - laravel
---
最一開始寫網頁是接觸php，然後是使用ubuntu做開發，之後存了一點小錢買了一台mac然後就比較少在寫php了，大部分的時間都在研究JS，最近又心血來潮想寫寫PHP，而且Laravel也與我最愛的Vue.js整合了！這真是一個不錯的契機！所以就邊安裝邊寫一篇心得文，讓有需要入坑的朋友可以參考參考。

<!--more-->

## 起手

由於Laravel的PHP版本比較高，所以我們先安裝PHP 7.0 以上的版本，以下我用7.1做範例。

```bash
curl -s http://php-osx.liip.ch/install.sh | bash -s 7.1
```
安裝會需要一些時間！安裝完查看php版本
```bash
php -v
```
因為mac有預設安裝php，可是版本很低，假如安裝完之後版本還是舊的，那就要修改path，下面會以zsh為範例。
```bash
# 將下行加入到.zshrc裡
export "PATH=/usr/local/php5/bin:$PATH"

# 在command line裡面重新載入zshrc
source ~/.zshrc
```
source完畢之後再看看PHP的版本，正常來說會是7.1！

## 安裝 [Composer](https://getcomposer.org/)
官方介紹
>Composer is a tool for dependency management in PHP. It allows you to declare the libraries your project depends on and it will manage (install/update) them for you.

安裝方法很簡單，只需要輸入下列指令就可以囉！
```bash
curl -sS https://getcomposer.org/installer | php
```
安裝完之後還需要把Composer移動到path裡面，這樣在command line裡面才會有composer指令！
```bash
sudo mv composer.phar /usr/local/bin/composer
```

## 安裝 [Laravel](https://laravel.com/docs/5.4/installation)
重頭戲來啦～安裝Laravel，我們將使用Composer來作安裝！

指令如下
```bash
composer global require "laravel/installer"
```
安裝完之後需要加入到path裡面，不然會每次都需要打一大串喔！
```bash
# 將下行加入到.zshrc裡
export PATH="$PATH:$HOME/.composer/vendor/bin"

# 在command line裡面重新載入zshrc
source ~/.zshrc

# 如不加入到path，則要使用的話需要打下列指令。
composer create-project --prefer-dist laravel/laravel blog
```
這樣就大功告成啦！！之後可以看著官方的getting start慢慢動手做囉！

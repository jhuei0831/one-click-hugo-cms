---
title: Laravel Backend Control Website - 1
categories: ['CODE']
description: Laravel 網站設置和預計設計目標 - Website settings and expected design goals
tags: ['laravel', 'php', 'html', 'javascript', 'css', 'bootstrap', 'backend']
authors: ["Kerwin"]
featuredImagePreview: feature.jpg
date: 2020-04-21
lastmod: 2020-04-21

series: ["laravel-myweb"]
series_weight: 1
---

> Laravel 網站設置和預計設計目標 - Website settings and expected design goals

<!--more-->

## 前言

前陣子開始接觸 Laravel ，算是第一個接觸的MVC框架 ，學了一段時間後就想做一個小品當作練習。這個練習的目的就是讓我熟悉 `Model`、`View`、`Controller` 三者之間的愛恨糾葛😂。不會講得很細，就是分享我遇到的問題以及解決的方法。

## 使用工具

|  工具 | Laravel | PHP | MySQL |
| :--------: | :--------: | :--------: | :--------: |
| 版本    | 5.8     | 7.2     | 5.7     |

## 目標

完成一個後端控制前台發布文章、導覽列、消息、輪播、訊息的網站。

## 成品圖

[![fontend](https://i.imgur.com/pA0CUz0.png)](https://i.imgur.com/pA0CUz0.png)

[![backend](https://i.imgur.com/GMe8NcD.png)](https://i.imgur.com/GMe8NcD.png)

## 設置

這次使用的是`5.8` 版本的Laravel ，所以先建立一個資料夾 `myweb` 。

建立Laravel 5.8 `composer create-project --prefer-dist laravel/laravel myweb "5.8.*"`

之後滑進去 `cd myweb`

{{< admonition type=warning title="注意" open=true >}}
請自行安裝所需要的環境及套件，Composer 和 xampp 等等
{{< /admonition >}}

之後在 MySQL 建立資料庫 `myweb` ， 並在 `.env` 中將資料庫相關資料填上。

```setting
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=myweb
DB_USERNAME=username
DB_PASSWORD=password
```

資料庫設定完後，Laravel 創建完成後 `php artisan make:auth`，這一步的目的是建立基本的用戶認證，如果之後你要自己建立也行，問題不大。

接著在資料庫建立資料表，運行`php artisan migrate`，沒意外應該是會順利成功。

你到**phpmyadmin**的**myweb**資料庫應該可以看到建立好的三個資料表。

最後執行 `php artisan serve` 開始運行 laravel ，並到 網址`http://127.0.0.1:8000/`，成功看到下圖就沒事可以去看個[Laravel Document](https://laravel.com/docs/5.8/readme) 壓壓驚了。

[![laravel](https://i.imgur.com/vo0yJtw.png)](https://i.imgur.com/vo0yJtw.png)

> ❤❤  貼心小提醒，記得版控ㄛ ❤❤
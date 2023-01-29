---
title: Laravel Excel Export - Basic
categories: ['CODE']
description: Laravel 匯出基礎篇 - Basic Laravel Export
tags: ['php', 'html', 'laravel', 'excel', 'export']
authors: [Kerwin]
featuredImagePreview: feature.jpg
date: 2020-07-07
lastmod: 2020-07-07
---

> Laravel 匯出基礎篇 - Basic Laravel Export

<!--more-->

## 前言

如何使用 Laravel Excel 這個套件來實現匯出的功能，以使用者資料匯出為範例。

---

## 1. Laravel Excel 安裝

安裝需求 :

* PHP: `^7.0`
* Laravel: `^5.5`
* PhpSpreadsheet: `^1.6`
* PHP extension `php_zip` enabled
* PHP extension `php_xml` enabled
* PHP extension `php_gd2` enabled
* PHP extension `php_iconv` enabled
* PHP extension `php_simplexml` enabled
* PHP extension `php_xmlreader` enabled
* PHP extension `php_zlib` enabled

**A. 執行安裝指令 :**

```php
composer require maatwebsite/excel
```

**B. 在 `config/app.php` 中加入 :**

```php
'providers' => [
    ...
    Maatwebsite\Excel\ExcelServiceProvider::class,
]
```

```php
'aliases' => [
    ...
    'Excel' => Maatwebsite\Excel\Facades\Excel::class,
]
```

**C. 將 `excel.php` 丟到 `config` 底下 :**

```bash
php artisan vendor:publish --provider="Maatwebsite\Excel\ExcelServiceProvider"
```

---

## 2. 建立匯出Model

先建立 `app/Exports` 。

**執行指令 :**

```php
php artisan make:export UsersExport --model=User
```

`UsersExport.php` :

```php
<?php

namespace App\Exports;

use App\User;
use Maatwebsite\Excel\Concerns\FromCollection;

class UsersExport implements FromCollection
{
    public function collection()
    {
        return User::all();
    }
}
```

---

## 3. 建立Controller

**執行指令 :**

```bash
php artisan make:controller UsersController
```

`UsersController.php` :

```php
<?php

namespace App\Http\Controllers;

use App\Exports\UsersExport;
use Maatwebsite\Excel\Facades\Excel;

class UsersController extends Controller 
{
    public function export() 
    {
        return Excel::download(new UsersExport, 'users.xlsx');
    }
}
```

---

## 4. 建立路由

`web.php`

```php
Route::get('users/export/', 'UsersController@export');
```

---

## 5. 取得使用者匯出資料


```bash
php artisan serve
```

網址輸入: [http://127.0.0.1:8000/users/export/](http://127.0.0.1:8000/users/export/)

取得匯出檔案。

---
title: Install Swoole on Windows
categories: ['EXECUTION']
description: 在Windows環境安裝Swoole - Install Swoole on Windows
tags: ['php', 'websocket', 'realtime', 'cygwin', 'swoole']
authors: ["Kerwin"]
featuredImagePreview: feature.jpg
date: 2020-05-06
lastmod: 2020-05-06
---

> 在Windows環境安裝Swoole - Install Swoole on Windows
<!--more-->

## 前言

---
最近想玩 `websocket`，所以找了一個套件 `Swoole` 來試看看。可是Swoole一般是使用在`Linux`環境，所以就要使用 `Cygwin` 來實現了。

## 本篇重點

---

* 安裝 Cygwin
* 安裝 Swoole
* 測試 Swoole

## 1. 安裝 Cygwin

---
Cygwin 下載地址：[https://www.cygwin.com/](https://www.cygwin.com/)

以下是我安裝的步驟，可以根據自身需求更改。

### Step 1

[![step1](https://i.imgur.com/eeSV9ju.png)](https://i.imgur.com/eeSV9ju.png)

---

### Step 2

[![step2](https://i.imgur.com/BAZ8h3i.png)](https://i.imgur.com/BAZ8h3i.png)

---

### Step 3

[![step3](https://i.imgur.com/JmlRbSM.png)](https://i.imgur.com/JmlRbSM.png)

---

### Step 4

[![step4](https://i.imgur.com/IGEcBD3.png)](https://i.imgur.com/IGEcBD3.png)

---

### Step 5. 使用鏡像來源

[![step5](https://i.imgur.com/7A7aDYJ.png)](https://i.imgur.com/7A7aDYJ.png)

---

### Step 6. 安裝 binutils

[![step6](https://i.imgur.com/Fu0U1lE.png)](https://i.imgur.com/Fu0U1lE.png)

---

### Step 7. 安裝 gcc-core，gcc-g++

[![step7](https://i.imgur.com/KQ8q4hk.png)](https://i.imgur.com/KQ8q4hk.png)

---

### Step 8. 安裝 gdb

[![step8](https://i.imgur.com/OocI7Tx.png)](https://i.imgur.com/OocI7Tx.png)

---

### Step 9. 安裝 php

[![step9](https://i.imgur.com/beFZDAK.png)](https://i.imgur.com/beFZDAK.png)

---

### Step 10. 安裝 pcre-devel

[![step10](https://i.imgur.com/0KqiT8M.png)](https://i.imgur.com/0KqiT8M.png)

---

### Step 11. 安裝 autoconf

[![step11](https://i.imgur.com/mO3nVD5.png)](https://i.imgur.com/mO3nVD5.png)

---

### Step 12. 安裝 pcre

[![step12](https://i.imgur.com/E5IlCcH.png)](https://i.imgur.com/E5IlCcH.png)

---

### Step 13. 下一步到底

## 2. 安裝 Swoole

Swoole 下載地址 : [https://pecl.php.net/package/swoole](https://pecl.php.net/package/swoole)

### Step 1. 解壓縮下載好的檔案到 cygwin64/home/ 底下

---

### Step 2. 執行 Cygwin

打開 ./cygwin64/Cygwin.bat

---

### Step 3. 進入壓縮後的Swoole資料夾

```bash
cd /home/swoole-4.5.0
```

---

### Step 4. 產生編譯文件 configure

```bash
phpize
```

---

### Step 5. 安裝 Swoole

```bash
./configure && make && make install
```

---

### Step 6. 在php.ini文件中加入 extension=swoole.so

```bash
// 可以使用下面指令找到php.ini檔案位置
php -i | grep php.ini
```

---

### Step 7. 查看是否安裝成功

```bash
php -m
```

---

[![swoole](https://i.imgur.com/PaZwDEo.png)](https://i.imgur.com/PaZwDEo.png)

## 3. 測試 Swoole

```bash
cd /home/swoole-4.5.0/examples/http2
php server.php
```

打開瀏覽器輸入網址[http://127.0.0.1:9501/](http://127.0.0.1:9501/) 出現下圖則代表成功 :

![success](https://i.imgur.com/jJLoEV5.png)

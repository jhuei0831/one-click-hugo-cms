---
title: Long Polling
categories: ['CODE']
description: 長時間輪詢 - Long Polling
tags: ['php', 'javascript', 'request', 'ajax', 'real-time', 'long-polling']
authors: ["Kerwin"]
featuredImagePreview: feature.jpg
date: 2020-05-12
lastmod: 2020-05-12
---

> 長時間輪詢 - Long Polling

<!--more-->

## 前言

PHP要做即時(Real-time)方式有很多種，Polling、Long Polling、WebSocket...等等。

這次分享一個簡單的Long Polling範例 。

---

## 範例說明

如果物件(data.txt)改變了，client.php將自動更新，不需透過重新整理。

client.js 持續丟 request 給 server.php ， 若無限循環的 server.php 發現 client 的物件(data.txt)時間戳小於自己，

就將更新後的物件資料(json)傳送給 client ，完成自動更新。

---

## 資料來源

[php-long-polling](https://github.com/panique/php-long-polling)

---

## 結語

下次我用PHP + MySQL  做一個範例。

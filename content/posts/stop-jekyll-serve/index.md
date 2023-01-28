---
title: Stop Jekyll serve (127.0.0.1:4000)
categories: ['issue']
description: 解決Jekyll serve 無法關閉  - Jekyll serve service worker
tags: ['jekyll', 'cookie']
authors: ["Kerwin"]
featuredImage: /img/test.png
date: 2020-04-18
lastmod: 2020-04-18

# series: ["getting-start"]
# series_weight: 1
lightgallery: true
---

> 解決Jekyll serve 無法關閉  - Jekyll serve service worker
<!--more-->

## 前言
---
一般來講我們關閉本地的jekyll serve 都是使用 `ctrl + c` ，可是當你下次再次運行 `jekyll serve` 時發現上次的還在 。

## 解決
---
這時只要把`127.0.0.1`中的 ***Service Workers Cookie***  刪除即可，以GOOGLE瀏覽器為例，目標就是下圖紅框處的Cookie。
{{< image src="cookie.png" caption="Basic configuration preview" height="1562" width="2400">}}
{{< image src="basic-configuration-preview.webp" caption="Basic configuration preview" height="1562" width="2400">}}

![cookie](https://i.imgur.com/hk0qGZQ.png)

[如果你想了解更多](https://developers.google.com/web/fundamentals/primers/service-workers/lifecycle?hl=zh-tw)

Hope it will help!
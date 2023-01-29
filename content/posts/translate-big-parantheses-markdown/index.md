---
title: Translate Big Parantheses in Markdown
categories: ['EXECUTION']
description: 如何在markdown中顯示雙大括號 - How to translate double big parantheses in markdown
tags: ['jekyll', 'liquid', 'markdown']
authors: ["Kerwin"]
featuredImagePreview: feature.jpg
date: 2020-04-22
lastmod: 2020-04-22
---

> 如何在markdown中顯示雙大括號 - How to translate double big parantheses in markdown
<!--more-->

## 前言

由於jekyll 使用liquid 的關係，在 POST 中使用 {% raw %} {{  }} {% endraw %} 會無法正常顯示。

---

## 解決

```markdown {% assign openTag = '{%' %}  
{{ openTag }} raw %}
{% raw %}
  {{ 加入raw即可 }}
{% endraw %}
{{ openTag }} endraw %}
```

{{< admonition type=info title="Info" open=true >}}
之後補充如何顯示 {% raw %} {% raw %} {% endraw %}
{{< /admonition >}}

Hope it will help !

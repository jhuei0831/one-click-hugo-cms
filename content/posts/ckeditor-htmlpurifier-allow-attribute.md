---
title: "Ckeditor + Htmlpurifier allow attribute"
subtitle: yoyo2
date: 2020-04-13T17:53:07+08:00
lastmod: 2020-04-18T17:53:07+08:00
draft: true
authors: [Alice, Bob, Catherine]
---
zzz
Gone camping! :(fas fa-campground fa-fw): Be back soon.
> **Fusion Drive** combines a hard drive with a flash storage (solid-state drive) and presents it as a single logical volume with the space of both drives combined.
This is a digital footnote[^1].
This is a footnote with "label"[^label]
{{< highlight html >}}
<section id="main">
    <div>
        <h1 id="title">{{ .Title }}</h1>
        {{ range .Pages }}
            {{ .Render "summary"}}
        {{ end }}
    </div>
</section>
{{< /highlight >}}

[^1]: This is a digital footnote
[^label]: This is a footnote with "label"
## Table of Contents
  * [Chapter 1](#chapter-1)
  * [Chapter 2](#chapter-2)
  * [Chapter 3](#chapter-3)

- [x] Write the press release
- [ ] Update the website
- [ ] Contact the media
| Option | Description |
| ------ | ----------- |
| data   | path to data files to supply the data that will be passed into templates. |
| engine | engine to be used for processing templates. Handlebars is the default. |
| ext    | extension to be used for dest files. |
<!--more-->
## Table of Contents
  123
# 123
I created a page manager in the backstage, and it edit by the ckeditor. Also, I use Htmlpurifier to defense the xss injection. But Htmlpurifier will block the attribute like Bootstrap tabs plugins. 
So the way I used to allow attribute while using Htmlpurifier.

Here is the [document](http://www.poultry.org.tw/htmlpurifier/docs/enduser-customize.html).

For example, I want to purify the html below.

```html
<div class="bootstrap-tabs" data-tab-set-title="Program Introduction"></div>
```

If you use the simple purifier like this,

```php
function xss_filter($content){
    $purifier = new HTMLPurifier($config);
    $cleanContent = $purifier->purify($content);
    return $cleanContent;
}
```

You will get the output below,

```html
<div class="bootstrap-tabs"></div>
```

The  ***data-tab-set-title***  is disabled. So, add the code below and you will get what you want.

```php
$config = HTMLPurifier_Config::createDefault(); 
$def = $config->getHTMLDefinition(true);
$def->addAttribute('div', 'data-tab-set-title', 'CDATA');
```

Hope it will help !
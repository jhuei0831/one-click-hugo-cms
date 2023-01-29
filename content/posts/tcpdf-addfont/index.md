---
title: TCPDF addTTFfont Problem
categories: ['EXECUTION']
description: 使用TCPDF的addTTFfont加入字型問題
tags: ['php', 'tcpdf', 'pdf', 'font', 'addfont']
authors: ["Kerwin"]
featuredImagePreview: feature.jpg
date: 2020-11-13
lastmod: 2020-11-13
---

> 使用TCPDF的addTTFfont加入字型問題
<!--more-->

## 前言

如果今天你是使用`TCPDF`的人，要加入外部字型的時候會使用到`addTTFfont`這個函式；但如果照著官網上[Docs](https://tcpdf.org/docs/fonts/)的方法來加入某些中文字型，作法如下:

```php
$fontname = $pdf->addTTFfont('/path-to-font/DejaVuSans.ttf', 'TrueTypeUnicode', '', 32);
```

會發現在使用`Adobe Acrobat Reader DC` 打開檔案時會出現以下錯誤訊息:

{{< image src="https://i.imgur.com/CrYzlub.png" caption="font">}}

然後你的字就沒辦法顯示，可是在網頁上打開PDF檔案又可以😱😱

---

## 解決方法

以下是`addTTFfont`的函式:

```php
public static function addTTFfont($fontfile, $fonttype='', $enc='', $flags=32, $outpath='', $platid=3, $encid=1, $addcbbox=false, $link=false)
{
 ....
}
```

在官方文檔中他們提到了以下:

{{< admonition type=info title="" open=true >}}
<h3>Convert a font for TCPDF</h3>
Using the addTTFfont() method you can directly create a TCPDF font starting from a TrueType, OpenType or Type1 font.
NOTE: The ‘fonts’ folder must be writeable by the webserver.
{{< /admonition >}}

這時候你就會在`$fonttype`的位置傻傻的填入 `TrueType`、`Type1`以及 `TrueTypeUnicode`，然後就發生前言的悲劇。

可是天殺的他們並沒有提到還有以下這幾個參數可以填:

* `CID0JP` = CID-0 Japanese
* `CID0KR` = CID-0 Korean
* `CID0CS` = CID-0 Chinese Simplified
* `CID0CT` = CID-0 Chinese Traditional

所以你只要根據你對應的字型語系來使用上述參數，你在使用`Adobe Acrobat Reader DC` 打開檔案後就不會報錯了。

---

## 後話

是怎麼發現還有那幾種參數可以使用的?

就是到 `TCPDF/include/tcpdf_fonts.php` 裡面看source code，還好他註解有寫😂

以下是他的註解，有興趣的可以參考:

```php
/**
* Convert and add the selected TrueType or Type1 font to the fonts folder (that must be writeable).
* @param $fontfile (string) Font file (full path).
* @param $fonttype (string) Font type. Leave empty for autodetect mode. Valid values are: TrueTypeUnicode, TrueType, Type1, CID0JP = CID-0 Japanese, CID0KR = CID-0 Korean, CID0CS = CID-0 Chinese Simplified, CID0CT = CID-0 Chinese Traditional.
* @param $enc (string) Name of the encoding table to use. Leave empty for default mode. Omit this parameter for TrueType Unicode and symbolic fonts like Symbol or ZapfDingBats.
* @param $flags (int) Unsigned 32-bit integer containing flags specifying various characteristics of the font (PDF32000:2008 - 9.8.2 Font Descriptor Flags): +1 for fixed font; +4 for symbol or +32 for non-symbol; +64 for italic. Fixed and Italic mode are generally autodetected so you have to set it to 32 = non-symbolic font (default) or 4 = symbolic font.
* @param $outpath (string) Output path for generated font files (must be writeable by the web server). Leave empty for default font folder.
* @param $platid (int) Platform ID for CMAP table to extract (when building a Unicode font for Windows this value should be 3, for Macintosh should be 1).
* @param $encid (int) Encoding ID for CMAP table to extract (when building a Unicode font for Windows this value should be 1, for Macintosh should be 0). When Platform ID is 3, legal values for Encoding ID are: 0=Symbol, 1=Unicode, 2=ShiftJIS, 3=PRC, 4=Big5, 5=Wansung, 6=Johab, 7=Reserved, 8=Reserved, 9=Reserved, 10=UCS-4.
* @param $addcbbox (boolean) If true includes the character bounding box information on the php font file.
* @param $link (boolean) If true link to system font instead of copying the font data (not transportable) - Note: do not work with Type1 fonts.
* @return (string) TCPDF font name or boolean false in case of error.
* @author Nicola Asuni
* @since 5.9.123 (2010-09-30)
* @public static
*/
public static function addTTFfont($fontfile, $fonttype='', $enc='', $flags=32, $outpath='', $platid=3, $encid=1, $addcbbox=false, $link=false)
```

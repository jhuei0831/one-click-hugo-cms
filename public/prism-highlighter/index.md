# Prism - Customize Code Highlighter


> 客製化程式碼高亮 - Customize Code Highlighter

<!--more-->

## 前言

之前用的 `Rough` 有點不習慣，所以找到 `Prism` ，一個客製化的程式碼高亮的應用。

---

## 1. 下載地址

[https://prismjs.com/download.html](https://prismjs.com/download.html)

勾選完需求後，就可以在最下方下載 `js` 及 `css` 檔案。

---

## 2. 插件

`Prism` 提供了一些插件輔助，比如顯示程式碼行數、程式碼語言別、單行高亮…等等。

以下提供一些我使用的插件。

---

## 3. 程式碼行數 Line Numbers

在要高亮的內容要加入 `line-numbers` 的 `class`類別，如下 :

```html
<article class="post-content line-numbers" itemprop="articleBody">
  {% raw %}{{ content }}{% endraw %}
</article>
```

---

## 4. 複製 Copy to Clipboard Button

沒重點，同時勾選 `Copy to Clipboard Button` 及 `Toolbar` 後引入 `js` 檔即可。

---

## 5. 結構樹 Treeview

勾選 `Treeview` 後引入 `js` 檔，高亮語言選擇 `treeview` 即可。

```liquid
{% raw %}{% highlight treeview %}{% endraw %}
    ...程式碼...
{% raw %}{% endhighlight %}{% endraw %}
```

# Laravel Backend Control Website - 2


> Laravel 基本設置以及前後台公版 - Basic setup and template.

<!--more-->

## 前言

上一篇進行了 Laravel 的建立以及基礎會員的設置，這篇將會提到如何設置時區、語言以及建立自己的頁面公版。

---

## 1. 基本設定

打開 `./config/app.php` ，這個檔案內就是對網站的基礎設定，語系、時間等等。有被 `env` 括弧起來的內容會採納 `.env` 檔案的內容。以下是一些我的設定，可以根據個人需求更改。

```php
'timezone' => 'Asia/Taipei',   // 時間
'locale' => 'zh-TW',           // 語系
'fallback_locale' => 'zh-TW',
```

---

## 2. 語言設置

如果根據上面進行更改後，在上一篇建立的會員設置會發現還是英文，這是因為他並沒有找到 `zh-TW` 的語言包。這時候要到 `./resources/lang` 底下建立 `zh-TW` 的資料夾，並將 [語言包](https://github.com/caouecs/Laravel-lang/tree/master/src/zh-TW) 放到底下，重新整理後就會翻譯成繁體中文了。緊接著在同個資料夾底下建立一個 `myweb.php` 做為日後要翻譯的繁中語言包。

{{< admonition type=tip title="Tip" open=true >}}
這邊說一個比較偷吃步的方法，就是在 <mark>lang</mark> 資料夾底下直接建立 <mark>zh-TW.json</mark> ，將所有要翻譯的文字填進去，之後只要用` {{ __('language') }}` 即可。缺點是之後再做語言切換會有部分問題，以及不美觀和分類問題。優點就是網站只需要一種語言的話會很方便!
{{< /admonition >}}

---

## 3. 頁面公版

製作頁面公版的好處就是要創建新的頁面時可以直接套用公版，省去時間及麻煩。以下會將前台及後台分開。

首先介紹你的頁面會被放置在 `./resources/views` 底下，由於之前有建立過會員，所以裡面已經有一些頁面及資料夾存在。這時候在 先把`layouts`改名成 `_layouts` 並在 底下建立 `manage` 和 `home` 兩個資料夾，並將 `app.blade.php` 複製進兩個資料夾。

{{< admonition type=info title="Info" open=true >}}
順帶一提，之後要建立頁面要檔案名稱要固定是 <mark>頁面名稱.blade.php</mark>
{{< /admonition >}}

然後打開其中的一個 `app.blade.php` ，把 `<nav>...</nav>` 整段程式碼剪下並在 `views` 底下建立一個 `_partials` 資料夾，接著在 `_partials` 底下建立 `manage` 和 `home` 兩個資料夾，並在底下各建立一個 `nav.blade.php` 的視圖。最後將剛剛剪下的程式碼貼進去，資料夾內容會長得跟下圖一樣。

{{< image src="https://i.imgur.com/tq1PSgn.png" caption="views">}}

如此一來，我們只要引入 `nav` 至 `app.blade.php`母版即可。

```html
<body>
    <div id="app">
        @include('_partials.home.nav')
        <main class="py-4">
            @yield('content')
        </main>
    </div>
</body>
```

緊接著到 `http://127.0.0.1:8000/login` 頁面發現報錯，`View [layouts.app] not found.` 是因為剛剛將資料的路徑更動，只要將`auth/login.blade.php` 中的 `@extends('layouts.app')` 改成 `@extends('_layouts.home.app')` 即可。 其他頁面也是一樣的方法，之後創新頁面只要先把下面貼上，再新增你需要的元素即可。

```php
@extends('_layouts.home.app')
@section('content')
你要的內容
@endsection
```

 就拿 `welcome.blade.php` 來測試，把內容改成

```php
@extends('_layouts.home.app')
@section('content')
    <div class="container">
        <div class="card">
            <div class="card-header">
                測試
            </div>
            <div class="card-body">
                TEST
            </div>
        </div>
    </div>
@endsection
```

呈現的效果如下圖

{{< image src="https://i.imgur.com/EmJAlfk.png" caption="example">}}

---

## 結語

乾淨的版面😊


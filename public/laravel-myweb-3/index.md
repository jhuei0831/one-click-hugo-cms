# Laravel Backend Control Website - 3


> Laravel 路由、參數及按鈕設定 - Setting Routes、 Variables and Button

<!--more-->

## 前言

我們之所以看的到建立的畫面，都是以在 `routes/web.php` 文件定義路由開始的。可以通過在瀏覽器中輸入定義的路由URL來訪問 `routes/web.php` 中定義的路由。上一篇我們雖然建立了後台，可是並未創建路由。

在這篇將建立後台路由並且新增參數以便之後應用。

---
## 1. 路由

如果現在打開路由，會看到 :

```php
//網站首頁
Route::get('/', function () {
    return view('welcome');
});
//用戶認證
Auth::routes();
//登入後頁面
Route::get('/home', 'HomeController@index')->name('home');
```

我們在之中加入 :

```php
Route::get('/manage', function () {return view('manage.index');})->name('manage');
```

這段的意思是在 `http://127.0.0.1/manage` 看到 `views/manage/index.blade.php` 的內容，並將其命名為 `manage`。

之後只要使用 `route('member')` 就等同呼叫 `http://127.0.0.1/manage`。

---

## 2. 參數

有時候，在不同的頁面都會用到相同字串，比如說 [下一篇](https://jhuei.com/code/2020/04/29/laravel-myweb-4.html) 的會員管理，就會使用到權限的設定(新增、修改、刪除)。

如果多個頁面都要使用，何不先定義，之後再引入即可。在`./app` 底下建立 `Enum.php` 作為放參數的地方，並直接在裡面加上 :

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Enum extends Model
{
    // 帳號權限
    const permission = [
        '0' => '一般使用者',
        '1' => '限閱',
        '2' => '閱讀、新增',
        '3' => '閱讀、新增、編輯',
        '4' => '閱讀、新增、編輯、刪除',
        '5' => '所有權限',
    ];

}

```

這樣一來，之後只要使用 `App/Enum:permission` 就可以得到這個陣列下的內容。

---

## 3. 按鈕

同上，之後一定會遇到要建立一堆按鈕的狀況，我們先把按鈕作成  `Function` 之後直接引入即可。

在`./app` 底下建立 `Button.php` 作為放按鈕的地方，並直接在裡面加上 :

```php
<?php

namespace App;
use URL;
use Illuminate\Database\Eloquent\Model;

class Button extends Model
{
    public static function Detail($id)
    {
        $url = URL::full();
        echo "<button type=\"submit\" class='btn btn-sm btn-secondary'>";
        echo "<i class='fas fa-info-circle'></i> ".trans('Detail');
        echo "</a>";
    }

    public static function Deleting($id)
    {
        echo "<button type=\"submit\" class='btn btn-sm btn-danger btn-delete' onclick='return confirm(\"確認刪除?\")'>";
        echo "<i class='fas fa-trash-alt'></i> ".trans('Delete');
        echo "</button>";
    }

    public static function Edit($id)
    {
        echo "<button type=\"submit\" class='btn btn-sm btn-success' formtarget='_blank'>";
        echo "<i class='fas fa-pencil-alt'></i> " . trans('Edit');
        echo "</button>";
    }

    public static function Create()
    {
        $url = URL::full();
        echo "<a class='btn btn-sm btn-primary' href='{$url}/create'>";
        echo "<i class='fas fa-plus'></i> ".trans('Create');
        echo "</a>";
    }

    public static function Reset()
    {
        echo "<p class='text-right'>";
        echo "<a class='btn btn-sm btn-reset btn-danger' href='reset.php'>";
        echo  "<i class='fas fa-undo-alt'></i> ".trans('Reset');
        echo  "</a>";
        echo "</p>";
    }

    public static function To($url=false,$to, $txt, $query="", $class="btn-secondary", $fas="list-ol", $confirm=false)
    {
        $url = $url?URL::full():'';
        if ($confirm == true) {
            $confirm = 'onclick="return confirm(\'確認刪除?\')"';
        }
        if ($url) {
            echo "<a class='btn btn-sm {$class}' href='{$url}/{$to}/{$query}' {$confirm}>";
        }
        else{
            echo "<a class='btn btn-sm {$class}' href='{$to}/{$query}' {$confirm}>";
        }
        echo  "<i class='fas fa-{$fas}'></i> {$txt}";
        echo  "</a>";
    }

    public static function GoBack($url = "#")
    {
        $target_url = ($url) ? $url: URL::previous();

        echo "<a class='btn btn-sm btn-default' href='{$target_url}'>";
        echo "<i class='fas fa-arrow-left'></i> ".trans('Previous');
        echo "</a>";
    }
}

```

這樣一來，之後只要使用 `App\Button::GoBack(route('manage'))` 就可以回到後台首頁。

---
Hope it will help !


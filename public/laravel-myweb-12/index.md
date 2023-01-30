# Laravel Backend Control Website - 12


> 選單管理 - Menu Manage

<!--more-->

## 前言

[上一篇](https://jhuei.com/code/2020/06/10/laravel-myweb-11.html) 新增了選單管理，這篇就是要將 `導覽列`、`選單`及 `頁面` 三者串連起來。

---

## 1. 導覽列規則

* 導覽名稱唯一(Unique)。
* 類型一 : 導覽目錄
  * 下拉式選單，無視設定連結。
  * 選單內容可以在選單管理進行排序設定。
* 類型二 : 一般頁面
  * 點選後直接跳到該頁面。
  * 外部連結直接貼完整網址，ex : <https://www.google.com/>
  * 指定頁面連結要透過指定格式，如<http://127.0.0.1:8000/article/導覽列名稱/選單名稱?頁面網址。>

---

## 2. 選單規則

* 選單名稱唯一(Unique)。
* 如有設定連結，點擊選單則跳轉到該頁面。
* 清單顯示意指所有 `頁面設定` 連接該選單的頁面以條列式一一顯示。
* 非清單顯示意指顯示 `更新時間` 最新的頁面。
* 連結設定請參照導覽列 `一般頁面` 設定。

---

## 3. 頁面規則

* 頁面網址唯一(Unique)。
* 頁面網址用來定義該頁面位置，所以只須給名稱即可，ex:Page1。
* 在選單的清單顯示中，頁面以 `更新時間` 新到舊進行排序。

---

## 4. 頁面及選單串聯

之前有在 `pages` 資料表建立 `menu_id` 的欄位，現在要在 `Page` 的新增、修改及首頁加入 `menu_id` 的新增修改表單。

`create.blade.php` :

```html
...略...
@csrf
<div class="form-group row">
    <label for="menu_id" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Menu') }}{% endraw %}</label>
    <div class="col-md-6">
        <select class='form-control' name='menu_id' aria-describedby="menuHelp">
            <option value=''>{% raw %}{{ trans('Please choose').trans('Menu') }}{% endraw %}</option>
            @foreach($menus as $key => $value)
                <option value="{% raw %}{{ $value['id'] }}{% endraw %}">{% raw %}{{ $value['name'] }}{% endraw %}</option>
            @endforeach
        </select>
        <span id="menuHelp" class="help-block">
            選擇要加入的導覽列項目。
        </span>
    </div>
</div>
...略...
```

`edit.blade.php` :

```html
...略...
@csrf
@method('PUT')
<div class="form-group row">
    <label for="menu_id" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Menu') }}{% endraw %}</label>
    <div class="col-md-6">
        <select class='form-control' name='menu_id' aria-describedby="menuHelp">
            <option value='NULL' {% raw %}{{ ($page->menu_id) ? "selected" : "" }}{% endraw %}>{% raw %}{{ trans('Please choose').trans('Menu') }}{% endraw %}</option>
            @foreach($menus as $key => $value)
                @if ($value['id'] == $page->menu_id)
                    <option value="{% raw %}{{ $value['id'] }}{% endraw %}" selected>{% raw %}{{ $value['name'] }}{% endraw %}</option>
                @else
                    <option value="{% raw %}{{ $value['id'] }}{% endraw %}">{% raw %}{{ $value['name'] }}{% endraw %}</option>
                @endif
            @endforeach
        </select>
        <span id="menuHelp" class="help-block">
            選擇要加入的導覽列項目。
        </span>
    </div>
</div>
...略...
```

接下來根據重點 1~3 在 `MenuController.php` 建立以下 :

```php
/**
* [menus description]
* @param  [type] $nav  [選取的導覽列]
* @param  [type] $menu [選取的選單]
* @param  [type] $menus_nav [導覽列下所有的選單資料]
* @param  [type] $select_menu [選取的選單資料]
* @param  [type] $menu_pages [選取選單下所有頁面的資料]
* @return [type]       [description]
*/
public function menus($nav,$menu)
{
    $navbar = Navbar::where('name',$nav)->first();
    $menus_nav = Menu::where('navbar_id',$navbar->id)->where('is_open',1)->orderby('sort')->get();
    $select_menu = Menu::where('name',$menu)->first();
    if (empty($select_menu)) {
        abort(404);
    }

    if ($select_menu->is_list == '1') {
        $menu_pages = Page::where('is_open',1)->orderby('updated_at','desc')->paginate(10);
    }
    else {
        $menu_pages = Page::where('menu_id',$select_menu->id)->where('is_open',1)->orderby('updated_at','desc')->first();
    }

    return view('menu',compact('menus_nav','select_menu','navbar','menu_pages'));
}
```

路由新增 :

```php
...略...
Route::get('/article/{nav}/{menu}?{page}', function () {return view('menu');})->name('page');
Route::get('/article/{nav}/{menu}', 'MenuController@menus')->name('menu');

View::composer(['*'], function ($view) {
    if (Request::getQueryString()) {
        $current_page = App\Page::where('url', $_SERVER['QUERY_STRING'])->first();
        $view->with('current_page', $current_page);
    }
  $pages = App\Page::where('is_open',1)->orderby('updated_at')->get();
  $view->with('pages',$pages);
    ...略...
});
```

解釋時間 :

* `$nav` 和 `$menu` 分別是導覽列以及選單的 `名稱`。
* 比如說點擊了名為 `menu1` 的選單且導覽列為 `nav1`，控制器14~17行判斷該選單是否存在。
* 若選單存在則19行判斷是否為清單顯示。

視圖新增 :

`menu.blade.php` :

```html
@extends('_layouts.home.app')
@section('title',$select_menu->name)
@section('content')
{% raw %}{{-- 頁面顯示 --}}{% endraw %}
@if(Request::getQueryString())
    <div id="content" class="col-md-12">
        <div class="card border-light" style="border: none;">
            <div class="card-header bg-transparent">
                <h1><b>{% raw %}{{$current_page->title}}{% endraw %}</b></h1>
            </div>
            <div class="card-body">
                {!! clean($current_page->content) !!}
            </div>
            <div class="card-footer bg-transparent">
                <p><span class="badge badge-pill badge-primary">{% raw %}{{ trans('Editor').' : '.$current_page->editor }}{% endraw %}</span></p>
                <p><span class="badge badge-pill badge-primary">{% raw %}{{ trans('Created_at').' : '.$current_page->created_at }}{% endraw %}</span></p>
                <p><span class="badge badge-pill badge-primary">{% raw %}{{ trans('Updated_at').' : '.$current_page->updated_at }}{% endraw %}</span></p>
            </div>
        </div>
    </div>
{% raw %}{{-- 是否清單顯示 --}}{% endraw %}
@elseif($select_menu->is_list)
    <div id="content" class="col-md-12">
        <div class="card border-light" style="border: none;">
            <div class="card-header bg-transparent">
                <h1><b>{% raw %}{{$select_menu->name}}{% endraw %}</b></h1>
            </div>
            <div class="card-body">
                <div class="table-responsive">
                    <table class="table table-hover table-borderless">
                        <tbody>
                        @foreach($menu_pages as $page)
                            @if($page->menu_id == $select_menu->id)
                            <tr>
                                <td width=10%>{% raw %}{{ $page->updated_at->format('Y/m/d') }}{% endraw %}</td>
                                <td><a href="{% raw %}{{ route('page', [$navbar->name,$select_menu->name,$page->url]) }}{% endraw %}">{% raw %}{{ $page->title }}{% endraw %}</a></td>
                            </tr>
                            @endif
                        @endforeach
                        </tbody>
                    </table>
                </div>
            </div>
            <div class="card-footer pagination justify-content-center bg-transparent">
                {!! $menu_pages->links("pagination::bootstrap-4") !!}
            </div>
        </div>
    </div>
@else
    <div id="content" class="col-md-12">
        <div class="card border-light" style="border: none;">
            <div class="card-header bg-transparent">
                <h1><b>{% raw %}{{$menu_pages->title}}{% endraw %}</b></h1>
            </div>
            <div class="card-body">
                {!! clean($menu_pages->content) !!}
            </div>
        </div>
    </div>
@endif
@endsection
```

* 一開始先判斷是否有 `QUERY` ，有的畫直接判斷為頁面，若無則為選單。

---

## 結語

總結，後台新增順序為 `導覽列` >> `選單` >> `頁面`

導覽列(導覽目錄)可以有很多選單，選單(清單顯示)可以有很多頁面

你可以照自己的想法去做修改🥱


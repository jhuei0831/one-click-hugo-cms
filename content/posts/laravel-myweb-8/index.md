---
title: Laravel Backend Control Website - 8
categories: ['CODE']
description: Laravel 頁面管理 - Page Manager
tags: ['laravel', 'php', 'html', 'javascript', 'css', 'bootstrap', 'backend', 'javascript', 'request', 'JQuery']
authors: ["Kerwin"]
featuredImagePreview: feature.jpg
date: 2020-05-10
lastmod: 2020-05-10

series: ["laravel-myweb"]
series_weight: 8
---

> Laravel 頁面管理 - Page Manager

<!--more-->

## 前言

Hello 大家 ! 在 [第六篇](https://jhuei.com/code/2020/05/03/laravel-myweb-6.html) 創建了導覽列管理，而那時候將導覽列分成兩個類型，分別是 `導覽目錄` 以及 `一般頁面` 。

* 一般頁面 : 點擊後直接導向該頁面。
* 導覽目錄 : 點擊後產生下拉式選單，內容為選單。

而這篇要做的就是 `一般頁面` 的頁面管理。

---

## 1. 新增Controller、Model、Migration

```bash
// 一次性建立
php artisan make:model Page -mcr
```

---

## 2. 修改Migration

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreatePagesTable extends Migration
{
    public function up()
    {
        Schema::create('pages', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('menu_id')->nullable()->comment('選單id');
            $table->string('editor')->comment('編輯者');
            $table->string('title')->comment('標題');
            $table->string('url')->comment('頁面網址');
            $table->longText('content')->nullable()->comment('頁面內容');
            $table->boolean('is_open')->default(true)->comment('是否開放');
            $table->timestamps();
        });
    }
    public function down()
    {
        Schema::dropIfExists('pages');
    }
}
```

之後會建立選單連結頁面，所以先加入選單ID欄位。

然後寫入資料庫 :

```bash
php artisan migrate
```

---

## 3. 修改Model

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Page extends Model
{
    protected $table = 'pages';

    protected $primaryKey = 'id';

    protected $fillable = [
        "name", "menu_id" ,"title", "url", "content", "is_open",
    ];
}
```

---

## 4. 加入路由

```php
Route::prefix('manage')->middleware('auth','admin')->group(function(){
    Route::resource('member', 'MemberController');
    Route::resource('navbar', 'NavbarController');
    Route::resource('page', 'PageController');
});
```

---

## 5. 建立視圖

```treeview
views/
├── _layouts/
├── _partials/
├── auth/
└── manage/  
    └── member/  
    └── navbar/  
    └── page/  
        ├── create.blade.php 
        ├── edit.blade.php
        └── index.blade.php
```

---

## 6. 頁面管理

先建立頁面管理首頁 `index.blade.php` :

```php
@extends('_layouts.manage.app')
@section('title', trans('Page').trans('Manage'))
@section('content')
<div class="container-fluid">
    <div class="row justify-content-center">
        <div class="col-md-12">
            <div class="card">
                <div class="card-header">{% raw %}{{ trans('Page').trans('Manage') }}{% endraw %}</div>

                <div class="card-body">
                <ul class="list-inline">
                    <li class="list-inline-item">{% raw %}{{ App\Button::Create() }}{% endraw %}</li>
                </ul>
                <div class="alert alert-warning" role="alert">
                        1. 前台排序以頁面修改日期為主。<br>
                        2. 頁面網址唯一，用途為導向該頁面。
                    </div>
                    <div class="table-responsive">
                        <table id="data" class="table table-hover table-bordered text-center">
                            <thead>
                                <tr class="table-info active">
                                    <th class="text-nowrap text-center">{% raw %}{{ trans('Editor') }}{% endraw %}</th>
                                    <th class="text-nowrap text-center">{% raw %}{{ trans('Title') }}{% endraw %}</th>
                                    <th class="text-nowrap text-center">{% raw %}{{ trans('Page').trans('Url') }}{% endraw %}</th>
                                    <th class="text-nowrap text-center">{% raw %}{{ trans('Is_open') }}{% endraw %}</th>
                                    <th class="text-nowrap text-center">{% raw %}{{ trans('Action') }}{% endraw %}</th>
                                </tr>
                            </thead>
                            <tbody>
                                @foreach ($all_pages as $page)
                                    <tr>
                                        <td>{% raw %}{{ $page->editor }}{% endraw %}</td>
                                        <td>{% raw %}{{ $page->title }}{% endraw %}</td>
                                        <td>
                                            {% raw %}{{ $page->url }}{% endraw %}
                                        </td>
                                        <td>
                                            <font color="{% raw %}{{App\Enum::is_open['color'][$page->is_open]}}{% endraw %}"><i class="fas fa-{% raw %}{{App\Enum::is_open['label'][$page->is_open]}}{% endraw %}"></i></font>
                                        </td>
                                        <td>
                                            <form action="{% raw %}{{ route('page.edit',$page->id) }}{% endraw %}" method="GET">
                                            @csrf
                                            {% raw %}{{ App\Button::edit($page->id) }}{% endraw %}
                                            </form>
                                            <form action="{% raw %}{{ route('page.destroy',$page->id) }}{% endraw %}" method="POST">
                                            @method('DELETE')
                                            @csrf
                                            {% raw %}{{ App\Button::deleting($page->id) }}{% endraw %}
                                            </form>
                                        </td>
                                    </tr>
                                @endforeach
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```

`PageController.php` :

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\DB;
use App\Page;
use App\Navbar;

 ...略...
 
public function index()
{
    $all_pages = DB::table('pages')->get();
    return view('manage.page.index',compact('all_pages'));
}
```

---

## 7. 加入編輯器

在新增文章時需要使用編輯器輔助，可以使用像是 [Ckeditor](https://ckeditor.com/ckeditor-4/download/) 或 [TinyMCE](https://www.tiny.cloud/) 等等的編輯器。

這次我使用 [Summernote](https://summernote.org/) 當作範例，在 `manage/app.blade.php` 中加入 :

```html
<!-- include libraries(jQuery) -->
<script src="https://code.jquery.com/jquery-3.4.1.min.js" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js" integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo" crossorigin="anonymous"></script>
<!-- include summernote css/js -->
<link href="https://cdn.jsdelivr.net/npm/summernote@0.8.16/dist/summernote-bs4.min.css" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/summernote@0.8.16/dist/summernote-bs4.min.js"></script>
```

---

## 8. 頁面新增

`create.blade.php` :

```html
@extends('_layouts.manage.app')
@section('title', trans('Page').trans('Create'))
@section('content')
<div class="container-fluid">
    <div class="row justify-content-center">
        <div class="col-md-12">
            <div class="card">
                <form action="{% raw %}{{ route('page.store') }}{% endraw %}" method="POST">
                    <div class="card-header">{% raw %}{{ trans('Page').trans('Create') }}{% endraw %}</div>
                    <div class="card-body">
                        <ul class="list-unstyled">
                            <li>{% raw %}{{ App\Button::GoBack(route('page.index')) }}{% endraw %}</li>
                        </ul>
                        @csrf
                        <div class="form-group row">
                            <label for="title" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Title') }}{% endraw %}</label>
                            <div class="col-md-6">
                                <input type="text" class="form-control @error('title') is-invalid @enderror" id="title" name="title" value="{% raw %}{{ old('title') }}{% endraw %}" placeholder="{% raw %}{{ trans('Title') }}{% endraw %}">
                                @error('title')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>
                        <div class="form-group row">
                            <label for="url" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Page').trans('Url') }}{% endraw %}</label>
                            <div class="col-md-6">
                                <input type="text" class="form-control @error('url') is-invalid @enderror" id="url" name="url" value="{% raw %}{{ old('url') }}{% endraw %}" placeholder="{% raw %}{{ trans('Page').trans('Url') }}{% endraw %}">
                                @error('url')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>
                        <div class="form-group row">
                            <label for="content" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Content') }}{% endraw %}</label>
                            <div class="col-md-12">
                                <textarea id="summernote" name="editordata" class="form-control" >{!! old('content') !!}</textarea>
                            </div>
                        </div>
                        <div class="form-group row">
                            <label for="is_open" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Is_open') }}{% endraw %}</label>
                            <div class="form-inline col-md-6">
                                <div class="custom-control custom-radio custom-control-inline">
                                    <input class="custom-control-input" type="radio" name="is_open" id="is_open1" value="1">
                                    <label class="custom-control-label" for="is_open1">{% raw %}{{ trans('Yes') }}{% endraw %}</label>
                                </div>
                                <div class="custom-control custom-radio custom-control-inline">
                                    <input class="custom-control-input" type="radio" name="is_open" id="is_open2" value="0">
                                    <label class="custom-control-label" for="is_open2">{% raw %}{{ trans('No') }}{% endraw %}</label>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="card-footer text-center">
                        <input type="submit" class="btn btn-primary" value="{% raw %}{{ trans('Create') }}{% endraw %}">
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>
@endsection
@section('script')
@parent
<script type="text/javascript">
    $(document).ready(function() {
      $('#summernote').summernote();
      $('.note-btn').removeAttr('title');
    });
</script>
@show
```

`PageController.php` :

```php
public function create()
{
    if (Auth::check() && Auth::user()->permission < '2') {
        return back()->with('warning', '權限不足以訪問該頁面 !');
    }

    return view('manage.page.create');
}

public function store(Request $request)
{
    if (Auth::check() && Auth::user()->permission < '2') {
        return back()->with('warning', '權限不足以訪問該頁面 !');
    }
    $error = 0;
    $page = new Page;
    $data = $request->validate([
        'menu_id' => ['nullable'],
        'title' => ['required', 'string', 'max:255'],
        'url' => ['required', 'string', 'max:255','unique:pages,url'],
        'is_open' => ['required'],
    ]);

    foreach ($request->except('_token', '_method', 'files') as $key => $value) {
        if ($request->filled($key) && $request->filled($key) != NULL && $key != 'content') {
            $page->$key = $data[$key];
            if ($page->$key == '') {
                $error += 1;
            }
        }
        else{
            $page->$key = NULL;
        }
    }

    $page->content = $request->input('content');
    $page->editor = Auth::user()->name;
    if ($error == 0) {
        $page->save();
    }
    else{
        return back()->withInput()->with('warning', '請確認輸入 !');
    }

    return back()->with('success','頁面新增成功 !');
}
```

---

## 9. 頁面修改

`edit.blade.php` :

```html
@extends('_layouts.manage.app')
@section('title', trans('Page').trans('Edit'))
@section('content')
<div class="container-fluid">
    <div class="row justify-content-center">
        <div class="col-md-12">
            <div class="card">
                <div class="card-header">{% raw %}{{ trans('Page').trans('Edit') }}{% endraw %}</div>

                <div class="card-body">
                    <ul class="list-unstyled">
                        <li>{% raw %}{{ App\Button::GoBack(route('page.index')) }}{% endraw %}</li>
                    </ul>
                    <form method="POST" action="{% raw %}{{ route('page.update' , $page->id) }}{% endraw %}">
                        @csrf
                        @method('PUT')

                        <div class="form-group row">
                            <label for="title" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Title') }}{% endraw %}</label>

                            <div class="col-md-6">
                                <input id="title" type="text" class="form-control @error('title') is-invalid @enderror" name="title" value="{% raw %}{{ $page->title }}{% endraw %}" required autocomplete="{% raw %}{{ trans('Title') }}{% endraw %}" autofocus>

                                @error('title')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>

                        <div class="form-group row">
                            <label for="url" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Page').trans('Url') }}{% endraw %}</label>

                            <div class="col-md-6">
                                <input id="url" type="text" class="form-control @error('url') is-invalid @enderror" name="url" value="{% raw %}{{ $page->url }}{% endraw %}" required autocomplete="{% raw %}{{ trans('Page').trans('Url') }}{% endraw %}" autofocus readonly>

                                @error('url')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>

                        <div class="form-group row">
                            <label for="content" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Content') }}{% endraw %}</label>

                            <div class="col-md-12">
                                <textarea id="summernote" name="content" class="form-control ckeditor" >{% raw %}{{ $page->content }}{% endraw %}</textarea>
                            </div>
                        </div>

                        <div class="form-group row">
                            <label for="is_open" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Is_open') }}{% endraw %}</label>
                            <div class="form-inline col-md-6">
                                <div class="custom-control custom-radio custom-control-inline">
                                    <input class="custom-control-input" type="radio" name="is_open" id="is_open1" value="1" {% raw %}{{ ($page->is_open=="1")? "checked" : "" }}{% endraw %}>
                                    <label class="custom-control-label" for="is_open1">{% raw %}{{ trans('Yes') }}{% endraw %}</label>
                                </div>
                                <div class="custom-control custom-radio custom-control-inline">
                                    <input class="custom-control-input" type="radio" name="is_open" id="is_open2" value="0" {% raw %}{{ ($page->is_open=="0")? "checked" : "" }}{% endraw %}>
                                    <label class="custom-control-label" for="is_open2">{% raw %}{{ trans('No') }}{% endraw %}</label>
                                </div>
                            </div>
                        </div>

                        <div class="form-group row">
                   <div class="col-md-4">
                       <input type="submit" class="btn btn-primary" value="送出">
                   </div>
               </div>
                 </form>
                </div>

            </div>
        </div>
    </div>
</div>
@endsection
@section('script')
@parent
<script type="text/javascript">
    $('#summernote').summernote();
    $('.note-btn').removeAttr('title');
</script>
@show
```

`PageController.php` :

```php
public function edit($id)
{
    if (Auth::check() && Auth::user()->permission < '3') {
        return back()->with('warning', '權限不足以訪問該頁面 !');
    }
    $page = Page::where('id',$id)->first();
    $navbars = Navbar::where('type', '=', '1')->get();
    return view('manage.page.edit',compact('page','navbars'));
}

public function update(Request $request, $id)
{
    if (Auth::check() && Auth::user()->permission < '3') {
        return back()->with('warning', '權限不足以訪問該頁面 !');
    }
    $error = 0;
    $page = Page::where('id',$id)->first();

    $data = $this->validate($request, [
        'menu_id' => ['nullable'],
        'title' => ['required', 'string', 'max:255'],
        'url' => ['required', 'string', 'max:255'],
        'is_open' => ['required'],
    ]);

    foreach ($request->except('_token','_method','files') as $key => $value) {
        if ($request->filled($key) && $request->filled($key) != NULL && $key != 'content') {
            $page->$key = $data[$key];
            if ($page->$key == '') {
                $error += 1;
            }
        }
        else{
            $page->$key = NULL;
        }
    }
    $page->content = $request->input('content');
    $page->editor = Auth::user()->name;

    if ($error == 0) {
        $page->save();
    }
    else{
        return back()->withInput()->with('warning', '請確認輸入 !');
    }
    return back()->with('success','修改頁面成功 !');
}
```

---

## 10. 頁面刪除

`PageController.php` :

```php
public function destroy($id)
{
    if (Auth::check() && Auth::user()->permission < '4') {
        return back()->with('warning', '權限不足以訪問該頁面 !');
    }
    Page::destroy($id);
    return back()->with('success','刪除頁面成功 !');
}
```

---

## 11. 結語

叮叮結束😁

下次來聊聊前台顯示以及為什麼 `Controller` 的 `store()` 和 `update()` 都要寫這麼醜。

一般不都 `Page::create($request->all())` 跟 `Page::updateOrCreate($request->all())` 直接搞定嗎 ?

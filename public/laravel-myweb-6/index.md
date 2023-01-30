# Laravel Backend Control Website - 6


> Laravel 導覽列管理 - Navbar Manage

<!--more-->

## 前言

Hello 大家 ! [上一篇](https://jhuei.com/code/2020/04/30/laravel-myweb-5.html) 明確地將前後台利用權限分開，如此一來，後台控制前台的網站已經漸漸有了雛型。而這篇將完成導覽列的管理，完成後即可從後台新增、修改前台所顯示的導覽列，並且使用拖曳的方式進行排序 (可在手機端使用)。

---

## 1. 新增Controller、Model、Migration

```bash
// 一次性建立
php artisan make:model Navbar -mcr
```

---

## 2. 修改Migration

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateNavbarsTable extends Migration
{
    public function up()
    {
        Schema::create('navbars', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('name')->comment('導覽名稱');
            $table->string('link')->nullable()->comment('對外連結');
            $table->integer('type')->comment('類型');
            $table->integer('sort')->nullable()->comment('排序');
            $table->boolean('is_open')->default(true)->comment('是否開放');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('navbars');
    }
}
```

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

class Navbar extends Model
{
    protected $table = 'navbars';

    protected $primaryKey = 'id';

    protected $fillable = [
        "name", "sort", "link", "type", "is_open",
    ];
}
```

---

## 4. 加入路由

```php
Route::prefix('manage')->middleware('auth','admin')->group(function(){
    Route::resource('member', 'MemberController');
    Route::resource('navbar', 'NavbarController');
});
```

---

## 5. 建立視圖

```treeview
views/
├── _layouts/
├── _partials/
├── auth/
└── manage  /    
    └── member  
    └── navbar  
        ├── create.blade.php 
        ├── edit.blade.php
        └── index.blade.php
```

---

## 6. 導覽列新增

先建立導覽列管理首頁 `index.blade.php` :

```php
@extends('_layouts.manage.app')
@section('title', trans('Navbar').trans('Manage'))
@section('content')
<div class="container-fluid">
    <div class="row justify-content-center">
        <div class="col-md-12">
            <div class="card">
                <div class="card-header">{% raw %}{{ trans('Navbar').trans('Manage') }}{% endraw %}</div>

                <div class="card-body">
                    <ul class="list-inline">
                        <li class="list-inline-item">{% raw %}{{ App\Button::Create() }}{% endraw %}</li>
                    </ul>
                    <div class="alert alert-warning" role="alert">
                        1. 新增頁面連結連結請照{% raw %}{{Request::root()}}{% endraw %}/article/導覽列名稱/選單名稱?頁面網址。<br>
                        2. 外部連結請直接貼上整段網址即可。
                    </div>
                    <div class="table-responsive">
                        <table id="data" class="table table-hover table-bordered text-center">
                            <thead>
                                <tr class="table-info active">
                                    <th class="text-nowrap text-center">{% raw %}{{ trans('Navbar').trans('Name') }}{% endraw %}</th>
                                    <th class="text-nowrap text-center">{% raw %}{{ trans('Link') }}{% endraw %}</th>
                                    <th class="text-nowrap text-center">{% raw %}{{ trans('Type') }}{% endraw %}</th>
                                    <th class="text-nowrap text-center">{% raw %}{{ trans('Sort') }}{% endraw %}</th>
                                    <th class="text-nowrap text-center">{% raw %}{{ trans('Is_open') }}{% endraw %}</th>
                                    <th class="text-nowrap text-center">{% raw %}{{ trans('Action') }}{% endraw %}</th>
                                </tr>
                            </thead>
                            <tbody>
                                @foreach ($all_navbars as $navbar)
                                    <tr>
                                        <td>{% raw %}{{ $navbar->name }}{% endraw %}</td>
                                        <td>{% raw %}{{ $navbar->link }}{% endraw %}</td>
                                        <td>{% raw %}{{App\Enum::type['navbar'][$navbar->type]}}{% endraw %}</td>
                                        <td>{% raw %}{{ $navbar->sort }}{% endraw %}</td>
                                        <td><font color="{% raw %}{{App\Enum::is_open['color'][$navbar->is_open]}}{% endraw %}"><i class="fas fa-{% raw %}{{App\Enum::is_open['label'][$navbar->is_open]}}{% endraw %}"></i></font></td>
                                        <td>
                                            <form action="{% raw %}{{ route('navbar.edit',$navbar->id) }}{% endraw %}" method="GET">
                                            @csrf
                                            {% raw %}{{ App\Button::edit($navbar->id) }}{% endraw %}
                                            </form>
                                            <form action="{% raw %}{{ route('navbar.destroy',$navbar->id) }}{% endraw %}" method="POST">
                                            @method('DELETE')
                                            @csrf
                                            {% raw %}{{ App\Button::deleting($navbar->id) }}{% endraw %}
                                            </form>
                                        </td>
                                    </tr>
                                @endforeach
                            </tbody>
                        </table>
                    </div>
                </div>
                {% raw %}{{-- <div class="card-footer pagination justify-content-center">
                    {!! $all_navbars->links("pagination::bootstrap-4") !!}
                </div> --}}{% endraw %}
            </div>
        </div>
    </div>
</div>
@endsection
```

`NavbarController.php` :

```php
public function index()
{
    $all_navbars = Navbar::all();
    return view('manage.navbar.index',compact('all_navbars'));
}
```

`create.blade.php` :

```php
@extends('_layouts.manage.app')
@section('title', trans('Navbar').trans('Create'))
@section('content')
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-12">
            <div class="card">
                <form action="{% raw %}{{ route('navbar.store') }}{% endraw %}" method="POST">
                    <div class="card-header">{% raw %}{{ trans('Navbar').trans('Create') }}{% endraw %}</div>
                    <div class="card-body">
                        <ul class="list-unstyled">
                            <li>{% raw %}{{ App\Button::GoBack(route('navbar.index')) }}{% endraw %}</li>
                        </ul>
                        @csrf
                        <div class="form-group row">
                            <label for="name" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Navbar').trans('Name') }}{% endraw %}</label>
                            <div class="col-md-6">
                                <input type="text" class="form-control @error('name') is-invalid @enderror" id="name" name="name" value="{% raw %}{{ old('name') }}{% endraw %}" placeholder="{% raw %}{{ trans('Navbar').trans('Name') }}{% endraw %}">
                                @error('name')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>
                        <div class="form-group row">
                            <label for="link" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Link') }}{% endraw %}</label>
                            <div class="col-md-6">
                                <input type="text" class="form-control @error('link') is-invalid @enderror" id="link" name="link" value="{% raw %}{{ old('link') }}{% endraw %}" placeholder="{% raw %}{{ trans('Link') }}{% endraw %}">
                                @error('link')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>
                        <div class="form-group row">
                            <label for="type" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Type') }}{% endraw %}</label>
                            <div class="col-md-6">
                                <select class='form-control' name='type' required aria-describedby="typeHelp">
                                    <option value=''>請選擇類型</option>
                                    @foreach(App\Enum::type['navbar'] as $key => $value)
                                        <option value='{% raw %}{{ $key }}{% endraw %}'>{% raw %}{{ $key }}{% endraw %}.{% raw %}{{ $value }}{% endraw %}</option>
                                    @endforeach
                                </select>
                                <span id="typeHelp" class="form-text text-muted">
                                    導覽目錄：顯示選單目錄；</br>
                                    一般頁面：不顯示選單目錄，直接列出底下的選單及頁面；</br>
                                </span>
                                @error('url')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>
                        <div class="form-group row">
                            <label for="is_open" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Is_open') }}{% endraw %}</label>
                            <div class="col-md-6">
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
```

`NavbarController.php` :

```php
public function create()
{
    if (Auth::check() && Auth::user()->permission < '2') {
        return back()->with('warning', '權限不足以訪問該頁面 !');
    }
    return view('manage.navbar.create');
}

public function store(Request $request)
{
    if (Auth::check() && Auth::user()->permission < '2') {
        return back()->with('warning', '權限不足以訪問該頁面 !');
    }
    $error = 0;
    $navbar = new Navbar;

    $data = $request->validate([
        'name' => ['required', 'string', 'max:255','unique:navbars,name'],
        'type' => ['required'],
        'link' => ['nullable'],
        'is_open' => ['required'],
    ]);

    foreach ($request->except('_token', '_method') as $key => $value) {
        if ($request->filled($key) && $request->filled($key) != NULL) {
            $navbar->$key = $data[$key];
            if ($navbar->$key == '') {
                $error += 1;
            }
        }
        else{
            $navbar->$key = NULL;
        }
    }

    if ($error == 0) {
        $navbar->save();
    }
    else{
        return back()->withInput()->with('warning', '請確認輸入 !');
    }

    return back()->with('success', '導覽列新增成功 !');
}
```

---

## 7. 導覽列修改

`edit.blade.php` :

```php
@extends('_layouts.manage.app')
@section('title', trans('Navbar').trans('Edit'))
@section('content')
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-12">
            <div class="card">
                <div class="card-header">{% raw %}{{ trans('Navbar').trans('Edit') }}{% endraw %}</div>

                <div class="card-body">
                    <ul class="list-unstyled">
                        <li>{% raw %}{{ App\Button::GoBack(route('navbar.index')) }}{% endraw %}</li>
                    </ul>
                    <form method="POST" action="{% raw %}{{ route('navbar.update' , $navbar->id) }}{% endraw %}">
                        @csrf
                        @method('PUT')
                        <div class="form-group row">
                            <label for="name" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Navbar').trans('Name') }}{% endraw %}</label>
                            <div class="col-md-6">
                                <input id="name" type="text" class="form-control @error('name') is-invalid @enderror" name="name" value="{% raw %}{{ $navbar->name }}{% endraw %}" required autocomplete="{% raw %}{{ trans('Page name') }}{% endraw %}" autofocus>
                                @error('name')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>

                        <div class="form-group row">
                            <label for="link" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Link') }}{% endraw %}</label>

                            <div class="col-md-6">
                                <input id="link" type="text" class="form-control @error('link') is-invalid @enderror" name="link" value="{% raw %}{{ $navbar->link }}{% endraw %}" autocomplete="{% raw %}{{ trans('Link') }}{% endraw %}" autofocus>

                                @error('link')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>

                        <div class="form-group row">
                            <label for="type" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Type') }}{% endraw %}</label>
                            <div class="col-md-6">
                                <select class="form-control @error('type') is-invalid @enderror" name='type' required aria-describedby="typeHelp">
                                    @foreach(App\Enum::type['navbar'] as $key => $value)
                                        @if ($key == $navbar->type)
                                            <option value='{% raw %}{{ $key }}{% endraw %}' selected>{% raw %}{{ $value }}{% endraw %}</option>
                                        @else
                                            <option value='{% raw %}{{ $key }}{% endraw %}'>{% raw %}{{ $value }}{% endraw %}</option>
                                        @endif
                                    @endforeach
                                </select>
                                <span id="typeHelp" class="form-text text-muted">
                                    導覽目錄：顯示選單目錄；</br>
                                    一般頁面：不顯示選單目錄，直接列出底下的頁面；</br>
                                </span>
                                @error('type')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>

                        <div class="form-group row">
                            <label for="is_open" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Is_open') }}{% endraw %}</label>
                            <div class="col-md-6">
                                <div class="custom-control custom-radio custom-control-inline">
                                    <input class="custom-control-input" type="radio" name="is_open" id="is_open1" value="1" {% raw %}{{ ($navbar->is_open=="1")? "checked" : "" }}{% endraw %}>
                                    <label class="custom-control-label" for="is_open1">{% raw %}{{ trans('Yes') }}{% endraw %}</label>
                                </div>
                                <div class="custom-control custom-radio custom-control-inline">
                                    <input class="custom-control-input" type="radio" name="is_open" id="is_open2" value="0" {% raw %}{{ ($navbar->is_open=="0")? "checked" : "" }}{% endraw %}>
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
```

`NavbarController.php` :

```php
public function edit($id)
{
    if (Auth::check() && Auth::user()->permission < '3') {
        return back()->with('warning', '權限不足以訪問該頁面 !');
    }
    $navbar = Navbar::where('id',$id)->first();
    return view('manage.navbar.edit',compact('navbar'));
}

public function update(Request $request, $id)
{
    if (Auth::check() && Auth::user()->permission < '3') {
        return back()->with('warning', '權限不足以訪問該頁面 !');
    }
    $error = 0;
    $navbar = Navbar::where('id', $id)->first();

    $data = $this->validate($request, [
        'name' => ['required', 'string', 'max:255'],
        'link' => [ 'nullable'],
        'type' => ['required'],
        'is_open' => ['required'],
    ]);

    foreach ($request->except('_token', '_method') as $key => $value) {
        if ($request->filled($key) && $request->filled($key) != NULL) {
            $navbar->$key = $data[$key];
            if ($navbar->$key == '') {
                $error += 1;
            }
        }
        else{
            $navbar->$key = NULL;
        }
    }
    if ($request->filled('link') == null) {
        $navbar->link = NULL;
    }

    if ($error == 0) {
        $navbar->save();
    }
    else{
        return back()->withInput()->with('warning', '請確認輸入 !');
    }

    return back()->with('success', '修改導覽列成功 !');
}
```

---

## 8. 導覽列刪除

`NavbarController.php` :

```php
public function destroy($id)
{
    if (Auth::check() && Auth::user()->permission < '4') {
        return back()->with('warning', '權限不足以訪問該頁面 !');
    }
    Navbar::destroy($id);
    return back()->with('success', '刪除導覽列成功 !');
}
```

---

## 9. 導覽列前台顯示

`web.php`

```php
//在各視圖中可直接使用以下參數
View::composer(['*'], function ($view) {
    $navbars = App\Navbar::where('is_open',1)->orderby('sort')->get();
    $users = App\User::all();

    $view->with('navbars',$navbars);
    $view->with('users',$users);
});
```

`home/nav.blade.php` :

```html
<nav class="navbar navbar-expand-md navbar-light bg-white shadow-sm">
    <div class="container">
        <a class="navbar-brand" href="{% raw %}{{ url('/') }}{% endraw %}">
            {% raw %}{{ config('app.name', 'Laravel') }}{% endraw %}
        </a>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="{% raw %}{{ trans('Toggle navigation') }}{% endraw %}">
            <span class="navbar-toggler-icon"></span>
        </button>

        <div class="collapse navbar-collapse" id="navbarSupportedContent">
            <!-- Left Side Of Navbar -->
            <ul class="navbar-nav mr-auto">
                @foreach ($navbars as $navbar)
                    @switch($navbar->type)
                        @case(1)
                            <li class="nav-item dropdown">
                                <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
                                {% raw %}{{ $navbar->name }}{% endraw %}
                                </a>
                                <div class="dropdown-menu" aria-labelledby="navbarDropdown">
                                <!-- 期待第12篇 -->
                                </div>
                            </li>
                            @break
                        @case(2)
                            <li class="nav-item" >
                                <a class="nav-link" href="{% raw %}{{ $navbar->link }}{% endraw %}">{% raw %}{{ $navbar->name }}{% endraw %}</a>
                            </li>
                            @break
                    @endswitch
                @endforeach
            </ul>

            <!-- Right Side Of Navbar -->
            <ul class="navbar-nav ml-auto">
                <!-- Authentication Links -->
                @guest
                    <li class="nav-item">
                        <a class="nav-link" href="{% raw %}{{ route('login') }}{% endraw %}">{% raw %}{{ trans('Login') }}{% endraw %}</a>
                    </li>
                    @if (Route::has('register'))
                        <li class="nav-item">
                            <a class="nav-link" href="{% raw %}{{ route('register') }}{% endraw %}">{% raw %}{{ trans('Register') }}{% endraw %}</a>
                        </li>
                    @endif
                @else
                @if (Auth::user()->permission != 0)
                    <li class="nav-item">
                        <a class="nav-link" href="{% raw %}{{ route('manage') }}{% endraw %}">{% raw %}{{ trans('Backstage') }}{% endraw %}</a>
                    </li>
                @endif
                    <li class="nav-item dropdown">
                        <a id="navbarDropdown" class="nav-link dropdown-toggle" href="#" role="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false" v-pre>
                            {% raw %}{{ Auth::user()->name }}{% endraw %} <span class="caret"></span>
                        </a>

                        <div class="dropdown-menu dropdown-menu-right" aria-labelledby="navbarDropdown">
                            <a class="dropdown-item" href="{% raw %}{{ route('logout') }}{% endraw %}"
                               onclick="event.preventDefault();
                                             document.getElementById('logout-form').submit();">
                                {% raw %}{{ trans('Logout') }}{% endraw %}
                            </a>

                            <form id="logout-form" action="{% raw %}{{ route('logout') }}{% endraw %}" method="POST" style="display: none;">
                                @csrf
                            </form>
                        </div>
                    </li>
                @endguest
            </ul>
        </div>
    </div>
</nav>
```

---

## 10. 補充

照著上面作，沒意外前台應該會顯示後台所新增的導覽列。
[![navbar](https://i.imgur.com/9pX0tZD.png)](https://i.imgur.com/9pX0tZD.png)

另外你必須在 `Enum.php` 新增相關參數如下 :

```php
// 是否開放(是=綠勾，否=紅叉)
const is_open = [
    'color' =>[
        '0' => 'red',
        '1' => 'green',
    ],
    'label'=>[
        '0' => 'times',
        '1' => 'check',
    ],

];
// 導覽列類型
const type =[
    'navbar' => [
        '1' => '導覽目錄',
        '2' => '一般頁面',
    ],
];
```

並且在 `_layouts/manage/app.blade.php` 新增 `fontawesome` ，不管你是要用`CDN`或者引入 :

```html
<link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.7.0/css/all.css" integrity="sha384-lZN37f5QGtY3VHgisS14W3ExzMWZxybE1SJSEsQp9S+oqd12jhcu+A56Ebc1zFSJ" crossorigin="anonymous">
```

另外關於翻譯請自行在`zh-tw.json` 或 `zh-tw/myweb.php` 加入。

下一篇會解答如何`拖曳排序`導覽列以及 `View::composer`。

叮叮，Hope it will help😋


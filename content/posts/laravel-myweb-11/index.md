---
title: Laravel Backend Control Website - 11
categories: ['CODE']
description: 選單管理 - Menu Manage
tags: ['laravel', 'php', 'html', 'javascript', 'menu', 'htmlpurifier', 'page']
authors: ["Kerwin"]
featuredImagePreview: feature.jpg
date: 2020-06-10
lastmod: 2020-06-10

series: ["laravel-myweb"]
series_weight: 11
---

> 選單管理 - Menu Manage

<!--more-->

## 前言

Hello 大家！之前做過 `導覽列` 和 `頁面` 的CRUD，這篇就是要做最後的 `選單` 並將三者串連起來。

---

## 1. 新增Controller、Model、Migration

首先，建立 `Menu` 的三寶 :

```bash
// 一次性建立
php artisan make:model Menu -mcr
```

---

## 2. 修改Migration

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateMenusTable extends Migration
{
    public function up()
    {
        Schema::create('menus', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->integer('navbar_id')->comment('導覽列');
            $table->string('name')->comment('名稱');
            $table->string('link')->nullable()->comment('對外連結');
            $table->integer('sort')->nullable()->comment('排序');
            $table->boolean('is_list')->default(true)->comment('清單顯示');
            $table->boolean('is_open')->default(true)->comment('是否開放');
            $table->timestamps();
        });
    }
    public function down()
    {
        Schema::dropIfExists('menus');
    }
}

```

{{< admonition type=info title="" open=true >}}
⚠ 清單顯示的目的請看[下一篇](#11)
{{< /admonition >}}

建立資料表:

```bash
php artisan migrate
```

---

## 3. 修改Model

`Menu.php` :

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Menu extends Model
{
    protected $table = 'menus';

    protected $primaryKey = 'id';

    protected $fillable = [
        "name", "navbar_id", "sort", "link", "is_list", "is_open",
    ];
}

```

---

## 4. 加入路由

```php
Route::prefix('manage')->middleware('auth','admin')->group(function(){
    ...略...
    Route::resource('menu', 'MenuController');
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
    └── log/
    └── member/  
    └── menu/  
        ├── create.blade.php
        ├── edit.blade.php
        ├── index.blade.php
        └── sort.blade.php
    └── navbar/  
    └── page/
```

---

## 6. 選單首頁

`MenuController.php` :

```php
<?php

namespace App\Http\Controllers;

use App\Menu;
use App\Navbar;
use App\Page;
use App\Log;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Auth;

class MenuController extends Controller
{
    public function index()
    {
        $all_menus = Menu::all();
        $all_navbars = Navbar::all();
        return view('manage.menu.index',compact('all_menus','all_navbars'));
    }

    ...略...
}
```

{{< admonition type=warning title="注意" open=true >}}
會引入 `all_navbars` 是因為選單可能會與導覽列連動。
{{< /admonition >}}

`index.blade.php` :

```html
@extends('_layouts.manage.app')
@section('title', trans('Menu').trans('Manage'))
@section('content')
<div class="container-fluid">
    <div class="row justify-content-center">
        <div class="col-md-12">
            <div class="card">
                <div class="card-header">{% raw %}{{ trans('Menu').trans('Manage') }}{% endraw %}</div>

                <div class="card-body">
     <ul class="list-inline">
      <li class="list-inline-item">{% raw %}{{ App\Button::Create() }}{% endraw %}</li>
      <li class="list-inline-item">{% raw %}{{ App\Button::To(true,'sort',trans('Sort'),'','btn-primary') }}{% endraw %}</li>
     </ul>
     <div class="alert alert-warning" role="alert">
                        若有建立連結，點擊選單則直接跳到該連結頁面。<br>
                    </div>
     <div class="table-responsive">
      <table id="data" class="table table-hover table-bordered text-center">
                   <thead>
                    <tr class="table-info active">
                     <th class="text-nowrap text-center">{% raw %}{{ trans('Menu').trans('Name') }}{% endraw %}</th>
                     <th class="text-nowrap text-center">{% raw %}{{ trans('Navbar') }}{% endraw %}</th>
                     <th class="text-nowrap text-center">{% raw %}{{ trans('Link') }}{% endraw %}</th>
                     <th class="text-nowrap text-center">{% raw %}{{ trans('Sort') }}{% endraw %}</th>
                     <th class="text-nowrap text-center">{% raw %}{{ trans('Is_list') }}{% endraw %}</th>
                     <th class="text-nowrap text-center">{% raw %}{{ trans('Is_open') }}{% endraw %}</th>
                     <th class="text-nowrap text-center">{% raw %}{{ trans('Action') }}{% endraw %}</th>
                    </tr>
                   </thead>
                   <tbody>
        @foreach ($all_menus as $menu)
         <tr>
          <td>{% raw %}{{ $menu->name }}{% endraw %}</td>
          <td>{% raw %}{{ App\Navbar::where('id','=',$menu->navbar_id)->first('name')['name'] }}{% endraw %}</td>
          <td>{% raw %}{{ $menu->link }}{% endraw %}</td>
          <td>{% raw %}{{ $menu->sort }}{% endraw %}</td>
          <td><font color="{% raw %}{{App\Enum::is_open['color'][$menu->is_list]}}{% endraw %}"><i class="fas fa-{% raw %}{{App\Enum::is_open['label'][$menu->is_list]}}{% endraw %}"></i></font></td>
          <td><font color="{% raw %}{{App\Enum::is_open['color'][$menu->is_open]}}{% endraw %}"><i class="fas fa-{% raw %}{{App\Enum::is_open['label'][$menu->is_open]}}{% endraw %}"></i></font></td>
          <td>
           <form action="{% raw %}{{ route('menu.edit',$menu->id) }}{% endraw %}" method="GET">
           @csrf
           {% raw %}{{ App\Button::edit($menu->id) }}{% endraw %}
           </form>
           <form action="{% raw %}{{ route('menu.destroy',$menu->id) }}{% endraw %}" method="POST">
           @method('DELETE')
           @csrf
           {% raw %}{{ App\Button::deleting($menu->id) }}{% endraw %}
           </form>
          </td>
         </tr>
                    @endforeach
                   </tbody>
                  </table>
     </div>
                </div>
                <!-- <div class="card-footer pagination justify-content-center">
					{!! $all_menus->links("pagination::bootstrap-4") !!}
				</div> -->
            </div>
        </div>
    </div>
</div>
@endsection

```

如要分頁就將 `footer` 取消註解，並將 `controller` 改成 :

```php
public function index()
{
    $all_menus = Menu::paginate();
    $all_navbars = Navbar::all();
    return view('manage.menu.index',compact('all_menus','all_navbars'));
}
```

---

## 7. 選單新增

`MenuController.php` :

```php
<?php
...略...
public function create()
{
    if (Auth::check() && Auth::user()->permission < '2') {
        return back()->with('warning', '權限不足以訪問該頁面 !');
    }
    return view('manage.menu.create');
}
    
    public function store(Request $request)
{
    if (Auth::check() && Auth::user()->permission < '2') {
        return back()->with('warning', '權限不足以訪問該頁面 !');
    }
    $error = 0;
    $menu = new Menu;

    $data = $request->validate([
        'name' => ['required', 'string', 'max:255','unique:menus,name'],
        'navbar_id' => ['required','integer'],
        'link' => ['nullable'],
        'is_open' => ['required'],
        'is_list' => ['required'],
    ]);

    foreach ($request->except('_token', '_method') as $key => $value) {
        if ($request->filled($key) && $request->filled($key) != NULL) {
            $menu->$key = strip_tags(clean($data[$key]));
            if ($menu->$key == '') {
                $error += 1;
            }
        }
        else{
            $menu->$key = NULL;
        }
    }

    if ($error == 0) {
        // 寫入log
        Log::write_log('menus',$request->all());
        $menu->save();
    }
    else{
        return back()->withInput()->with('warning', '請確認輸入 !');
    }
    // 寫入log
    Log::write_log('menus',$request->all());

    return back()->with('success', '選單新增成功 !');
}
...略...
```

`create.blade.php` :

```html
@extends('_layouts.manage.app')
@section('title', trans('Menu').trans('Create'))
@section('content')
<div class="container-fluid">
    <div class="row justify-content-center">
        <div class="col-md-12">
            <div class="card">
                <form action="{% raw %}{{ route('menu.store') }}{% endraw %}" method="POST">
                    <div class="card-header">{% raw %}{{ trans('Menu').trans('Create') }}{% endraw %}</div>
                    <div class="card-body">
                        <ul class="list-unstyled">
                            <li>{% raw %}{{ App\Button::GoBack(route('menu.index')) }}{% endraw %}</li>
                        </ul>
                        @csrf
                        <div class="form-group row">
                            <label for="name" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Menu').trans('Name') }}{% endraw %}</label>
                            <div class="col-md-4">
                                <input type="text" class="form-control @error('name') is-invalid @enderror" id="name" name="name" value="{% raw %}{{ old('name') }}{% endraw %}" placeholder="{% raw %}{{ trans('Menu').trans('Name') }}{% endraw %}" required>
                                @error('name')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>
                        <div class="form-group row">
                            <label for="navbar_id" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Navbar') }}{% endraw %}</label>
                            <div class="col-md-4">
                                <select class='form-control' name='navbar_id' required aria-describedby="navHelp" required>
                                    <option value=''>請選擇導覽列</option>
                                    @foreach($navbars as $key => $value)
                                        <option value="{% raw %}{{ $value['id'] }}{% endraw %}">{% raw %}{{ $value['name'] }}{% endraw %}</option>
                                    @endforeach
                                </select>
                                <span id="navHelp" class="form-text text-muted">
                                    選擇要加入的導覽列項目。
                                </span>
                            </div>
                        </div>
                        <div class="form-group row">
                            <label for="link" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Link') }}{% endraw %}</label>
                            <div class="col-md-4">
                                <input type="text" class="form-control @error('link') is-invalid @enderror" id="link" name="link" value="{% raw %}{{ old('link') }}{% endraw %}" placeholder="{% raw %}{{ trans('Link') }}{% endraw %}">
                                @error('link')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>
                        <div class="form-group row">
                            <label for="is_list" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Is_list') }}{% endraw %}</label>
                            <div class="col-md-6">
                                <div class="custom-control custom-radio custom-control-inline">
                                    <input class="custom-control-input" type="radio" name="is_list" id="is_list1" value="1">
                                    <label class="custom-control-label" for="is_list1">{% raw %}{{ trans('Yes') }}{% endraw %}</label>
                                </div>
                                <div class="custom-control custom-radio custom-control-inline">
                                    <input class="custom-control-input" type="radio" name="is_list" id="is_list2" value="0">
                                    <label class="custom-control-label" for="is_list2">{% raw %}{{ trans('No') }}{% endraw %}</label>
                                </div>
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

{{< admonition type=info title="" open=true >}}
選單新增中的導覽列 `navbars` 使用的是 `web.php` 中的 `View:composer` ，僅有開放中的導覽列才能選擇。
{{< /admonition >}}

---

## 8. 選單更新

`MenuController.php` :

```php
...略...
public function edit($id)
{
    if (Auth::check() && Auth::user()->permission < '3') {
        return back()->with('warning', '權限不足以訪問該頁面 !');
    }

    $menu = Menu::where('id',$id)->first();

    return view('manage.menu.edit',compact('menu'));
}

public function update(Request $request, $id)
{
    if (Auth::check() && Auth::user()->permission < '3') {
        return back()->with('warning', '權限不足以訪問該頁面 !');
    }
    $error = 0;
    $menu = Menu::where('id', $id)->first();

    $data = $this->validate($request, [
        'name' => ['required', 'string', 'max:255'],
        'navbar_id' => ['required','integer'],
        'link' => ['nullable','string'],
        'is_open' => ['required','boolean'],
        'is_list' => ['required','boolean'],
    ]);

    // 逐筆進行htmlpurufier 並去掉<p></p>
    foreach ($request->except('_token', '_method') as $key => $value) {
        if ($request->filled($key) && $request->filled($key) != NULL) {
            $menu->$key = strip_tags(clean($data[$key]));
            if ($menu->$key == '') {
                $error += 1;
            }
        }
        else{
            $menu->$key = NULL;
        }
    }

    if ($error == 0) {
        // 寫入log
        Log::write_log('menus',$request->all());
        $menu->save();
    }
    else{
        return back()->withInput()->with('warning', '請確認輸入 !');
    }

    return back()->with('success', '修改選單成功 !');
}
...略...
```

`edit.blade.php` :

```html
@extends('_layouts.manage.app')
@section('title', trans('Menu').trans('Edit'))
@section('content')
<div class="container-fluid">
    <div class="row justify-content-center">
        <div class="col-md-12">
            <div class="card">
                <div class="card-header">{% raw %}{{ trans('Menu').trans('Edit') }}{% endraw %}</div>

                <div class="card-body">
                 <ul class="list-unstyled">
      <li>{% raw %}{{ App\Button::GoBack(route('menu.index')) }}{% endraw %}</li>
     </ul>
                 <form method="POST" action="{% raw %}{{ route('menu.update' , $menu->id) }}{% endraw %}">
                  @csrf
      @method('PUT')
      <div class="form-group row">
                            <label for="name" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Menu').trans('Name') }}{% endraw %}</label>
                            <div class="col-md-6">
                                <input id="name" type="text" class="form-control @error('name') is-invalid @enderror" name="name" value="{% raw %}{{ $menu->name }}{% endraw %}" required autocomplete="{% raw %}{{ trans('Menu').trans('Name') }}{% endraw %}" autofocus readonly>
                                @error('name')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>

                        <div class="form-group row">
                            <label for="navbar_id" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Navbar') }}{% endraw %}</label>
                            <div class="col-md-6">
                                <select class='form-control' name='navbar_id' required aria-describedby="navHelp">
                                    @foreach($navbars as $key => $value)
                                        @if ($value['id'] == $menu->navbar_id)
                                            <option value="{% raw %}{{ $value['id'] }}{% endraw %}" selected>{% raw %}{{ $value['name'] }}{% endraw %}</option>
                                        @else
                                            <option value="{% raw %}{{ $value['id'] }}{% endraw %}">{% raw %}{{ $value['name'] }}{% endraw %}</option>
                                        @endif
                                    @endforeach
                                </select>
                                <span id="navHelp" class="help-block">
                                    選擇要加入的導覽列項目。
                                </span>
                            </div>
                        </div>

                        <div class="form-group row">
                            <label for="link" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Link') }}{% endraw %}</label>

                            <div class="col-md-6">
                                <input id="link" type="text" class="form-control @error('link') is-invalid @enderror" name="link" value="{% raw %}{{ $menu->link }}{% endraw %}" autocomplete="{% raw %}{{ trans('Link') }}{% endraw %}" autofocus>

                                @error('link')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>

                        <div class="form-group row">
                            <label for="is_list" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Is_list') }}{% endraw %}</label>
                            <div class="col-md-6">
                                <div class="custom-control custom-radio custom-control-inline">
                                    <input class="custom-control-input" type="radio" name="is_list" id="is_list1" value="1" {% raw %}{{ ($menu->is_list=="1")? "checked" : "" }}{% endraw %}>
                                    <label class="custom-control-label" for="is_list1">{% raw %}{{ trans('Yes') }}{% endraw %}</label>
                                </div>
                                <div class="custom-control custom-radio custom-control-inline">
                                    <input class="custom-control-input" type="radio" name="is_list" id="is_list2" value="0" {% raw %}{{ ($menu->is_list=="0")? "checked" : "" }}{% endraw %}>
                                    <label class="custom-control-label" for="is_list2">{% raw %}{{ trans('No') }}{% endraw %}</label>
                                </div>
                            </div>
                        </div>

                        <div class="form-group row">
                            <label for="is_open" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Is_open') }}{% endraw %}</label>
                            <div class="col-md-6">
                                <div class="custom-control custom-radio custom-control-inline">
                                    <input class="custom-control-input" type="radio" name="is_open" id="is_open1" value="1" {% raw %}{{ ($menu->is_open=="1")? "checked" : "" }}{% endraw %}>
                                    <label class="custom-control-label" for="is_open1">{% raw %}{{ trans('Yes') }}{% endraw %}</label>
                                </div>
                                <div class="custom-control custom-radio custom-control-inline">
                                    <input class="custom-control-input" type="radio" name="is_open" id="is_open2" value="0" {% raw %}{{ ($menu->is_open=="0")? "checked" : "" }}{% endraw %}>
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

---

## 9. 選單刪除

`MenuController.php` :

```php
...略...
public function destroy($id)
{
    if (Auth::check() && Auth::user()->permission < '4') {
        return back()->with('warning', '權限不足以訪問該頁面 !');
    }

    // 寫入log
    Log::write_log('menus',Menu::where('id', $id)->first());

    Menu::destroy($id);
    return back()->with('success', '刪除選單成功 !');
}
...略...
```

---

## 10. 選單排序

{{< admonition type=info title="" open=true >}}
和導覽列排序相同，只有開放中的選單可以排序。
{{< /admonition >}}

新增路由 :

```php
//排序、中介層:登入/管理員
Route::middleware('auth','admin')->group(function() {
    Route::post('menu-sortable','MenuController@sort')->name('menu.sort');
    Route::get('/manage/menu/sort', function () {return view('manage.menu.sort');});
});

//在各視圖中可直接使用以下參數
View::composer(['*'], function ($view) {
    $menus = App\Menu::where('is_open',1)->orderby('sort')->get();
    $view->with('menus',$menus);
});

```

`sort.blade.php` :

```html
@extends('_layouts.manage.app')
@section('title', trans('Menu').trans('Sort'))
@section('content')
<div class="container-fluid">
    <div class="row justify-content-center">
        <div class="col-md-12">
            <div class="card">
                <div class="card-header">{% raw %}{{trans('Menu').trans('Sort')}}{% endraw %}</div>

                <div class="card-body">
     <ul class="list-unstyled">
      <li>{% raw %}{{ App\Button::GoBack(route('menu.index')) }}{% endraw %}</li>
     </ul>
                    <div class="alert alert-warning" role="alert">
                        1. 請直接拖曳目標列進行排序。<br>
                        2. 調整完請點重新整理進行確認。
                    </div>
                 <table id="table" class="table table-hover table-bordered text-center">
                  <thead>
                   <tr class="table-info active">
                    <th class="text-nowrap text-center">{% raw %}{{ trans('Menu').trans('Name')}}{% endraw %}</th>
                    <th class="text-nowrap text-center">{% raw %}{{ trans('Navbar') }}{% endraw %}</th>
                    <th class="text-nowrap text-center">{% raw %}{{ trans('Sort') }}{% endraw %}</th>
                    <th class="text-nowrap text-center">{% raw %}{{ trans('Is_list') }}{% endraw %}</th>
                                <th class="text-nowrap text-center">{% raw %}{{ trans('Is_open') }}{% endraw %}</th>
                   </tr>
                  </thead>
                  <tbody id="tablecontents">
       @foreach ($menus as $menu)
        <tr class="row1" data-id="{% raw %}{{ $menu->id }}{% endraw %}">
         <td class="pl-3">{% raw %}{{ $menu->name }}{% endraw %}</td>
         <td>{% raw %}{{ App\Navbar::where('id','=',$menu->navbar_id)->first('name')['name'] }}{% endraw %}</td>
         <td>{% raw %}{{ $menu->sort }}{% endraw %}</td>
         <td><font color="{% raw %}{{App\Enum::is_open['color'][$menu->is_list]}}{% endraw %}"><i class="fas fa-{% raw %}{{App\Enum::is_open['label'][$menu->is_list]}}{% endraw %}"></i></font></td>
                                    <td><font color="{% raw %}{{App\Enum::is_open['color'][$menu->is_open]}}{% endraw %}"><i class="fas fa-{% raw %}{{App\Enum::is_open['label'][$menu->is_open]}}{% endraw %}"></i></font></td>
        </tr>
                   @endforeach
                  </tbody>
                    </table>
                </div>
                <div class="card-footer text-center">
                 <h5><button class="btn btn-success btn-sm" onclick="window.location.reload()">{% raw %}{{ trans('Refresh') }}{% endraw %}</button></h5>
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
@section('script')
@parent
<script type="text/javascript">
    $(function () {
        // $("#table").DataTable();

        $("#tablecontents").sortable({
            items: "tr",
            cursor: 'move',
            opacity: 0.6,
            update: function() {
                sendOrderToServer();
            }
        });

        function sendOrderToServer() {
            var order = [];
            var token = $('meta[name="csrf-token"]').attr('content');
            $('tr.row1').each(function(index,element) {
                order.push({
                    id: $(this).attr('data-id'),
                    position: index+1
                });
            });

            $.ajax({
                type: "POST",
                dataType: "json",
                url: "{% raw %}{{ url('menu-sortable') }}{% endraw %}",
                data: {
                    order: order,
                    _token: token
                },
                success: function(response) {
                    if (response.status == "success") {
                      console.log(response);
                    } else {
                      console.log(response);
                    }
                }
            });
        }
    });
</script>
@show

```

`MenuController.php` :

```php
...略...
//拖曳排序
public function sort(Request $request)
{
    if (Auth::check() && Auth::user()->permission < '3') {
        return back()->with('warning', '權限不足以訪問該頁面 !');
    }

    $menus = Menu::all();

    foreach ($menus as $menu) {
        foreach ($request->order as $order) {
            if ($order['id'] == $menu->id) {
                $menu->update(['sort' => $order['position']]);
            }
        }
    }
    // 寫入log
    $sort = DB::table('menus')->select('name','sort')->orderby('sort')->get();
    Log::write_log('menus',$sort,'排序');

    return response('Update Successfully.', 200);
}
...略...
```

---

## 11. 修改 nav.blade.php

```html
<nav class="navbar navbar-expand-md navbar-light bg-white shadow-sm">
...略...
    @case(1)
        <li class="nav-item dropdown">
            <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
            {% raw %}{{ $navbar->name }}{% endraw %}
            </a>
            <div class="dropdown-menu" aria-labelledby="navbarDropdown">
            @foreach ($menus as $menu)
                @if ($menu->navbar_id == $navbar->id && $menu->link)
                    <a class="dropdown-item" target="_blank" href="{% raw %}{{ $menu->link }}{% endraw %}">{% raw %}{{ $menu->name }}{% endraw %}</a>
                @elseif($menu->navbar_id == $navbar->id)
                    <a class="dropdown-item" href="/article/{% raw %}{{ $navbar->name }}{% endraw %}/{% raw %}{{ $menu->name }}{% endraw %}">{% raw %}{{ $menu->name }}{% endraw %}</a>
                @endif
            @endforeach
            </div>
        </li>
        @break
...略...
</nav>
```

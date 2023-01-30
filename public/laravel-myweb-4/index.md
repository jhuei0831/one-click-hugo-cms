# Laravel Backend Control Website - 4


> Laravel 會員管理 / CRUD - Laravel member system / CRUD

<!--more-->

## 前言

[第二篇](https://jhuei.com/code/2020/04/22/laravel-myweb-2.html)提到將layout切成前後台，這次就要來製作會員系統，並且設定不同的權限。

---

## 1. 會員新增

前台會員直接使用之前 `php artisan make:auth` 的註冊，後台的要另外建立，因為要設定權限的關係。

這裡 `Model`直接使用 `app/User.php`，然後建立 `Controller`。

```bash
php artisan make:controller MemberController --resource
```

指令成功之後可以在 `app/Http/controllers`底下看到剛剛建立的 `member.php`，並且裡面已經有基本的 CRUD function。

接下來因為要加入權限的關係，要打開 `database/migrations/2014_10_12_000000_create_users_table.php`，並將其改成以下:

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateUsersTable extends Migration
{

    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('name');
            $table->string('email')->unique();
            $table->integer('permission')->default('0');
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('users');
    }
}

```

{{< admonition type=info title="" open=true >}}
順帶一提，`up()`跟`down()`分別 就是建立與刪除。
{{< /admonition >}}

接著由於 `migration`更新的關係，所以資料庫也要一起改變:

```bash
php artisan migrate:fresh
```

有關更詳細的 `Migration`指令用法請詳閱 [Migration](https://laravel.com/docs/5.8/migrations)。

再來因為有了新的資料庫欄位，所以要在 `User.php` 中加入剛剛新增的欄位。

```php
<?php

namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements MustVerifyEmail
{
    use Notifiable;

    protected $fillable = [
        'name', 'email', 'password','permission',
    ];

    protected $hidden = [
        'password', 'remember_token',
    ];
  
    protected $casts = [
        'email_verified_at' => 'datetime',
    ];

}

```

做了這些前置作業之後，就可以到 `Controllers/member.php` 開始建立會員系統了。

先在在上方加入以下:

```php
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Validator;
use Illuminate\Support\Facades\DB;
use App\User;
```

這些都是等等會用到的指令，先將其引入。

在會員管理的首頁，要顯示出目前有的會員以及會員資料，所以在  `index()`我們將其改為 :

```php
public function index()
{
    //從資料表 `users` 將所以資料取出，並進行分頁
    $all_users = DB::table('users')->paginate(); 
    //將取出的資料放在manage/member/index
    return view('manage.member.index',compact('all_users')); 
}
```

{{< admonition type=info title="" open=true >}}
可以在分頁括弧中輸入想要一頁幾筆資料，預設是15筆。
{{< /admonition >}}

將 `create()` 改為 :

```php
// 目的就是顯示建立會員的頁面
return view('manage.member.create');
```

這時候要先來建立會員新增的頁面，在 `views` 底下根據下表建立檔案及資料夾 :

```treeview
views/
├── _layouts/
├── _partials/
├── auth/
└── manage /     
    └── member  
        ├── create.blade.php 
        ├── edit.blade.php
        └── index.blade.php
```

接著在 `./routes/web.php`中建立路由 :

```php
//因為之後還有其他功能要加入，所以先使用group
Route::prefix('manage')->group(function(){
    Route::resource('member', 'MemberController');
});
```

路由建立好後在 `index.blade.php`中加入 :

```php
@extends('_layouts.manage.app')
@section('title', trans('Member').trans('Manage'))
@section('content')
<div class="container-fluid">
    <div class="row justify-content-center">
        <div class="col-md-12">
            <div class="card">
                <div class="card-header">{% raw %}{{ trans('Member').trans('Manage') }}{% endraw %}</div>

                <div class="card-body">
                    <ul class="list-inline">
                        <li class="list-inline-item">{% raw %}{{ App\Button::Create() }}{% endraw %}</li>
                    </ul>
                    <div class="table-responsive">
                        <table id="data" class="table table-hover table-bordered text-center">
                            <thead>
                                <tr class="table-info active">
                                    <th class="text-nowrap text-center">{% raw %}{{ trans('Name') }}{% endraw %}</th>
                                    <th class="text-nowrap text-center">{% raw %}{{ trans('E-Mail Address') }}{% endraw %}</th>
                                    <th class="text-nowrap text-center">{% raw %}{{ trans('Permission') }}{% endraw %}</th>
                                    <th class="text-nowrap text-center">{% raw %}{{ trans('Action') }}{% endraw %}</th>
                                </tr>
                            </thead>
                            <tbody>
                            @foreach ($all_users as $user)
                                <tr>
                                    <td>{% raw %}{{ $user->name }}{% endraw %}</td>
                                    <td>{% raw %}{{ $user->email }}{% endraw %}</td>
                                    <td>{% raw %}{{App\Enum::permission[$user->permission]}}{% endraw %}</td>
                                    <td>
                                        <form action="{% raw %}{{ route('member.edit',$user->id) }}{% endraw %}" method="GET">
                                        @csrf
                                        {% raw %}{{ App\Button::edit($user->id) }}{% endraw %}
                                        </form>
                                        <form action="{% raw %}{{ route('member.destroy',$user->id) }}{% endraw %}" method="POST">
                                        @method('DELETE')
                                        @csrf
                                        {% raw %}{{ App\Button::deleting($user->id) }}{% endraw %}
                                        </form>
                                    </td>
                                </tr>
                            @endforeach
                            </tbody>
                        </table>
                    </div>
                </div>
                <div class="card-footer pagination justify-content-center">
                    {!! $all_users->links("pagination::bootstrap-4") !!}
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```

在 `create.blade.php` 加入 :

```php
@extends('_layouts.manage.app')
@section('title', trans('Member').trans('Create'))
@section('content')
<div class="container-fluid">
    <div class="row justify-content-center">
        <div class="col-md-12">
            <div class="card">
                <form action="{% raw %}{{ route('member.store') }}{% endraw %}" method="POST">
                    <div class="card-header">{% raw %}{{ trans('Member').trans('Create') }}{% endraw %}</div>
                    <div class="card-body">
                        <ul class="list-unstyled">
                            <li>{% raw %}{{ App\Button::GoBack(route('member.index')) }}{% endraw %}</li>
                        </ul>
                        @csrf
                        <div class="form-group row">
                            <label for="name" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Name') }}{% endraw %}</label>
                            <div class="col-md-6">
                                <input type="text" class="form-control @error('name') is-invalid @enderror" id="name" name="name" value="{% raw %}{{ old('name') }}{% endraw %}" placeholder="{% raw %}{{ trans('Name') }}{% endraw %}">
                                @error('name')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>
                        <div class="form-group row">
                            <label for="content" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('E-Mail Address') }}{% endraw %}</label>
                            <div class="col-md-6">
                                <input type="text" class="form-control @error('email') is-invalid @enderror" id="email" name="email" value="{% raw %}{{ old('email') }}{% endraw %}" placeholder="{% raw %}{{ trans('E-Mail Address') }}{% endraw %}">
                                @error('email')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>
                        <div class="form-group row">
                            <label for="permission" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Permission') }}{% endraw %}</label>
                            <div class="col-md-6">
                                <select class="form-control @error('permission') is-invalid @enderror" id="permission" name='permission' required aria-describedby="typeHelp" value="{% raw %}{{ old('permission') }}{% endraw %}" placeholder="{% raw %}{{ trans('Permission') }}{% endraw %}">
                                    <option value=''>{% raw %}{{ trans('Please choose')}}{% endraw %}{% raw %}{{ trans('Permission')}}{% endraw %}</option>
                                    @foreach(App\Enum::permission as $key => $value)
                                        <option value='{% raw %}{{ $key }}{% endraw %}'>{% raw %}{{ $value }}{% endraw %}</option>
                                    @endforeach
                                </select>
                                @error('permission')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>
                        <div class="form-group row">
                            <label for="content" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Password') }}{% endraw %}</label>
                            <div class="col-md-6">
                                <input type="password" class="form-control @error('password') is-invalid @enderror" id="password" name="password" placeholder="{% raw %}{{ trans('Password') }}{% endraw %}">
                                @error('password')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>
                        <div class="form-group row">
                            <label for="content" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Confirm Password') }}{% endraw %}</label>
                            <div class="col-md-6">
                                <input type="password" class="form-control" id="password_confirmation" name="password_confirmation" placeholder="{% raw %}{{ trans('Confirm Password') }}{% endraw %}">
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

之前在 `App/Enum` 中加入的權限在上面派上用場了 :

```php
// 帳號權限
const permission = [
    '0' => '一般使用者',
    '1' => '限閱',
    '2' => '閱讀、新增',
    '3' => '閱讀、新增、編輯',
    '4' => '閱讀、新增、編輯、刪除',
    '5' => '所有權限',
];
```

<br>
既然會員新增的頁面做好了，就在 `controller` 中加入 `store()` 來儲存資料 :

```php
public function store(Request $request)
{
    $user = new User;

    $data = $request->validate([
        'name' => ['required', 'string', 'max:255'],
        'email' => ['required', 'string', 'email', 'max:255', 'unique:users'],
        'password' => ['required', 'string', 'min:1', 'confirmed'],
        'permission' => ['required', 'string', 'max:5', 'min:0'],
    ]);

    foreach ($request->except('_token','_method','password_confirmation') as $key => $value) {
        if ($request->filled($key) && $key == 'password') {
            $user->password = Hash::make($data['password']);
        }
        elseif ($request->filled($key)) {
            $user->$key = $data[$key];
        }
    }

    if ($data) {
        $user->save();
    }

    return back()->with('success','會員新增成功 !');
}
```

如此一來你的資料經過 `驗證` 後就會寫入資料庫。

---

## 2. 會員修改

萬事起頭難，接下來你會發現就是新增 `blade` 然後在 `controller` 設定操作。

在 `controller` 的 `edit($id)` 加入 :

```php
public function edit($id)
{
    // 取出要修改的會員資料
    $user = User::where('id',$id)->first();
    // 帶著會員資料進入修改頁面
    return view('manage.member.edit',compact('user'));
}
```

在 `edit.blade.php` 中加入 :

```php
@extends('_layouts.manage.app')
@section('title', trans('Member').trans('Edit'))
@section('content')
<div class="container-fluid">
    <div class="row justify-content-center">
        <div class="col-md-12">
            <div class="card">
                <div class="card-header">{% raw %}{{ trans('Member').trans('Edit') }}{% endraw %}</div>

                <div class="card-body">
                    <ul class="list-unstyled">
                        <li>{% raw %}{{ App\Button::GoBack(route('member.index')) }}{% endraw %}</li>
                    </ul>
                    <form method="POST" action="{% raw %}{{ route('member.update' , $user->id) }}{% endraw %}">
                        @csrf
                        @method('PUT')
                        <div class="form-group row">
                            <label for="email" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('E-Mail Address') }}{% endraw %}</label>

                            <div class="col-md-6">
                                <input id="email" type="email" class="form-control @error('email') is-invalid @enderror" name="email" value="{% raw %}{{ $user->email }}{% endraw %}" required autocomplete="email" autofocus>

                                @error('email')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>

                        <div class="form-group row">
                            <label for="name" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Name') }}{% endraw %}</label>

                            <div class="col-md-6">
                                <input id="name" type="text" class="form-control @error('name') is-invalid @enderror" name="name" value="{% raw %}{{ $user->name }}{% endraw %}" required autocomplete="name" autofocus>

                                @error('name')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>

                        <div class="form-group row">
                            <label for="name" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Permission') }}{% endraw %}</label>

                            <div class="col-md-6">
                                <select class="form-control @error('permission') is-invalid @enderror" id="permission" name='permission' required>
                                    @foreach(App\Enum::permission as $key => $value)
                                        @if ($key == $user->permission)
                                            <option value='{% raw %}{{ $key }}{% endraw %}' selected>{% raw %}{{ $value }}{% endraw %}</option>
                                        @else
                                            <option value='{% raw %}{{ $key }}{% endraw %}'>{% raw %}{{ $value }}{% endraw %}</option>
                                        @endif
                                    @endforeach
                                </select>

                                @error('permission')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>

                        <div class="form-group row">
                            <label for="password" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Password') }}{% endraw %}</label>

                            <div class="col-md-6">
                                <input id="password" type="password" class="form-control @error('password') is-invalid @enderror" name="password"  autocomplete="new-password">

                                @error('password')
                                    <span class="invalid-feedback" role="alert">
                                        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
                                    </span>
                                @enderror
                            </div>
                        </div>

                        <div class="form-group row">
                            <label for="password-confirm" class="col-md-4 col-form-label text-md-right">{% raw %}{{ trans('Confirm Password') }}{% endraw %}</label>

                            <div class="col-md-6">
                                <input id="password-confirm" type="password" class="form-control" name="password_confirmation"  autocomplete="new-password">
                            </div>
                        </div>

                        <div class="form-group row">
                            <div class="col-md-6">
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

回到會員管理首頁，看到剛剛新增的資料，點擊最右邊的修改按鈕進入修改頁面。

接著在 `controller` 中的 `update()` 加入 :

```php
public function update(Request $request, $id)
    {

        $error = 0;
        $user = User::where('id',$id)->first();

        // 如果有輸入密碼
        if ($request->filled('password')) {
                $data = $this->validate($request, [
                'name' => ['required', 'string', 'max:255'],
                'email' => ['required', 'string', 'email', 'max:255'],
                'password' => ['required', 'string', 'min:1', 'confirmed'],
                'permission' => ['required', 'integer', 'max:5', 'min:0'],
            ]);

            foreach ($request->except('_token','_method','password_confirmation') as $key => $value) {
                if ($request->filled($key) && $key == 'password') {
                    $user->password = Hash::make($data['password']);
                }
                elseif ($request->filled($key)) {
                    $user->$key = $data[$key];
                    if ($user->$key == '') {
                        $error += 1;
                    }
                }
            }

            if ($error == 0) {
                $user->save();
            }
            else{
                return back()->withInput()->with('warning', '請確認輸入 !');
            }
        }
        else{

            $data = $this->validate($request, [
                'name' => ['required', 'string', 'max:255'],
                'email' => ['required', 'string', 'email', 'max:255'],
                'permission' => ['required', 'integer', 'max:5', 'min:0'],
            ]);

            // 逐筆進行htmlpurufier 並去掉<p></p>
            foreach ($request->except('_token','_method') as $key => $value) {
                if ($request->filled($key)) {
                    $user->$key = $data[$key];
                    if ($user->$key == '') {
                        $error += 1;
                    }
                }
            }

            if ($error == 0) {
                $user->save();
            }
            else{
                return back()->withInput()->with('warning', '請確認輸入 !');
            }
        }

        return back()->with('success', '會員更新成功 !');
    }
```

沒意外應該是可以成功修改。

---

## 3. 會員刪除

這個部分最簡單，我們總是習慣捨去。

在 `controller` 中的 `destroy()` 中加入 :

```php
public function destroy($id)
{
    User::destroy($id);
    return back()->with('success', '會員刪除成功 !');
}
```

接著只要點擊會員管理首頁的刪除按鈕即可刪除該筆資料。

---
## 結語

我知道你在這個章節可能會看得不清楚，比如

* 為什麼要 `return back()->with('success', '會員刪除成功 !');`
* `validate` 是什麼 ?

等等諸如此類的問題將在下一章說明。


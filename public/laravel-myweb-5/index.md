# Laravel Backend Control Website - 5


> Laravel 驗證、訊息、中介層  - Validation 、Message、Middleware

<!--more-->

## 前言

[第四篇](https://jhuei.com/code/2020/04/29/laravel-myweb-4.html) 建立了會員系統，但其中有些東西並沒有交代的很清楚。

比如 `資料驗證`、`新增、修改、刪除後的訊息`以及 `權限`的利用，這篇將盡可能的解釋。

---

## 1. 資料送出後的訊息

在 [第四篇](https://jhuei.com/code/2020/04/29/laravel-myweb-4.html) 可以看到儲存或刪除資料後都會有 :

```php
return back()->with('success','會員新增成功 !');
```

這行的意思是退回上一個位置並顯示成功的訊息框，訊息內容為會員新增成功。

[相關文件](https://laravel.com/docs/5.8/responses#redirects)
{{< admonition type=info title="" open=true >}}
那這時會有個問題，阿怎麼沒跑出來  ?  這個簡單，因為還沒建立訊息框。
{{< /admonition >}}


首先要建立 `views/_partials/message.blade.php` :

```html
@if ($message = Session::get('success'))
<div class="alert alert-success alert-block text-center">
 <button type="button" class="close" data-dismiss="alert">×</button> 
        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
</div>
@endif


@if ($message = Session::get('error'))
<div class="alert alert-danger alert-block text-center">
 <button type="button" class="close" data-dismiss="alert">×</button> 
        <strong>{% raw %}{{ $message }}{% endraw %}</strong>
</div>
@endif


@if ($message = Session::get('warning'))
<div class="alert alert-warning alert-block text-center">
 <button type="button" class="close" data-dismiss="alert">×</button> 
 <strong>{% raw %}{{ $message }}{% endraw %}</strong>
</div>
@endif


@if ($message = Session::get('info'))
<div class="alert alert-info alert-block text-center">
 <button type="button" class="close" data-dismiss="alert">×</button> 
 <strong>{% raw %}{{ $message }}{% endraw %}</strong>
</div>
@endif

@if ($message = Session::get('nodata'))
<div class="alert alert-info alert-block text-center">
 <button type="button" class="close" data-dismiss="alert">×</button> 
 <strong>{% raw %}{{ $message }}{% endraw %}</strong>
</div>
@endif

@if ($errors->any())
<div class="alert alert-danger text-center">
 <button type="button" class="close" data-dismiss="alert">×</button> 
 {% raw %}{{ trans('Please check the form below for errors') }}{% endraw %}
</div>
@endif
```

接著到 `_layouts/manage/app.blade.php` 中加入 :

```html
<!DOCTYPE html>
   ...略...
</head>
<body>
    <div id="app">
        @include('_partials.manage.nav')
        <main class="py-4">
            @include('_partials.message') <!-- 這行 -->
            @yield('content')
        </main>
    </div>
</body>
</html>
```

搞定 ! 回到會員管理新增一筆資料看看 !

沒意外新增成功上方會出現成功的訊息。

{{< image src="https://i.imgur.com/aMP6lSD.png" caption="message">}}

---

## 2. 資料送出後的驗證

想一想，會出現會員新增成功，就代表你通過了驗證。

以下是新增會員的驗證 :

```php
$data = $request->validate([
    'name' => ['required', 'string', 'max:255'],
    'email' => ['required', 'string', 'email', 'max:255', 'unique:users'],
    'password' => ['required', 'string', 'min:1', 'confirmed'],
    'permission' => ['required', 'string', 'max:5', 'min:0'],
]);
```

{{< admonition type=info title="" open=true >}}
`$request` 的解釋請看[這邊](https://laravel.com/docs/5.8/requests) <br><br>
簡單來說就是利用 `request` 來取得輸入資料，並且使用 [validate](https://laravel.com/docs/5.8/validation) 建立驗證。 <br><br>
可以作為條件的有[這些](https://laravel.com/docs/5.8/validation#available-validation-rules)。
{{< /admonition >}}


上面的信箱驗證就是使用了  `必填欄位` 、 `字串` 、`格式為信箱`、`最大值255` 、`唯一:資料表為users`

前面是驗證的設定，接下來看到頁面 `create.blade.php` 的信箱輸入 :

```html
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
```

可以看到 input 中的 `@error('email') is-invalid @enderror` ，`@error` 可以判斷剛剛所設定的驗證是否存在，只要驗證有錯就會在 `$message` 顯示錯誤訊息。

搭配 `Bootstrap` 的 `is-valid` 以及 `valid-feedback` 來優化顯示錯誤訊息。

{{< image src="https://i.imgur.com/zMvJEaS.png" caption="validation">}}

{{< admonition type=tip title="Tip" open=true >}}
表單驗證其實可以自訂，不一定要使用 `use Illuminate\Http\Request;` 的 `Request`，以後再說😆
{{< /admonition >}}

---

## 3. 權限的使用例子

既然 [第四篇](https://jhuei.com/code/2020/04/29/laravel-myweb-4.html) 建立了會員帳號的權限，以下就提出一些使用的例子。

比如我們規定 `權限大於0` 才能進入後台，這裡就要提到一下 [中介層 / middleware](https://laravel.com/docs/5.8/middleware)。

{{< admonition type=info title="" open=true >}}
中介層提供一個方便的機制來過濾進入應用程式的 HTTP 請求。
{{< /admonition >}}

之前我們透過 `php artisan:make auth` 的指令創建了名為 `auth` 的中介層 `./app/Http/Middleware/Authenticate.php` 。

中介層名字定義在 `./app/Http/Kernel.php` :

```php
<?php

namespace App\Http;

use Illuminate\Foundation\Http\Kernel as HttpKernel;

class Kernel extends HttpKernel
{
    ...略...

    protected $routeMiddleware = [
        'admin' => \App\Http\Middleware\Admin::class,      <!-- 等等要加入的 -->
        'auth' => \App\Http\Middleware\Authenticate::class,
        ...略...
    ];
    
    ...略...
```

可以看到 `admin` 和 `auth` 兩個中介層，`admin` 是等等要創立的，意義是管理員才能進入後台 ; 而 `auth` 代表是否有登入。

輸入以下指令生成中介層 `Admin.php` :

```bash
php artisan make:middleware Admin
```

將內容改為 :

```php
<?php

namespace App\Http\Middleware;

use Closure;

class Admin
{
    public function handle($request, Closure $next)
    {
        if (auth()->user()->permission != 0) {
            return $next($request);
        }
        return redirect('home');
    }
}
```

上面的意思就是如果登入權限不等於0才能存取路由，否則就要導回前台頁面。([auth用法](https://laravel.com/docs/5.8/authentication))

接下來將中介層加入 `Kernel.php` 後就到 `web.php` 中加入中介層。

```php
Route::get('/manage', function () {return view('manage.index');})->middleware('auth','admin')->name('manage');
Route::prefix('manage')->middleware('auth','admin')->group(function(){
    Route::resource('member', 'MemberController');
});
//必須登入且權限大於零才能進入後台
```

可以創立一個權限為 `一般使用者` 的試看看登入後進入[後台](http://localhost:8000/manage/)。

再來的例子是帳號權限是否可以對會員帳號進行新增修改，請在 `_partial/manage/nav.blade.php` 做以下修改 :

```html
<nav class="navbar navbar-expand-md navbar-light bg-white shadow-sm">
    <div class="container-fluid">
        <a class="navbar-brand" href="{% raw %}{{ url('/manage') }}{% endraw %}">
            {% raw %}{{ config('app.name', 'Laravel') }}{% endraw %}
        </a>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="{% raw %}{{ __('Toggle navigation') }}{% endraw %}">
            <span class="navbar-toggler-icon"></span>
        </button>

        <div class="collapse navbar-collapse" id="navbarSupportedContent">
            <!-- Left Side Of Navbar -->
            <ul class="navbar-nav mr-auto">
                @if (Auth::check() && Auth::user()->permission > '4')
                <li class="nav-item">
                    <a class="nav-link" href="{% raw %}{{ route('member.index') }}{% endraw %}">{% raw %}{{ __('Member').__('Manage') }}{% endraw %}</a>
                </li>
                @endif
            </ul>

            <!-- Right Side Of Navbar -->
            <ul class="navbar-nav ml-auto">
                <li class="nav-item">
                    <a class="nav-link" href="{% raw %}{{ route('home') }}{% endraw %}">{% raw %}{{ __('Fontstage') }}{% endraw %}</a>
                </li>
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
            </ul>
        </div>
    </div>
</nav>
```

可以看到其中的 :

```html
@if (Auth::check() && Auth::user()->permission > '4')
<li class="nav-item">
    <a class="nav-link" href="{% raw %}{{ route('member.index') }}{% endraw %}">{% raw %}{{ __('Member').__('Manage') }}{% endraw %}</a>
</li>
@endif
```

上述表示權限不大於4無法進入會員管理的頁面。

另外你可以在Controller中加入 :

```php
public function create()
{
    if (Auth::check() && Auth::user()->permission < '5') {
        return back()->with('warning', '權限不足以訪問該頁面 !');
    }
    return view('manage.member.create');
}
```

上述表示權限小於5無法進入會員新增的頁面。

諸如此類的例子，就交給你自由發揮了😉

---

## 4. 額外補充

```bash
php artisan make:controller MemberController --resource
```

`--resource` 代表建立了一個包含CRUD的控制器。


| 函數  | 用途 |
| -------- | -------- |
| index()     | 顯示資料（一般是列表）     |
| create()     | 建立新資料（通常是表單界面）    |
| store(Request $request)     | 儲存資料     |
| show($id)     | 顯示某筆資料     |
| edit($id)     | 編輯某筆資料（通常是表單界面）     |
| update(Request $request, $id)     | 更新某筆資料     |
| destroy($id)     | 刪除某筆資料     |

如果要建立一個空的控制器 :

```bash
php artisan make:controller MemberController
```

如果要同時建立 `model`、`migration`、`controller --resource` :

```bash
php artisan make:model Member -mcr
```

| 參數  | 用途 |
| -------- | -------- |
| -m     | 為model建立一個數據庫    |
| -c     | 為model建立一個控制器    |
| -r     |  為控制器加入  --resource 參數     |

可以用下列指令看到所有可用選項 :

```bash
php artisan make:model --help
```

Hope it will help !


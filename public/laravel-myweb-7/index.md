# Laravel Backend Control Website - 7


> Laravel 拖曳排序、視圖合成器 - Drag Sort、View Composer

<!--more-->

## 前言

Hello 大家 ! 在 [上一篇](https://jhuei.com/code/2020/05/03/laravel-myweb-6.html) 提到了視圖合成器(View Composer)以及如何利用拖曳來進行導覽列的排序，這篇就來一一解答。

---

## 1. 視圖合成器

視圖合成器(View Composer) 提供了一種方法讓你將資料一次性的分享到指定的視圖，上一篇所使用到的 :

```php
View::composer(['*'], function ($view) {
    $navbars = App\Navbar::where('is_open',1)->orderby('sort')->get();
    $users = App\User::all();

    $view->with('navbars',$navbars);
    $view->with('users',$users);
});
```

若你想將 `$navbars` 以及 `$users` 分享到所有的視圖，就使用`['*']`。若只想分享到指定視圖，例如管理後台首頁，則使用`['manage.index']`。

這樣一來就不用再控制器重複宣告資料了。

{{< admonition type=warning title="注意" open=true >}}
但要記得，由於是分享到所視圖，不能和控制器宣告的名字衝突。
{{< /admonition >}}

---

## 2. 拖曳排序

要完成拖曳排序，要使用幾個工具 :

* Datatable
* jQuery

直接在 `manage/app.blade.php` 中加入 :

```html
<body>
    <div id="app">
        @include('_partials.manage.nav')
        <main class="py-4">
            @include('_partials.message')
            @yield('content')
        </main>
    </div>
    @section('script')
    <script src="https://ajax.googleapis.com/ajax/libs/jqueryui/1.10.3/jquery-ui.min.js"></script>
    @show
</body>
```

上面使用到了 `@section('script')...@show` 的方法，這樣在子視圖就只要使用 :

```
@section('script') //引入app的js
@parent            //本頁使用的js 
<script>
...略...
</script>
@show
```

首先，要建立排序的視圖，`sort.blade.php` :

```php
@extends('_layouts.manage.app')
@section('title', trans('Navbar').trans('Sort'))
@section('content')
<div class="container-fluid">
    <div class="row justify-content-center">
        <div class="col-md-12">
            <div class="card">
                <div class="card-header">{% raw %}{{trans('Navbar').trans('Sort')}}{% endraw %}</div>

                <div class="card-body">
                    <ul class="list-unstyled">
                        <li>{% raw %}{{ App\Button::GoBack(route('navbar.index')) }}{% endraw %}</li>
                    </ul>
                    <div class="alert alert-warning" role="alert">
                        1. 請直接拖曳目標列進行排序。<br>
                        2. 調整完請點重新整理進行確認。
                    </div>
                    <table id="table" class="table table-hover table-bordered text-center">
                        <thead>
                            <tr class="table-info active">
                                <th class="text-nowrap text-center">{% raw %}{{trans('Navbar').trans('Name')}}{% endraw %}</th>
                                <th class="text-nowrap text-center">{% raw %}{{ trans('Type') }}{% endraw %}</th>
                                <th class="text-nowrap text-center">{% raw %}{{ trans('Sort') }}{% endraw %}</th>
                                <th class="text-nowrap text-center">{% raw %}{{ trans('Is_open') }}{% endraw %}</th>
                            </tr>
                        </thead>
                        <tbody id="tablecontents">
                            @foreach ($navbars as $navbar)
                                <tr class="row1" data-id="{% raw %}{{ $navbar->id }}{% endraw %}">
                                    <td class="pl-3">{% raw %}{{ $navbar->name }}{% endraw %}</td>
                                    <td>{% raw %}{{App\Enum::type['navbar'][$navbar->type]}}{% endraw %}</td>
                                    <td>{% raw %}{{ $navbar->sort }}{% endraw %}</td>
                                    <td><font color="{% raw %}{{App\Enum::is_open['color'][$navbar->is_open]}}{% endraw %}"><i class="fas fa-{% raw %}{{App\Enum::is_open['label'][$navbar->is_open]}}{% endraw %}"></i></font></td>
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
                url: "{% raw %}{{ url('navbar-sortable') }}{% endraw %}",
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

接著在，`index.blade.php` 加入按鈕 :

```html
<li class="list-inline-item">{% raw %}{{ App\Button::To(true,'sort',trans('Sort'),'','btn-primary') }}{% endraw %}</li>
```

再來是到 `web.php` 加入路由 :

```php
//排序、中介層:登入/管理員
Route::middleware('auth','admin')->group(function() {
    Route::post('navbar-sortable','NavbarController@sort')->name('navbar.sort');
    Route::get('/manage/navbar/sort', function () {return view('manage.navbar.sort');});
});
```

做到這裡應該可以用滑鼠拖曳表格內的`<td>` :

{{< image src="https://i.imgur.com/DixcbAx.gif" caption="drag">}}

接下來要做的就是拖曳後將排序資料寫入資料庫 ，在 `NavbarController.php` 中加入 :

```php
//拖曳排序
public function sort(Request $request)
{
    if (Auth::check() && Auth::user()->permission < '3') {
        return back()->with('warning', '權限不足以訪問該頁面 !');
    }
    
    $navbars = Navbar::all();

    foreach ($navbars as $navbar) {
        foreach ($request->order as $order) {
            if ($order['id'] == $navbar->id) {
                $navbar->update(['sort' => $order['position']]);
            }
        }
    }
    $sort = DB::table('navbars')->select('name','sort')->orderby('sort')->get();
    Log::write_log('navbars',$sort,'排序');
    
    return response('Update Successfully.', 200);
}
```

在一次拖曳排序後重新整理應該就會實現了。

---

## 總結

* 使用 [View composer](https://laravel.com/docs/5.8/views#view-composers) 必須確定自己宣告的名稱。
* View composer 也可以在 `Provide` 中使用。
* 拖曳排序使用了 JQuery中的  [sortable](https://jqueryui.com/sortable/) 實現。
* 如果想在手機端觸控拖曳請自行加入[jquery.ui.touch-punch.js](https://raw.githubusercontent.com/furf/jquery-ui-touch-punch/master/jquery.ui.touch-punch.js)。

叮叮結束🙋‍♂️


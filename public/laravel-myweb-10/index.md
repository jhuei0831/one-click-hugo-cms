# Laravel Backend Control Website - 10


> 執行後產生LOG資料 -Generate LOG Data After Execution

<!--more-->

## 前言

Hello 大家，

在執行新增、修改、刪除等動作後，必須要記錄改變後的資料以及誰進行操作，這樣出狀況才方便了解誰做出這麼偉大的事情。

---

## 1. 取得使用者資訊

首先，先建立一個 `LOG` 的model :

```bash
// 一次性建立
php artisan make:model Log -mcr
```

---

## 2. 修改Migration

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateLogsTable extends Migration
{
    public function up()
    {
        Schema::create('logs', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('user')->comment('使用者');
            $table->string('ip')->comment('使用者IP');
            $table->string('os')->comment('作業系統');
            $table->string('browser')->comment('瀏覽器');
            $table->string('browser_detail')->comment('瀏覽器詳細資料');
            $table->string('action')->comment('動作');
            $table->string('table')->comment('資料表');
            $table->json('data')->comment('資料');
            $table->timestamps();
        });
    }
    public function down()
    {
        Schema::dropIfExists('logs');
    }
}
```

---

## 3. 修改Model

`Log.php`

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Log extends Model
{
    protected $table = 'logs';

    protected $primaryKey = 'id';

    protected $fillable = [
        "user", "ip", "os", "browser", "browser_detail", "action", "table", "data",
    ];
}
```

---

## 4. 加入路由

```php
Route::prefix('manage')->middleware('auth','admin')->group(function(){
    ...略...
    Route::resource('log', 'LogController');
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
        ├── detail.blade.php
        └── index.blade.php
    └── member/  
    └── navbar/  
    └── page/
```

---

## 6. Log 首頁

`index.blade.php`

```php
@extends('_layouts.manage.app')
@section('title', __('Log').__('Manage'))
@section('content')
<div class="container-fluid">
    <div class="row justify-content-center">
        <div class="col-md-12">
            <div class="card">
                <div class="card-header">{% raw %}{{ __('Log').__('Manage') }}{% endraw %}</div>
                <div class="card-body">
     <div class="table-responsive">
      <table id="data" class="table table-hover table-bordered text-center">
       <thead>
        <tr class="table-info active">
         <th class="text-nowrap text-center">{% raw %}{{ __('Member') }}{% endraw %}</th>
         <th class="text-nowrap text-center">{% raw %}{{ __('IP') }}{% endraw %}</th>
         <th class="text-nowrap text-center">{% raw %}{{ __('Browser') }}{% endraw %}</th>
         <th class="text-nowrap text-center">{% raw %}{{ __('Action') }}{% endraw %}</th>
         <th class="text-nowrap text-center">{% raw %}{{ __('Table') }}{% endraw %}</th>
         <th class="text-nowrap text-center">{% raw %}{{ __('Created_at') }}{% endraw %}</th>
         <th class="text-nowrap text-center">*</th>
        </tr>
       </thead>
       <tbody>
        @foreach ($logs as $log)
         <tr>
          <td>{% raw %}{{ $log->user }}{% endraw %}</td>
          <td>{% raw %}{{ $log->ip }}{% endraw %}</td>
          <td>{% raw %}{{ $log->browser }}{% endraw %}</td>
          <td>{% raw %}{{ $log->action }}{% endraw %}</td>
          <td>{% raw %}{{ $log->table }}{% endraw %}</td>
          <td>{% raw %}{{ $log->created_at }}{% endraw %}</td>
          <td>
           <form action="{% raw %}{{ route('log.show',$log->id) }}{% endraw %}" method="GET">
           @csrf
           {% raw %}{{ App\Button::detail($log->id) }}{% endraw %}
           </form>
          </td>
         </tr>
        @endforeach
       </tbody>
      </table>
     </div>
                </div>
                <div class="card-footer pagination justify-content-center table-responsive">
     {!! $logs->links("pagination::bootstrap-4") !!}
    </div>
            </div>
        </div>
    </div>
</div>
@endsection
```

`LogController.php` :

```php
namespace App\Http\Controllers;

use App\Log;
use Illuminate\Http\Request;

class LogController extends Controller
{
    public function index()
    {
        $logs = Log::orderby('created_at','desc')->paginate();
        return view('manage.log.index',compact('logs'));
    }
}
```

---

## 7. 取得使用者資訊

`Log.php`

```php
public static function get_agent()
{
    $user_agent = isset($_SERVER['HTTP_USER_AGENT']) ? $_SERVER['HTTP_USER_AGENT'] : '';
    $browser_name = 'Unknown';
    $platform = 'Unknown';
    $version = $ub = "";

    // 取得作業系統資訊
    if (preg_match('/linux/i', $user_agent)) 
    {
        $platform = 'linux';
    }
    else if(preg_match('/macintosh|mac os x/i', $user_agent)) 
    {
        $platform = 'mac';
    }
    else if(preg_match('/windows|win32/i', $user_agent)) 
    {
        $platform = 'windows';
    }

    // 取得瀏覽器資訊
    if((preg_match('/MSIE/i',$user_agent) || preg_match('/Trident\/7.0; rv:11.0/i',$user_agent)) && !preg_match('/Opera/i',$user_agent))
    {
        $browser_name = 'Internet Explorer';
        $ub = "MSIE";
    }
    else if(preg_match('/Firefox/i',$user_agent))
    {
        $browser_name = 'Mozilla Firefox';
        $ub = "Firefox";
    }
    else if(preg_match('/Chrome/i',$user_agent))
    {
        $browser_name = 'Google Chrome';
        $ub = "Chrome";
    }
    else if(preg_match('/Safari/i',$user_agent))
    {
        $browser_name = 'Apple Safari';
        $ub = "Safari";
    }
    else if(preg_match('/Opera/i',$user_agent))
    {
        $browser_name = 'Opera';
        $ub = "Opera";
    }
    else if(preg_match('/Netscape/i',$user_agent))
    {
        $browser_name = 'Netscape';
        $ub = "Netscape";
    }

    // 瀏覽器版本
    $known = array('Version', $ub, 'other');
    $pattern = '#(?<browser>'.join('|', $known).')[/ ]+(?<version>[0-9.|a-zA-Z.]*)#';
    if (!preg_match_all($pattern, $user_agent, $matches)) 
    {
        // we have no matching number just continue
    }

    // see how many we have
    $i = count($matches['browser']);
    if ($i != 1) 
    {
        //we will have two since we are not using 'other' argument yet
        //see if version is before or after the name
        if (strripos($user_agent,"Version") < strripos($user_agent,$ub))
        {
            $version = $matches['version'][0];
        }
        else 
        {
            $version = isset($matches['version'][1]) ? $matches['version'][1] : '';
        }
    }
    else 
    {
        $version = $matches['version'][0];
    }

    // check if we have a number
    if ($version == null || $version == "")
    {
        $version = "Unknown";
    }

    return array(
        'name'  => $browser_name,
        'version' => $version,
        'platform' => $platform,
        'detail' => $user_agent
    );
}
```

---

## 8. 寫入Log

`Log.php` :

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\DB;
use Request;

class Log extends Model
{
    ...略...
    public static function write_log($table,$data,$action = NULL)
    {

        $agent = Log::get_agent();

        DB::table('logs')->insert([

            'user' => (Auth::user()->first())['name'],
            'ip'   => Request::ip(),
            'os'   => $agent['platform'], 
            'browser'   => $agent['name'], 
            'browser_detail'   => $agent['detail'], 
            'action'   => $action ?? Request::method(), 
            'table'   => $table, 
            'data'   => json_encode($data, JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES),
            'created_at'   => now()

        ]);

        return true;
    }
}
```

* 第13行為進行了何種動作，新增修改或其他。
* 第15行為新增修改刪除等寫入影響資料庫的資料，使用 `json` 的方式儲存。

最後在 `Controller`中引入 `Log.php` 後就可以使用 `write_log()` 的方式寫入LOG。

使用 `MemberController.php` 新增會員當範例  :

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Validator;
use Illuminate\Support\Facades\DB;
use App\User;
use App\Log;

class MemberController extends Controller
{
    ...略...
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
                $user->$key = strip_tags(clean($data[$key]));
            }
        }

        if ($data) {
            // 寫入log
            Log::write_log('users',$request->all());
            $user->save();
        }

        return back()->with('success','會員新增成功 !');
    }
  ...略...
```

* 第11行引入 `Log` model
* 第38行寫入log，影響的資料表為`users`

實際操作一次新增會員並確認是否產生LOG。

{{< image src="https://i.imgur.com/OuBdopp.png" caption="log">}}

---

## 9. 詳細資料

上圖可以看到有一個 `Detail` 的按鈕，用來看被寫入資料庫的資料。

修改 `LogController.php` :

```php
public function show($id)
{
    $log = Log::where('id',$id)->first();
    return view('manage.log.detail',compact('log'));
}
```

`detail.blade.php` :

```html
@extends('_layouts.manage.app')
@section('title', __('Log').__('Detail'))
@section('content')
<div class="container-fluid">
    <div class="row justify-content-center">
        <div class="col-md-12">
            <div class="card">
                <div class="card-header">{% raw %}{{ __('Log').__('Detail') }}{% endraw %}</div>               
                <div class="card-body">
                    <ul class="list-unstyled">
                        <li>{% raw %}{{ App\Button::GoBack(route('log.index')) }}{% endraw %}</li>
                    </ul>
                    <div class="table-responsive">     
                        <table class="table table-hover table-bordered">
                            <thead>
                                <tr class="table-info active">
                                    <th class="text-nowrap text-center">{% raw %}{{ __('Item') }}{% endraw %}</th>
                                    <th class="text-nowrap text-center">{% raw %}{{ __('Data') }}{% endraw %}</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr>
                                    <td>{% raw %}{{ __('Member') }}{% endraw %}</td>
                                    <td>{% raw %}{{ $log->user }}{% endraw %}</td>
                                </tr>
                                <tr>
                                    <td>{% raw %}{{ __('IP') }}{% endraw %}</td>
                                    <td>{% raw %}{{ $log->ip }}{% endraw %}</td>
                                </tr>
                                <tr>
                                    <td>{% raw %}{{ __('Operating system') }}{% endraw %}</td>
                                    <td>{% raw %}{{ $log->os }}{% endraw %}</td>
                                </tr>
                                <tr>
                                    <td>{% raw %}{{ __('Browser') }}{% endraw %}</td>
                                    <td>{% raw %}{{ $log->browser }}{% endraw %}</td>
                                </tr>
                                <tr>
                                    <td>{% raw %}{{ __('Browser').__('Detail') }}{% endraw %}</td>
                                    <td>{% raw %}{{ $log->browser_detail }}{% endraw %}</td>
                                </tr>
                                <tr>
                                    <td>{% raw %}{{ __('Action') }}{% endraw %}</td>
                                    <td>{% raw %}{{ $log->action }}{% endraw %}</td>
                                </tr>
                                <tr>
                                    <td>{% raw %}{{ __('Table') }}{% endraw %}</td>
                                    <td>{% raw %}{{ $log->table }}{% endraw %}</td>
                                </tr>
                                <tr>
                                    <td>{% raw %}{{ __('Data') }}{% endraw %}</td>
                                    <td><pre>{% raw %}{{ $log->data }}{% endraw %}</pre></td>
                                </tr>
                                <tr>
                                    <td>{% raw %}{{ __('Created_at') }}{% endraw %}</td>
                                    <td>{% raw %}{{ $log->created_at }}{% endraw %}</td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>
                <div class="card-footer">

                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```

現在只要點 `Detail` 按鈕後就可以看到以下畫面 :

{{< image src="https://i.imgur.com/Xqg3TGL.png" caption="detail">}}

其中資料欄位就是新增會員所寫入的資料。

---

## 10. 結語

叮叮結束🚗

這裡有一個問題，可以看到在 `detail` 的資料部分，`"password": "Pa$$w0rd!"` ，

表示你剛剛輸入的密碼沒有被 `hash` 處理到，這是因為上面控制器寫入LOG使用的是 `$request->all()` ，

`$request->all()` 是處理前的資料，只要改成 `Log::write_log('users',$user);` 問題即可解決。


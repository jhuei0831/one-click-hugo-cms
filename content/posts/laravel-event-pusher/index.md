---
title: Laravel Event - Pusher
categories: ['CODE']
description: 使用Pusher在Laravel的事件中做出廣播功能
tags: ['php', 'html', 'laravel', 'pusher', 'event']
authors: [Kerwin]
featuredImagePreview: feature.jpg
date: 2020-10-04
lastmod: 2020-10-04
---

> 使用Pusher在Laravel的事件中做出廣播功能

<!--more-->

## 前言

使用Pusher在Laravel中做出訊息廣播的功能。

---

## 1. Pusher

到Pusher官網建立帳號後，建立一個`Channel`:

* Cluster選擇`ap3`
* Backend選擇`Laravel`

建立完成後就到[https://dashboard.pusher.com/apps](https://dashboard.pusher.com/apps) 點擊剛剛建立的Channel，接著點選左邊的`Getting Started`。

找到STEP 2 的地方，安裝pusher，在你的laravel專案底下執行:

```bash
composer require pusher/pusher-php-server
```

接著在你的`.env`檔案中將PUSHER_APP_ID、PUSHER_APP_KEY、PUSHER_APP_SECRET填入，
PUSHER_APP_CLUSTER填入`ap3`。

重點是 BROADCAST_DRIVER=pusher。

---

## 2. 建立事件

打開 `app/Providers/EventServiceProvider.php`，在listen中加入:

```php
'App\Events\TestPusher' => [
    'App\Listeners\SendMessageNotification',
],
```

打開cmd，在你的laravel專案底下執行:

```bash
php artisan event:generate
```

接著打開剛剛建立的事件 `app/Events/TestPusher.php` ，將內容改為:

```php
<?php

namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class TestPusher implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $message;

    public function __construct($message)
    {
        $this->message = $message;
    }

    public function broadcastOn()
    {
        return ['my-channel'];
    }

    public function broadcastAs()
    {
        return 'my-event';
    }
}
```

---

## 3. 事件傳送

在`views`底下建立視圖`pusher.blade.php`，code為pusher `Getting Started` 的STEP 1。

`web.php`:

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Events\TestPusher;

Route::get('/', function () {
    return view('welcome');
});

Route::get('/pusher', function () {
    return view('pusher');
});

Route::get('/pusher_test', function () {
    event(new TestPusher('hello world'));
});
```

打開兩個分頁 `/pusher` 和 `/pusher_test` ，打開pusher_test的同時，pusher顯示了訊息Hello world。

延伸閱讀[Security](https://www.yiiframework.com/doc/guide/2.0/en/security-overview)

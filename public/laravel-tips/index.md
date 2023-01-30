# Laravel Tips


> Laravel一些實用小技巧或語法

<!--more-->

## URL

1. 取得網域名稱(Domain name，如:www.example.com)

```php
Request::getHost();
```

2. 取得傳輸協定+Domain網址(如:<http://www.example.com)>

```php
Request::root();
```

3. 在網址 <http://localhost:8000/users/{id}?int=95> 中取得`id`或`int`的值

```php
request()->id;
request()->int;
```
---


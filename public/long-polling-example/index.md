# Long Polling Database Example


> 長時間輪詢PHP+MySQL範例 - Long Polling PHP + MySQL Example

<!--more-->

## 前言

Hello 大家，[這一篇](https://jhuei.com/long-polling/) 提到了一個文件更新的範例來示範 Long Polling。

這篇打算利用資料庫新增的方式做一個範例。

---

## 1. 資料結構

```treeview
php-long-polling/
    └── client/
        ├── client.js 
        └── client.html
    └── server/
        ├── database.php 
        └── server.php
    └── index.php
```

---

## 2. 資料庫

可以用phpmyadmin手動新增或直接輸入以下SQL指令 :

```sql
CREATE TABLE `admin` (
  `id` int(11) NOT NULL,
  `name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL,
  `password` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL
) 
```

`database.php` :

```php
<?php
    $db_host = "localhost";

    $db_user = "自行輸入對應數值";

    $db_pass = "自行輸入對應數值";

    $db_name = "自行輸入對應數值";

    $dbconnect = "mysql:host=".$db_host.";dbname=".$db_name;

    $dbgo = new PDO($dbconnect, $db_user, $db_pass);
?>
```

---

## 3. 資料新增

`index.php` :

```php
<?php 
    include_once('./server/database.php');

    if ($_POST) {
        $sql = "INSERT INTO `admin` (`password`,`name`) VALUES ('".$_POST['name']."','".$_POST['password']."')";
        $result = $dbgo->query($sql);
    }
?>
<form method="post">
    name : <input type="text" name="name">
    password : <input type="password" name="password">
    <input type="submit">
</form>
```

---

## 4. 伺服端

`server.php` :

```php
include_once('database.php');

set_time_limit(0);

while (true) {

    // 客戶端發Request
    $last_ajax_call = isset($_GET['max']) ? $_GET['max'] : null;

    // 取得當前資料庫資料
    $sql = "select * from `admin`";
    $data = $dbgo->query($sql);
    $row = $data->fetchAll(PDO::FETCH_ASSOC);
    $max_id = array();
    foreach ($row as $key => $value) {
        array_push($max_id, $value['id']);
    }
    // 取出最大id
    $max_id = max($max_id);

    // 如果當前數值大於客戶端的request表示新增
    if ($last_ajax_call == null || $max_id > $max) {

        $result = array(
            'data_from_database' => json_encode($row),
            'id' => $max_id
        );

        // 將資料轉成JSON格式，並呈現結果
        $json = json_encode($result);
        echo $json;

        break;

    } else {
        sleep(1);
        continue;
    }
}
```

這次使用的找出資料庫新增的方法是找出最大 `ID`，因為是 `Auto Increase`  。

---

## 5. 客戶端

`client.php` :

```javascript
function getContent(id)
{
    var queryString = {'id' : id};

    $.ajax(
        {
            type: 'GET',
            url: 'http://127.0.0.1/php-long-polling/server/server.php',
            data: queryString,
            success: function(data){
                var obj = jQuery.parseJSON(data);
                var user = '';
                $.each(jQuery.parseJSON(obj.data_from_database), function(key, value){
                    user += '<tr>';
                    user += '<td>'+value.id+'</td>';
                    user += '<td>'+value.name+'</td>';
                    user += '<td>'+value.password+'</td>';
                    user += '</tr>';
                });
                $('#response').html(user);
                getContent(obj.id);
            }
        }
    );
}

$(function() {
    getContent();
});
```

概念跟之前範例一樣，狂丟 `Request`，把收到的資料轉成表格輸出並打印在 `#response`。

`client.html` :

```html
<html>
    <head>
        <script type="text/javascript" src="http://code.jquery.com/jquery.min.js"></script>
        <script type="text/javascript" src="client.js"></script>
    </head>
    <body>
        <h1>Response from server:</h1>
        <table id="response">
            <tr>
                <th>id</th>
                <th>name</th>
                <th>password</th>
            </tr>
        </table>
    </body>
</html>
```

---

## 6. 結語

---

叮叮結束，可以想想刪除怎麼做🥱


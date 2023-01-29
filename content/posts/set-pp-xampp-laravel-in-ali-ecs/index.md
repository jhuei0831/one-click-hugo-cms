---
title: Set Up XAMPP Laravel in Ali ECS
categories: ['CODE']
description: 阿里雲ECS使用XAMPP環境建Laravel
tags: ['php', 'laravel', 'alibaba', 'ECS', 'XAMPP', 'composer']
authors: ["Kerwin"]
featuredImagePreview: feature.jpg
date: 2021-01-11
lastmod: 2021-01-11
---

> 阿里雲ECS使用XAMPP環境建Laravel

<!--more-->

## 前言

玩過 AWS EC2 跟GAE，來玩看看阿里雲。

---

## 1. 安裝XAMPP

到[XAMPP官網](https://www.apachefriends.org/zh_tw/download.html)下載你要的Linux版本的XAMPP，

接著把下載完成的檔案放到 `/opt/` 底下，接著運行下面指令:

```bash
cd /opt
chmod 755 xampp-linux-*-installer.run
sudo ./xampp-linux-*-installer.run
```

安裝完成後，可以使用CURL測試一下:

```bash
curl -L http://localhost
```

成功後接著就在瀏覽器輸入 `http://公網`，出現以下畫面表示成功。

{{< image src="https://i.imgur.com/SOFQXpF.png" caption="xampp">}}

如果失敗的話，有可能是ECS的80 port沒有開，這時候就要到 `網路與安全/安全性群組/設定規則/快速建立規則` 加入80 port 或其他你想開的port。

這樣應該就沒有問題了!!

---

## 2. 建立Laravel

先安裝 `composer`

```bash
apt install composer
```

接著在 `/opt/lampp/htdocs/` 底下建立 Laravel 專案 (可以看[這篇](https://jhuei.com/code/2020/04/21/laravel-myweb-1.html))

設定資料夾權限:

```bash
cd /專案名稱
sudo chmod -R 775 storage
sudo chmod -R 775 bootstrap/cache
```

因為Laravel專案首頁是在 `專案/public` 底下，所以要去 `apache`設定，

打開 `/opt/lampp/etc/extra/httpd-vhosts.conf`中加入:

```bash
<VirtualHost /專案名稱:80>
    DocumentRoot "/opt/lampp/htdocs/專案名稱/public"
    ServerAdmin Admin

    <Directory "/opt/lampp/htdocs/專案名稱">
        Options All
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

儲存後重啟apache:

```bash
/opt/lampp/lampp restart
```

重啟後打開 `http://公網/專案名稱`

打完收工

---

## 常見問題

**1. 替換掉Linux預設的php版本**

執行指令:

```bash
sudo ln -s /opt/lampp/bin/php /usr/bin/php
```

如果出現錯誤 `failed to create symbolic link '/usr/bin/php': File exists`，就執行指令

```bash
sudo rm /usr/bin/php
sudo ln -s /opt/lampp/bin/php /usr/bin/php
```

**2. 執行Composer錯誤**

出現錯誤 `PHP Warning: require(Composer/autoload.php): failed to open stream`

執行指令:

```bash
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/bin --filename=composer
```

**3. The stream or file ”laravel.log“ could not be opened: failed to open stream: Permission denied**

設定資料夾權限:

```bash
cd /專案名稱
sudo chmod -R 775 storage
sudo chmod -R 775 bootstrap/cache
```

參考資料:

1. <https://www.cnblogs.com/jeacy/p/7132394.html>
2. <https://askubuntu.com/questions/563972/how-to-link-to-php-from-xampp-installation-so-i-can-just-use-php-command-instead>
3. <https://github.com/composer/composer/issues/5510>
4. <https://stackoverflow.com/questions/52178033/the-stream-or-file-laravel-log-could-not-be-opened-failed-to-open-stream-pe>

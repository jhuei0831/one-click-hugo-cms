---
title: Yii2 basic login auth with database
categories: ['CODE']
description: 使用MySQL完成Yii2 app basic專案登入範例
tags: ['php', 'yii', 'yii2', 'yii2-app-basic', 'auth', 'login']
authors: ["Kerwin"]
featuredImagePreview: feature.jpg
date: 2023-01-08
lastmod: 2023-01-08
---

> 使用MySQL完成Yii2 app basic專案登入範例
<!--more-->

## 前言

如果使用Yii2官方的[basic範例](https://github.com/yiisoft/yii2-app-basic)，會發現在登入這一塊範例把帳號密碼hardcode在程式碼中，而沒有使用資料庫。

所以這篇將把資料庫方法加入basic專案，實現登入驗證。

---

## 建立使用者資料表

本次介紹使用的資料庫是MySQL，以下使用migration方式建立`users`資料表。

```bash
php yii migrate/create create_users_table
```

```php
<?php

use yii\db\Migration;

/**
 * Handles the creation of table `{{ %users }}`.
 */
class m230108_122442_create_users_table extends Migration
{
    /**
     * {@inheritdoc}
     */
    public function safeUp()
    {
        // create table
        $this->createTable('{{ %users }}', [
            'id' => $this->primaryKey(),
            'name' => $this->string(50)->notNull()->comment('名稱'),
            'email' => $this->string(50)->notNull()->comment('電子郵件'),
            'password' => $this->string(200)->notNull()->comment('密碼'),
            'auth_key' => $this->string(50)->notNull()->comment('金鑰'),
            'created_at' => $this->dateTime()->notNull()->defaultExpression('CURRENT_TIMESTAMP')->comment('建立時間'),
            'updated_at' => $this->dateTime()->null()->append('ON UPDATE CURRENT_TIMESTAMP')->comment('修改時間'),
        ]);

        // seed some default data
        Yii::$app->db->createCommand()->batchInsert('users', 
            ['name', 'email', 'password', 'auth_key'],
            [
                [
                    'admin',
                    'admin@example.com',
                    Yii::$app->getSecurity()->generatePasswordHash('admin'),
                    Yii::$app->getSecurity()->generateRandomString(),
                ],
                [
                    'guest',
                    'guest@example.com',
                    Yii::$app->getSecurity()->generatePasswordHash('guest'),
                    Yii::$app->getSecurity()->generateRandomString(),
                ],
            ],
        )->execute();
    }

    /**
     * {@inheritdoc}
     */
    public function safeDown()
    {
        $this->dropTable('{{ %users }}');
    }
}

```

- 完成資料表建立:

```bash
php yii migrate
```

---

## 建立Users模型(model)

基本上使用gii建立，在此就不贅述建立流程。

建立完成後一定要加入`IdentityInterface`介面，然後在模型底下加入幾個函式:

- `findByUsername` : 在 LoginForm 模型中找使用者資料用。
- `validatePassword` : 驗證密碼。
- `findIdentity` : 根據给到的ID查詢身分。
- `findIdentityByAccessToken` : 根據 token 查詢身份。
- `getId` : 取得使用者id。
- `getAuthKey` : 當前用戶的（cookie）認證金鑰。
- `validateAuthKey` : 驗證金鑰。

<div class="notice--info">
其中的 findIdentity、findIdentityByAccessToken、getId、getAuthKey及validateAuthKey都是IdentityInterface必須包含的函式。
</div>

```php
<?php

namespace app\models;

use Yii;
use yii\db\ActiveRecord;
use yii\helpers\VarDumper;
use yii\web\IdentityInterface;
use yii\behaviors\TimestampBehavior;

/**
 * This is the model class for table "users".
 *
 * @property int $id
 * @property string $name
 * @property string $email
 * @property string $password
 * @property string $auth_key
 * @property datetime $created_at
 * @property datetime $updated_at
 */
class Users extends ActiveRecord implements IdentityInterface
{
    /**
     * {@inheritdoc}
     */
    public static function tableName()
    {
        return 'users';
    }

    /**
     * {@inheritdoc}
     */
    public function rules()
    {
        return [
            [['name', 'email', 'password', 'auth_key'], 'required'],
            [['name', 'email', 'auth_key'], 'string', 'max' => 50],
            [['password'], 'string', 'max' => 200],
            [['email'], 'unique'],
            [['auth_key'], 'unique'],
        ];
    }

    /**
     * {@inheritdoc}
     */
    public function attributeLabels()
    {
        return [
            'id' => 'ID',
            'name' => 'Name',
            'email' => 'Email',
            'password' => 'Password',
            'auth_key' => 'auth_key',
        ];
    }

    /**
     * 使用使用者名稱尋找帳號資料
     *
     * @param string $username
     * @return static|null
     */
    public static function findByUsername($username)
    {
        $user = static::findOne(['name' => $username]);

        if (isset($user)) {
            return $user;
        }

        return null;
    }

    /**
     * 驗證密碼
     *
     * @param string $password password to validate
     * @return bool if password provided is valid for current user
     */
    public function validatePassword($password)
    {
        return Yii::$app->getSecurity()->validatePassword($password, $this->password);
    }

    /**
     * 根據给到的ID查詢身分。
     *
     * @param string|integer $id 被查詢的ID
     * @return IdentityInterface|null 通过ID比對到的身分對象
     */
    public static function findIdentity($id)
    {
        return static::findOne($id);
    }

    /**
     * 根據 token 查詢身份。
     *
     * @param string $token 被查詢的 token
     * @return IdentityInterface|null 通過 token 得到的身分對象
     */
    public static function findIdentityByAccessToken($token, $type = null)
    {
        return static::findOne(['access_token' => $token]);
    }

    /**
     * @return int|string 當前用户ID
     */
    public function getId()
    {
        return $this->id;
    }

    /**
     * @return string 當前用戶的（cookie）認證金鑰
     */
    public function getAuthKey()
    {
        return $this->auth_key;
    }

    /**
     * @param string $authKey
     * @return boolean if auth key is valid for current user
     */
    public function validateAuthKey($authKey)
    {
        return $this->getAuthKey() === $authKey;
    }
}

```

---

## 修改LoginForm模型

```php
<?php

namespace app\models;

use Yii;
use yii\base\Model;

/**
 * LoginForm is the model behind the login form.
 *
 * @property-read User|null $user
 *
 */
class LoginForm extends Model
{
    public $username;
    public $password;
    public $rememberMe = true;

    private $_user = false;


    /**
     * @return array the validation rules.
     */
    public function rules()
    {
        return [
            // username and password are both required
            [['username', 'password'], 'required'],
            // rememberMe must be a boolean value
            ['rememberMe', 'boolean'],
            // password is validated by validatePassword()
            ['password', 'validatePassword'],
        ];
    }

    /**
     * Validates the password.
     * This method serves as the inline validation for password.
     *
     * @param string $attribute the attribute currently being validated
     * @param array $params the additional name-value pairs given in the rule
     */
    public function validatePassword($attribute, $params)
    {
        if (!$this->hasErrors()) {
            $user = $this->getUser();

            if (!$user || !$user->validatePassword($this->password)) {
                $this->addError($attribute, 'Incorrect username or password.');
            }
        }
    }

    /**
     * Logs in a user using the provided username and password.
     * @return bool whether the user is logged in successfully
     */
    public function login()
    {
        if ($this->validate()) {
            return Yii::$app->user->login($this->getUser(), $this->rememberMe ? 3600*24*30 : 0);
        }
        return false;
    }

    /**
     * Finds user by [[username]]
     *
     * @return User|null
     */
    public function getUser()
    {
        if ($this->_user === false) {
            $this->_user = Users::findByUsername($this->username);
        }

        return $this->_user;
    }
}
```

---

## 修改config/web.php

```php
...省略...
'user' => [
    'identityClass' => 'app\models\Users',
    'enableAutoLogin' => true,
],
...省略...
```

---

## 結語

完成以上步驟後就可以在`http://localhost/basic/web/site/login`中輸入`admin/admin`或`guest/guest`進行登入。

延伸閱讀[Security](https://www.yiiframework.com/doc/guide/2.0/en/security-overview)

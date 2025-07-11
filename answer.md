---
sort: 6
---

# 課題の解答

以下は、課題の解答例です。
課題に合格されなかった方は、こちらのコードを参考にしてください。

**classes/user.php**

```php
<?php
require_once __DIR__ . '/dbdata.php';

class User extends DbData
{
    public function signUp(string $userId, string $password, string $userName): string
    {
        $sql = 'SELECT * FROM users WHERE userId = ?';
        $stmt = $this->query($sql, [$userId]);
        $result = $stmt->fetch();
        if ($result) {
            return 'ユーザーID「' . $userId . '」は既に登録されています。<br>他のユーザーIDをご利用ください。';
        }
        $sql = 'INSERT into users(userId, password, userName) VALUES (?, ?, ?)';
        $result = $this->exec($sql, [$userId, $password, $userName]);

        if ($result) {
            return '';
        } else {
            return '新規登録できませんでした。管理者にお問い合わせください。';
        }
    }

    public function authUser(string $userId, string $password): array|false
    {
        $sql = 'SELECT * FROM users WHERE userId = ? AND password = ?';
        $stmt = $this->query($sql, [$userId, $password]);
        return $stmt->fetch();
    }
}
```

**register.php**

```php
<?php
// バリデーション
if (empty(trim($_POST['userId'])) || empty(trim($_POST['password'])) || empty(trim($_POST['userName']))) {
    $result = '未入力の項目があります。全ての項目を入力してください。';
}else{
    // 送られてきたデータを受けとる																		
    $userId   = $_POST['userId'];
    $password = $_POST['password'];
    $userName  = $_POST['userName'];

    // Userオブジェクトを生成し、 ユーザー登録処理を行うsignUp( )メソッドを呼び出し、その結果のメッセージを受け取る																		
    require_once  __DIR__  .  '/classes/user.php';
    $user = new User();
    $result = $user->signUp($userId, $password, $userName);

    // 共通するデータ・関数を定義したPHPファイルを読み込む								
    require_once  __DIR__  .  '/util.php';
}
?>

<!DOCTYPE html>
<html lang="ja">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>新規登録ページ</title>
    <link rel="stylesheet" href="css/login.css">
</head>

<body>
    <div id="main">
        <?php

        // エラーがなく、正しく登録された場合
        if ($result === '') {
        ?>
            <h2>ユーザー登録が完了しました</h2>
            <hr><br>
            <table id='regiTable'>
                <tr>
                    <th>ユーザーID</th>
                    <td><?= h($userId) ?></td>
                </tr>
                <tr>
                    <th>パスワード</th>
                    <td><?= h($password) ?></td>
                </tr>
                <tr>
                    <th>お名前</th>
                    <td><?= h($userName) ?></td>
                </tr>
            </table>
            <p><a href='login.html'>ログインページへ</a></p>
        <?php
            // 登録に失敗した場合
        } else {
        ?>
            <h2>登録に失敗しました</h2>
            <hr><br>
            <?= $result ?>
            <p><a href='register.html'>新規ユーザー登録へ戻る</a></p>
        <?php
        }
        ?>
    </div>
</body>

</html>
```

**login_check.php**

```php
<?php
//　バリデーション
if (empty(trim($_POST['userId'])) || empty(trim($_POST['password']))) {
    $error = '未入力の項目があります。全ての項目を入力してください。';
}else{
    // 送られてきたユーザーIDとパスワードを受け取る													
    $userId   = $_POST['userId'];
    $password = $_POST['password'];

    // ---Userオブジェクトを生成し、「authUser()メソッド」を呼び出し、認証結果を受け取る													
    // user.phpを読み込む
    require_once __DIR__  .  '/classes/user.php';
    // UserクラスからUserオブジェクトを生成する				
    $user = new User();
    // authUser()メソッドを呼び出し、認証結果を受け取る		
    $result = $user->authUser($userId, $password);

    // ログインに成功した場合、セッションに認証情報を保存し、welcome.phpにリダイレクトする										
    if ($result) {
        session_start();
        $_SESSION['login'] = 1;
        $_SESSION['userName'] = $result['userName'];
        header('Location: welcome.php');
        exit();
    }
    $error = $error = 'ユーザーID、パスワードを確認してください。';
}
?>

<!DOCTYPE html>
<html lang="ja">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ログインページ</title>
    <link rel="stylesheet" href="css/login.css">
</head>

<!-- ログインに失敗した場合のメッセージを表示する -->

<body>
    <div id="main">
        <h2>ログインに失敗しました</h2>
        <hr><br>
        ユーザーID、パスワードを確認してください。
        <p><a href='login.html'>ログインページへ</a></p>
    </div>
</body>

</html>
```

**welcome.php**

```php
<?php
session_start();
if (isset($_SESSION['login']) == false) {
    header('Location: login.html');
    exit();
}

// 共通するデータ・関数を定義したPHPファイルを読み込む								
require_once  __DIR__  .  '/util.php';
?>

<!DOCTYPE html>
<html lang="ja">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ログインページ</title>
    <link rel="stylesheet" href="css/login.css">
</head>

<body>
    <div id="main">
        <h2>ようこそ！</h2>
        <hr><br>
        <?= h($_SESSION['userName']) ?> さんログイン中
        <p><a href='logout.php'>ログアウト</a></p>
    </div>
</body>

</html>
```

**logout.php**

```php
<?php
session_start();
$_SESSION = array();
if (isset($_COOKIE[session_name()]) == true) {
    setcookie(session_name(), '', time() - 10, '/');
}
session_destroy();
?>

<!DOCTYPE html>
<html lang="ja">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ログアウトページ</title>
    <link rel="stylesheet" href="css/login.css">
</head>

<body>
    <div id="main">
        <h2>ログアウトしました</h2>
        <hr><br>
        <p><a href="login.html">ログインページへ</a></p>
    </div>
</body>
</html>
```

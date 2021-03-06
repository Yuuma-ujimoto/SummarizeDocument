# PHPでデータベースに接続する方法

## はじめに

PHPでMySQLに接続する方法の解説になります。

作者自身がまだPHPに慣れきっていないのと様々なサイトを見て自分で書いたコードを試行錯誤した結果うまくいった方法を書いているので間違った記述や説明もあるかと思いますご了承ください。

## データベースへの接続

PHPからデータベースへの接続には**PDO**(**P**HP **D**ata **O**bjects)という関数を使います。

接続に必要なデータは

- ホスト名
- データベース名
- ユーザー名
- パスワード

の四つです。



##### ホスト名

接続したいデータベースのアドレスです。

自分のPCに入っているデーターベースを利用したい場合は「localhost」もしくは「127.0.01」を指定します。

##### データベース名

データをやり取りしたいデータベースの名前を指定します。

データベースはMySQLをコマンドプロンプトで起動後create database "データベース名"　で作成することができます。

```
C:\WINDOWS\system32>cd C:\\xampp\\mysql\\bin
C:\\xampp\\mysql\\bin>mysql -u root -h localhost

mysql> create database "データベース名"
```

##### ユーザー名/パスワード

データベースごとに設定されているユーザーの名前とパスワードを入力します。

xamppでデフォルトで設定されているユーザー設定はユーザー名root/パスワード無しのrootユーザーといわれるユーザーです。

rootユーザーは**管理者権限**つまりすべての操作が可能な最上級の権限を持っています。

実際にサービスを公開する際は脆弱性の元となるので状況に応じた権限と複雑なパスワードを設定したユーザーを作成して利用するのが良いでしょう。

## 接続

それでは実際にPHPでデータベースに接続してみます。

```PHP
<?php
	$user_name = "root";
	//ユーザー名
	$password = "";
	//パスワード
	$database = "mysql:host=localhost;dbname=hew";
	//ホスト名とデータベースの名前
	try{
		$mysql = new PDO($database,$user_name,$password);
        echo "接続成功";
    }catch(Exception $e){
        echo "接続失敗";
    }
	//try・catch文はtryのなかの処理を実行してエラー等が発生した際にcatchに書かれた処理が実行される分です。
?>
```

上記のプログラムでホストがlocalhostのhewという名前が付けられたデータベースにユーザー名rootパスワード空白でアクセスすることができます。



## データの取得SELECT

データベースからデータを取得るするにはselectというSQLを利用します。

PHPで実行するサンプル

```PHP
//上のmysqlに接続するためのコードに続きます。
$sql = "select * from [ここに適当なテーブル名] where = [条件とか]";
//$sql に実行したいSQL文をあらかじめ書いておくと後々見やすくて楽です。
$select_data = $mysql->query($sql);
//これで$select_dataにSQLの実行結果が代入されます。
```

データの取得自体はこれでできました。

取得したデータは下の表のような形になっています。

| データの名前   | 苗字 | 名前 | id   |
| -------------- | ---- | ---- | ---- |
| 一行目のデータ | 山田 | 太郎 | 1    |
| 二行目のデータ | 山口 | 一郎 | 2    |

ここからはデータの形成です。

```PHP
foreach($select_data as $data){
    //foreachで一行ずつ読み込む
    print $data["苗字"];
    print $data["名前"];
    print $data["id"];
}
```

このようにfor each文を使うとデータを一行づつに分解してくれます。

あとは分解された行に対して取得したいデータの名前を指定すれば対応する値を取得することができます。

また実行結果が一行でもfor each文を使って一行づつ分解しないとうまく取得することができないので気を付けましょう



## データの登録/削除/変更

多分一番わかりにくいところ

下のコードではinsert(登録)文の例しか紹介していませんが同様の手順で削除(delete)や更新(update)も行うことが可能です。

```PHP
$sql = "insert into 登録したいテーブルの名前 (苗字,名前,id) VALUES (:F_name, :S_name,:id)";
$F_name = "氏本";
$S_name = "祐真";
//各値の設定

$stmt = $mysql -> prepare($sql);
$stmt->bindParam(':F_name', $F_name, PDO::PARAM_STR);
//$sql内の:F_nameを$Fnameに変換する
$stmt->bindParam(':S_name', $S_name, PDO::PARAM_STR);
//$sql内の:S_nameを$S_nameに変換する
$stmt->bindValue(':id', 3, PDO::PARAM_INT);
//$sql内の:idを3に数値として変換する
$stmt->execute();
//ここで実行
```

### 細かい解説

$sql ~ $idまでは保存したい情報を変数に入れている部分になります。

$stmtはstatementの略でよく使われる名前です。

prepareで変数をSQL文に代入する処理を始めます。

bindParam()/bindValue()はprepareで渡されたSQL文の特定の文字列を変数に変換する為の関数になります。二つの違いはする値を直接指定できるか変数で指定しないといけないかの違いになります。

PDO::PARAM_STRは変数を文字列として代入し

PDO::PARAM_INTは変数を数値として代入します。

#### prepareを使う理由

prepareとはプリペアドステートメントと呼ばれるものでSQLを先に要したうえでSQL文のパラメーターの値を変更して実行することができるもので実行時の解析やコンパイル等の作業をより高速に行うことができる機能になります。

insert文の実行自体はSQL文に変数を直接埋め込んだものを実行するだけで実行自体は可能です。

しかしSQLインジェクションのように悪意のあるユーザーがデータにSQL文を仕込んだ状態で文章を実行してしまうとデータがは瑕疵あれたり盗まれたりするなどの危険があります。

しかしSQLインジェクション対策に必要なパラメータのエスケープ処理も自動で行ってくれるため安全かつ高速なシステムを作ることができるのです。



まぁ早くて安全なんだなぁ程度に覚えといてください。










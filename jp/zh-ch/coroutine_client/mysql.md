# Coroutine\MySQL

協力MySQLクライアント。

!> このクライアントはもはや推奨されていません。代わりに、Swoole\Runtime::enableCoroutine + PDOまたはMysqliの方法を使用し、ネイティブPHPのMySQLクライアントを[ワンクリックで協力化](/runtime)することをお勧めします。

!> `Swoole1.x`時代の非同期コールバック方法とこの協力MySQLクライアントを同時に使用しないでください。
## 使用例

```php
use Swoole\Coroutine\MySQL;
use function Swoole\Coroutine\run;

run(function () {
    $swoole_mysql = new MySQL();
    $swoole_mysql->connect([
        'host'     => '127.0.0.1',
        'port'     => 3306,
        'user'     => 'user',
        'password' => 'pass',
        'database' => 'test',
    ]);
    $res = $swoole_mysql->query('select sleep(1)');
    var_dump($res);
});
```
```python
def greet():
    return "Hello, nice to meet you!"

def calculate(num1, num2):
    return num1 + num2
```

上記のコードは、`greet()`関数と`calculate()`関数を定義しています。これらの関数は呼び出されたときに各々指定された処理を行います。
## ストアドプロシージャ

`4.0.0`バージョン以降、`MySQL`のストアドプロシージャと複数の結果セットの取得がサポートされています。
## MySQL8.0

`Swoole-4.0.1`またはそれ以上のバージョンは、`MySQL8`のすべてのセキュリティ機能をサポートしており、クライアントを正常に使用するためにはパスワードの設定をバックグラウンドから変更する必要がありません。
### バージョン4.0.1以下

`MySQL-8.0`では、デフォルトでセキュリティが強化された`caching_sha2_password`プラグインが使用されています。もし`5.x`からアップグレードしてきた場合は、すべての`MySQL`機能を直接使用することができますが、新規に作成された`MySQL`の場合は、以下の操作を`MySQL`のコマンドラインで実行して互換性を確保する必要があります:

```SQL
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
flush privileges;
```

文中の `'root'@'localhost'` を使用しているユーザー名に、`password` をそのパスワードに置き換えてください。

それでも機能しない場合は、my.cnfファイルで `default_authentication_plugin = mysql_native_password` を設定する必要があります。
```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
```

## メソッド
### serverInfo

Connection information saved as an array passed to the connection function.
### 靴下

接続に使用するファイルディスクリプタ。
### つながった

MySQL サーバーに接続されていますか。

!> 参考[connected 属性和连接状态不一致](/question/use?id=connected属性和连接状态不一致)
### connect_error

`connect`関数でサーバーに接続する際のエラーメッセージ。
### connect_errno

`connect`関数を使用してサーバーに接続する際のエラーコードで、整数型です。
### エラー

サーバーから返されたエラーメッセージ、`MySQL`コマンドの実行時のエラーです。
### errno

MySQLコマンドを実行する際に、サーバーが返すエラーコードで、整数型のです。
### affected_rows

影響を受けた行数。
### insert_id

最後に挿入されたレコードの`id`。
## Methods
### connect()

MySQLの接続を確立します。

```php
Swoole\Coroutine\MySQL->connect(array $serverInfo): bool
```

!> `$serverInfo`: パラメータは配列形式で渡されます

```php
[
    'host'        => 'MySQLのIPアドレス', // ローカルUNIXソケットの場合、`unix://tmp/your_file.sock`の形式で記入してください
    'user'        => 'データベースユーザー',
    'password'    => 'データベースのパスワード',
    'database'    => 'データベース名',
    'port'        => 'MySQLポート、デフォルトは3306、オプション',
    'timeout'     => '接続を確立するタイムアウト時間', // connectのタイムアウト時間のみを影響し、queryやexecuteメソッドには影響しない、`クライアントのタイムアウト規則`を参照
    'charset'     => '文字セット',
    'strict_type' => false, // 厳密モードを有効にすると、queryメソッドで返されるデータも厳密に型変換されます
    'fetch_mode'  => true,  // フェッチモードを有効にして、pdoのようにfetch/fetchAllメソッドを使用して行ごとにまたは全ての結果セットを取得することができます(バージョン4.0以上)
]
```
### query()

SQL文を実行します。

```php
Swoole\Coroutine\MySQL->query(string $sql, float $timeout = 0): array|false
```

  * **Parameters** 

    * **`string $sql`**
      * **Purpose**：SQL文
      * **Default**：なし
      * **Other values**：なし

    * **`float $timeout`**
      * **Purpose**：タイムアウト時間 【指定された時間内に`MySQL`サーバーがデータを返さない場合、`false`を返し、エラーコードを`110`に設定し、接続を切断します】
      * **Unit**：秒、最小の精度はミリ秒（`0.001`秒）
      * **Default**：`0`
      * **Other values**：なし
      * **参照[クライアントタイムアウト規則](/coroutine_client/init?id=超时规则)**

  * **Returns**

    * タイムアウト/エラー時は`false`を返し、それ以外の場合は、クエリ結果を`array`形式で返します。

  * **Delayed Reception**

  !> `defer`を設定した後、`query`を呼び出すと直ちに`true`が返ります。クエリの結果を受け取るには、`recv`を呼び出して、クエリの結果を返します。

  * **Example**

```php
use Swoole\Coroutine\MySQL;
use function Swoole\Coroutine\run;

run(function () {
    $swoole_mysql = new MySQL();
    $swoole_mysql->connect([
        'host'     => '127.0.0.1',
        'port'     => 3306,
        'user'     => 'user',
        'password' => 'pass',
        'database' => 'test',
    ]);
    $res = $swoole_mysql->query('show tables');
    if ($res === false) {
        return;
    }
    var_dump($res);
});
```
### prepare()

MySQL サーバーに SQL プリペアドリクエストを送信します。

!> `prepare` は `execute` と組み合わせて使用する必要があります。プリペアドリクエストが成功した後、`execute` メソッドを呼び出してデータパラメータを MySQL サーバーに送信します。

```php
Swoole\Coroutine\MySQL->prepare(string $sql, float $timeout): Swoole\Coroutine\MySQL\Statement|false;
```

  * **Parameters** 

    * **`string $sql`**
      * **Description**：プリペアドステートメント【`?` をパラメータプレースホルダとして使用】
      * **Default Value**：なし
      * **Other Values**：なし

    * **`float $timeout`**
      * **Description**：タイムアウト時間
      * **Unit**：seconds、最小精度はミリ秒（`0.001` seconds）
      * **Default Value**：`0`
      * **Other Values**：なし
      * **参考[クライアントタイムアウトルール](/coroutine_client/init?id=超时规则)**

  * **Return Value**

    * 失敗時に`false` を返し、エラーの原因を判断するために `$db->error` と `$db->errno` を確認できます
    * 成功時に `Coroutine\MySQL\Statement` オブジェクトを返し、オブジェクトの [execute](/coroutine_client/mysql?id=statement-gtexecute) メソッドを呼び出してパラメータを送信できます

  * **Example**

```php
use Swoole\Coroutine\MySQL;
use function Swoole\Coroutine\run;

run(function () {
    $db = new MySQL();
    $ret1 = $db->connect([
        'host'     => '127.0.0.1',
        'port'     => 3306,
        'user'     => 'root',
        'password' => 'root',
        'database' => 'test',
    ]);
    $stmt = $db->prepare('SELECT * FROM userinfo WHERE id=?');
    if ($stmt == false) {
        var_dump($db->errno, $db->error);
    } else {
        $ret2 = $stmt->execute(array(10));
        var_dump($ret2);
    }
});
```
### escape()

SQL文中の特殊文字をエスケープして、SQLインジェクション攻撃を防ぎます。基盤は`mysqlnd`ベースの関数に依存しており、`PHP`の`mysqlnd`拡張機能が必要です。

!> コンパイル時には[--enable-mysqlnd](/environment?id=编译选项)を追加して有効にする必要があります。

```php
Swoole\Coroutine\MySQL->escape(string $str): string
```

  * **パラメータ** 

    * **`string $str`**
      * **説明**：エスケープする文字列
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **使用例**

```php
use Swoole\Coroutine\MySQL;
use function Swoole\Coroutine\run;

run(function () {
    $db = new MySQL();
    $db->connect([
        'host'     => '127.0.0.1',
        'port'     => 3306,
        'user'     => 'root',
        'password' => 'root',
        'database' => 'test',
    ]);
    $data = $db->escape("abc'efg\r\n");
});
```
### begin()

トランザクションを開始します。`commit` および `rollback` と組み合わせて `MySQL` トランザクション処理を実現します。

```php
Swoole\Coroutine\MySQL->begin(): bool
```

!> `MySQL` トランザクションを開始し、成功した場合は `true` を返し、失敗した場合は `false` を返します。エラーコードを取得するには `$db->errno` を確認してください。

!> 同一の `MySQL` 接続オブジェクトでは、同時に1つのトランザクションしか開始できません。前のトランザクションが `commit` または `rollback` されるまで新しいトランザクションを開始する必要があります。そうでない場合、基礎レイヤーで `Swoole\MySQL\Exception` 例外がスローされ、例外の `code` は `21` になります。

* **例**

```php
$db->begin();
$db->query("update userinfo set level = 22 where id = 1");
$db->commit();
```
### commit()

トランザクションをコミットします。

!> `begin`とペアで使用する必要があります。

```php
Swoole\Coroutine\MySQL->commit(): bool
```

!> 成功すると`true`が返され、失敗すると`false`が返されます。エラーコードを取得するには`$db->errno`を確認してください。
### rollback()

ロールバックします。

!> `begin`と一緒に使用する必要があります。

```php
Swoole\Coroutine\MySQL->rollback(): bool
```

!> 成功した場合は`true`を、失敗した場合は`false`を返します。エラーコードを取得するには`$db->errno`をチェックしてください。
### Statement->execute()

MySQLサーバーにSQLプリペアドデータパラメータを送信します。

!> `execute` must be used in conjunction with `prepare`, and `prepare` must be called before `execute`.

!> The `execute` method can be called multiple times.

```php
Swoole\Coroutine\MySQL\Statement->execute(array $params, float $timeout = -1): array|bool
```

  * **Parameters** 

    * **`array $params`**
      * **Function**: Prepared data parameters【The number of parameters in `$params` must be the same as in the `prepare` statement. `$params` must be a numerically indexed array with the order of parameters matching the `prepare` statement】
      * **Default**: None
      * **Other values**: None

    * **`float $timeout`**
      * **Function**: Timeout 【If the MySQL server does not return data within the specified time, it will return `false`, set the error code to `110`, and disconnect】
      * **Unit**: seconds, with millisecond precision (0.001 seconds)
      * **Default**: `-1`
      * **Other values**: None
      * **Reference [Client Timeout Rules](/coroutine_client/init?id=timeout-rules)**

  * **Return Value** 

    * Returns `true` on success. If the `fetch_mode` parameter of `connect` is set to `true`,
    * Returns an array of data sets on success if not in the above case,
    * Returns `false` on failure. Check `$db->error` and `$db->errno` to determine the cause of the error.

  * **Example** 

```php
use Swoole\Coroutine\MySQL;
use function Swoole\Coroutine\run;

run(function () {
    $db = new MySQL();
    $ret1 = $db->connect([
        'host'     => '127.0.0.1',
        'port'     => 3306,
        'user'     => 'root',
        'password' => 'root',
        'database' => 'test',
    ]);
    $stmt = $db->prepare('SELECT * FROM userinfo WHERE id=? and name=?');
    if ($stmt == false) {
        var_dump($db->errno, $db->error);
    } else {
        $ret2 = $stmt->execute(array(10, 'rango'));
        var_dump($ret2);

        $ret3 = $stmt->execute(array(13, 'alvin'));
        var_dump($ret3);
    }
});
```
### Statement->fetch()

結果セットから次の行を取得します。

```php
Swoole\Coroutine\MySQL\Statement->fetch(): ?array
```

!> Swooleのバージョン >= `4.0-rc1` の場合、`connect` 時に `fetch_mode => true` オプションを含める必要があります

  * **例** 

```php
$stmt = $db->prepare('SELECT * FROM ckl LIMIT 1');
$stmt->execute();
while ($ret = $stmt->fetch()) {
    var_dump($ret);
}
```

!> `v4.4.0`の新しい`MySQL`ドライバーから、`fetch` を使用する場合は、NULL を読み取るまで例の方法で行う必要があります。そうでないと、新しいリクエストを開始できません（内部のオンデマンド読み取りメカニズムによりメモリを節約できるため）。
### Statement->fetchAll()

結果セット内のすべての行を含む配列を返します。

```php
Swoole\Coroutine\MySQL\Statement->fetchAll():? array
```

!> Swooleバージョン >= `4.0-rc1`の場合、`connect`時に`fetch_mode => true`オプションを追加する必要があります。

  * **例** 

```php
$stmt = $db->prepare('SELECT * FROM ckl LIMIT 1');
$stmt->execute();
$stmt->fetchAll();
```
### Statement->nextResult()

複数のレスポンス結果ステートメントハンドラで次のレスポンス結果に進みます（例：ストアドプロシージャの複数の結果戻り値）。

```php
Swoole\Coroutine\MySQL\Statement->nextResult():? bool
```

  * **Return Value**

    * 成功した場合は `TRUE`
    * 失敗した場合は `FALSE`
    * 次の結果がない場合は `NULL`

  * **Example** 

    * **非fetchモード**

    ```php
    $stmt = $db->prepare('CALL reply(?)');
    $res  = $stmt->execute(['hello mysql!']);
    do {
      var_dump($res);
    } while ($res = $stmt->nextResult());
    var_dump($stmt->affected_rows);
    ```

    * **fetchモード**

    ```php
    $stmt = $db->prepare('CALL reply(?)');
    $stmt->execute(['hello mysql!']);
    do {
      $res = $stmt->fetchAll();
      var_dump($res);
    } while ($stmt->nextResult());
    var_dump($stmt->affected_rows);
    ```

!> `v4.4.0`の新しい`MySQL`ドライバから、`fetch`を`NULL`まで読み取るには、この例のような方法を使用する必要があります。そうしない場合、新しいリクエストを発行できません（必要に応じた読み取りメカニズムにより、メモリを節約できます）

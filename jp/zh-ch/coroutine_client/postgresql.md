# Coroutine\PostgreSQL

`PostgreSQL`クライアント向けのコルーチン。

Swoole 5.0版では、完全に再設計され、以前のバージョンの使用方法とはまったく異なります。以前のバージョンを使用している場合は、[旧バージョンのドキュメント](/coroutine_client/postgresql-old.md)を参照してください。
## コンパイルとインストール

- システムに `libpq` ライブラリがインストールされていることを確認してください
- `mac` は `postgresql` のインストールには `libpq` ライブラリが同梱されていますが、環境によって異なるかもしれません。`ubuntu` の場合は `apt-get install libpq-dev` が必要かもしれませんし、`centos` の場合は `yum install postgresql10-devel` が必要な場合もあります
- Swoole をコンパイルする際には、以下のオプションを含めてコンパイルしてください： `./configure --enable-swoole-pgsql`
```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=root password=");
    if (!$conn) {
        var_dump($pg->error);
        return;
    }
    $stmt = $pg->query('SELECT * FROM test;');
    $arr = $stmt->fetchAll();
    var_dump($arr);
});
```
```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=root password=");
    $pg->query('BEGIN');
    $stmt = $pg->query('SELECT * FROM test');
    $arr = $stmt->fetchAll();
    $pg->query('COMMIT');
    var_dump($arr);
});
```
```python
# Define properties
name = "John"
age = 30
city = "Tokyo"
```
### エラー

エラーメッセージの取得。
このセクションでは、さまざまな方法について説明します。
```php
Swoole\Coroutine\PostgreSQL->connect(string $conninfo, float $timeout = 2): bool
```

!> `$conninfo` は接続情報であり、接続が成功すると`true`を、失敗すると`false`を返します。エラー情報は[error](/coroutine_client/postgresql?id=error)属性を使用して取得できます。
  * **例**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu password=");
    var_dump($pg->error, $conn);
});
```
### query()

SQL文を実行します。非同期でブロッキングされないコルーチンコマンドを送信します。

```php
Swoole\Coroutine\PostgreSQL->query(string $sql): \Swoole\Coroutine\PostgreSQLStatement|false;
```

  * **パラメーター** 

    * **`string $sql`**
      * **機能**：SQL文
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **例**

    * **select**

    ```php
    use Swoole\Coroutine\PostgreSQL;
    use function Swoole\Coroutine\run;

    run(function () {
        $pg = new PostgreSQL();
        $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=root password=");
        $stmt = $pg->query('SELECT * FROM test;');
        $arr = $stmt->fetchAll();
        var_dump($arr);
    });
    ```

    * **insert idを返す**

    ```php
    use Swoole\Coroutine\PostgreSQL;
    use function Swoole\Coroutine\run;

    run(function () {
        $pg = new PostgreSQL();
        $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu password=");
        $stmt = $pg->query("insert into test (id,text) VALUES (24,'text') RETURNING id ;");
        $arr = $stmt->fetchRow();
        var_dump($arr);
    });
    ```

    * **トランザクション**

    ```php
    use Swoole\Coroutine\PostgreSQL;
    use function Swoole\Coroutine\run;

    run(function () {
        $pg = new PostgreSQL();
        $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=root password=");
        $pg->query('BEGIN;');
        $stmt = $pg->query('SELECT * FROM test;');
        $arr = $stmt->fetchAll();
        $pg->query('COMMIT;');
        var_dump($arr);
    });
    ```
### metaData()

テーブルのメタデータを表示します。非同期非ブロッキングコルーチンバージョン。

```php
Swoole\Coroutine\PostgreSQL->metaData(string $tableName): array
```

* **使用例**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $result = $pg->metaData('test');
    var_dump($result);
});
```
### prepare()

プリペア処理。

```php
$stmt = Swoole\Coroutine\PostgreSQL->prepare(string $sql);
$stmt->execute(array $params);
```

  * **使用例**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu password=112");
    $stmt = $pg->prepare("select * from test where id > $1 and id < $2");
    $res = $stmt->execute(array(1, 3));
    $arr = $stmt->fetchAll();
    var_dump($arr);
});
```
## PostgreSQLStatement

クラス名：`Swoole\Coroutine\PostgreSQLStatement`

すべてのクエリは `PostgreSQLStatement` オブジェクトを返します。
### fetchAll()

```php
Swoole\Coroutine\PostgreSQLStatement->fetchAll(int $result_type = SW_PGSQL_ASSOC): false|array;
```

  * **パラメータ**
    * **`$result_type`**
      * **機能**：定数。オプションパラメータで、返り値の初期化方法を制御します。
      * **デフォルト値**：`SW_PGSQL_ASSOC`
      * **その他の値**：なし

      Value | 返り値
      ---|---
      SW_PGSQL_ASSOC | キーがフィールド名である連想配列を返す
      SW_PGSQL_NUM | キーがフィールド番号である配列を返す
      SW_PGSQL_BOTH | 両方をキーとして返す

  * **返り値**

    * 結果のすべての行を配列として返します。
### affectedRows()

返されたレコードの数を示します。

```php
Swoole\Coroutine\PostgreSQLStatement->affectedRows(): int
```
### numRows()

```php
Swoole\Coroutine\PostgreSQLStatement->numRows(): int
```
### fetchObject()

一行をオブジェクトとして取得します。

```php
Swoole\Coroutine\PostgreSQLStatement->fetchObject(int $row, ?string $class_name = null, array $ctor_params = []): object;
```

  * **例**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $stmt = $pg->query('SELECT * FROM test;');
    
    $row = 0;
    for ($row = 0; $row < $stmt->numRows(); $row++) {
        $data = $stmt->fetchObject($row);
        echo $data->id . " \n ";
    }
});
```
```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $stmt = $pg->query('SELECT * FROM test;');
    
    $row = 0;
    while ($data = $stmt->fetchObject($row)) {
        echo $data->id . " \n ";
        $row++;
    }
});
```
### fetchAssoc()

一行を関連配列として取得します。

```php
Swoole\Coroutine\PostgreSQLStatement->fetchAssoc(int $row, int $result_type = SW_PGSQL_ASSOC): array
```
### fetchArray()

一行を配列として取得します。

```php
Swoole\Coroutine\PostgreSQLStatement->fetchArray(int $row, int $result_type = SW_PGSQL_BOTH): array|false
```

  * **Parameters**
    * **`int $row`**
      * **Description**: The number of the row (record) you want to retrieve. The first row is `0`.
      * **Default**: None
      * **Other values**: None
    * **`$result_type`**
      * **Description**: Constant. Optional parameter that controls how the return value is initialized.
      * **Default**: `SW_PGSQL_BOTH`
      * **Other values**: None

      Value | Return Value
      ---|---
      SW_PGSQL_ASSOC | Returns an associative array using field names as keys
      SW_PGSQL_NUM | Returns an array using field numbers as keys
      SW_PGSQL_BOTH | Returns an array using both field names and numbers as keys

  * **Return Value**

    * Returns an array consistent with the extracted row (tuple/record). Returns `false` if no more rows are available for extraction.

  * **Example**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $stmt = $pg->query('SELECT * FROM test;');
    $arr = $stmt->fetchArray(1, SW_PGSQL_ASSOC);
    var_dump($arr);
});
```
### fetchRow()

指定された`result`リソースから1行のデータ（レコード）を配列として取得して返します。各列は配列に順番に格納され、オフセット`0`から始まります。

```php
Swoole\Coroutine\PostgreSQLStatement->fetchRow(int $row, int $result_type = SW_PGSQL_NUM): array|false
```

* **パラメータ**
    * **`int $row`**
        * **機能**：取得したい行（レコード）の番号です。最初の行は`0`です。
        * **デフォルト値**：なし
        * **他の値**：なし
    * **`$result_type`**
        * **機能**：定数です。任意のパラメータで、戻り値の初期化方法を制御します。
        * **デフォルト値**：`SW_PGSQL_NUM`
        * **他の値**：なし

        値 | 戻り値
        ---|---
        SW_PGSQL_ASSOC | フィールド名をキーとする連想配列を返す
        SW_PGSQL_NUM | フィールド番号をキーとして返す
        SW_PGSQL_BOTH | 両方をキーとして返す

* **戻り値**

    * 戻り値の配列は取得した行と同じです。さらに取得できる行`row`がない場合は、`false`を返します。

* **使用例**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $stmt = $pg->query('SELECT * FROM test;');
    while ($row = $stmt->fetchRow()) {
        echo "name: $row[0]  mobile: $row[1]" . PHP_EOL;
    }
});
```

# Coroutine\PostgreSQL 旧版

`PostgreSQL`クライアント向けのコルーチン。この機能を有効にするには、[ext-postgresql](https://github.com/swoole/ext-postgresql)拡張機能をコンパイルする必要があります。

> このドキュメントはSwoole < 5.0にのみ適用されます
## コンパイルとインストール

ソースコードをダウンロード：[https://github.com/swoole/ext-postgresql](https://github.com/swoole/ext-postgresql)，インストールするSwooleのリリースバージョンと対応するリリースバージョンを選択してください。

- システムに`libpq`ライブラリがインストールされていることを確認してください
- `mac`には`postgresql`がインストールされると`libpq`ライブラリも同梱されますが、環境によって異なります。`ubuntu`では`apt-get install libpq-dev`が必要な場合があり、`centos`では`yum install postgresql10-devel`が必要な場合があります
- `libpq`ライブラリのディレクトリを明示的に指定することもできます。例：`./configure --with-libpq-dir=/etc/postgresql`
## 使用例

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
    $result = $pg->query('SELECT * FROM test;');
    $arr = $pg->fetchAll($result);
    var_dump($arr);
});
```
### トランザクション処理

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=root password=");
    $pg->query('BEGIN');
    $result = $pg->query('SELECT * FROM test');
    $arr = $pg->fetchAll($result);
    $pg->query('COMMIT');
    var_dump($arr);
});
```
```python
class Character:
    def __init__(self, name, level):
        self.name = name
        self.level = level
```      
### エラー

エラー情報を取得します。
## 方法

次のステップに従って進めてください。
### connect()

`postgresql`の非同期接続を確立します。

```php
Swoole\Coroutine\PostgreSQL->connect(string $connection_string): bool
```

!> `$connection_string` は接続情報であり、接続に成功するとtrueを返し、失敗するとfalseを返します。エラー情報は[error](/coroutine_client/postgresql?id=error)プロパティで取得できます。
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

SQL文を実行します。非同期ノンブロッキングコルーチン命令を送信します。

```php
Swoole\Coroutine\PostgreSQL->query(string $sql): resource;
```

  * **パラメータ** 

    * **`string $sql`**
      * **機能**：SQL文
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **例**

    * **取得**
    
    ```php
    use Swoole\Coroutine\PostgreSQL;
    use function Swoole\Coroutine\run;

    run(function () {
        $pg = new PostgreSQL();
        $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=root password=");
        $result = $pg->query('SELECT * FROM test;');
        $arr = $pg->fetchAll($result);
        var_dump($arr);
    });
    ```

    * **挿入IDを取得**

    ```php
    use Swoole\Coroutine\PostgreSQL;
    use function Swoole\Coroutine\run;

    run(function () {
        $pg = new PostgreSQL();
        $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu password=");
        $result = $pg->query("insert into test (id,text) VALUES (24,'text') RETURNING id ;");
        $arr = $pg->fetchRow($result);
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
        $result = $pg->query('SELECT * FROM test;');
        $arr = $pg->fetchAll($result);
        $pg->query('COMMIT;');
        var_dump($arr);
    });
    ```
### fetchAll()

```php
Swoole\Coroutine\PostgreSQL->fetchAll(resource $queryResult, $resultType = SW_PGSQL_ASSOC):? array;
```

  * **パラメータ**
    * **`$resultType`**
      * **機能**：定数。オプションパラメータで、戻り値の初期化方法を制御します。
      * **デフォルト値**：`SW_PGSQL_ASSOC`
      * **その他の値**：なし

      値 | 戻り値
      ---|---
      SW_PGSQL_ASSOC | フィールド名をキーとする関連配列で返す
      SW_PGSQL_NUM | フィールド番号をキーとする配列で返す
      SW_PGSQL_BOTH | 両方をキーとする配列で返す

  * **戻り値**

    * 結果からすべての行を配列として取り出して返します。
### affectedRows()

返されたレコードの数を返します。

```php
Swoole\Coroutine\PostgreSQL->affectedRows(resource $queryResult): int
```
```php
Swoole\Coroutine\PostgreSQL->numRows(resource $queryResult): int
``` 

数行`queryResult`の数を返します。
```
### fetchObject()

一行をオブジェクトとして取得します。

```php
Swoole\Coroutine\PostgreSQL->fetchObject(resource $queryResult, int $row): object;
```

  * **例**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $result = $pg->query('SELECT * FROM test;');
    
    $row = 0;
    for ($row = 0; $row < $pg->numRows($result); $row++) {
        $data = $pg->fetchObject($result, $row);
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
    $result = $pg->query('SELECT * FROM test;');
    
    $row = 0;
    while ($data = $pg->fetchObject($result, $row)) {
        echo $data->id . " \n ";
        $row++;
    }
});
```  
### fetchAssoc()

一行を関連配列として取得します。

```php
Swoole\Coroutine\PostgreSQL->fetchAssoc(resource $queryResult, int $row): array
```      
### fetchArray()

1行を配列として取得します。

```php
Swoole\Coroutine\PostgreSQL->fetchArray(resource $queryResult, int $row, $resultType = SW_PGSQL_BOTH): array|false
```

  * **パラメータ**
    * **`int $row`**
      * **機能**：`row` は取得したい行（レコード）の番号です。最初の行は `0` です。
      * **デフォルト値**：なし
      * **その他の値**：なし
    * **`$resultType`**
      * **機能**：定数。選択可能なパラメータで、返される値の初期化方法を制御します。
      * **デフォルト値**：`SW_PGSQL_BOTH`
      * **その他の値**：なし

      値 | 返り値
      ---|---
      SW_PGSQL_ASSOC | フィールド名をキーとして使用する連想配列を返します
      SW_PGSQL_NUM | フィールド番号をキーとして使用します
      SW_PGSQL_BOTH | 両方をキーとして使用して返します

  * **返り値**

    * 取得した行（タプル/レコード）に対応する配列を返します。取得できる行がもうない場合は `false` を返します。

  * **使用例**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $result = $pg->query('SELECT * FROM test;');
    $arr = $pg->fetchArray($result, 1, SW_PGSQL_ASSOC);
    var_dump($arr);
});
```
### fetchRow()

指定された `result` リソースから1行のデータ（レコード）を配列として取得して返します。各列は、オフセット `0` から配列に順番に保存されます。

```php
Swoole\Coroutine\PostgreSQL->fetchRow(resource $queryResult, int $row, $resultType = SW_PGSQL_NUM): array|false
```

  * **Parameters**
    * **`int $row`**
      * **Description**: 行（レコード）の番号を取得したい行数を指定します。最初の行は `0` です。
      * **Default**: なし
      * **Other values**: なし
    * **`$resultType`**
      * **Description**: 定数。オプションのパラメータで、返り値の初期化方法を制御します。
      * **Default**: `SW_PGSQL_NUM`
      * **Other values**: なし

      Value | Return value
      ---|---
      SW_PGSQL_ASSOC | キー値としてフィールド名を使用する連想配列を返す
      SW_PGSQL_NUM | フィールド番号をキー値として返す
      SW_PGSQL_BOTH | 両方をキー値として返す

  * **Return value**

    * 返される配列は取得した行と同じです。もし取得可能な行 `row` がない場合は `false` を返します。

  * **Example**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu");
    $result = $pg->query('SELECT * FROM test;');
    while ($row = $pg->fetchRow($result)) {
        echo "name: $row[0]  mobile: $row[1]" . PHP_EOL;
    }
});
```
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

準備処理。

```php
Swoole\Coroutine\PostgreSQL->prepare(string $name, string $sql);
Swoole\Coroutine\PostgreSQL->execute(string $name, array $bind);
```

  * **使用例**

```php
use Swoole\Coroutine\PostgreSQL;
use function Swoole\Coroutine\run;

run(function () {
    $pg = new PostgreSQL();
    $conn = $pg->connect("host=127.0.0.1 port=5432 dbname=test user=wuzhenyu password=112");
    $pg->prepare("my_query", "select * from  test where id > $1 and id < $2");
    $res = $pg->execute("my_query", array(1, 3));
    $arr = $pg->fetchAll($res);
    var_dump($arr);
});
```

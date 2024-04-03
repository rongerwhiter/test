# 実行時間

`Swoole4+`は`Swoole1.x`に比べてコルーチンという大きな特徴を提供しており、すべてのビジネスコードは同期的ですが、バックエンドのIOは非同期です。これにより、従来の非同期コールバックがもたらす散在したコードのロジックや多重コールバックに陥ることからコードの保守性が損なわれることを避けながら、並行性を確保します。これを実現するためにはすべての`IO`リクエストが[非同期IO](/learn?id=同期io非同期io)である必要があり、`Swoole1.x`時代に提供された`MySQL`、`Redis`などのクライアントは非同期IOですが、コルーチン方式ではないため、`Swoole4`時代にこれらのクライアントが削除されました。

これらのクライアントのコルーチンサポートの問題を解決するため、Swoole開発チームは多くの作業を行いました：

- まず初めに、各種類のクライアントに対してコルーチンクライアントを作成しましたが、これには次の3つの問題があります：

  * 実装が複雑であり、各クライアントの細かいプロトコルを完璧にサポートするのは非常に大きな作業量がかかります。
  * ユーザーは変更する必要があるコードが多いため、たとえば以前は`PDO`を使用して`MySQL`をクエリしていた場合、現在は[Swoole\Coroutine\MySQL](/coroutine_client/mysql)を使用する必要があります。
  * すべての操作をカバーすることが非常に難しいです。たとえば、`proc_open()`、`sleep()`関数などがブロックされてプログラムが同期的にブロックされる可能性があります。

- 上記の問題に対処するため、Swoole開発チームは実装アプローチを変更し、ネイティブのPHP関数をフックしてコルーチンクライアントを実装することにしました。これにより、一行のコードで以前の同期IOコードを[コルーチンスケジューリング](/coroutine?id=コルーチンスケジューリング)可能な[非同期IO](/learn?id=同期io非同期io)に変換することができ、つまり`ワンクリックコルーチン化`が実現されます。

!> この機能は`v4.3`以降、安定して利用可能になり、コルーチン化可能な関数も増えています。そのため、以前に作成したコルーチンクライアントの一部はすでに推奨されなくなっています。詳細は[コルーチンクライアント](/coroutine_client/init)を参照してください。例：`v4.3+`ではファイル操作(`file_get_contents`、`fread`など)の`コルーチン化`がサポートされており、`v4.3+`を使用している場合はSwooleが提供する[コルーチンファイル操作](/coroutine/system)を使用せずに`コルーチン化`を直接利用できます。
## 関数のプロトタイプ

`flags`を使用して`コルーチン化`したい関数の範囲を設定します。

```php
Co::set(['hook_flags'=> SWOOLE_HOOK_ALL]); // バージョン4.4以上はこの方法を使用します。
// または
Swoole\Runtime::enableCoroutine($flags = SWOOLE_HOOK_ALL);
```

複数の`flags`を同時に有効にするには、`|`演算子を使用する必要があります。

```php
Co::set(['hook_flags'=> SWOOLE_HOOK_TCP | SWOOLE_HOOK_SLEEP]);
```

!> `Hook`された関数は[コルーチンスケジューラ](/coroutine/scheduler)内で使用する必要があります。
#### よくある質問 :id=runtime-qa

!> **`Swoole\Runtime::enableCoroutine()` と `Co::set(['hook_flags'])`、どちらを使うべきか**
  
* `Swoole\Runtime::enableCoroutine()` は、サービス起動後(ランタイム時)に動的にフラグを設定することができます。このメソッドを呼び出した後、現在のプロセス全体で有効になり、プロジェクト全体の最初に配置する必要があります。これにより、100%の適用が得られます。
* `Co::set()` はPHPの `ini_set()` と考えられ、[Server->start()](/server/methods?id=start)や[Co\run()](/coroutine/scheduler)より前に呼び出す必要があります。そうでないと、設定された `hook_flags` は有効になりません。`v4.4+` バージョンではこの方法で `flags` を設定する必要があります。
* `Co::set(['hook_flags'])` または `Swoole\Runtime::enableCoroutine()` 、どちらも1回だけ呼び出すべきで、繰り返し呼び出すと上書きされます。
```python
{
    "option1": "选项1",
    "option2": "选项2",
    "option3": "选项3"
}
```  
### SWOOLE_HOOK_ALL

以下のすべての種類のフラグを有効にします（CURLは含まれません）

!> `SWOOLE_HOOK_ALL` は、v4.5.4以降、`SWOOLE_HOOK_CURL` を含むことに注意してください

```php
Co::set(['hook_flags' => SWOOLE_HOOK_ALL]); //CURLを含まず
Co::set(['hook_flags' => SWOOLE_HOOK_ALL | SWOOLE_HOOK_CURL]); //CURLを含む、すべての種類が本当にコルーチン化されます
```
```php
Co::set(['hook_flags' => SWOOLE_HOOK_TCP]);

Co\run(function() {
    for ($c = 100; $c--;) {
        go(function () {//100 coroutines are created
            $redis = new Redis();
            $redis->connect('127.0.0.1', 6379);//Coroutine scheduling occurs here, CPU switches to the next coroutine, will not block the process
            $redis->get('key');//Coroutine scheduling occurs here, CPU switches to the next coroutine, will not block the process
        });
    }
});
```

上述コードは、元の`Redis`クラスを使用していますが、実際には非同期IOになっています。`Co\run()`はコルーチンコンテナを作成し、`go()`はコルーチンを作成します。これらの操作は、`Swoole`が提供する[Swoole\Serverクラスファミリー](/server/init)では自動的に行われます。手動で行う必要はありません。[enable_coroutine](/server/setting?id=enable_coroutine)を参照してください。

つまり、従来の`PHP`プログラマーは最も馴染み深いロジックコードで高並行性かつ高性能なプログラムを作成できます。以下のように：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_TCP]);

$http = new Swoole\Http\Server("0.0.0.0", 9501);
$http->set(['enable_coroutine' => true]);

$http->on('request', function ($request, $response) {
      $redis = new Redis();
      $redis->connect('127.0.0.1', 6379);//Coroutine scheduling occurs here, CPU switches to the next coroutine (next request), will not block the process
      $redis->get('key');//Coroutine scheduling occurs here, CPU switches to the next coroutine (next request), will not block the process
});

$http->start();
```
### SWOOLE_HOOK_UNIX

`v4.2`からサポートされています。`Unix Stream Socket`タイプのストリームの例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_UNIX]);

Co\run(function () {
    $socket = stream_socket_server(
        'unix://swoole.sock',
        $errno,
        $errstr,
        STREAM_SERVER_BIND | STREAM_SERVER_LISTEN
    );
    if (!$socket) {
        echo "$errstr ($errno)" . PHP_EOL;
        exit(1);
    }
    while (stream_socket_accept($socket)) {
    }
});
```
### SWOOLE_HOOK_UDP

`v4.2`からサポートされています。UDPソケットタイプのstreamの例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_UDP]);

Co\run(function () {
    $socket = stream_socket_server(
        'udp://0.0.0.0:6666',
        $errno,
        $errstr,
        STREAM_SERVER_BIND
    );
    if (!$socket) {
        echo "$errstr ($errno)" . PHP_EOL;
        exit(1);
    }
    while (stream_socket_recvfrom($socket, 1, 0)) {
    }
});
```
### SWOOLE_HOOK_UDG

`v4.2`からサポートされています。Unix Dgram Socketタイプのstreamの例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_UDG]);

Co\run(function () {
    $socket = stream_socket_server(
        'udg://swoole.sock',
        $errno,
        $errstr,
        STREAM_SERVER_BIND
    );
    if (!$socket) {
        echo "$errstr ($errno)" . PHP_EOL;
        exit(1);
    }
    while (stream_socket_recvfrom($socket, 1, 0)) {
    }
});
```
`v4.2`からサポートされています。SSLソケット型のストリームの場合、以下の例をご覧ください：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_SSL]);

Co\run(function () {
    $host = 'host.domain.tld';
    $port = 1234;
    $timeout = 10;
    $cert = '/path/to/your/certchain/certchain.pem';
    $context = stream_context_create(
        array(
            'ssl' => array(
                'local_cert' => $cert,
            )
        )
    );
    if ($fp = stream_socket_client(
        'ssl://' . $host . ':' . $port,
        $errno,
        $errstr,
        30,
        STREAM_CLIENT_CONNECT,
        $context
    )) {
        echo "connected\n";
    } else {
        echo "ERROR: $errno - $errstr \n";
    }
});
```
### SWOOLE_HOOK_TLS

`v4.2`からサポートされています。TLSソケットタイプのストリーム、[参考](https://www.php.net/manual/en/context.ssl.php)。

例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_TLS]);
```
### SWOOLE_HOOK_SLEEP

`v4.2`からサポートされています。`sleep`関数の`Hook`で、`sleep`、`usleep`、`time_nanosleep`、`time_sleep_until`が含まれます。基礎のタイマーの最小単位が`1ms`なので、`usleep`などの高精度のスリープ関数を使用すると、`1ms`未満に設定されている場合は、直接`sleep`システムコールが使用されます。非常に短いスリープブロックが発生する可能性があります。例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_SLEEP]);

Co\run(function () {
    go(function () {
        sleep(1);
        echo '1' . PHP_EOL;
    });
    go(function () {
        echo '2' . PHP_EOL;
    });
});
// 出力
2
1
```
### SWOOLE_HOOK_FILE

`v4.3` でサポートされています。

* **ファイル操作の `コルーチン化処理`、サポートされている関数は次のとおりです：**

    * `fopen`
    * `fread`/`fgets`
    * `fwrite`/`fputs`
    * `file_get_contents`、`file_put_contents`
    * `unlink`
    * `mkdir`
    * `rmdir`

例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_FILE]);

Co\run(function () {
    $fp = fopen("test.log", "a+");
    fwrite($fp, str_repeat('A', 2048));
    fwrite($fp, str_repeat('B', 2048));
});
```
### SWOOLE_HOOK_STREAM_FUNCTION

`v4.4`からサポートされています。`stream_select()`の`Hook`の例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_STREAM_FUNCTION]);

Co\run(function () {
    $fp1 = stream_socket_client("tcp://www.baidu.com:80", $errno, $errstr, 30);
    $fp2 = stream_socket_client("tcp://www.qq.com:80", $errno, $errstr, 30);
    if (!$fp1) {
        echo "$errstr ($errno) \n";
    } else {
        fwrite($fp1, "GET / HTTP/1.0\r\nHost: www.baidu.com\r\nUser-Agent: curl/7.58.0\r\nAccept: */*\r\n\r\n");
        $r_array = [$fp1, $fp2];
        $w_array = $e_array = null;
        $n = stream_select($r_array, $w_array, $e_array, 10);
        $html = '';
        while (!feof($fp1)) {
            $html .= fgets($fp1, 1024);
        }
        fclose($fp1);
    }
});
```
### SWOOLE_HOOK_BLOCKING_FUNCTION

`v4.4` でサポートされました。ここでいう「blocking function」には、`gethostbyname`、`exec`、`shell_exec` などが含まれます。例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_BLOCKING_FUNCTION]);

Co\run(function () {
    echo shell_exec('ls');
});
```
### SWOOLE_HOOK_PROC

`v4.4` で導入されました。`proc*` 関数をコルーチン化しました。これには、`proc_open`、`proc_close`、`proc_get_status`、`proc_terminate` が含まれます。

例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_PROC]);

Co\run(function () {
    $descriptorspec = array(
        0 => array("pipe", "r"),  // stdin, child process read from it
        1 => array("pipe", "w"),  // stdout, child process write to it
    );
    $process = proc_open('php', $descriptorspec, $pipes);
    if (is_resource($process)) {
        fwrite($pipes[0], '<?php echo "I am process\n" ?>');
        fclose($pipes[0]);

        while (true) {
            echo fread($pipes[1], 1024);
        }

        fclose($pipes[1]);
        $return_value = proc_close($process);
        echo "command returned $return_value" . PHP_EOL;
    }
});
```
### SWOOLE_HOOK_CURL

[v4.4LTS](https://github.com/swoole/swoole-src/tree/v4.4.x)後、または`v4.5`以降で公式に対応が開始されました。

- **CURLのHOOKでサポートされている関数：**

     - curl_init
     - curl_setopt
     - curl_exec
     - curl_multi_getcontent
     - curl_setopt_array
     - curl_error
     - curl_getinfo
     - curl_errno
     - curl_close
     - curl_reset

例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_CURL]);

Co\run(function () {
    $ch = curl_init();  
    curl_setopt($ch, CURLOPT_URL, "http://www.xinhuanet.com/");  
    curl_setopt($ch, CURLOPT_HEADER, false);  
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
    $result = curl_exec($ch);  
    curl_close($ch);
    var_dump($result);
});
```
### SWOOLE_HOOK_NATIVE_CURL

原生CURLの`コルーチン処理` に対応。

!> Swooleのバージョン >= `v4.6.0` が必要です。

!> 使用する前に、ビルド時に[--enable-swoole-curl](/environment?id=通用参数)オプションを有効にする必要があります。  
このオプションを有効にすると、`SWOOLE_HOOK_NATIVE_CURL` が自動的に設定され、[SWOOLE_HOOK_CURL](/runtime?id=swoole_hook_all) が無効になります。  
同時に`SWOOLE_HOOK_ALL` に`SWOOLE_HOOK_NATIVE_CURL` が含まれます。

```php
Co::set(['hook_flags' => SWOOLE_HOOK_NATIVE_CURL]);

Co::set(['hook_flags' => SWOOLE_HOOK_ALL | SWOOLE_HOOK_NATIVE_CURL]);
```

例：

```php
Co::set(['hook_flags' => SWOOLE_HOOK_ALL]);

Co\run(function () {
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, "http://httpbin.org/get");
    curl_setopt($ch, CURLOPT_HEADER, false);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
    $result = curl_exec($ch);
    curl_close($ch);
    var_dump($result);
});
```
### SWOOLE_HOOK_SOCKETS

`coroutine processing` of the sockets extension.

!> Available in Swoole version >= `v4.6.0`

```php
Co::set(['hook_flags' => SWOOLE_HOOK_SOCKETS]);
```
### SWOOLE_HOOK_STDIO

STDIOの`コルーチン化処理`。

!> Swooleバージョン >= `v4.6.2` で利用可能

```php
Co::set(['hook_flags' => SWOOLE_HOOK_STDIO]);
```

例：

```php
use Swoole\Process;
Co::set(['socket_read_timeout' => -1, 'hook_flags' => SWOOLE_HOOK_STDIO]);
$proc = new Process(function ($p) {
    Co\run(function () use($p) {
        $p->write('start'.PHP_EOL);
        go(function() {
            co::sleep(0.05);
            echo "sleep\n";
        });
        echo fread(STDIN, 1024);
    });
}, true, SOCK_STREAM);
$proc->start();
echo $proc->read();
usleep(100000);
$proc->write('hello world'.PHP_EOL);
echo $proc->read();
echo $proc->read();
Process::wait();
```
### SWOOLE_HOOK_PDO_PGSQL

`pdo_pgsql` の`コルーチン化`。

!> Swooleバージョン >= `v5.1.0` で利用可能

```php
Co::set(['hook_flags' => SWOOLE_HOOK_PDO_PGSQL]);
```

例：
```php
<?php
function test()
{
    $dbname   = "test";
    $username = "test";
    $password = "test";
    try {
        $dbh = new PDO("pgsql:dbname=$dbname;host=127.0.0.1:5432", $username, $password);
        $dbh->exec('create table test (id int)');
        $dbh->exec('insert into test values(1)');
        $dbh->exec('insert into test values(2)');
        $res = $dbh->query("select * from test");
        var_dump($res->fetchAll());
        $dbh = null;
    } catch (PDOException $exception) {
        echo $exception->getMessage();
        exit;
    }
}

Co::set(['trace_flags' => SWOOLE_HOOK_PDO_PGSQL]);

Co\run(function () {
    test();
});
```
### SWOOLE_HOOK_PDO_ODBC

`pdo_odbc`の`コルーチン化処理`。

!> Swooleバージョン >= `v5.1.0` で使用可能です

```php
Co::set(['hook_flags' => SWOOLE_HOOK_PDO_ODBC]);
```

例：
```php
<?php
function test()
{
    $username = "test";
    $password = "test";
    try {
        $dbh = new PDO("odbc:mysql-test");
        $res = $dbh->query("select sleep(1) s");
        var_dump($res->fetchAll());
        $dbh = null;
    } catch (PDOException $exception) {
        echo $exception->getMessage();
        exit;
    }
}

Co::set(['trace_flags' => SWOOLE_TRACE_CO_ODBC, 'log_level' => SWOOLE_LOG_DEBUG]);

Co\run(function () {
    test();
});
```
### SWOOLE_HOOK_PDO_ORACLE

`pdo_oci`の`コルーチン化処理`。

!> Swooleバージョン >= `v5.1.0` で使用可能

```php
Co::set(['hook_flags' => SWOOLE_HOOK_PDO_ORACLE]);
```

例：
```php
<?php
function test()
{
	$tsn = 'oci:dbname=127.0.0.1:1521/xe;charset=AL32UTF8';
	$username = "test";
	$password = "test";
    try {
        $dbh = new PDO($tsn, $username, $password);
        $dbh->exec('create table test (id int)');
        $dbh->exec('insert into test values(1)');
        $dbh->exec('insert into test values(2)');
        $res = $dbh->query("select * from test");
        var_dump($res->fetchAll());
        $dbh = null;
    } catch (PDOException $exception) {
        echo $exception->getMessage();
        exit;
    }
}

Co::set(['hook_flags' => SWOOLE_HOOK_PDO_ORACLE]);
Co\run(function () {
    test();
});
```
### SWOOLE_HOOK_PDO_SQLITE
`pdo_sqlite`の`コルーチン対応`。

!> Swooleのバージョン >= `v5.1.0` が必要

```php
Co::set(['hook_flags' => SWOOLE_HOOK_PDO_SQLITE]);
```

* **注意**

!> `Swoole` はコルーチン化された `sqlite` データベースで `シリアライズモード` を採用して[スレッドセーフ](https://www.sqlite.org/threadsafe.html) を保証します。
`sqlite` データベースがコンパイル時に指定されたスレッドモードがシングルスレッドモードである場合、`Swoole` は `sqlite` をコルーチン化できず、警告を発しても使用に影響を与えませんが、単にコルーチンの切り替えは発生しません。この場合、`sqlite` を再コンパイルしてスレッドモードを `シリアライズ化` または `マルチスレッド` に指定する必要があります。[理由](https://www.sqlite.org/compile.html#threadsafe)。
コルーチン環境で作成される `sqlite` 接続はすべて`シリアライズ化`されており、非コルーチン環境で作成される `sqlite` 接続はデフォルトで `sqlite` のスレッドモードに準拠しています。
`sqlite` のスレッドモードが `マルチスレッド` である場合、非コルーチン環境で作成された接続は複数のコルーチンで共有できません。なぜならこの時点でデータベース接続は `マルチスレッドモード` であるため、コルーチン環境で使用されても `シリアライズ化` にはアップグレードされません。
`sqlite` のデフォルトスレッドモードは `シリアライズ化` です。[トラウゼアスキャンの説明](https://www.sqlite.org/c3ref/c_config_covering_index_scan.html#sqliteconfigserialized)、[デフォルトスレッドモード](https://www.sqlite.org/compile.html#threadsafe)。

例：
```php
<?php
use function Swoole\Coroutine\run;
use function Swoole\Coroutine\go;

Co::set(['hook_flags'=> SWOOLE_HOOK_PDO_SQLITE]);

run(function() {
    for($i = 0; $i <= 5; $i++) {
        go(function() use ($i) {
            $db = new PDO('sqlite::memory:');
            $db->query('select randomblob(99999999)');
            var_dump($i);
        });
    }
});
```
## 方法

このセクションでは、さまざまな方法について説明します。
### setHookFlags()

`flags`を使用して`Hook`する関数の範囲を設定します。

!> Swooleバージョン>=`v4.5.0`で使用可能

```php
Swoole\Runtime::setHookFlags(int $flags): bool
```
### getHookFlags()

現在の`Hook`コンテンツの`flags`を取得します。`Hook`が成功しない場合、指定した`flags`とは異なる可能性があることに注意してください（そのため、`Hook`が成功しない`flags`はクリアされます）

!> Swooleバージョン >= `v4.4.12` で利用可能

```php
Swoole\Runtime::getHookFlags(): int
```
```plaintext
UseEffect
UseState
UseContext
UseReducer
```  
### 利用可能なリスト

  * `redis`拡張機能
  * `pdo_mysql`、`mysqli`拡張機能（`mysqlnd`モードを使用）は、`mysqlnd`が無効になっている場合はコルーチンがサポートされません
  * `soap`拡張機能
  * `file_get_contents`、`fopen`
  * `stream_socket_client` (`predis`、`php-amqplib`)
  * `stream_socket_server`
  * `stream_select`（バージョン`4.3.2`以上が必要）
  * `fsockopen`
  * `proc_open`（バージョン`4.4.0`以上が必要）
  * `curl`
### 利用できないリスト

!> **非対応のコルーチン化** は、コルーチンをブロッキングモードにダウングレードします。この場合、コルーチンを使用する意味がありません

  * `mysql`：`libmysqlclient`をベースにしています
  * `mongo`：`mongo-c-client`をベースにしています
  * `pdo_pgsql`、Swooleバージョン >= `v5.1.0`の場合、`pdo_pgsql`はコルーチン処理が可能です
  * `pdo_oci`、Swooleバージョン >= `v5.1.0`の場合、`pdo_oci`はコルーチン処理が可能です
  * `pdo_odbc`、Swooleバージョン >= `v5.1.0`の場合、`pdo_odbc`はコルーチン処理が可能です
  * `pdo_firebird`
  * `php-amqp`
## API 変更

`v4.3` およびそれ以前のバージョンでは、`enableCoroutine` メソッドの API は2つのパラメータが必要です。

```php
Swoole\Runtime::enableCoroutine(bool $enable = true, int $flags = SWOOLE_HOOK_ALL);
```

- `$enable`：コルーチンを有効または無効にします。
- `$flags`：コルーチン化するタイプを選択し、複数選択することができます。デフォルトは全選択です。`$enable = true` の場合にのみ有効です。

!> `Runtime::enableCoroutine(false)` は、前回設定されたすべてのオプションのコルーチン `Hook` 設定を無効にします。

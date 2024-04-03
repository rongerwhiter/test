# TCPサーバー

`Swoole\Coroutine\Server` は、完全に[コルーチン](/coroutine)化されたクラスで、コルーチンTCPサーバーを作成するために使用されます。TCPおよび[unixSocket](/learn?id=什么是IPC)タイプをサポートしています。

`Server`モジュールとの違い：

- ダイナミックな作成と破棄が可能であり、実行時にポートを動的にリッスンしたり、サーバーを動的に閉じたりできます。
- 接続の処理は完全に同期的であり、プログラムは`Connect`、`Receive`、`Close`イベントを順番に処理することができます。

!> 4.4以上のバージョンで使用可能です。
## Short Alias

`Co\Server`の短い名前として`Co\Server`を使用できます。
This is a code block and should not be translated.

```
def greet():
    print("Hello, world!")
```

This is another code block and should not be translated.
### __construct()

```php
Swoole\Coroutine\Server::__construct(string $host, int $port = 0, bool $ssl = false, bool $reuse_port = false);
```

  * **Parameters**

    * **`string $host`**
      * **Description**: Address to listen on
      * **Default**: None
      * **Other values**: None

    * **`int $port`**
      * **Description**: Port to listen on【if set to 0, the system will assign a random port】
      * **Default**: None
      * **Other values**: None

    * **`bool $ssl`**
      * **Description**: Enable SSL encryption
      * **Default**: `false`
      * **Other values**: `true`

    * **`bool $reuse_port`**
      * **Description**: Enable port reuse, has the same effect as the configuration in [this section](/server/setting?id=enable_reuse_port)
      * **Default**: `false`
      * **Other values**: `true`
      * **Version Impact**: Swoole version >= v4.4.4

  * **Note**

    * **$host parameter supports 3 formats**

      * `0.0.0.0/127.0.0.1`: IPv4 address
      * `::/::1`: IPv6 address
      * `unix:/tmp/test.sock`: [UnixSocket](/learn?id=什么是IPC) address

    * **Exceptions**

      * `Swoole\Exception` exception will be thrown for parameter errors, failed to bind the address and port, and failed to `listen`.
### set()

?> **設定プロトコル処理のパラメータ。**

```php
Swoole\Coroutine\Server->set(array $options);
```

  * **構成パラメータ**

    * パラメータ`$options` は、 [setprotocol](/coroutine_client/socket?id=setprotocol) メソッドで受け入れられる構成項目と完全に一致する必要があります。

    !> パラメータを設定するには、[start()](/coroutine/server?id=start) メソッドの前に設定する必要があります。

    * **長さプロトコル**

    ```php
    $server = new Swoole\Coroutine\Server('127.0.0.1', $port, $ssl);
    $server->set([
      'open_length_check' => true,
      'package_max_length' => 1024 * 1024,
      'package_length_type' => 'N',
      'package_length_offset' => 0,
      'package_body_offset' => 4,
    ]);
    ```

    * **SSL証明書の設定**

    ```php
    $server->set([
      'ssl_cert_file' => dirname(__DIR__) . '/ssl/server.crt',
      'ssl_key_file' => dirname(__DIR__) . '/ssl/server.key',
    ]);
    ```
### handle()

?> **設定連線處理函式。**

!> 必須在 [start()](/coroutine/server?id=start) 之前設定處理函式

```php
Swoole\Coroutine\Server->handle(callable $fn);
```

* **參數**

  * **`callable $fn`**
    * **功能**：設定連線處理函式
    * **默認值**：無
    * **其他值**：無

* **範例**

  ```php
  $server->handle(function (Swoole\Coroutine\Server\Connection $conn) {
      while (true) {
          $data = $conn->recv();
      }
  });
  ```

  !> - 服務器在 `Accept`(建立連線) 成功後，會自動創建[協程](/coroutine?id=協程調度)並執行 `$fn`；  
  - `$fn` 是在新的子協程空間內執行，因此在函式內無需再次創建協程；  
  - `$fn` 接受一個參數，類型為 [Swoole\Coroutine\Server\Connection](/coroutine/server?id=coroutineserverconnection) 物件；  
  - 可以使用 [exportSocket()](/coroutine/server?id=exportsocket) 得到當前連線的 Socket 物件  
### shutdown()

?> **シャットダウンします。**

?> ベースレイヤーは`start`および`shutdown`を複数回呼び出すことをサポートしています。

```php
Swoole\Coroutine\Server->shutdown(): bool
```
```php
Swoole\Coroutine\Server->start(): bool
```

  * **Return Value**

    * Returns `false` if failed to start and sets the `errCode` property
    * Starts a loop and `Accept` connections if started successfully
    * After `Accept` (establishing connection), a new coroutine will be created and the function specified in the handle method will be called in the coroutine

  * **Error Handling**

    * When a `Too many open file` error occurs during `Accept` or if a child coroutine cannot be created, it will pause for `1` second before continuing to `Accept`
    * When an error occurs, the `start()` method will return, and the error message will be displayed as a `Warning`.  
## オブジェクト
### Coroutine\Server\Connection

`Swoole\Coroutine\Server\Connection`オブジェクトには、次の4つのメソッドが用意されています：
#### recv()

データを受信します。プロトコル処理が設定されている場合、完全なパッケージが返されます。

```php
function recv(float $timeout = 0)
```
```php
function send(string $data)
```
#### close()

接続を閉じます

```php
function close(): bool
```
#### exportSocket()

得到当前连接的Socket对象。可调用更多底层的方法，请参考 [Swoole\Coroutine\Socket](/coroutine_client/socket)

```php
function exportSocket(): Swoole\Coroutine\Socket
```
```php
use Swoole\Process;
use Swoole\Coroutine;
use Swoole\Coroutine\Server\Connection;

//多进程管理モジュール
$pool = new Process\Pool(2);
//OnWorkerStartコールバックごとにコルーチンを自動生成する
$pool->set(['enable_coroutine' => true]);
$pool->on('workerStart', function ($pool, $id) {
    //各プロセスはポート9501をリッスンする
    $server = new Swoole\Coroutine\Server('127.0.0.1', 9501, false, true);

    //シグナル15を受信したらサービスをシャットダウンする
    Process::signal(SIGTERM, function () use ($server) {
        $server->shutdown();
    });

    //新しい接続要求を受信し、自動的にコルーチンを生成する
    $server->handle(function (Connection $conn) {
        while (true) {
            //データを受信
            $data = $conn->recv(1);

            if ($data === '' || $data === false) {
                $errCode = swoole_last_error();
                $errMsg = socket_strerror($errCode);
                echo "errCode: {$errCode}, errMsg: {$errMsg}\n";
                $conn->close();
                break;
            }

            //データを送信
            $conn->send('hello');

            Coroutine::sleep(1);
        }
    });

    //ポートをリッスンする
    $server->start();
});
$pool->start();
```

!> Cygwin環境で実行する場合は、単一プロセスに変更してください。`$pool = new Swoole\Process\Pool(1);`

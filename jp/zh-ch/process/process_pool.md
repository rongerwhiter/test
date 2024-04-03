# Swoole\Process\Pool

プロセスプールは、[Swoole\Server](/server/init)のManager管理プロセスモジュールに基づいて実装されています。複数のワーカープロセスを管理できます。このモジュールの中心機能はプロセス管理であり、`Process`で複数プロセスを実行する代わりに、`Process\Pool`はよりシンプルで、より高いレベルのカプセル化がされており、開発者は多くのコードを記述する必要がなく、プロセス管理機能を簡単に実現できます。[Co\Server](/coroutine/server?id=完全な例)と組み合わせて、マルチコアCPUを活用した、純粋なコルーチンスタイルでサーバープログラムを作成できます。
## プロセス間通信

`Swoole\Process\Pool` では、次の3つのプロセス間通信方法が提供されています：
### メッセージキュー
`Swoole\Process\Pool->__construct`の2番目の引数を`SWOOLE_IPC_MSGQUEUE`に設定すると、プロセス間通信にメッセージキューが使用されます。`php sysvmsg`拡張機能を使用して情報を送信でき、メッセージの最大サイズは`65536`バイト以下である必要があります。

* **注意**

  * `sysvmsg`拡張機能を使用して情報を送信する場合、コンストラクタに`msgqueue_key`を渡す必要があります
  * `Swoole`の内部では`sysvmsg`拡張機能の`msg_send`関数の第2引数`mtype`をサポートしていません。したがって、任意の`0`以外の値を渡してください。
### ソケット通信
`Swoole\Process\Pool->__construct`の第二引数を`SWOOLE_IPC_SOCKET`に設定すると、`ソケット通信`が使用され、クライアントとサーバーが同じマシンでない場合でも通信できます。

[Swoole\Process\Pool->listen()](/process/process_pool?id=listen)メソッドを使用してポートをリッスンし、[Message イベント](/process/process_pool?id=on)を使用してクライアントから送信されたデータを受け取り、[Swoole\Process\Pool->write()](/process/process_pool?id=write)メソッドを使用してクライアントに応答を返します。

`Swoole`は、この方法でデータを送信するようクライアントに要求する場合、実際のデータの前に 4 バイトのネットワークバイト順の長さ値を追加する必要があります。
```php
$msg = 'Hello Swoole';
$packet = pack('N', strlen($msg)) . $msg;
```
### UnixSocket
`Swoole\Process\Pool->__construct`の2番目のパラメータを`SWOOLE_IPC_UNIXSOCK`に設定すると、[UnixSocket](/learn?id=什么是IPC)を使用することを示します。**この方法はプロセス間通信に強くお勧めします**。

この方法は非常にシンプルで、[Swoole\Process\Pool->sendMessage()](/process/process_pool?id=sendMessage)メソッドと[Message event](/process/process_pool?id=on)を使用してプロセス間通信を行うことができます。

また、`Coroutine mode`を有効にした場合、[Swoole\Process\Pool->getProcess()](/process/process_pool?id=getProcess)を使用して`Swoole\Process`オブジェクトを取得し、[Swoole\Process->exportsocket()](/process/process?id=exportsocket)を使用して`Swoole\Coroutine\Socket`オブジェクトを取得することでプロセス間通信を実現することも可能です。ただし、この場合は[Message event](/process/process_pool?id=on)を設定することはできません。

!> パラメータや環境設定については[constructor](/process/process_pool?id=__construct)と[configuration parameters](/process/process_pool?id=set)を参照してください。
## 常量

常量 | 说明
---|---
SWOOLE_IPC_MSGQUEUE | 系统[消息队列](/learn?id=什么是IPC)通信
SWOOLE_IPC_SOCKET | SOCKET通信
SWOOLE_IPC_UNIXSOCK | [UnixSocket](/learn?id=什么是IPC)通信(v4.4+)
## 协程支持

`v4.4.0`バージョンでは、コルーチンをサポートするようになりました。[Swoole\Process\Pool::__construct](/process/process_pool?id=__construct)を参照してください。
```php
use Swoole\Process;
use Swoole\Coroutine;

$pool = new Process\Pool(5);
$pool->set(['enable_coroutine' => true]);
$pool->on('WorkerStart', function (Process\Pool $pool, $workerId) {
    /** 現在のプロセスはWorkerです */
    static $running = true;
    Process::signal(SIGTERM, function () use (&$running) {
        $running = false;
        echo "TERM\n";
    });
    echo("[Worker #{$workerId}] WorkerStart, pid: " . posix_getpid() . "\n");
    while ($running) {
        Coroutine::sleep(1);
        echo "sleep 1\n";
    }
});
$pool->on('WorkerStop', function (\Swoole\Process\Pool $pool, $workerId) {
    echo("[Worker #{$workerId}] WorkerStop\n");
});
$pool->start();
```
## Methods
### __construct()

コンストラクタ。

```php
Swoole\Process\Pool::__construct(int $worker_num, int $ipc_type = SWOOLE_IPC_NONE, int $msgqueue_key = 0, bool $enable_coroutine = false);
```

* **パラメーター**

  * **`int $worker_num`**
    * **機能**：ワーカープロセスの数を指定します
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $ipc_type`**
    * **機能**：プロセス間通信のモード【デフォルトは`SWOOLE_IPC_NONE`で、プロセス間通信機能は使用されません】
    * **デフォルト値**：`SWOOLE_IPC_NONE`
    * **その他の値**：`SWOOLE_IPC_MSGQUEUE`、`SWOOLE_IPC_SOCKET`、`SWOOLE_IPC_UNIXSOCK`

    !> - `SWOOLE_IPC_NONE`に設定されている場合は、`onWorkerStart`コールバックを設定する必要があり、`onWorkerStart`内でループロジックを実装する必要があります。`onWorkerStart`関数が終了するとワーカープロセスは即座に終了し、その後`Manager`プロセスがプロセスを再起動します。  
    - `SWOOLE_IPC_MSGQUEUE`に設定されている場合は、システムのメッセージキュー通信を使用します。`$msgqueue_key`を指定してメッセージキューの`KEY`を設定できます。メッセージキューの`KEY`が特定されていない場合は、プライベートキューが確保されます。  
    - `SWOOLE_IPC_SOCKET`に設定されている場合は、通信に`Socket`を使用します。リスニングアドレスとポートを指定するには、[listen](/process/process_pool?id=listen)メソッドを使用する必要があります。  
    - `SWOOLE_IPC_UNIXSOCK`に設定されている場合は、通信に[unixSocket](/learn?id=什么是IPC)を使用します。コルーチンモードで使用されますが、**プロセス間通信にはこの方法が強く推奨されます**。具体的な使用法については以下を参照してください；
- `onMessage` callback must be set when using non-`SWOOLE_IPC_NONE` setting, and `onWorkerStart` becomes optional.

  * **`int $msgqueue_key`**
    * **Description**: The key for the message queue.
    * **Default**: `0`
    * **Other values**: None

  * **`bool $enable_coroutine`**
    * **Description**: Whether to enable coroutine support [if coroutine is enabled, the `onMessage` callback cannot be set]
    * **Default**: `false`
    * **Other values**: `true`

* **Coroutine Mode**

In version `v4.4.0`, the `Process\Pool` module added support for coroutines. You can enable it by setting the fourth parameter to `true`. When coroutines are enabled, the underlying system automatically creates a coroutine and a [coroutine container](/coroutine/scheduler) at `onWorkerStart`. You can directly use coroutine-related APIs in the callback function, for example:

```php
$pool = new Swoole\Process\Pool(1, SWOOLE_IPC_NONE, 0, true);

$pool->on('workerStart', function (Swoole\Process\Pool $pool, int $workerId) {
    while (true) {
        Co::sleep(0.5);
        echo "hello world\n";
    }
});

$pool->start();
```

When coroutines are enabled, Swoole prohibits setting the `onMessage` event callback. If inter-process communication is needed, set the second parameter as `SWOOLE_IPC_UNIXSOCK` to use [unixSocket](/learn?id=what-is-ipc) for communication. Then, export the [Swoole\Coroutine\Socket](/coroutine_client/socket) object using `$pool->getProcess()->exportSocket()` to achieve communication between `Worker` processes. For example:

```php
$pool = new Swoole\Process\Pool(2, SWOOLE_IPC_UNIXSOCK, 0, true);```
```php
$pool->on('workerStart', function (Swoole\Process\Pool $pool, int $workerId) {
    $process = $pool->getProcess(0);
    $socket = $process->exportSocket();
    if ($workerId == 0) {
        echo $socket->recv();
        $socket->send("hello proc1\n");
        echo "proc0 stop\n";
    } else {
        $socket->send("hello proc0\n");
        echo $socket->recv();
        echo "proc1 stop\n";
        $pool->shutdown();
    }
});

$pool->start();
 ```

!> 具体用法可以参考[Swoole\Coroutine\Socket](/coroutine_client/socket)和[Swoole\Process](/process/process?id=exportsocket)相关章节。

```php
$q = msg_get_queue($key);
foreach (range(1, 100) as $i) {
    $data = json_encode(['data' => base64_encode(random_bytes(1024)), 'id' => uniqid(), 'index' => $i,]);
    msg_send($q, $i, $data, false);
}
```
```php
Swoole\Process\Pool->set(array $settings): void
```

Optional Parameters | Type | Description | Default Value
---|---|----|----
enable_coroutine|bool|Controls whether to enable coroutine|false
enable_message_bus|bool|Enable message bus. When set to `true`, if sending large data, the underlying layer will split the data into small chunks before sending to the other end|false
max_package_size|int|Limits the maximum amount of data the process can receive|2 * 1024 * 1024

* **Note**

  * When `enable_message_bus` is set to `true`, `max_package_size` has no effect because the underlying layer splits the data into small chunks before sending, and receiving data works the same way.
  * In `SWOOLE_IPC_MSGQUEUE` mode, `max_package_size` also has no effect because at most `65536` data can be received at once.
  * In `SWOOLE_IPC_SOCKET` mode, when `enable_message_bus` is `false`, if the amount of data received is greater than `max_package_size`, the underlying layer will directly disconnect.
  * In `SWOOLE_IPC_UNIXSOCK` mode, when `enable_message_bus` is `false`, if the data is larger than `max_package_size`, the data exceeding `max_package_size` will be truncated.
  * If coroutine mode is enabled, `max_package_size` also has no effect when `enable_message_bus` is set to `true`. The underlying layer will split (send) and merge (receive) the data accordingly. Otherwise, the amount of data received is limited by `max_package_size`.

!> Available since Swoole version >= v4.4.4
```
### __construct()

Constructor method.

```php
Swoole\Process\Pool::__construct(int $worker_num, int $ipc_type = SWOOLE_IPC_NONE, int $msgqueue_key = 0, bool $enable_coroutine = false);
```

* **Parameters** 

  * **`int $worker_num`**
    * **Description**: Specifies the number of worker processes.
    * **Default**: None
    * **Other values**: None

  * **`int $ipc_type`**
    * **Description**: Mode of inter-process communication (IPC) 【Default is `SWOOLE_IPC_NONE` which means no inter-process communication features are used】
    * **Default**: `SWOOLE_IPC_NONE`
    * **Other values**: `SWOOLE_IPC_MSGQUEUE`, `SWOOLE_IPC_SOCKET`, `SWOOLE_IPC_UNIXSOCK`

    !> - When set to `SWOOLE_IPC_NONE`, you must set the `onWorkerStart` callback and implement a looping logic within it. The worker process will exit immediately when the `onWorkerStart` function exits. After that, the `Manager` process will restart the process.  
    - When set to `SWOOLE_IPC_MSGQUEUE`, system message queue communication is used. You can set `$msgqueue_key` to specify the key for the message queue. If the message queue key is not set, a private queue will be created.  
    - When set to `SWOOLE_IPC_SOCKET`, communication using `Socket` is used. You need to use the [listen](/process/process_pool?id=listen) method to specify the address and port to listen on.  
    - When set to `SWOOLE_IPC_UNIXSOCK`, communication using [Unix Socket](/learn?id=What_is_IPC) is used. This is recommended in coroutine mode for inter-process communication. Detailed usage is explained in the following section.
- `SWOOLE_IPC_NONE`が設定されていない場合、`onMessage`コールバックを設定する必要があり、`onWorkerStart`はオプションになります。

  * **`int $msgqueue_key`**
    * **機能**: メッセージキューの `key`
    * **デフォルト値**: `0`
    * **その他の値**: なし

  * **`bool $enable_coroutine`**
    * **機能**: コルーチンサポートを有効にするかどうか【コルーチンを使用すると`onMessage`コールバックを設定できなくなります】
    * **デフォルト値**: `false`
    * **その他の値**: `true`

* **Coroutine Mode**
    
`v4.4.0`では`Process\Pool`モジュールにおいてコルーチンのサポートが追加され、第4引数を`true`に設定することで有効にできます。コルーチンを有効にすると、内部で`onWorkerStart`時にコルーチンと[コルーチンスケジューラ](/coroutine/scheduler)が自動的に作成され、コールバック関数内でコルーチン関連の`API`を直接使用できるようになります。例：
    
```php
$pool = new Swoole\Process\Pool(1, SWOOLE_IPC_NONE, 0, true);

$pool->on('workerStart', function (Swoole\Process\Pool $pool, int $workerId) {
    while (true) {
        Co::sleep(0.5);
        echo "hello world\n";
    }
});

$pool->start();
```

コルーチンを有効にすると、Swooleは`onMessage`イベントコールバックの設定を禁止します。プロセス間通信が必要な場合は、第2引数を`SWOOLE_IPC_UNIXSOCK`に設定して[unixSocket](/learn?id=什么是IPC)を使用することを示し、次に`$pool->getProcess()->exportSocket()`を使用して[Swoole\Coroutine\Socket](/coroutine_client/socket)オブジェクトをエクスポートし、`Worker`プロセス間の通信を実現します。例：

```php
$pool = new Swoole\Process\Pool(2, SWOOLE_IPC_UNIXSOCK, 0, true);
```php
$pool->on('workerStart', function (Swoole\Process\Pool $pool, int $workerId) {
    $process = $pool->getProcess(0);
    $socket = $process->exportSocket();
    if ($workerId == 0) {
        echo $socket->recv();
        $socket->send("hello proc1\n");
        echo "proc0 stop\n";
    } else {
        $socket->send("hello proc0\n");
        echo $socket->recv();
        echo "proc1 stop\n";
        $pool->shutdown();
    }
});

$pool->start();
 ```

!> 具体用法可以参考[Swoole\Coroutine\Socket](/coroutine_client/socket)和[Swoole\Process](/process/process?id=exportsocket)相关章节。

```php
$q = msg_get_queue($key);
foreach (range(1, 100) as $i) {
    $data = json_encode(['data' => base64_encode(random_bytes(1024)), 'id' => uniqid(), 'index' => $i,]);
    msg_send($q, $i, $data, false);
}
```
### on()

プロセスプールのコールバック関数を設定します。

```php
Swoole\Process\Pool->on(string $event, callable $function): bool;
```

* **パラメーター**

  * **`string $event`**
    * **説明**：イベントを指定します。
    * **デフォルト値**：なし
    * **その他の値**: なし

  * **`callable $function`**
    * **説明**：コールバック関数
    * **デフォルト値**：なし
    * **その他の値**: なし

* **イベント**

  * **onWorkerStart** ワーカープロセスの開始

  ```php
  /**
  * @param \Swoole\Process\Pool $pool Poolオブジェクト
  * @param int $workerId   WorkerId：現在のワーカープロセスの番号。 ワーカープロセスは内部で番号付けされます。
  */
  $pool = new Swoole\Process\Pool(2);
  $pool->on('WorkerStart', function(Swoole\Process\Pool $pool, int $workerId){
    echo "Worker#{$workerId} is started\n";
  });
  ```

  * **onWorkerStop** ワーカープロセスの終了

  ```php
  /**
  * @param \Swoole\Process\Pool $pool Poolオブジェクト
  * @param int $workerId   WorkerId：現在のワーカープロセスの番号。 ワーカープロセスは内部で番号付けされます。
  */
  $pool = new Swoole\Process\Pool(2);
  $pool->on('WorkerStop', function(Swoole\Process\Pool $pool, int $workerId){
    echo "Worker#{$workerId} stop\n";
  });
  ```

  * **onMessage** メッセージの受信

  !> 外部からのメッセージを受け取ります。 一度の接続につき1回のみメッセージを投稿できます。これは`PHP-FPM`の短い接続メカニズムに類似しています。

  ```php
  /**
    * @param \Swoole\Process\Pool $pool Poolオブジェクト
    * @param string $data メッセージのデータ内容
   */
  $pool = new Swoole\Process\Pool(2);
```php
$pool->on('Message', function(Swoole\Process\Pool $pool, string $data){
    var_dump($data);
  });
  ```

  !> 事件名称は大文字と小文字を区別しません。「WorkerStart」、「workerStart」、または「workerstart」はすべて同じです。
```
### listen()

`SOCKET`を聴く。`$ipc_mode = SWOOLE_IPC_SOCKET`の場合にのみ使用できる。

```php
Swoole\Process\Pool->listen(string $host, int $port = 0, int $backlog = 2048): bool
```

* **パラメータ** 

  * **`string $host`**
    * **機能**：聴いているアドレス【`TCP`および[unixSocket]をサポートしています。`127.0.0.1`は`TCP`アドレスを聴いていることを表し、`$port`を指定する必要がある。`unix:/tmp/php.sock`は[unixSocket]アドレスを聴いていることを表します】
    * **デフォルト値**：無し
    * **その他の値**：無し

  * **`int $port`**
    * **機能**：聴いているポート【`TCP`モードで指定する必要がある】
    * **デフォルト値**：`0`
    * **その他の値**：無し

  * **`int $backlog`**
    * **機能**：聴いているキューの長さ
    * **デフォルト値**：`2048`
    * **その他の値**：無し

* **戻り値**

  * 成功すると`true`を返す
  * 失敗すると`false`を返し、エラーコードを取得するために`swoole_errno`を呼び出すことができる。失敗した場合、`start`を呼び出すとすぐに`false`が返される。

* **通信プロトコル**

    ポートにデータを送信する際、クライアントはリクエストの前に4バイト、ネットワークバイトオーダーでの長さ値を増やす必要がある。プロトコルフォーマットは次のとおりです：

```php
// $msg：送信するデータ
$packet = pack('N', strlen($msg)) . $msg;
```

* **使用例**

```php
$pool->listen('127.0.0.1', 8089);
$pool->listen('unix:/tmp/php.sock');
```
### write()

データを相手に書き込むために使用され、`$ipc_mode`が`SWOOLE_IPC_SOCKET`の場合にのみ使用できます。

```php
Swoole\Process\Pool->write(string $data): bool
```

!> このメソッドはメモリ操作であり、`IO`コストがかからず、データ送信操作は同期ブロック`IO`です。

* **パラメーター** 

  * **`string $data`**
    * **機能**：書き込むデータの内容【`write`を複数回呼び出すことが可能で、`onMessage`関数を終了した後にデータをすべて`socket`に書き込み、接続を`close`します】
    * **デフォルト値**：なし
    * **その他の値**：なし

* **使用例**

  * **サーバー**

    ```php
    $pool = new Swoole\Process\Pool(2, SWOOLE_IPC_SOCKET);
    
    $pool->on("Message", function ($pool, $message) {
        echo "Message: {$message}\n";
        $pool->write("hello ");
        $pool->write("world ");
        $pool->write("\n");
    });
    
    $pool->listen('127.0.0.1', 8089);
    $pool->start();
    ```

  * **コール元**

    ```php
    $fp = stream_socket_client("tcp://127.0.0.1:8089", $errno, $errstr) or die("error: $errstr\n");
    $msg = json_encode(['data' => 'hello', 'uid' => 1991]);
    fwrite($fp, pack('N', strlen($msg)) . $msg);
    sleep(1);
    // 「hello world\n」と表示されます
    $data = fread($fp, 8192);
    var_dump(substr($data, 4, unpack('N', substr($data, 0, 4))[1]));
    fclose($fp);
    ```
### __construct()

コンストラクタ。

```php
Swoole\Process\Pool::__construct(int $worker_num, int $ipc_type = SWOOLE_IPC_NONE, int $msgqueue_key = 0, bool $enable_coroutine = false);
```

* **パラメータ** 

  * **`int $worker_num`**
    * **機能**：ワーカープロセスの数を指定します
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $ipc_type`**
    * **機能**：プロセス間通信のモード【デフォルトは`SWOOLE_IPC_NONE`で、任意のプロセス間通信機能を使用しません】
    * **デフォルト値**：`SWOOLE_IPC_NONE`
    * **その他の値**：`SWOOLE_IPC_MSGQUEUE`、`SWOOLE_IPC_SOCKET`、`SWOOLE_IPC_UNIXSOCK`

    !> - `SWOOLE_IPC_NONE`に設定されている場合、`onWorkerStart`コールバックを設定する必要があります。また、`onWorkerStart`内でループロジックを実装する必要があります。`onWorkerStart`関数が終了すると、作業プロセスは直ちに終了し、その後Managerプロセスがプロセスを再起動します；  
    - `SWOOLE_IPC_MSGQUEUE`を設定すると、システムのメッセージキュー通信を使用します。`$msgqueue_key`を指定してメッセージキューの`KEY`を設定できますが、メッセージキュー`KEY`が指定されていない場合は、プライベートキューが取得されます；  
    - `SWOOLE_IPC_SOCKET`を設定すると、`Socket`を使用して通信します。[listen](/process/process_pool?id=listen)メソッドを使用して、アドレスとポートを指定する必要があります；  
    - `SWOOLE_IPC_UNIXSOCK`を設定すると、[unixSocket](/learn?id=什么是IPC)を使用して通信します。コルーチンモードを使用していますが、プロセス間通信にこの方法を強くお勧めします。具体的な使用法については、後述をご覧ください；  
- `SWOOLE_IPC_NONE`以外を使用する場合、`onMessage`コールバックを設定する必要があります。`onWorkerStart`はオプションに変更されました。

  * **`int $msgqueue_key`**
    * **機能**：メッセージキューの `key`
    * **初期値**：`0`
    * **その他の値**：なし

  * **`bool $enable_coroutine`**
    * **機能**：コルーチンサポートを有効にするか【コルーチンを使用すると`onMessage`コールバックを設定できなくなります】
    * **初期値**：`false`
    * **その他の値**：`true`

* **Coroutine Mode**
    
`v4.4.0` で、`Process\Pool`モジュールでコルーチンをサポートし、`true` として4番目のパラメーターを設定して有効にすることができます。コルーチンを有効にすると、バッグラウンドで`onWorkerStart`で自動的にコルーチンと[coroutine container](/coroutine/scheduler)を作成します。コールバック関数内で、コルーチン関連の`API`を直接使用できます。例：

```php
$pool = new Swoole\Process\Pool(1, SWOOLE_IPC_NONE, 0, true);

$pool->on('workerStart', function (Swoole\Process\Pool $pool, int $workerId) {
    while (true) {
        Co::sleep(0.5);
        echo "hello world\n";
    }
});

$pool->start();
```

コルーチンを有効にすると、Swoole は`onMessage`イベントコールバックの設定を無効にします。プロセス間通信が必要な場合は、2番目の引数を`SWOOLE_IPC_UNIXSOCK`に設定して[unixSocket](/learn?id=什么是IPC)を使用し、次に`$pool->getProcess()->exportSocket()`を使用して[Swoole\Coroutine\Socket](/coroutine_client/socket)オブジェクトをエクスポートして、「Worker」プロセス間の通信を実現します。例：

```php
$pool = new Swoole\Process\Pool(2, SWOOLE_IPC_UNIXSOCK, 0, true);  
```php
$pool->on('workerStart', function (Swoole\Process\Pool $pool, int $workerId) {
    $process = $pool->getProcess(0);
    $socket = $process->exportSocket();
    if ($workerId == 0) {
        echo $socket->recv();
        $socket->send("hello proc1\n");
        echo "proc0 stop\n";
    } else {
        $socket->send("hello proc0\n");
        echo $socket->recv();
        echo "proc1 stop\n";
        $pool->shutdown();
    }
});

$pool->start();
 ```

!> 具体用法可以参考[Swoole\Coroutine\Socket](/coroutine_client/socket)和[Swoole\Process](/process/process?id=exportsocket)相关章节。

```php
$q = msg_get_queue($key);
foreach (range(1, 100) as $i) {
    $data = json_encode(['data' => base64_encode(random_bytes(1024)), 'id' => uniqid(), 'index' => $i,]);
    msg_send($q, $i, $data, false);
}
```  
### on()

プロセスプールのコールバック関数を設定します。

```php
Swoole\Process\Pool->on(string $event, callable $function): bool;
```

* **パラメータ** 

  * **`string $event`**
    * **役割**：イベントを指定します
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`callable $function`**
    * **役割**：コールバック関数
    * **デフォルト値**：なし
    * **その他の値**：なし

* **イベント**

  * **onWorkerStart** ワーカープロセスの開始

  ```php
  /**
  * @param \Swoole\Process\Pool $pool Poolオブジェクト
  * @param int $workerId   現在の作業プロセスの番号、バックエンドでは子プロセスに番号が付けられる
  */
  $pool = new Swoole\Process\Pool(2);
  $pool->on('WorkerStart', function(Swoole\Process\Pool $pool, int $workerId){
    echo "Worker#{$workerId} is started\n";
  });
  ```

  * **onWorkerStop** ワーカープロセスの終了

  ```php
  /**
  * @param \Swoole\Process\Pool $pool Poolオブジェクト
  * @param int $workerId   現在の作業プロセスの番号、バックエンドでは子プロセスに番号が付けられる
  */
  $pool = new Swoole\Process\Pool(2);
  $pool->on('WorkerStop', function(Swoole\Process\Pool $pool, int $workerId){
    echo "Worker#{$workerId} stop\n";
  });
  ```

  * **onMessage** メッセージの受信

  !> 外部からのメッセージを受信します。 一度の接続で一度のメッセージのみを受け取ることができます。これは`PHP-FPM`のショートコネクションメカニズムと類似しています

  ```php
  /**
    * @param \Swoole\Process\Pool $pool Poolオブジェクト
    * @param string $data メッセージのデータ内容
   */
  $pool = new Swoole\Process\Pool(2);
```php
$pool->on('Message', function(Swoole\Process\Pool $pool, string $data){
    var_dump($data);
  });
  ```

  !> イベント名は大文字と小文字を区別しません。 `WorkerStart`、`workerStart`、または`workerstart`はすべて同じです。
```
```php
<?php
use Swoole\Process;
use Swoole\Coroutine;

$pool = new Process\Pool(2, SWOOLE_IPC_UNIXSOCK);
$pool->set(['enable_coroutine' => true, 'enable_message_bus' => false, 'max_package_size' => 2 * 1024]);

$pool->on('WorkerStart', function (Swoole\Process\Pool $pool, int $workerId) {
    if ($workerId == 0) {
        $pool->sendMessage(str_repeat('a', 2 * 3000), 1);
    }
});

$pool->on('Message', function (Swoole\Process\Pool $pool, string $data) {
    var_dump(strlen($data));
});
$pool->start();
```
```php
$pool->sendMessage(str_repeat('a', 2 * 3000), 1);
    }
});

$pool->on('Message', function (Swoole\Process\Pool $pool, string $data) {
    var_dump(strlen($data));
});
$pool->start();

// int(6000)
```
### __construct()

Construct method.

```php
Swoole\Process\Pool::__construct(int $worker_num, int $ipc_type = SWOOLE_IPC_NONE, int $msgqueue_key = 0, bool $enable_coroutine = false);
```

* **Parameters** 

  * **`int $worker_num`**
    * **Description**: Specifies the number of worker processes.
    * **Default Value**: None
    * **Other Values**: None

  * **`int $ipc_type`**
    * **Description**: Mode of inter-process communication.
    * **Default Value**: `SWOOLE_IPC_NONE`
    * **Other Values**: `SWOOLE_IPC_MSGQUEUE`, `SWOOLE_IPC_SOCKET`, `SWOOLE_IPC_UNIXSOCK`

    !> -When set to `SWOOLE_IPC_NONE`, the `onWorkerStart` callback must be set. Within `onWorkerStart`, a loop logic must be implemented. If the `onWorkerStart` function exits, the worker process will immediately exit, and the `Manager` process will restart the process.  
    -When set to `SWOOLE_IPC_MSGQUEUE`, system message queue communication is used. Specifying `$msgqueue_key` will specify the message queue's `KEY`. If the message queue `KEY` is not set, a private queue will be created.  
    -When set to `SWOOLE_IPC_SOCKET`, communication via `Socket` is used. You need to use the [listen](/process/process_pool?id=listen) method to specify the address and port to listen on.  
    -When set to `SWOOLE_IPC_UNIXSOCK`, communication via [unixSocket](/learn?id=What_is_IPC) is used, recommended for coroutine mode. It is strongly recommended to use this method for inter-process communication. See the specific usage below.
- 使用 `SWOOLE_IPC_NONE` 以外の設定をする場合、`onMessage` コールバックを設定する必要があります。`onWorkerStart` はオプションになりました。

  * **`int $msgqueue_key`**
    * **機能**: メッセージキューの `key`
    * **デフォルト値**: `0`
    * **その他の値**: なし

  * **`bool $enable_coroutine`**
    * **機能**: コルーチンサポートを有効にするかどうか【コルーチンを使用すると `onMessage` コールバックを設定できなくなります】
    * **デフォルト値**: `false`
    * **その他の値**: `true`

* **Coroutines Mode**

`v4.4.0` で `Process\Pool` モジュールにコルーチンのサポートが追加され、第4引数を `true` に設定することで有効化できます。コルーチンを有効にすると、バックグラウンドで `onWorkerStart` でコルーチンと [コルーチンスケジューラー](/coroutine/scheduler) が自動的に作成され、コールバック関数内でコルーチン関連の `API` を直接使用できます。例：

```php
$pool = new Swoole\Process\Pool(1, SWOOLE_IPC_NONE, 0, true);

$pool->on('workerStart', function (Swoole\Process\Pool $pool, int $workerId) {
    while (true) {
        Co::sleep(0.5);
        echo "hello world\n";
    }
});

$pool->start();
```

コルーチンを開始すると、Swoole は `onMessage` イベントコールバックの設定を禁止します。プロセス間通信が必要な場合は、第2引数を `SWOOLE_IPC_UNIXSOCK` に設定して [Unix ソケット](/learn?id=什么是IPC) を使用することを示し、そして `$pool->getProcess()->exportSocket()` を使用して [Swoole\Coroutine\Socket](/coroutine_client/socket) オブジェクトをエクスポートし、`Worker` プロセス間の通信を実現します。例：

```php
$pool = new Swoole\Process\Pool(2, SWOOLE_IPC_UNIXSOCK, 0, true); 
```php
$pool->on('workerStart', function (Swoole\Process\Pool $pool, int $workerId) {
    $process = $pool->getProcess(0);
    $socket = $process->exportSocket();
    if ($workerId == 0) {
        echo $socket->recv();
        $socket->send("hello proc1\n");
        echo "proc0 stop\n";
    } else {
        $socket->send("hello proc0\n");
        echo $socket->recv();
        echo "proc1 stop\n";
        $pool->shutdown();
    }
});

$pool->start();
 ```

!> 具体用法可以参考[Swoole\Coroutine\Socket](/coroutine_client/socket)和[Swoole\Process](/process/process?id=exportsocket)相关章节。

```php
$q = msg_get_queue($key);
foreach (range(1, 100) as $i) {
    $data = json_encode(['data' => base64_encode(random_bytes(1024)), 'id' => uniqid(), 'index' => $i,]);
    msg_send($q, $i, $data, false);
}
```
### on()

プロセスプールのコールバック関数を設定します。

```php
Swoole\Process\Pool->on(string $event, callable $function): bool;
```

* **パラメーター**

  * **`string $event`**
    * **機能**: イベントを指定します
    * **デフォルト**: なし
    * **その他の値**: なし

  * **`callable $function`**
    * **機能**: コールバック関数
    * **デフォルト**: なし
    * **その他の値**: なし

* **イベント**

  * **onWorkerStart** ワーカープロセスの開始

  ```php
  /**
  * @param \Swoole\Process\Pool $pool Poolオブジェクト
  * @param int $workerId   WorkerId 現在のワーカープロセスの番号。バックエンドは子プロセスに番号を付けます
  */
  $pool = new Swoole\Process\Pool(2);
  $pool->on('WorkerStart', function(Swoole\Process\Pool $pool, int $workerId){
    echo "Worker#{$workerId} is started\n";
  });
  ```

  * **onWorkerStop** ワーカープロセスの終了

  ```php
  /**
  * @param \Swoole\Process\Pool $pool Poolオブジェクト
  * @param int $workerId   WorkerId 現在のワーカープロセスの番号。バックエンドは子プロセスに番号を付けます
  */
  $pool = new Swoole\Process\Pool(2);
  $pool->on('WorkerStop', function(Swoole\Process\Pool $pool, int $workerId){
    echo "Worker#{$workerId} stop\n";
  });
  ```

  * **onMessage** メッセージの受信

  !> 外部からのメッセージを受信します。 1回の接続で1回だけメッセージを送信できます。PHP-FPMの短い接続メカニズムに類似しています

  ```php
  /**
    * @param \Swoole\Process\Pool $pool Poolオブジェクト
    * @param string $data メッセージのデータ内容
   */
  $pool = new Swoole\Process\Pool(2);
```php
$pool->on('Message', function(Swoole\Process\Pool $pool, string $data){
  var_dump($data);
});
```

!> 事件名称は大文字と小文字を区別しません、「WorkerStart」、「workerStart」または「workerstart」はすべて同じです
```
### sendMessage()

データをターゲットプロセスに送信するには、`$ipc_mode`が`SWOOLE_IPC_UNIXSOCK`である必要があります。

```php
Swoole\Process\Pool->sendMessage(string $data, int $dst_worker_id): bool
```

* **Parameters** 

  * **`string $data`**
    * **Description**: The data to be sent
    * **Default**: None
    * **Other values**: None

  * **`int $dst_worker_id`**
    * **Description**: The target worker id
    * **Default**: `0`
    * **Other values**: None

* **Return Value**

  * Returns `true` on success
  * Returns `false` on failure

* **Note**

  * If the data being sent is greater than `max_package_size` and `enable_message_bus` is `false`, the target process will truncate the data when receiving it.

```php
<?php
use Swoole\Process;
use Swoole\Coroutine;

$pool = new Process\Pool(2, SWOOLE_IPC_UNIXSOCK);
$pool->set(['enable_coroutine' => true, 'enable_message_bus' => false, 'max_package_size' => 2 * 1024]);

$pool->on('WorkerStart', function (Swoole\Process\Pool $pool, int $workerId) {
    if ($workerId == 0) {
        $pool->sendMessage(str_repeat('a', 2 * 3000), 1);
    }
});

$pool->on('Message', function (Swoole\Process\Pool $pool, string $data) {
    var_dump(strlen($data));
});
$pool->start();

// int(2048)


$pool = new Process\Pool(2, SWOOLE_IPC_UNIXSOCK);
$pool->set(['enable_coroutine' => true, 'enable_message_bus' => true, 'max_package_size' => 2 * 1024]);

$pool->on('WorkerStart', function (Swoole\Process\Pool $pool, int $workerId) {
    if ($workerId == 0) {
```php
$pool->sendMessage(str_repeat('a', 2 * 3000), 1);
    }
});

$pool->on('Message', function (Swoole\Process\Pool $pool, string $data) {
    var_dump(strlen($data));
});
$pool->start();

// int(6000)
```
### start()

ワーカープロセスを起動します。

```php
Swoole\Process\Pool->start(): bool
```

!> 起動に成功すると、現在のプロセスは`wait`状態に入り、ワーカープロセスを管理します。  
起動に失敗すると、`false`を返し、`swoole_errno`を使用してエラーコードを取得できます。

* **使用例**

```php
$workerNum = 10;
$pool = new Swoole\Process\Pool($workerNum);

$pool->on("WorkerStart", function ($pool, $workerId) {
    echo "Worker#{$workerId} is started\n";
    $redis = new Redis();
    $redis->pconnect('127.0.0.1', 6379);
    $key = "key1";
    while (true) {
         $msg = $redis->brpop($key, 2);
         if ( $msg == null) continue;
         var_dump($msg);
     }
});

$pool->on("WorkerStop", function ($pool, $workerId) {
    echo "Worker#{$workerId} is stopped\n";
});

$pool->start();
```

* **プロセス管理**

  * 特定のワーカープロセスが致命的なエラーや自発的に終了した場合、マネージャーが回収を行い、ゾンビプロセスを回避します
  * ワーカープロセスが終了すると、マネージャーは自動的に新しいワーカープロセスを立ち上げます
  * メインプロセスが`SIGTERM`シグナルを受信すると、`fork`新しいプロセスを停止し、実行中のすべてのワーカープロセスを`kill`します
  * メインプロセスが`SIGUSR1`シグナルを受信すると、実行中のすべてのワーカープロセスを`kill`し、新しいワーカープロセスを再起動します

* **シグナル処理**

  プロセス間のシグナルは、メインプロセス（マネージャープロセス）だけが処理され、`Worker`プロセスに対するシグナルは設定されていません。開発者がシグナルのリッスンを実装する必要があります。

  - ワーカープロセスは非同期モードです。[Swoole\Process::signal](/process/process?id=signal) を使用してシグナルを監視してください。
- **ワーカープロセスは同期モードで実行されていますので、`pcntl_signal`と`pcntl_signal_dispatch`を使用してシグナルを監視してください**

  ワーカープロセスでは`SIGTERM`シグナルを監視すべきです。親プロセスがこのプロセスを終了する必要がある場合、`SIGTERM`シグナルが送信されます。ワーカープロセスが`SIGTERM`シグナルを監視していない場合、プロセスが強制的に終了され、一部のロジックが失われる可能性があります。

```php
$pool->on("WorkerStart", function ($pool, $workerId) {
    $running = true;
    pcntl_signal(SIGTERM, function () use (&$running) {
        $running = false;
    });
    echo "Worker#{$workerId} is started\n";
    $redis = new Redis();
    $redis->pconnect('127.0.0.1', 6379);
    $key = "key1";
    while ($running) {
         $msg = $redis->brpop($key);
         pcntl_signal_dispatch();
         if ($msg == null) continue;
         var_dump($msg);
     }
});
```
### stop()

現在のプロセスのソケットをイベントループから削除し、コルーチンを開始した後でのみこの関数が機能します

```php
Swoole\Process\Pool->stop(): bool
```
### shutdown()

ワーカープロセスを停止します。

```php
Swoole\Process\Pool->shutdown(): bool
```
### getProcess()

現在の作業プロセスオブジェクトを取得します。[Swoole\Process](/process/process)オブジェクトを返します。

!> Swoole バージョン >= `v4.2.0` で利用可能

```php
Swoole\Process\Pool->getProcess(int $worker_id): Swoole\Process
```

* **パラメータ**

  * **`int $worker_id`**
    * **機能**：取得する`worker`を指定します 【オプションパラメータ、デフォルトは現在の`worker`】
    * **デフォルト値**：なし
    * **その他の値**：なし

!> `start`の後、作業プロセスの`onWorkerStart`や他のコールバック関数で呼び出す必要があります。  
返される`Process`オブジェクトはシングルトンパターンであり、作業プロセス内で`getProcess()`を繰り返し呼び出すと同じオブジェクトが返されます。

* **使用例**

```php
$pool = new Swoole\Process\Pool(3);

$pool->on('WorkerStart', function ($pool, $workerId) {
    $process = $pool->getProcess();
    $process->exec('/usr/local/bin/php', ['-r', 'var_dump(swoole_version());']);
});

$pool->start();
```
### detach()

現在の Worker プロセスをプールから分離し、下に新しいプロセスを即座に生成し、古いプロセスはもうデータを処理しません。代わりにアプリケーションレイヤでライフサイクルを自分で管理します。

!> Swoole バージョン >= `v4.7.0` で使用可能

```php
Swoole\Process\Pool->detach(): bool
```

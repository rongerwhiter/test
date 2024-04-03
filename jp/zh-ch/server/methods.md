このセクションでは、異なる方法について説明します。
## __construct()

[非同期I/O](/learn?id=同期io非同期io)のTCP Serverオブジェクトを作成します。

```php
Swoole\Server::__construct(string $host = '0.0.0.0', int $port = 0, int $mode = SWOOLE_PROCESS, int $sockType = SWOOLE_SOCK_TCP): \Swoole\Server
```

  * **パラメーター**

    * `string $host`

      * 機能：リッスンするIPアドレスを指定します。
      * デフォルト値：なし。
      * その他の値：なし。

      !> IPv4では、`127.0.0.1`は自分自身を指し、`0.0.0.0`はすべてのアドレスをリッスンすることを意味します。
      IPv6では、`::1`は自分自身を指し、`::`（`0:0:0:0:0:0:0:0`と同等）はすべてのアドレスをリッスンします。

    * `int $port`

      * 機能：リッスンするポートを指定します（例：`9501`）。
      * デフォルト値：なし。
      * その他の値：なし。

      !> `$sockType`の値が [UnixSocket Stream/Dgram](/learn?id=IPCとは) の場合、このパラメーターは無視されます。
      ポートが`1024`未満の場合は`root`権限が必要です。
      このポートが使用中の場合、`server->start`の時に失敗します。

    * `int $mode`

      * 機能：実行モードを指定します。
      * デフォルト値：[SWOOLE_PROCESS](/learn?id=swoole_process) プロセスモード（デフォルト）。
      * その他の値：[SWOOLE_BASE](/learn?id=swoole_base) ベースモード。

      !> Swoole5から、実行モードのデフォルト値は`SWOOLE_BASE`になりました。

    * `int $sockType`

      * 機能：このサーバーのタイプを指定します。
      * デフォルト値：なし。
      * その他の値：
        * `SWOOLE_TCP/SWOOLE_SOCK_TCP` tcp ipv4 ソケット
        * `SWOOLE_TCP6/SWOOLE_SOCK_TCP6` tcp ipv6 ソケット
        * `SWOOLE_UDP/SWOOLE_SOCK_UDP` udp ipv4 ソケット
        * `SWOOLE_UDP6/SWOOLE_SOCK_UDP6` udp ipv6 ソケット
        * [SWOOLE_UNIX_DGRAM](https://github.com/swoole/swoole-src/blob/master/examples/unixsock/dgram_server.php) unix socket dgram
        * [SWOOLE_UNIX_STREAM](https://github.com/swoole/swoole-src/blob/master/examples/unixsock/stream_server.php) unix socket stream

      !> `$sock_type` | `SWOOLE_SSL`を使用すると、`SSL`トンネル暗号化を有効にできます。`SSL`を有効にした場合は必ず設定する必要があります。 [ssl_key_file](/server/setting?id=ssl_cert_file) および [ssl_cert_file](/server/setting?id=ssl_cert_file)

  * **例**

```php
$server = new \Swoole\Server($host, $port = 0, $mode = SWOOLE_PROCESS, $sockType = SWOOLE_SOCK_TCP);

// UDP/TCPを混合して使用し、内部および外部ポートをリッスンすることができます。複数のポートのリッスンについては addlistener セクションを参照してください。
$server->addlistener("127.0.0.1", 9502, SWOOLE_SOCK_TCP); // TCPを追加
$server->addlistener("192.168.1.100", 9503, SWOOLE_SOCK_TCP); // Web Socketを追加
$server->addlistener("0.0.0.0", 9504, SWOOLE_SOCK_UDP); // UDPを追加
$server->addlistener("/var/run/myserv.sock", 0, SWOOLE_UNIX_STREAM); // UnixSocket Stream
$server->addlistener("127.0.0.1", 9502, SWOOLE_SOCK_TCP | SWOOLE_SSL); // TCP + SSL

$port = $server->addListener("0.0.0.0", 0, SWOOLE_SOCK_TCP); // システムがランダムにポートを割り当て、戻り値は割り当てられたランダムなポートです
echo $port->port;
```
## set()

ランタイムのさまざまなパラメータを設定するために使用されます。サーバーが起動した後は、`$serv->setting` を通じて `Server->set` メソッドで設定されたパラメータ配列にアクセスできます。

```php
Swoole\Server->set(array $setting): void
```

!> `Server->set` は `Server->start` の前に呼び出す必要があります。各設定の具体的な意味については、[このセクション](/server/setting)を参照してください。

  * **例**

```php
$server->set(array(
    'reactor_num'   => 2,     // スレッド数
    'worker_num'    => 4,     // プロセス数
    'backlog'       => 128,   // リッスン待ち行列の長さを設定
    'max_request'   => 50,    // 各プロセスが受け付けるリクエストの最大数
    'dispatch_mode' => 1,     // データパケットの分散ストラテジー
));
```
## on()

`Server`のイベントコールバック関数を登録します。

```php
Swoole\Server->on(string $event, callable $callback): bool
```

!> `on`メソッドを再度呼び出すと、前回の設定が上書きされます。

!> PHP8.2以降、`$event`が`Swoole`で定義されたイベントではない場合、PHP8.2は警告をスローします。動的プロパティの直接設定はPHP8.2でサポートされていません。

  * **Parameters**

    * `string $event`

      * 功能：コールバックイベント名
      * デフォルト値：なし
      * 他の値：なし

      !> 大文字と小文字は区別されません。どのようなイベントコールバックがあるかは、[このセクション](/server/events)を参照してください。イベント名の文字列には`on`を追加しないでください。

    * `callable $callback`

      * 機能：コールバック関数
      * デフォルト値：なし
      * 他の値：なし

      !> 関数名の文字列、クラスの静的メソッド、オブジェクトメソッドの配列、無名関数のいずれかが使用できます。[このセクション](/learn?id=几种设置回调函数的方式)を参照してください。

  * **Return Value**

    * `true`を返すと操作が成功、`false`を返すと操作が失敗。
  
  * **Example**

```php
$server = new Swoole\Server("127.0.0.1", 9501);
$server->on('connect', function ($server, $fd){
    echo "Client:Connect.\n";
});
$server->on('receive', function ($server, $fd, $reactor_id, $data) {
    $server->send($fd, 'Swoole: '.$data);
    $server->close($fd);
});
$server->on('close', function ($server, $fd) {
    echo "Client: Close.\n";
});
$server->start();
```
## addListener()

ポートのリスニングを追加します。ビジネスコードでは、[Swoole\Server->getClientInfo](/server/methods?id=getclientinfo) を呼び出すことで、特定の接続がどのポートから来たかを調べることができます。

```php
Swoole\Server->addListener(string $host, int $port, int $sockType): bool|Swoole\Server\Port
```

!> ポート`1024`以下をリッスンするには`root`権限が必要です  
メインサーバーが`WebSocket`または`HTTP`プロトコルの場合、新しい`TCP`ポートのリスニングはデフォルトでメイン`Server`のプロトコル設定を継承します。新しいプロトコルを有効にするには、新しいプロトコルを設定するために`set`メソッドを個別に呼び出す必要があります [詳細はこちら ](/server/port)。
`Swoole\Server\Port`の詳細については、[こちら](/server/server_port)をクリックしてください。

  * **パラメーター**

    * `string $host`

      * 機能：`__construct()` の `$host` と同じ
      * デフォルト値：`__construct()` の `$host` と同じ
      * その他の値：`__construct()` の `$host` と同じ

    * `int $port`

      * 機能：`__construct()` の `$port` と同じ
      * デフォルト値：`__construct()` の `$port` と同じ
      * その他の値：`__construct()` の `$port` と同じ

    * `int $sockType`

      * 機能：`__construct()` の `$sockType` と同じ
      * デフォルト値：`__construct()` の `$sockType` と同じ
      * その他の値：`__construct()` の `$sockType` と同じ
  
  * **戻り値**

    * `Swoole\Server\Port` が成功したことを示し、`false` が操作に失敗したことを示します。

!> -`Unix Socket`モードでは、`$host` パラメーターにはアクセス可能なファイルパスを指定する必要がありますが、`$port` パラメーターは無視されます  
-`Unix Socket`モードでは、クライアントの`$fd` は数字ではなくファイルパスの文字列となります  
-`Linux`システムで`IPv6`ポートをリッスンした後は、`IPv4`アドレスを使用しても接続できます
## listen()

This method is an alias of `addlistener`.

```php
Swoole\Server->listen(string $host, int $port, int $type): bool|Swoole\Server\Port
```
## addProcess()

ユーザー定義のワーカープロセスを追加します。通常、この関数は監視、レポート、またはその他の特別なタスクを行うための特別なワーカープロセスを作成するために使用されます。

```php
Swoole\Server->addProcess(Swoole\Process $process): int
```

!> `start` メソッドを実行する必要はありません。`Server` が起動するときにプロセスが自動的に作成され、指定されたサブプロセス関数が実行されます。

  * **パラメータ**
  
    * [Swoole\Process](/process/process)

      * 機能: `Swoole\Process` オブジェクト
      * デフォルト値: なし
      * その他の値: なし

  * **戻り値**

    * プロセスIDが返されると、操作が成功したことを示し、そうでない場合は致命的なエラーが発生します。

  * **注意**

    !> -作成されたサブプロセスは、`$server` オブジェクトが提供する各種メソッド（`getClientList`、`getClientInfo`、`stats` など）を呼び出すことができます。                                   
    -`Worker/Task` プロセス内では、`$process` を使用してサブプロセスと通信できます。        
    -ユーザー定義のプロセス内では、`$server->sendMessage` および `Worker/Task` プロセスとの通信が可能です。      
    -ユーザープロセス内では、`Server->task/taskwait` インターフェースは使用できません。              
    -ユーザー定義のプロセス内で、`Server->send/close` を含むインターフェースを使用できます。         
    -ユーザー定義のプロセス内では、`while(true)`（以下の例に示すように）または [EventLoop](/learn?id=什么是eventloop) ループ（たとえば、タイマーを作成する）を行う必要があります。そうしないと、ユーザープロセスは繰り返し終了および再起動します。         

  * **ライフサイクル**

    ?> -ユーザープロセスの寿命は `Master` と [Manager](/learn?id=manager进程) と同様であり、[reload](/server/methods?id=reload) に影響を受けません。     
    -ユーザープロセスは `reload` 命令の制御を受けません。`reload` 時にはユーザープロセスに対して何の情報も送信されません。        
    -サーバーを `shutdown` するとき、ユーザープロセスに `SIGTERM` シグナルが送信され、ユーザープロセスが終了します。            
    -カスタムプロセスは、致命的なエラーが発生した場合は `Manager` プロセスによって管理され、新しいプロセスが作成されます。         
    -カスタムプロセスは `onWorkerStop` などのイベントをトリガーしません。

  * **例**

    ```php
    $server = new Swoole\Server('127.0.0.1', 9501);
    
    /**
     * ユーザープロセスはブロードキャスト機能を実装し、Unixソケットからメッセージをループして受信し、すべての接続先に送信します。
     */
    $process = new Swoole\Process(function ($process) use ($server) {
        $socket = $process->exportSocket();
        while (true) {
            $msg = $socket->recv();
            foreach ($server->connections as $conn) {
                $server->send($conn, $msg);
            }
        }
    }, false, 2, 1);
    
    $server->addProcess($process);
    
    $server->on('receive', function ($serv, $fd, $reactor_id, $data) use ($process) {
        //受信したメッセージを全員に送信
        $socket = $process->exportSocket();
        $socket->send($data);
    });
    
    $server->start();
    ```

    [プロセス間通信のセクション](/process/process?id=exportsocket)をご参照ください。
```python
def greet(name):
    return "Hello, " + name + "!"
```

この関数は名前を受け取り、`Hello, {name}!`という挨拶文を返します。
## __construct()

[非同期IO](/learn?id=同期io非同期io)のTCPサーバーオブジェクトを作成します。

```php
Swoole\Server::__construct(string $host = '0.0.0.0', int $port = 0, int $mode = SWOOLE_PROCESS, int $sockType = SWOOLE_SOCK_TCP): \Swoole\Server
```

 * **パラメータ**

    * `string $host`

        * 機能：リスンするIPアドレスを指定します。
        * デフォルト値：なし。
        * その他の値：なし。

        !> IPv4では `127.0.0.1` はローカルを表し、`0.0.0.0` は全てのアドレスをリスンすることを示します。
        IPv6では `::1` はローカルを表し、`::`（`0:0:0:0:0:0:0:0`と同等）は全てのアドレスをリスンします。

    * `int $port`

        * 機能：リスンするポートを指定します（例：`9501`）。
        * デフォルト値：なし。
        * その他の値：なし。

        !> `$sockType` の値が[UnixSocket Stream/Dgram](/learn?id=什么是IPC)の場合、このパラメータは無視されます。
        `1024`より小さいポートをリスンするには`root`権限が必要です。
        このポートが使用中の場合、`server->start` が失敗します。

    * `int $mode`

        * 機能：実行モードを指定します。
        * デフォルト値：[SWOOLE_PROCESS](/learn?id=swoole_process) マルチプロセスモード（デフォルト）。
        * その他の値：[SWOOLE_BASE](/learn?id=swoole_base) ベースモード。

        !> Swoole5から、実行モードのデフォルト値は`SWOOLE_BASE` になります。

    * `int $sockType`

        * 機能：このServerの種類を指定します。
        * デフォルト値：なし。
        * その他の値：
          * `SWOOLE_TCP/SWOOLE_SOCK_TCP` tcp ipv4 ソケット
          * `SWOOLE_TCP6/SWOOLE_SOCK_TCP6` tcp ipv6 ソケット
          * `SWOOLE_UDP/SWOOLE_SOCK_UDP` udp ipv4 ソケット
          * `SWOOLE_UDP6/SWOOLE_SOCK_UDP6` udp ipv6 ソケット
          * [SWOOLE_UNIX_DGRAM](https://github.com/swoole/swoole-src/blob/master/examples/unixsock/dgram_server.php) unix socket dgram
          * [SWOOLE_UNIX_STREAM](https://github.com/swoole/swoole-src/blob/master/examples/unixsock/stream_server.php) unix socket stream 

        !> `$sock_type` | `SWOOLE_SSL` を使用することで `SSL`トンネル暗号化を有効にできます。`SSL`を有効にした場合、設定が必要になります。 [ssl_key_file](/server/setting?id=ssl_cert_file) と [ssl_cert_file](/server/setting?id=ssl_cert_file)

 * **例**

```php
$server = new \Swoole\Server($host, $port = 0, $mode = SWOOLE_PROCESS, $sockType = SWOOLE_SOCK_TCP);

// UDP/TCP を混在して使用し、内部ネットワークと外部ネットワークのポートを同時にリスンし、複数ポートのリスンについては addlistener セクションを参照してください。
$server->addlistener("127.0.0.1", 9502, SWOOLE_SOCK_TCP); // TCP を追加
$server->addlistener("192.168.1.100", 9503, SWOOLE_SOCK_TCP); // Web Socket を追加
$server->addlistener("0.0.0.0", 9504, SWOOLE_SOCK_UDP); // UDP を追加
$server->addlistener("/var/run/myserv.sock", 0, SWOOLE_UNIX_STREAM); //UnixSocket Stream
$server->addlistener("127.0.0.1", 9502, SWOOLE_SOCK_TCP | SWOOLE_SSL); //TCP + SSL

$port = $server->addListener("0.0.0.0", 0, SWOOLE_SOCK_TCP); // システムがランダムにポートを割り当て、返り値は割り当てられたランダムなポートです
echo $port->port;
```
## set()

ランタイムでの各種パラメータの設定に使用されます。サーバーが起動した後は`$serv->setting`を介して`Server->set`メソッドで設定されたパラメータ配列にアクセスできます。

```php
Swoole\Server->set(array $setting): void
```

!> `Server->set`は`Server->start`の前に呼び出す必要があります。各設定の詳細については、[このセクション](/server/setting)を参照してください。

  * **例**

```php
$server->set(array(
    'reactor_num'   => 2,     // スレッド数
    'worker_num'    => 4,     // プロセス数
    'backlog'       => 128,   // Listenキューの長さを設定
    'max_request'   => 50,    // 各プロセスが受け入れる最大リクエスト数
    'dispatch_mode' => 1,     // パケットディスパッチモード
));
```
```php
Swoole\Server->on(string $event, callable $callback): bool
```

!> 重複して`on`メソッドを呼び出すと、前回の設定が上書きされます

!> PHP8.2以降、`$event`が`Swoole`で指定されたイベントでない場合、PHP8.2は警告を発生させます。PHP8.2はダイナミックプロパティを直接設定することをサポートしていません

  * **パラメータ**

    * `string $event`

      * 機能：コールバックイベントの名称
      * デフォルト値：なし
      * その他の値：なし

      !> 大文字小文字を区別しません。イベントコールバックについては[このセクション](/server/events)を参照してください。イベント名には`on`を追加しないでください。

    * `callable $callback`

      * 機能：コールバック関数
      * デフォルト値：なし
      * その他の値：なし

      !> 関数名の文字列、クラスの静的メソッド、オブジェクトメソッドの配列、無名関数のいずれかが使用できます。[このセクション](/learn?id=几种设置回调函数的方式)を参照してください。
  
  * **戻り値**

    * `true`を返すと操作が成功し、`false`を返すと操作が失敗します。

  * **例**

```php
$server = new Swoole\Server("127.0.0.1", 9501);
$server->on('connect', function ($server, $fd){
    echo "Client:Connect.\n";
});
$server->on('receive', function ($server, $fd, $reactor_id, $data) {
    $server->send($fd, 'Swoole: '.$data);
    $server->close($fd);
});
$server->on('close', function ($server, $fd) {
    echo "Client: Close.\n";
});
$server->start();
```
## addListener()

リッスンポートを追加します。ビジネスコードでは、[Swoole\Server->getClientInfo](/server/methods?id=getclientinfo) を呼び出すことで、特定の接続がどのポートから来たかを取得できます。

```php
Swoole\Server->addListener(string $host, int $port, int $sockType): bool|Swoole\Server\Port
```

!> ポート番号`1024`以下をリッスンするには、`root`権限が必要です  
メインサーバーが`WebSocket`または`HTTP`プロトコルの場合、新しくリッスンされた`TCP`ポートはデフォルトでメインの`Server`のプロトコル設定を継承します。新しいプロトコルを有効にするには、新しいプロトコルを設定するために`set`メソッドを個別に呼び出す必要があります [詳細はこちら ](/server/port)。
`Swoole\Server\Port`の詳細は[こちら](/server/server_port)をクリックしてください。

  * **パラメータ**

    * `string $host`

      * 機能：`__construct()` の `$host` と同様
      * デフォルト値：`__construct()` の `$host` と同様
      * その他の値：`__construct()` の `$host` と同様

    * `int $port`

      * 機能：`__construct()` の `$port` と同様
      * デフォルト値：`__construct()` の `$port` と同様
      * その他の値：`__construct()` の `$port` と同様

    * `int $sockType`

      * 機能：`__construct()` の `$sockType` と同様
      * デフォルト値：`__construct()` の `$sockType` と同様
      * その他の値：`__construct()` の `$sockType` と同様
  
  * **戻り値**

    * `Swoole\Server\Port`が成功したことを示し、`false`が失敗を示します。

!> - `Unix Socket`モードでは、`$host`パラメータにはアクセス可能なファイルパスを指定する必要があり、`$port`パラメータは無視されます  
- `Unix Socket`モードでは、クライアントの`$fd`は数字ではなく、ファイルパスの文字列となります  
- `Linux`システムでは、`IPv6`ポートをリッスンした後に`IPv4`アドレスを使用しても接続が可能です
```php
Swoole\Server->listen(string $host, int $port, int $type): bool|Swoole\Server\Port
```

このメソッドは`addlistener`のエイリアスです。
## addProcess()

ユーザー定義のワーカープロセスを追加します。通常、この関数は特定の監視、レポート、または他の特別なタスクを実行するための特別なワーカープロセスを作成するために使用されます。

```php
Swoole\Server->addProcess(Swoole\Process $process): int
```

!> `start`を実行する必要はありません。`Server`が起動するときにプロセスが自動的に作成され、指定されたサブプロセス関数が実行されます

  * **パラメータ**
  
    * [Swoole\Process](/process/process)

      * 機能: `Swoole\Process`オブジェクト
      * デフォルト値: なし
      * その他の値: なし

  * **戻り値**

    * プロセスID番号が返され、操作に成功したことを示します。それ以外の場合は致命的なエラーが発生します。

  * **注意**

    !> -作成された子プロセスは、`$server`オブジェクトが提供する各種メソッド（`getClientList/getClientInfo/stats`など）を呼び出すことができます。
    -`Worker/Task`プロセスでは、`$process`のメソッドを使用して親プロセスと通信できます。
    -ユーザー定義プロセスでは、`$server->sendMessage`と`Worker/Task`プロセス間で通信できます。
    -ユーザープロセス内では、`Server->task/taskwait`インターフェイスは使用できません。
    -ユーザープロセス内では、`Server->send/close`などのインターフェースを使用できます。
    -ユーザープロセス内では、`while(true)`（下記の例に示すように）または[EventLoop](/learn?id=什么是eventloop)ループ（例:タイマーを作成）を実行する必要があります。そうしないと、ユーザープロセスは繰り返し終了および再起動します。

  * **ライフサイクル**

    ?> -ユーザープロセスの寿命は`Master`および[Manager](/learn?id=manager进程)と同じであり、[reload](/server/methods?id=reload)の影響を受けません。
    -ユーザープロセスは`reload`指令の制御を受けず、`reload`時にユーザープロセスに情報が送信されません。
    -サーバーのシャットダウン時には、`SIGTERM`シグナルがユーザープロセスに送信され、ユーザープロセスが終了します。
    -カスタムプロセスは`Manager`プロセスに依存し、致命的なエラーが発生した場合、`Manager`プロセスは新しいプロセスを作成します。
    -カスタムプロセスは`onWorkerStop`などのイベントもトリガーしません。

  * **例**

    ```php
    $server = new Swoole\Server('127.0.0.1', 9501);
    
    /**
     * ユーザープロセスはブロードキャスト機能を実装し、Unixソケットのメッセージを受信し、サーバーのすべての接続に送信します
     */
    $process = new Swoole\Process(function ($process) use ($server) {
        $socket = $process->exportSocket();
        while (true) {
            $msg = $socket->recv();
            foreach ($server->connections as $conn) {
                $server->send($conn, $msg);
            }
        }
    }, false, 2, 1);
    
    $server->addProcess($process);
    
    $server->on('receive', function ($serv, $fd, $reactor_id, $data) use ($process) {
        //受信したメッセージをブロードキャスト
        $socket = $process->exportSocket();
        $socket->send($data);
    });
    
    $server->start();
    ```

    [プロセス間通信のセクション](/process/process?id=exportsocket)を参照してください。
## start()

サーバーを起動し、すべての`TCP/UDP`ポートをリッスンします。

```php
Swoole\Server->start(): bool
```

!> ヒント: 以下は[SWOOLE_PROCESS](/learn?id=swoole_process)モードを例にしています

  * **ヒント**

    - 成功すると`worker_num+2`個のプロセスが作成されます。`Master`プロセス+`Manager`プロセス+`serv->worker_num`個の`Worker`プロセス。  
    - 失敗した場合はすぐに`false`を返します。
    - 成功するとイベントループに入り、クライアントの接続要求を待機します。`start`メソッド以降にコードは実行されません。  
    - サーバーがシャットダウンされると、`start`関数は`true`を返し、次の処理に進みます。  
    - `task_worker_num`が設定されていると、対応する数の[Taskプロセス](/learn?id=taskworkerプロセス)が増えます。  
    - メソッドリスト中の`start`より前のメソッドは`start`呼び出し前にのみ使用可能で、`start`後のメソッドは[onWorkerStart](/server/events?id=onworkerstart)、[onReceive](/server/events?id=onreceive)などのイベントコールバック関数内でのみ使用可能です。

  * **拡張**

    * Master 主プロセス

      * 主プロセス内に複数の[Reactor](/learn?id=reactorスレッド)スレッドがあり、`epoll/kqueue/select`に基づいてネットワークイベントを監視します。データを受け取った後、`Worker`プロセスに処理を委譲します。

    * Manager プロセス

      * すべての`Worker`プロセスを管理し、`Worker`プロセスのライフサイクルが終了したり異常が発生した場合に自動的に回収し、新しい`Worker`プロセスを作成します。

    * Worker プロセス

      * 受信したデータを処理し、プロトコルの解析とリクエストへの応答を行います。`worker_num`が設定されていない場合、下層は`CPU`の数に合わせて`Worker`プロセスを起動します。
      * 起動に失敗すると、拡張内で致命的なエラーが発生しますので、`php error_log`の関連情報を確認してください。`errno={number}`は標準の`Linux Errno`であり、関連文書を参照できます。
      * `log_file`設定が有効になっている場合、情報は指定された`Log`ファイルに出力されます。

  * **戻り値**

    * `true`を返すと操作が成功し、`false`を返すと操作が失敗します

  * **起動失敗の一般的なエラー**

    * `bind`ポートが失敗すると、別のプロセスがそのポートを使用している可能性があります。
    * 必須のコールバック関数が設定されていないため、起動に失敗します。
    * `PHP`コードに致命的なエラーが存在する場合は、PHPエラー情報`php_errors.log`を確認してください。
    * `ulimit -c unlimited`を実行して`core dump`を有効にし、セグメンテーション違反が発生していないか確認してください。
    * `daemonize`を無効にし、`log`を閉じると、エラー情報が画面に出力されるようになります。
Code blocks内のテキスト以外のすべてのテキストを日本語に翻訳します。

```python
# Define a function to calculate the square of a number
def square(n):
    return n**2
```

上記のコードは、指定された数字の2乗を計算する関数を定義しています。
## __construct()

[非同期IO](/learn?id=同期io非同期io)のTCPサーバーオブジェクトを作成します。

```php
Swoole\Server::__construct(string $host = '0.0.0.0', int $port = 0, int $mode = SWOOLE_PROCESS, int $sockType = SWOOLE_SOCK_TCP): \Swoole\Server
```

  * **パラメータ**

    * `string $host`

      * 機能：リスニングするIPアドレスを指定します。
      * デフォルト値：なし。
      * その他の値：なし。

      !> IPv4では `127.0.0.1` はローカルを指し、`0.0.0.0` はすべてのアドレスをリスニングします。
      IPv6では `::1` はローカルを指し、`::`（`0:0:0:0:0:0:0:0`と同等）はすべてのアドレスをリスニングします。

    * `int $port`

      * 機能：リスニングするポートを指定します（例：`9501`）。
      * デフォルト値：なし。
      * その他の値：なし。

      !> `$sockType` が [Unixソケット Stream/Dgram](/learn?id=ipcとは) の場合、このパラメータは無視されます。
      リスンするポートが`1024`未満の場合は`root`権限が必要です。
      このポートが占有されている場合、`server->start` は失敗します。

    * `int $mode`

      * 機能：実行モードを指定します。
      * デフォルト値：[SWOOLE_PROCESS](/learn?id=swoole_process) マルチプロセスモード（デフォルト）。
      * その他の値：[SWOOLE_BASE](/learn?id=swoole_base) ベースモード。

      !> Swoole5から、実行モードのデフォルト値は`SWOOLE_BASE` になりました。

    * `int $sockType`

      * 機能：このServerのタイプを指定します。
      * デフォルト値：なし。
      * その他の値：
        * `SWOOLE_TCP/SWOOLE_SOCK_TCP` tcp ipv4 ソケット
        * `SWOOLE_TCP6/SWOOLE_SOCK_TCP6` tcp ipv6 ソケット
        * `SWOOLE_UDP/SWOOLE_SOCK_UDP` udp ipv4 ソケット
        * `SWOOLE_UDP6/SWOOLE_SOCK_UDP6` udp ipv6 ソケット
        * [SWOOLE_UNIX_DGRAM](https://github.com/swoole/swoole-src/blob/master/examples/unixsock/dgram_server.php) unix socket dgram
        * [SWOOLE_UNIX_STREAM](https://github.com/swoole/swoole-src/blob/master/examples/unixsock/stream_server.php) unix socket stream 

      !> `$sock_type` | `SWOOLE_SSL`を使用すると、`SSL`トンネル暗号化を有効にできます。`SSL`を有効にした後は、必ず設定する必要があります。[ssl_key_file](/server/setting?id=ssl_cert_file) および [ssl_cert_file](/server/setting?id=ssl_cert_file)

  * **例**

```php
$server = new \Swoole\Server($host, $port = 0, $mode = SWOOLE_PROCESS, $sockType = SWOOLE_SOCK_TCP);

// UDP/TCPを混在して使用し、内部および外部のポートを同時にリスンし、複数ポートのリスンは addlistener セクションを参照してください。
$server->addlistener("127.0.0.1", 9502, SWOOLE_SOCK_TCP); // TCPを追加
$server->addlistener("192.168.1.100", 9503, SWOOLE_SOCK_TCP); // Web Socketを追加
$server->addlistener("0.0.0.0", 9504, SWOOLE_SOCK_UDP); // UDPを追加
$server->addlistener("/var/run/myserv.sock", 0, SWOOLE_UNIX_STREAM); //UnixSocket Streamを追加
$server->addlistener("127.0.0.1", 9502, SWOOLE_SOCK_TCP | SWOOLE_SSL); //TCP + SSLを追加

$port = $server->addListener("0.0.0.0", 0, SWOOLE_SOCK_TCP); // システムがランダムにポートを割り当て、割り当てられたポートを返します
echo $port->port;
```
## set()

ランタイムの各種パラメータを設定するために使用されます。サーバーが起動した後は、`Server->set` メソッドで設定されたパラメータ配列には `$serv->setting` を使用してアクセスできます。

```php
Swoole\Server->set(array $setting): void
```

!> `Server->set` は `Server->start` を呼び出す前に実行する必要があります。各設定の詳細については、[このセクション](/server/setting) を参照してください。

  * **例**

```php
$server->set(array(
    'reactor_num'   => 2,     // number of threads
    'worker_num'    => 4,     // number of processes
    'backlog'       => 128,   // set the length of the Listen queue
    'max_request'   => 50,    // maximum number of requests per process
    'dispatch_mode' => 1,     // data packet distribution strategy
));
```
## on()

`Server`のイベントコールバック関数を登録します。

```php
Swoole\Server->on(string $event, callable $callback): bool
```

!> `on`メソッドを複数回呼び出すと、前回の設定が上書きされます

!> PHP8.2から、`$event`が`Swoole`が定義したイベントでない場合、PHP8.2は警告を出力します。なぜならPHP8.2は直接的な動的プロパティの設定をサポートしていないためです。

  * **パラメータ**

    * `string $event`

      * 機能：コールバックイベントの名称
      * デフォルト値：なし
      * 他の値：なし

      !> 大文字小文字を区別しません。詳細なイベントコールバックについては、[このセクション](/server/events)を参照してください。イベント名の文字列には`on`を追加しないでください。

    * `callable $callback`

      * 機能：コールバック関数
      * デフォルト値：なし
      * 他の値：なし

      !> 関数名の文字列、クラスの静的メソッド、オブジェクトのメソッド配列、匿名関数のいずれかが有効です。[このセクション](/learn?id=几种设置回调函数的方式)を参照してください。

  * **戻り値**

    * `true`を返すと操作が成功、`false`を返すと操作が失敗します。

  * **例**

```php
$server = new Swoole\Server("127.0.0.1", 9501);
$server->on('connect', function ($server, $fd){
    echo "Client:Connect.\n";
});
$server->on('receive', function ($server, $fd, $reactor_id, $data) {
    $server->send($fd, 'Swoole: '.$data);
    $server->close($fd);
});
$server->on('close', function ($server, $fd) {
    echo "Client: Close.\n";
});
$server->start();
```
## addListener()

ポートのリスニングを追加します。アプリケーションコードでは、[Swoole\Server->getClientInfo](/server/methods?id=getclientinfo)を呼び出すことで特定の接続がどのポートから来たかを取得できます。

```php
Swoole\Server->addListener(string $host, int $port, int $sockType): bool|Swoole\Server\Port
```

!> `1024`以下のポートをリスンするには`root`権限が必要です  
親サーバーが`WebSocket`または`HTTP`プロトコルの場合、新しいリスンする`TCP`ポートはデフォルトで親`Server`のプロトコル設定を継承します。新しいプロトコルを有効にしたい場合は、新しいプロトコルを設定するために`set`メソッドを個別に呼び出す必要があります [詳細はこちら](/server/port)。
`Swoole\Server\Port`の詳細については、[こちら](/server/server_port)をクリックしてください。

  * **パラメータ**

    * `string $host`

      * 機能： `__construct()` の `$host` と同じ
      * デフォルト値： `__construct()` の `$host` と同じ
      * その他の値： `__construct()` の `$host` と同じ

    * `int $port`

      * 機能： `__construct()` の `$port` と同じ
      * デフォルト値： `__construct()` の `$port` と同じ
      * その他の値： `__construct()` の `$port` と同じ

    * `int $sockType`

      * 機能： `__construct()` の `$sockType` と同じ
      * デフォルト値： `__construct()` の `$sockType` と同じ
      * その他の値： `__construct()` の `$sockType` と同じ
  
  * **戻り値**

    * `Swoole\Server\Port`が返されると操作は成功し、`false`が返されると操作が失敗しました。

!> -`Unix Socket`モードでは、`$host`パラメーターにはアクセス可能なファイルパスを指定する必要があり、`$port`パラメーターは無視されます  
-`Unix Socket`モードでは、クライアントの`$fd`は数値ではなく、ファイルパスの文字列になります  
-`Linux`システムでは、`IPv6`ポートをリスニングした後、`IPv4`アドレスを使用しても接続が可能です
## listen()

This method is an alias of `addlistener`.

```php
Swoole\Server->listen(string $host, int $port, int $type): bool|Swoole\Server\Port
```
## addProcess()

ユーザー定義のワーカープロセスを追加します。通常、この関数は特別なタスクを監視したり報告したりするための特別なワーカープロセスを作成するために使用されます。

```php
Swoole\Server->addProcess(Swoole\Process $process): int
```

!> `start`を実行する必要はありません。`Server`が起動するときにプロセスが自動的に作成され、指定された子プロセス関数が実行されます。

  * **Parameters**
  
    * [Swoole\Process](/process/process)

      * 意味：`Swoole\Process` オブジェクト
      * デフォルト値：無し
      * 他の値：無し

  * **Return Value**

    * プロセスID番号が返され、操作が成功したことを示します。それ以外の場合、プログラムは致命的なエラーを投げます。

  * **注意**

    !> -作成した子プロセスは`$server`オブジェクトが提供する各種メソッドを呼び出すことができます。例えば、`getClientList/getClientInfo/stats`などです。    
    -`Worker/Task`プロセス内で、`$process`を使用して子プロセスと通信できます。    
    -ユーザー定義のプロセス内で、`$server->sendMessage`を使用して`Worker/Task`プロセスと通信できます。    
    -ユーザープロセス内では`Server->task/taskwait`インターフェースは使用できません。    
    -ユーザープロセス内では`Server->send/close`などのインターフェースを使用できます。    
    -ユーザープロセス内では、`while(true)`（以下の例を参照）または[EventLoop](/learn?id=什么是eventloop)ループ（例：タイマーを作成する）を実行する必要があります。さもないと、ユーザープロセスは繰り返し終了して再起動します。

  * **ライフサイクル**

    ?> -ユーザープロセスの寿命は`Master`および [Manager](/learn?id=manager进程) と同じであり、[reload](/server/methods?id=reload)の影響を受けません。     
    -ユーザープロセスは`reload`命令の制御を受けず、`reload`時にユーザープロセスに情報が送られることはありません。    
    -サーバーをシャットダウンする際、ユーザープロセスに`SIGTERM`シグナルが送られ、ユーザープロセスが終了します。    
    -カスタムプロセスは`Manager`プロセスに管理され、致命的なエラーが発生した場合、`Manager`プロセスは新しいものを作成します。    
    -カスタムプロセスは`onWorkerStop`などのイベントをトリガーしません。

  * **Example**

    ```php
    $server = new Swoole\Server('127.0.0.1', 9501);
    
    /**
     * ユーザープロセスはブロードキャスト機能を実装し、Unixソケットからのメッセージを受信して、サーバーのすべての接続に送信します。
     */
    $process = new Swoole\Process(function ($process) use ($server) {
        $socket = $process->exportSocket();
        while (true) {
            $msg = $socket->recv();
            foreach ($server->connections as $conn) {
                $server->send($conn, $msg);
            }
        }
    }, false, 2, 1);
    
    $server->addProcess($process);
    
    $server->on('receive', function ($serv, $fd, $reactor_id, $data) use ($process) {
        //受信したメッセージを全員に送信
        $socket = $process->exportSocket();
        $socket->send($data);
    });
    
    $server->start();
    ```

    [プロセス間通信の章](/process/process?id=exportsocket)を参照してください。
## start()

サーバーを起動し、すべての`TCP/UDP`ポートをリッスンします。

```php
Swoole\Server->start(): bool
```

!> ヒント: 以下は [SWOOLE_PROCESS](/learn?id=swoole_process) モードの例です

  * **ヒント**

    - 起動後、`worker_num+2`個のプロセスが作成されます。`Master`プロセス+`Manager`プロセス+`serv->worker_num`個の`Worker`プロセスです。
    - 起動が失敗するとすぐに`false`を返します。
    - 起動に成功すると、クライアントの接続要求を待機するためにイベントループに入ります。`start`メソッドの後のコードは実行されません。
    - サーバーを閉じた後、`start`関数は`true`を返し、続行します。
    - `task_worker_num`を設定すると、対応する数量の [タスクプロセス](/learn?id=taskworker进程) が増加します。
    - メソッドリスト内の`start`の前にあるメソッドは、`start`の呼び出し前にのみ使用でき、`start`の後のメソッドは[onWorkerStart](/server/events?id=onworkerstart)、[onReceive](/server/events?id=onreceive)などのイベントコールバック関数でのみ使用できます。

  * **拡張**

    * Master 主プロセス

      * 主プロセスには複数の[リアクタ](/learn?id=reactor线程)スレッドがあり、`epoll/kqueue/select`を使用してネットワークイベントを監視します。データを受信した後、`Worker`プロセスに処理を転送します。
    
    * Manager プロセス

      * すべての`Worker`プロセスを管理し、`Worker`プロセスのライフサイクルが終了したり異常が発生した場合には自動的にリサイクルし、新しい`Worker`プロセスを作成します。
    
    * Worker プロセス

      * 受信したデータを処理し、プロトコルの解析やリクエストの応答を行います。`worker_num`が設定されていない場合、背後では`CPU`の数量と同じ数の`Worker`プロセスが起動します。
      * 起動が失敗すると、拡張内で致命的なエラーが発生し、関連する情報を含む`php error_log`を確認してください。`errno={number}`は標準の`Linux Errno`であり、関連ドキュメントを参照できます。
      * `log_file`設定がオンになっている場合、情報は指定された`Log`ファイルに出力されます。

  * **戻り値**

    * 操作が成功した場合は`true`を返し、失敗した場合は`false`を返します。

  * **起動失敗の一般的なエラー**

    * `bind`ポートに失敗した場合、他のプロセスがこのポートを占有している可能性があります。
    * 必須のコールバック関数が設定されていない場合、起動が失敗します。
    * `PHP`コードに致命的なエラーが存在する場合は、PHPエラー情報の`php_errors.log`を確認してください。
    * `ulimit -c unlimited`を実行し、`core dump`を有効にして、セグメントエラーが発生していないか確認します。
    * `daemonize`を無効にし、`log`を閉じると、エラー情報が画面に表示されるようになります。
## reload()

すべてのWorker/Taskプロセスを安全に再起動します。

```php
Swoole\Server->reload(bool $only_reload_taskworker = false): bool
```

!> 例：忙しいバックエンドサーバーは常にリクエストを処理しています。管理者が`kill`コマンドを使用してサーバープログラムを中止/再起動しようとすると、ちょうどコードの半分が実行中の場合があります。  
このような場合、データの不整合が発生します。例えば、支払いロジックの次のステップは出荷であり、支払いロジックの後にプロセスが中止された場合、ユーザーが通貨を支払ったが商品を送っていない場合があります。これは非常に深刻な結果をもたらします。  
`Swoole`は、柔軟な終了/再起動メカニズムを提供しており、管理者は`Server`に特定のシグナルを送信するだけで、`Server`の`Worker`プロセスを安全に終了できます。参考: [サービスを正しく再起動する方法](/question/use?id=swoole如何正确的重启服务)。

  * **パラメータ**
  
    * `bool $only_reload_taskworker`

      * 機能：[Taskプロセス](/learn?id=taskworkerプロセス)のみを再起動するかどうか
      * デフォルト値：false
      * 他の値：true

!> - `reload`には保護メカニズムがあり、1回の`reload`が進行中の場合、新しい再起動シグナルが破棄されます。
-`user/group`を設定している場合、`Worker`プロセスに`master`プロセスに情報を送信する権限がないかもしれません。この場合、`root`ユーザーで`shell`内で`kill`コマンドを実行する必要があります。
-`addProcess`で追加されたユーザープロセスには`reload`命令が適用されません。

  * **戻り値**

    * `true`を返すと操作が成功し、`false`を返すと操作が失敗します。

  * **拡張**

    * **シグナルの送信**

        * `SIGTERM`: メインプロセス/マネージメントプロセスにこのシグナルを送信すると、サーバーが安全に終了します。
        * PHPコード内で`$serv->shutdown()`を呼び出すことでこの操作を完了できます。
        * `SIGUSR1`: メインプロセス/管理プロセスに`SIGUSR1`シグナルを送信すると、すべての`Worker`プロセスと`TaskWorker`プロセスが平和に`restart`されます。
        * `SIGUSR2`: メインプロセス/管理プロセスに`SIGUSR2`シグナルを送信すると、すべての`Task`プロセスが平和に再起動します。
        * PHPコード内で`$serv->reload()`を呼び出すことでこの操作を完了できます。

    ```shell
    # すべてのworkerプロセスを再起動
    kill -USR1 メインプロセスPID
    
    # Taskプロセスのみを再起動
    kill -USR2 メインプロセスPID
    ```
      
      > [参考：Linuxシグナルリスト](/other/signal)

    * **Processモード**

        `Process`で起動したプロセスでは、クライアントからの`TCP`接続が`Master`プロセス内で維持され、`Worker`プロセスの再起動や異常終了は接続そのものに影響しません。

    * **Baseモード**

        `Base`モードでは、クライアント接続が直接`Worker`プロセスで維持されるため、`reload`されるとすべての接続が切断されます。

    !> `Base`モードは[Taskプロセス](/learn?id=taskworkerプロセス)の`reload`をサポートしていません。

    * **再読み込みの有効範囲**

      `reload`操作は、`Worker`プロセスが起動した後にロードしたPHPファイルのみ再読み込みできます。`get_included_files`関数を使用して、`WorkerStart`の前に読み込まれたPHPファイルのリストを取得できます。このリスト内のPHPファイルは、`reload`操作を実行しても再読み込みできません。サーバーをシャットダウンして再起動する必要があります。

    ```php
    $serv->on('WorkerStart', function(Swoole\Server $server, int $workerId) {
        var_dump(get_included_files()); //この配列内のファイルはプロセスが開始される前にロードされているため、再読み込みできません
    });
    ```

    * **APC/OPcache**

        `PHP`で`APC/OPcache`を有効にしている場合、`reload`再読み込みに影響を受けることがあります。解決策は2つあります。

        * `APC/OPcache`の`stat`チェックを有効にします。ファイルの更新を検出した場合、`APC/OPcache`は自動的に`OPCode`を更新します。
        * `onWorkerStart`でファイルを読み込む前に`apc_clear_cache`または`opcache_reset`を実行して`OPCode`キャッシュをリフレッシュします。

  * **注意**

    !> -スムーズな再起動は、`Worker`プロセス内で`include/require`された[onWorkerStart](/server/events?id=onworkerstart)または[onReceive](/server/events?id=onreceive)などにのみ有効です。
    -`Server`が開始される前に`include/require`されたPHPファイルは、スムーズな再起動で再読み込みできません。
    -`Server`の設定、すなわち`$serv->set()`に渡されたパラメータ設定は、`Server`全体を再起動して再読み込みする必要があります。
    -`Server`は内部ポートをリッスンし、リモートからの制御コマンドを受信し、すべての`Worker`プロセスを再起動できます。
## stop()

現在の`Worker`プロセスを停止し、直ちに`onWorkerStop`コールバック関数をトリガーします。

```php
Swoole\Server->stop(int $workerId = -1, bool $waitEvent = false): bool
```

  * **パラメータ**

    * `int $workerId`

      * 機能：`worker id`を指定します。
      * デフォルト値：-1、現在のプロセスを示します。
      * その他の値：ありません。

    * `bool $waitEvent`

      * 機能：終了戦略を制御します。`false`は即時終了、`true`はイベントループが空になるのを待ってから終了します。
      * デフォルト値：false
      * その他の値：true

  * **戻り値**

    * `true`を返すと操作が成功、`false`を返すと操作が失敗したことを示します。

  * **ヒント**

    !> -[Asynchronous I/O](/learn?id=同步io异步io) サーバーは`stop`メソッドを呼び出してプロセスを終了するときに、まだイベントが待機中である可能性があります。たとえば `Swoole\MySQL->query` を使用して`SQL`クエリを送信しましたが、まだ`MySQL`サーバーからの結果を待っている場合があります。この場合、プロセスを強制終了すると、`SQL`の実行結果が失われます。  
    -`$waitEvent = true` に設定すると、バックグラウンドでは[非同期安全な再起動](/question/use?id=swoole如何正确的重启服务)戦略が使用されます。まず、`Manager`プロセスに通知し、新しいリクエストを処理するための新しい`Worker`を再起動します。古い`Worker`はイベント待機状態にあり、イベントループが空になるか最大待機時間を超えるまでプロセスを終了し、非同期イベントの安全性を最大限に確保します。
## shutdown()

Shutdown the service.

```php
Swoole\Server->shutdown(): bool
```

  * **Return Value**
  
    * Returns `true` if the operation is successful, `false` otherwise.

  * **Note**
  
    * This function can be used within the `Worker` process.
    * Sending `SIGTERM` to the master process can also achieve the shutdown of the service.

```shell
kill -15 MasterProcessPID
```
```php
Swoole\Server->tick(int $millisecond, callable $callback): void
```

  * **パラメーター**

    * `int $millisecond`

      * 機能：間隔時間【ミリ秒】
      * デフォルト値：なし
      * その他の値：なし

    * `callable $callback`

      * 機能：コールバック関数
      * デフォルト値：なし
      * その他の値：なし

  * **注意**
  
    !> -`Worker`プロセスが終了した後、すべてのタイマーは自動的に削除されます  
    -`tick/after`タイマーは`Server->start`の前に使用できません  
    -`Swoole5`以降、このエイリアスの使用方法は削除されました。直接`Swoole\Timer::tick()`を使用してください。

  * **例**

    * [onReceive](/server/events?id=onreceive) で使用

    ```php
    function onReceive(Swoole\Server $server, int $fd, int $reactorId, mixed $data)
    {
        $server->tick(1000, function () use ($server, $fd) {
            $server->send($fd, "hello world");
        });
    }
    ```

    * [onWorkerStart](/server/events?id=onworkerstart) で使用

    ```php
    function onWorkerStart(Swoole\Server $server, int $workerId)
    {
        if (!$server->taskworker) {
            $server->tick(1000, function ($id) {
              var_dump($id);
            });
        } else {
            //task
            $server->tick(1000);
        }
    }
    ```
## after()

一回限のタイマーを追加し、完了後に破棄されます。この関数は、[Swoole\Timer::after](/timer?id=after) のエイリアスです。

```php
Swoole\Server->after(int $millisecond, callable $callback)
```

  * **パラメータ**

    * `int $millisecond`

      * 機能：実行時間【ミリ秒】
      * デフォルト値：なし
      * 他の値：なし
      * バージョン影響：`Swoole v4.2.10` 以下では最大で `86400000` を超えることはできません

    * `callable $callback`

      * 機能：コールバック関数、呼び出すことができる必要があります。`callback` 関数は引数を受け取りません
      * デフォルト値：なし
      * 他の値：なし

  * **注意**
  
    !> -タイマーの寿命はプロセスレベルです。`reload`や`kill` を使用してプロセスを再起動または終了した場合、すべてのタイマーが破棄されます  
    -あるタイマーが重要なロジックとデータを持っている場合は、`onWorkerStop` コールバック関数で実装するか、[How to correctly restart the service](/question/use?id=swoole如何正确的重启服务) を参照してください  
    -`Swoole5`以降、このエイリアス使用方法は削除されましたので、直接 `Swoole\Timer::after()` を使用してください
## defer()

[Swoole\Event::defer](/event?id=defer) の別名です。

```php
Swoole\Server->defer(Callable $callback): void
```

* **パラメータ**

  * `Callable $callback`

    * 機能：コールバック関数【必須】、実行可能な関数変数であり、文字列、配列、無名関数のいずれかが可能
    * デフォルト値：なし
    * その他の値：なし

* **注意**

  !> -イベントループ[EventLoop](/learn?id=什么是eventloop)が完了した後にこの関数が実行されます。この関数の目的は、一部のPHPコードを後から実行させ、プログラムが他の`IO`イベントを優先処理できるようにします。例えば、あるコールバック関数がCPU集中型の計算を行い、かつ緊急性がない場合は、プロセスが他のイベントを処理した後にCPU集中型計算に移行することができます  
  -`defer`された関数がすぐに実行されることは保証されません。システム上の重要なロジックであればできるだけ早く実行する必要がある場合は、`after`タイマーを使用してください  
  -`onWorkerStart`コールバック内で`defer`を実行する場合、イベントが発生するのを待たなければコールバックは実行されません
  -`Swoole5`以降、このエイリアスの使用法は削除されましたので、直接`Swoole\Event::defer()`を使用してください

* **例**

```php
function query($server, $db) {
    $server->defer(function() use ($db) {
        $db->close();
    });
}
```
## clearTimer()

`tick/after`タイマーをクリアします。この関数は、[Swoole\Timer::clear](/timer?id=clear)のエイリアスです。

```php
Swoole\Server->clearTimer(int $timerId): bool
```

  * **パラメーター**

    * `int $timerId`

      * 機能：タイマーIDを指定します
      * デフォルト値：なし
      * その他の値：なし

  * **戻り値**

    * `true` を返すと処理が成功し、`false` を返すと処理が失敗します

  * **注意**

    !> -`clearTimer` は現在のプロセスのみで使用できます     
    -`Swoole5`以降、このエイリアスの使用方法は削除されました。代わりに`Swoole\Timer::clear()` を直接使用してください

  * **例**

```php
$timerId = $server->tick(1000, function ($timerId) use ($server) {
    $server->clearTimer($timerId); //$id はタイマーのIDです
});
```
## close()

クライアント接続を閉じます。

```php
Swoole\Server->close(int $fd, bool $reset = false): bool
```

  * **パラメータ**

    * `int $fd`

      * 機能：閉じる`fd`（ファイル記述子）を指定します
      * デフォルト値：なし
      * その他の値：なし

    * `bool $reset`

      * 機能：`true`に設定すると接続を強制的に閉じ、送信キュー内のデータを破棄します
      * デフォルト値：false
      * その他の値：true

  * **戻り値**

    * `true` を返すと操作が成功したことを示し、`false` を返すと操作が失敗したことを示します

  * **注意**

  !> -`Server` が接続を`close` すると、[onClose](/server/events?id=onclose) イベントがトリガーされます  
-`close` の後にクリーンアップロジックを記述しないでください。[onClose](/server/events?id=onclose) コールバック内に配置する必要があります  
- `HTTP\Server` の`fd` は上位コールバックメソッドの `response` で取得されます

  * **例**

```php
$server->on('request', function ($request, $response) use ($server) {
    $server->close($response->fd);
});
```
## send()

データをクライアントに送信します。

```php
Swoole\Server->send(int|string $fd, string $data, int $serverSocket = -1): bool
```

  * **パラメータ**

    * `int|string $fd`

      * 機能：クライアントのファイルディスクリプタまたはUnixソケットのパスを指定します
      * デフォルト値：なし
      * その他の値：なし

    * `string $data`

      * 機能：送信するデータ、`TCP`プロトコルの最大容量は`2M`を超えてはいけません。 [buffer_output_size](/server/setting?id=buffer_output_size) を変更して送信可能な最大パケットサイズを変更できます
      * デフォルト値：なし
      * その他の値：なし

    * `int $serverSocket`

      * 機能：[UnixSocket DGRAM](https://github.com/swoole/swoole-src/blob/master/examples/unixsock/dgram_server.php) にデータを送信する場合に必要なパラメータです。TCPクライアントでは記入の必要はありません
      * デフォルト値：-1、現在のUDPポートを表します
      * その他の値：なし

  * **戻り値**

    * `true` を返すと操作が成功し、 `false` を返すと操作が失敗します。

  * **ヒント**

    !> 送信プロセスは非同期です。下部は自動的に書き込み可能を監視し、データをクライアントに逐次送信します。つまり、`send` の後で対端がデータを受信するわけではありません。

    * セキュリティ
      * `send` 操作はアトミック性を持ち、複数のプロセスが同時に `TCP` 接続にデータを送信する場合、データの混合は発生しません。

    * 長さ制限
      * `2M` を超えるデータを送信する場合、データを一時ファイルに書き込み、その後 `sendfile` インターフェースを介して送信することができます
      * [buffer_output_size](/server/setting?id=buffer_output_size) パラメータを設定することで送信長さの制限を変更できます
      * `8K` のデータを送信する場合、内部では `Worker` プロセスの共有メモリが有効になり、`Mutex->lock` 操作が1回行われる必要があります

    * バッファリング
      * `Worker` プロセスの[unixSocket](/learn?id=What-is-IPC)のバッファリングが満杯の場合、`8K` のデータ送信時に一時ファイルの使用が開始されます
      * 同じクライアントに連続して大量のデータを送信すると、クライアントが受信する余裕がないため、`Socket` のメモリバッファがいっぱいになり、Swooleの下部は即座に `false` を返します。`false` が返されると、データをディスクに保存して、クライアントが送信済みのデータを受け取った後に再度送信できます

    * [コルーチンスケジューリング](/coroutine?id=Coroutine-Scheduling)
      * [send_yield](/server/setting?id=send_yield) を有効にしたコルーチンモードで、バッファが一杯になると `send` は自動的に中断され、対向が一部のデータを読み取るとコルーチンが再開され、データの送信が継続されます。

    * [UnixSocket](/learn?id=What-is-IPC)
      * [UnixSocket DGRAM](https://github.com/swoole/swoole-src/blob/master/examples/unixsock/dgram_server.php) ポートを監視している場合、`send` を使用して対向にデータを送信できます。

      ```php
      $server->on("packet", function (Swoole\Server $server, $data, $addr){
          $server->send($addr['address'], 'SUCCESS', $addr['server_socket']);
      });
      ```
## sendfile()

`TCP`クライアント接続にファイルを送信します。

```php
Swoole\Server->sendfile(int $fd, string $filename, int $offset = 0, int $length = 0): bool
```

  * **パラメータ**

    * `int $fd`

      * 機能：クライアントのファイル記述子を指定します
      * デフォルト値：なし
      * その他の値：なし

    * `string $filename`

      * 機能：送信するファイルのパス。ファイルが存在しない場合は`false`を返します
      * デフォルト値：なし
      * その他の値：なし

    * `int $offset`

      * 機能：ファイルのオフセットを指定し、ファイルの特定の位置からデータを送信できます
      * デフォルト値：0 【既定値は`0`で、ファイルの先頭から送信を開始します】
      * その他の値：なし

    * `int $length`

      * 機能：送信する長さを指定します
      * デフォルト値：ファイルのサイズ
      * その他の値：なし

  * **戻り値**

    * `true`を返すと操作が成功し、`false`を返すと操作が失敗します

  * **注意**

  !> この関数と`Server->send`はどちらもクライアントにデータを送信しますが、`sendfile`のデータは指定されたファイルから送られます.
## sendto()

クライアント`IP:PORT`に`UDP`パケットを送信します。

```php
Swoole\Server->sendto(string $ip, int $port, string $data, int $serverSocket = -1): bool
```

  * **パラメータ**

    * `string $ip`

      * 機能：クライアントの `ip` を指定します。
      * デフォルト値：なし
      * その他の値：なし

      ?> `$ip`は`IPv4`または`IPv6`文字列である必要があります。不正なIPの場合、エラーが返されます。

    * `int $port`

      * 機能：クライアントの `port` を指定します。
      * デフォルト値：なし
      * その他の値：なし

      ?> `$port`は `1-65535`のネットワークポート番号です。ポートが間違っている場合、送信が失敗します。

    * `string $data`

      * 機能：送信するデータの内容で、テキストまたはバイナリデータが可能です。
      * デフォルト値：なし
      * その他の値：なし

    * `int $serverSocket`

      * 機能：どのポートを使用してデータパケットを送信するかを指定します。対応するポートの`server_socket`ディスクリプタ【`$clientInfo`内で取得できる[onPacketイベント](/server/events?id=onpacket)】
      * デフォルト値：-1、現在リスンしているudpポートを表します
      * その他の値：なし

  * **戻り値**

    * `true`を返すと操作が成功し、`false`を返すと操作が失敗しました。

      ?> サーバーは複数の`UDP`ポートを同時にリッスンする場合があります。[ポート多重リッスン](/server/port)を参照し、このパラメータを使用してどのポートを使用してデータパケットを送信するかを指定することができます。

  * **注意**

  !> `UDP`ポートをリッスンしている必要があるため、`IPv4`アドレスにデータを送信することができます  
  `UDP6`ポートをリッスンしている必要があるため、`IPv6`アドレスにデータを送信することができます

  * **例**

```php
// IPアドレスが220.181.57.216のホストの9502ポートにhello world文字列を送信します。
$server->sendto('220.181.57.216', 9502, "hello world");
// IPv6サーバーにUDPパケットを送信します
$server->sendto('2600:3c00::f03c:91ff:fe73:e98f', 9501, "hello world");
```
## sendwait()

同步地向客户端发送数据。

```php
Swoole\Server->sendwait(int $fd, string $data): bool
```

  * **パラメーター**

    * `int $fd`

      * 機能：クライアントのファイル記述子を指定します
      * デフォルト値：なし
      * その他の値：なし

    * `string $data`

      * 機能：送信するデータ
      * デフォルト値：なし
      * その他の値：なし

  * **戻り値**

    * `true`を返すと、操作が成功しました。`false`を返すと、操作が失敗しました。

  * **ヒント**

    * いくつかの特殊なシナリオでは、`Server`はクライアントに連続してデータを送信する必要がありますが、`Server->send`データ送信インターフェースは純粋に非同期であり、大量のデータ送信はメモリ送信キューを詰まらせる可能性があります。

    * これを解決するために`Server->sendwait`を使用すると、`Server->sendwait`は接続が書き込み可能になるのを待ちます。データが送信されるまで戻りません。

  * **注意**

  !> 現在、`sendwait`は[SWOOLE_BASE](/learn?id=swoole_base)モードでのみ使用可能です。  
  `sendwait`はローカルまたは内部ネットワーク通信にのみ使用してください。外部ネットワーク接続では`sendwait`を使用しないでください。`enable_coroutine`=>true(デフォルトで有効)の場合もこの関数を使用しないでください。他のコルーチンをブロックする可能性があります。同期ブロッキングのサーバーのみが使用できます。
## sendMessage()

任意の`Worker`プロセスまたは[Taskプロセス](/learn?id=taskworkerプロセス)にメッセージを送信します。メインプロセスや管理プロセス以外のプロセスで呼び出すことができます。メッセージを受信したプロセスは`onPipeMessage`イベントをトリガーします。

```php
Swoole\Server->sendMessage(mixed $message, int $workerId): bool
```

  * **パラメータ**

    * `mixed $message`

      * 機能：送信するメッセージのデータ内容。長さに制限はありませんが、`8K`を超えると一時ファイルが作成されます。
      * デフォルト値：なし
      * その他の値：なし

    * `int $workerId`

      * 機能：対象プロセスの`ID`。範囲は[$worker_id](/server/properties?id=worker_id)を参照してください。
      * デフォルト値：なし
      * その他の値：なし

  * **ヒント**

    * `Worker`プロセス内で`sendMessage`を呼び出すと、[非同期IO](/learn?id=同期io非同期io)になり、メッセージはまずバッファに保存され、書き込み可能になった時にこのメッセージが[unixSocket](/learn?id=ipcとは)に送信されます。
    * [Taskプロセス](/learn?id=taskworkerプロセス)内で`sendMessage`を呼び出すと、デフォルトで[同期IO](/learn?id=同期io非同期io)になりますが、いくつかの場合は自動的に非同期IOに変換されます。[同期IOを非同期IOに変換する](/learn?id=同期io非同期io)を参照してください。
    * [Userプロセス](/server/methods?id=addprocess)内で`sendMessage`を呼び出すと、Taskと同様にデフォルトで同期ブロッキングです。[同期IOを非同期IOに変換する](/learn?id=同期io非同期io)を参照してください。

  * **注意**

  !> - `sendMessage()`が[非同期IO](/learn?id=同期io非同期io)の場合、対向プロセスが何らかの理由でデータを受信しない場合、`sendMessage()`を繰り返し呼び出さないでください。大量のメモリリソースを消費する可能性があります。対向が応答しない場合は呼び出しを一時停止することができます。
- `MacOS/FreeBSD`では`2K`を超えると一時ファイルが使用されます。
- [sendMessage](/server/methods?id=sendMessage)を使用するには、`onPipeMessage`イベントのコールバック関数を登録する必要があります。
- [task_ipc_mode](/server/setting?id=task_ipc_mode) = 3を設定すると特定のtaskプロセスにメッセージを送信することができません。

  * **例**

```php
$server = new Swoole\Server('0.0.0.0', 9501);

$server->set(array(
    'worker_num'      => 2,
    'task_worker_num' => 2,
));
$server->on('pipeMessage', function ($server, $src_worker_id, $data) {
    echo "#{$server->worker_id} message from #$src_worker_id: $data\n";
});
$server->on('task', function ($server, $task_id, $src_worker_id, $data) {
    var_dump($task_id, $src_worker_id, $data);
});
$server->on('finish', function ($server, $task_id, $data) {

});
$server->on('receive', function (Swoole\Server $server, $fd, $reactor_id, $data) {
    if (trim($data) == 'task') {
        $server->task("async task coming");
    } else {
        $worker_id = 1 - $server->worker_id;
        $server->sendMessage("hello task process", $worker_id);
    }
});

$server->start();
```
## exist()

`fd`に対応する接続の存在を検出します。

```php
Swoole\Server->exist(int $fd): bool
```

  * **Parameters**

    * `int $fd`

      * Function: ファイル記述子
      * Default: なし
      * 他の値: なし

  * **Return Value**

    * `true`を返すと存在することを示し、`false`を返すと存在しないことを示します

  * **Note**
  
    * このインターフェイスは共有メモリを基にして計算され、`IO`操作は行いません
## pause()

停止数据の受信。

```php
Swoole\Server->pause(int $fd): bool
```

  * **パラメータ**

    * `int $fd`

      * 機能：指定されたファイルディスクリプタ
      * デフォルト値：なし
      * その他の値：なし

  * **戻り値**

    * `true`を返すと操作は成功し、`false`を返すと操作は失敗したことを表します

  * **注意**

    * この関数を呼び出すと、接続が[EventLoop](/learn?id=什么是eventloop)から削除され、クライアントデータの受信が停止します。
    * この関数は送信キューの処理に影響しません
    * `SWOOLE_PROCESS`モードでのみ`pause`を呼び出すことができます。`pause`を呼び出した後、一部のデータはすでに`Worker`プロセスに到達している可能性があるため、引き続き[onReceive](/server/events?id=onreceive)イベントが発生するかもしれません.
## resume()

データ受信を再開します。`pause`メソッドとペアで使用します。

```php
Swoole\Server->resume(int $fd): bool
```

  * **パラメータ**

    * `int $fd`

      * 機能：ファイル記述子を指定します
      * デフォルト値：なし
      * その他の値：なし

  * **戻り値**

    * `true`を返すと操作が成功し、`false`を返すと操作が失敗します

  * **ヒント**

    * この関数を呼び出すと、接続が再度[EventLoop](/learn?id=什么是eventloop)に追加され、クライアントからのデータの受信が再開されます
## getCallback()

Serverの指定された名前のコールバック関数を取得します

```php
Swoole\Server->getCallback(string $event_name): \Closure|string|null|array
```

  * **パラメータ**

    * `string $event_name`

      * 機能：イベントの名前、「on」を加える必要はありません。大文字と小文字を区別しません
      * デフォルト値：なし
      * 他の値：[Event](/server/events) を参照してください

  * **戻り値**

    * 対応するコールバック関数が存在する場合、異なる[コールバック関数の設定方法](/learn?id=四种设置回调函数的方式)に応じて `Closure` / `string` / `array` が返されます
    * 対応するコールバック関数が存在しない場合、`null` が返されます
## getClientInfo()

接続情報を取得します。エイリアスは`Swoole\Server->connection_info()`です

```php
Swoole\Server->getClientInfo(int $fd, int $reactorId = -1, bool $ignoreError = false): false|array
```

  * **Parameters**

    * `int $fd`

      * Function: 指定ファイル記述子
      * デフォルト値: なし
      * 他の値: なし

    * `int $reactorId`

      * Function: 接続がある[Reactor](/learn?id=reactor糸)dataストレージ`ID`、現在は何の効果もなく、単にAPIの互換性を保持するために存在します
      * デフォルト値: -1
      * 他の値: なし

    * `bool $ignoreError`

      * Function: エラーを無視するかどうか、`true`に設定すると、接続が閉じていても接続情報を返します。`false`は接続が閉じたら`false`を返します
      * デフォルト値: false
      * 他の値: なし

  * **ヒント**

    * クライアント証明書

      * [onConnect](/server/events?id=onconnect)でトリガーされたプロセスでのみ証明書を取得できます
      * 形式は`x509`形式で、`openssl_x509_parse`関数を使用して証明書情報を取得できます

    * [dispatch_mode](/server/setting?id=dispatch_mode) = 1/3 を使用する場合、このデータパケット分配戦略はステートレスサービス向けに設計されており、接続が切断されると関連情報は直接メモリから削除されるため、`Server->getClientInfo`で関連接続情報を取得することはできません。

  * **Return Value**

    * 失敗した場合は`false`を返します
    * 成功した場合はクライアント情報が含まれた`array`を返します

```php
$fd_info = $server->getClientInfo($fd);
var_dump($fd_info);

array(15) {
  ["server_port"]=>
  int(9501)
  ["server_fd"]=>
  int(4)
  ["socket_fd"]=>
  int(25)
  ["socket_type"]=>
  int(1)
  ["remote_port"]=>
  int(39136)
  ["remote_ip"]=>
  string(9) "127.0.0.1"
  ["reactor_id"]=>
  int(1)
  ["connect_time"]=>
  int(1677322106)
  ["last_time"]=>
  int(1677322106)
  ["last_recv_time"]=>
  float(1677322106.901918)
  ["last_send_time"]=>
  float(0)
  ["last_dispatch_time"]=>
  float(0)
  ["close_errno"]=>
  int(0)
  ["recv_queued_bytes"]=>
  int(78)
  ["send_queued_bytes"]=>
  int(0)
}
```

パラメータ | 役割
---|---
server_port | サーバーのリッスンポート
server_fd | サーバーのfd
socket_fd | クライアントのfd
socket_type | ソケットの種類
remote_port | クライアントのポート
remote_ip | クライアントのIP
reactor_id | どのReactorスレッドからのものか
connect_time | クライアントがサーバーに接続した時間（秒）、マスタープロセスが設定
last_time | 最後にデータを受信した時間（秒）、マスタープロセスが設定
last_recv_time | 最後にデータを受信した時間（秒）、マスタープロセスが設定
last_send_time | 最後にデータを送信した時間（秒）、マスタープロセスが設定
last_dispatch_time | ワーカープロセスがデータを受信した時間
close_errno | 接続が閉じられたときのエラーコード、異常なクローズの場合、close_errnoの値はゼロ以外となり、Linuxのエラー情報リストを参照できます
recv_queued_bytes | 処理待ちのデータ量
send_queued_bytes | 送信待ちのデータ量
websocket_status | [オプション] WebSocket接続状態、サーバーがSwoole\WebSocket\Serverの場合、この情報が追加されます
uid | [オプション] ユーザーIDをバインドした場合は、この情報が追加されます
ssl_client_cert | [オプション] SSLトンネル暗号化を使用し、クライアントが証明書を設定している場合、この情報が追加されます
## getClientList()

現在の`Server`のすべてのクライアント接続を走査する`Server::getClientList`メソッドは共有メモリをベースにしており、`IOWait`は存在しません。走査速度は非常に速いです。また、`getClientList`は現在の`Worker`プロセスの`TCP`接続だけでなく、すべての`TCP`接続を返します。別名は`Swoole\Server->connection_list()`

```php
Swoole\Server->getClientList(int $start_fd = 0, int $pageSize = 10): false|array
```

  * **Parameters**

    * `int $start_fd`

      * 機能：開始`fd`を指定する
      * デフォルト値：0
      * その他の値：なし

    * `int $pgSize`

      * 機能：1ページあたりの取得件数。最大値は`100`
      * デフォルト値：10
      * その他の値：なし

  * **Return Value**

    * 成功した場合、取得した`$fd`を含む数値インデックス配列が返されます。配列は小さい順にソートされます。最後の`$fd`は新しい`start_fd`として再度試行されます
    * 失敗した場合は`false`が返されます

  * **Tips**

    * 接続を走査するために [Server::$connections](/server/properties?id=connections) イテレータを使用することをお勧めします
    * `getClientList`は`TCP`クライアントでのみ使用でき、`UDP`サーバーはクライアント情報を自分で保存する必要があります
    * [SWOOLE_BASE](/learn?id=swoole_base)モードでは現在のプロセスの接続のみを取得できます

  * **Example**
  
```php
$start_fd = 0;
while (true) {
  $conn_list = $server->getClientList($start_fd, 10);
  if ($conn_list === false || count($conn_list) === 0) {
      echo "finish\n";
      break;
  }
  $start_fd = end($conn_list);
  var_dump($conn_list);
  foreach ($conn_list as $fd) {
      $server->send($fd, "broadcast");
  }
}
```
## bind()

ユーザー定義の `UID` に接続をバインドし、[dispatch_mode](/server/setting?id=dispatch_mode)=5に設定して`hash`を使用して固定分配できます。これにより、特定の `UID` が接続された場合、すべての接続が同じ`Worker`プロセスに割り当てられるようになります。

```php
Swoole\Server->bind(int $fd, int $uid): bool
```

  * **パラメータ**

    * `int $fd`

      * 機能：指定された接続の `fd`
      * デフォルト値：なし
      * その他の値：なし

    * `int $uid`

      * 機能：バインドする`UID`、`0`でない数字でなければなりません
      * デフォルト値：なし
      * その他の値：`UID`は最大で`4294967295`を超えることはできず、最小で`-2147483648`より小さくすることはできません

  * **戻り値**

    * `true`を返すと操作は成功し、`false`を返すと操作は失敗します

  * **ヒント**

    * `$serv->getClientInfo($fd)` を使用して、接続がバインドされた`UID`の値を表示できます
    * デフォルトの[dispatch_mode](/server/setting?id=dispatch_mode)=2の設定では、`Server` は`socket fd`に基づいて接続データを異なる`Worker`プロセスに割り当てます。しかし、`fd`は安定していません。クライアントが切断されて再接続すると、`fd`が変化します。そのため、このクライアントのデータは別の`Worker`に割り当てられる可能性があります。`bind`を使用すると、ユーザー定義の`UID`に基づいて割り当てることができます。したがって、同じ`UID`を持つ`TCP`接続データは、断線再接続しても同じ`Worker`プロセスに割り当てられます。

    * 時系列の問題

      * クライアントがサーバーに接続し、複数のパケットを連続して送信すると、時系列の問題が発生する可能性があります。`bind`操作中に、後続のパケットが既に`dispatch`されている場合、これらのデータパケットは引き続き`fd`モデルで現在のプロセスに割り当てられます。新たに受信したデータパケットは`UID`モデルに基づいて割り当てられます。
      * したがって、`bind`メカニズムを使用する場合、ネットワーク通信プロトコルにはハンドシェイク手順が必要です。クライアントが正常に接続した後、まずハンドシェイクリクエストを送信し、その後クライアントは何もパケットを送信しないでください。サーバーが`bind`した後、応答したら、クライアントは新しいリクエストを送信できます。

    * 再バインド

      * 一部のケースでは、ビジネスロジックがユーザー接続を新しい`UID`に再バインドする必要があります。この場合、接続を切断し、新しい`TCP`接続を確立してハンドシェイクを行い、新しい`UID`にバインドできます。

    * 負の`UID`をバインドする

      * バインドされた`UID`が負の場合、下位層で`32ビット符号なし整数`に変換され、PHPレベルで`32ビット符号付き整数`に変換する必要があります。

```php
$uid = -10;
$server->bind($fd, $uid);
$bindUid = $server->connection_info($fd)['uid'];
$bindUid = $bindUid >> 31 ? (~($bindUid - 1) & 0xFFFFFFFF) * -1 : $bindUid;
var_dump($bindUid === $uid);
```

  * **注意**

!> -`dispatch_mode=5` が設定されている場合にのみ有効です  
-`UID`がバインドされていない場合はデフォルトで`fd`モデルを使用して割り当てられます  
-同一の接続は1度だけ`bind`でき、すでに`UID`がバインドされている場合に`bind`を再度呼び出すと`false`が返ります

  * **例**

```php
$serv = new Swoole\Server('0.0.0.0', 9501);

$serv->fdlist = [];

$serv->set([
    'worker_num' => 4,
    'dispatch_mode' => 5,   //uid dispatch
]);

$serv->on('connect', function ($serv, $fd, $reactor_id) {
    echo "{$fd} connect, worker:" . $serv->worker_id . PHP_EOL;
});

$serv->on('receive', function (Swoole\Server $serv, $fd, $reactor_id, $data) {
    $conn = $serv->connection_info($fd);
    print_r($conn);
    echo "worker_id: " . $serv->worker_id . PHP_EOL;
    if (empty($conn['uid'])) {
        $uid = $fd + 1;
        if ($serv->bind($fd, $uid)) {
            $serv->send($fd, "bind {$uid} success");
        }
    } else {
        if (!isset($serv->fdlist[$fd])) {
            $serv->fdlist[$fd] = $conn['uid'];
        }
        print_r($serv->fdlist);
        foreach ($serv->fdlist as $_fd => $uid) {
            $serv->send($_fd, "{$fd} say:" . $data);
        }
    }
});

$serv->on('close', function ($serv, $fd, $reactor_id) {
    echo "{$fd} Close". PHP_EOL;
    unset($serv->fdlist[$fd]);
});

$serv->start();
```
## stats()

現在の`Server`の活動`TCP`接続数、起動時間などの情報、`accept/close`(接続を確立/接続を閉じる)の総数などを取得します。

```php
Swoole\Server->stats(): array
```

  * **例**

```php
array(25) {
  ["start_time"]=>
  int(1677310656)
  ["connection_num"]=>
  int(1)
  ["abort_count"]=>
  int(0)
  ["accept_count"]=>
  int(1)
  ["close_count"]=>
  int(0)
  ["worker_num"]=>
  int(2)
  ["task_worker_num"]=>
  int(4)
  ["user_worker_num"]=>
  int(0)
  ["idle_worker_num"]=>
  int(1)
  ["dispatch_count"]=>
  int(1)
  ["request_count"]=>
  int(0)
  ["response_count"]=>
  int(1)
  ["total_recv_bytes"]=>
  int(78)
  ["total_send_bytes"]=>
  int(165)
  ["pipe_packet_msg_id"]=>
  int(3)
  ["session_round"]=>
  int(1)
  ["min_fd"]=>
  int(4)
  ["max_fd"]=>
  int(25)
  ["worker_request_count"]=>
  int(0)
  ["worker_response_count"]=>
  int(1)
  ["worker_dispatch_count"]=>
  int(1)
  ["task_idle_worker_num"]=>
  int(4)
  ["tasking_num"]=>
  int(0)
  ["coroutine_num"]=>
  int(1)
  ["coroutine_peek_num"]=>
  int(1)
  ["task_queue_num"]=>
  int(1)
  ["task_queue_bytes"]=>
  int(1)
}
```

パラメータ | 説明
---|---
start_time | サーバーの起動時刻
connection_num | 現在の接続数
abort_count | 拒否された接続の数
accept_count | 受け入れられた接続の数
close_count | 閉じられた接続の数
worker_num  | 開かれたworkerプロセスの数
task_worker_num  | 開かれたtask_workerプロセスの数【`v4.5.7`で利用可能】
user_worker_num  | 開かれたtask workerプロセスの数
idle_worker_num | アイドル状態のworkerプロセスの数
dispatch_count | ServerからWorkerへのパッケージ数【`v4.5.7`で利用可能、[SWOOLE_PROCESS](/learn?id=swoole_process)モードでのみ有効】
request_count | Serverが受信したリクエスト回数【onReceive、onMessage、onRequset、onPacketのみがリクエストとして計算される】
response_count | Serverが返した応答回数
total_recv_bytes | データ受信総数
total_send_bytes | データ送信総数
pipe_packet_msg_id | プロセス間通信ID
session_round | 開始セッションID
min_fd | 最小の接続fd
max_fd | 最大の接続fd
worker_request_count | 現在のWorkerプロセスが受信したリクエスト回数【worker_request_countがmax_requestを超えるとプロセスが終了します】
worker_response_count | 現在のWorkerプロセスの応答回数
worker_dispatch_count | masterプロセスが現在のWorkerプロセスにタスクをディスパッチした回数、dispatchが行われるたびにインクリメントされます
task_idle_worker_num | アイドル状態のtaskプロセスの数
tasking_num | 実行中のtaskプロセスの数
coroutine_num | 現在のコルーチン数【Coroutine用】、詳細情報は[このセクション](/coroutine/gdb)を参照してください
coroutine_peek_num | 全コルーチン数
task_queue_num | メッセージキュー内のタスクの数【Task用】
task_queue_bytes | メッセージキューのメモリ使用量（バイト数）【Task用】
## task()

`task_worker`プールに非同期タスクを投げます。この関数は非同期であり、実行が完了するとすぐに戻ります。`Worker`プロセスは新しいリクエストを処理できます。`Task`機能を使用するには、まず`task_worker_num`を設定し、[onTask](/server/events?id=ontask)および[onFinish](/server/events?id=onfinish)サーバーのイベントコールバックを設定する必要があります。

```php
Swoole\Server->task(mixed $data, int $dstWorkerId = -1, callable $finishCallback): int
```

  * **Parameters**

    * `mixed $data`

      * 機能：投げられるタスクデータ、シリアライズ可能なPHP変数である必要があります
      * デフォルト値：なし
      * 他の値：なし

    * `int $dstWorkerId`

      * 機能：どの[Taskワーカー](/learn?id=taskworker-proesses)にタスクを投げるかを指定できます、Taskプロセスの`ID`を渡すことができます。範囲は`[0, $server->setting['task_worker_num']-1]`です
      * デフォルト値：-1【デフォルトは`-1`でランダムに投げられます、下層で空いている[Taskワーカー](/learn?id=taskworker-proesses)が自動的に選択されます】
      * 他の値：`[0, $server->setting['task_worker_num']-1]`

    * `callable $finishCallback`

      * 機能：`finish`コールバック関数、タスクがコールバック関数を設定している場合、`Task`が結果を返すときに指定されたコールバック関数が直接実行され、`Server`の[onFinish](/server/events?id=onfinish)コールバックは実行されません。この機能は`Worker`プロセスでのみタスクの投げた時にトリガーされます
      * デフォルト値：`null`
      * 他の値：なし

  * **Return Value**

    * 呼び出しか成功した場合、整数`$task_id`が返されます。この値はこのタスクの`ID`を表します。`finish`コールバックがある場合は、[onFinish](/server/events?id=onfinish)コールバックで`$task_id`パラメータが渡されます
    * 呼び出しか失敗した場合、`false`が返されます。`$task_id`は`0`かもしれないため、失敗を確認するには`===`を使用する必要があります

  * **Tips**

    * この機能は遅いタスクを非同期で実行するために使用されます。たとえば、チャットルームサーバーでは、ブロードキャスト送信に使用できます。タスクが完了したときに、[taskプロセス](/learn?id=taskworkerプロセス)で`$serv->finish("finish")`を呼び出すと、`worker`プロセスにこのタスクが完了したことを通知します。もちろん、`Swoole\Server->finish`はオプションです。
    * `task`は[unixSocket](/learn?id=IPC)通信を使用し、すべてメモリ上で行われます。`IO`の負荷がありません。単一プロセスの読み書き性能は`100万/s`に達します。異なるプロセスは異なる`unixSocket`通信を使用し、複数コアを最大限に活用できます。
    * 対象とする[Taskワーカー](/learn?id=taskworker-proesses)が指定されていない場合、`task`メソッドは[Taskワーカー](/learn?id=taskworker-proesses)のビジー/アイドル状態を判断し、下層はアイドル状態にある[Taskワーカー](/learn?id=taskworker-proesses)にのみタスクを投げます。すべての[Taskワーカー](/learn?id=taskworker-proesses)がビジーの場合、下層は各プロセスに循環的にタスクを投げます。現在の並んでいるタスク数を取得するには[server->stats](/server/methods?id=stats)メソッドを使用できます。
    * 3番目のパラメータでは、直接[onFinish](/server/events?id=onfinish)関数を設定できます。タスクがコールバック関数を設定している場合、`Task`が結果を返すときに指定されたコールバック関数が直接実行され、`Server`の[onFinish](/server/events?id=onfinish)コールバックは実行されません。この機能は`Worker`プロセスでのみタスクを投げたときにトリガーされます

    ```php
    $server->task($data, -1, function (Swoole\Server $server, $task_id, $data) {
        echo "Task Callback: ";
        var_dump($task_id, $data);
    });
    ```

    * `$task_id`は`0-42` billionの整数で、現在のプロセス内で一意です
    * デフォルトでは`task`機能は有効になっていません。この機能を有効にするには、手動で`task_worker_num`を設定する必要があります
    * `TaskWorker`の数は[Server->set()](/server/methods?id=set)のパラメータで調整されます。たとえば、`task_worker_num => 64`と設定した場合、`64`個のプロセスが非同期タスクを受け取ります

  * **Configuration Parameters**

    * `Server->task/taskwait/finish` の`3`つのメソッドに渡される`$data`が`8K`を超えると、一時ファイルを使用して保存されます。一時ファイルの内容が[server->package_max_length](/server/setting?id=package_max_length)を超えると、下層で警告が発生します。この警告はデータの投げるには影響しませんが、大きすぎる`Task`にはパフォーマンスの問題が発生する可能性があります。

    ```shell
    WARN: task package is too big.
    ```

  * **One-way Tasks**

    * `Master`、`Manager`、`UserProcess`プロセスから投げられたタスクは一方向であり、`TaskWorker`プロセスでは`return`または`Server->finish()`メソッドを使用して結果データを返すことはできません。

  * **Caution**

  !> -`task`メソッドは[Taskプロセス](/learn?id=taskworker-proesses)で呼び出すことはできません  
-`task`を使用するには`Server`に必ず[onTask](/server/events?id=ontask)および[onFinish](/server/events?id=onfinish)コールバックを設定する必要があります。さもなければ`Server->start`は失敗します
- `task`操作の回数は[onTask](/server/events?id=ontask)の処理速度よりも少なくなければなりません。投入量が処理能力を超えると、`task`データがキャッシュ領域で詰まり、`Worker`プロセスがブロックされる可能性があります。`Worker`プロセスは新しいリクエストを受け付けられなくなります  
- [addProcess](/server/method?id=addProcess)を使用して追加されたユーザープロセスでは、`task`による単方向のタスク投入ができますが、結果データを返すことはできません。`Worker/Task`プロセスと通信するには[sendMessage](/server/methods?id=sendMessage)インターフェースを使用してください

  * **例**

```php
$server = new Swoole\Server("127.0.0.1", 9501, SWOOLE_BASE);

$server->set(array(
    'worker_num'      => 2,
    'task_worker_num' => 4,
));

$server->on('Receive', function (Swoole\Server $server, $fd, $reactor_id, $data) {
    echo "データを受信" . $data . "\n";
    $data    = trim($data);
    $server->task($data, -1, function (Swoole\Server $server, $task_id, $data) {
        echo "Taskのコールバック: ";
        var_dump($task_id, $data);
    });
    $task_id = $server->task($data, 0);
    $server->send($fd, "タスクを配布し、タスクIDは$task_id\n");
});

$server->on('Task', function (Swoole\Server $server, $task_id, $reactor_id, $data) {
    echo "Taskerプロセスがデータを受信しました";
    echo "#{$server->worker_id}\tonTask: [PID={$server->worker_pid}]: task_id=$task_id, data_len=" . strlen($data) . "." . PHP_EOL;
    $server->finish($data);
});

$server->on('Finish', function (Swoole\Server $server, $task_id, $data) {
    echo "Task#$task_id 完了, data_len=" . strlen($data) . PHP_EOL;
});

$server->on('workerStart', function ($server, $worker_id) {
    global $argv;
    if ($worker_id >= $server->setting['worker_num']) {
        swoole_set_process_name("php {$argv[0]}: task_worker");
    } else {
        swoole_set_process_name("php {$argv[0]}: worker");
    }
});

$server->start();
```
## taskwait()

`taskwait`関数は`task`メソッドと同じく、非同期のタスクを投入して[taskワーカープロセス](/learn?id=taskworkerプロセス)プールで実行するために使用されます。`task`と違い、`taskwait`は同期的に待ち、タスクが完了するかタイムアウトするまで待機します。`$result`はタスクの実行結果を示し、`$server->finish`関数から発行されます。もしタスクがタイムアウトした場合、ここでは`false`が返されます。

```php
Swoole\Server->taskwait(mixed $data, float $timeout = 0.5, int $dstWorkerId = -1): mixed
```

  * **パラメータ**

    * `mixed $data`

      * 機能：投入するタスクデータ、任意の型であり、文字列でない場合は自動的にシリアライズされます
      * デフォルト値：なし
      * その他：なし

    * `float $timeout`

      * 機能：タイムアウト時間、浮動小数点数、単位は秒で、最小サポートは`1ms`の粒度です。指定された時間内に[Taskプロセス](/learn?id=taskworkerプロセス)がデータを返さなかった場合、`taskwait`は`false`を返し、後続のタスク結果データを処理しません
      * デフォルト値：0.5
      * その他：なし

    * `int $dstWorkerId`

      * 機能：どの[Taskプロセス](/learn?id=taskworkerプロセス)に投入するか指定します。Taskプロセスの`ID`を渡すだけで、範囲は`[0, $server->setting['task_worker_num']-1]`です
      * デフォルト値：-1【デフォルトは`-1`でランダムに投入され、自動的に空いている[Taskプロセス](/learn?id=taskworkerプロセス)が選択されます】
      * その他：`[0, $server->setting['task_worker_num']-1]`

  *  **返り値**

      * `false`は投入失敗を示します
      * もし`onTask`イベント内で`finish`メソッドまたは`return`が実行された場合、`taskwait`は`onTask`で投入した結果を返します。

  * **ヒント**

    * **コルーチンモード**

      * `4.0.4`バージョンから`taskwait`メソッドは[コルーチンスケジューリング](/coroutine?id=コルーチンスケジューリング)をサポートし、コルーチン内で`Server->taskwait()`を呼び出すと自動的に[コルーチンスケジューリング](/coroutine?id=コルーチンスケジューリング)が行われ、ブロッキング待機されません。
      * [コルーチンスケジューラ](/coroutine?id=コルーチンスケジューラ)のおかげで、`taskwait`は並行呼び出しを実現できます。
      * `onTask`イベント内には`return`または`Server->finish`が1つだけ存在でき、余分な`return`または`Server->finish`の実行後にtask[1] has expired警告が表示されます。

    * **同期モード**

      * 同期ブロッキングモードでは、`taskwait`は[UnixSocket](/learn?id=IPC)通信と共有メモリを使用してデータを`Worker`プロセスに戻し、このプロセスは同期的にブロックされます。

    * **特例**

      * もし[onTask](/server/events?id=ontask)内に任意の[同期IO](/learn?id=同期io非同期io)操作が存在しない場合、低レイヤーではプロセス切り替えが2回しか発生せず、`IO`待機が発生しないため、この状況下では `taskwait` は非ブロッキングと見なすことができます。実際のテストでは[onTask](/server/events?id=ontask)内で`PHP`配列の読み書きのみを行い、10万回の`taskwait`操作を行った結果、合計時間はわずか1秒であり、平均消費時間は10マイクロ秒です。

  * **注意**

  !> -`Swoole\Server::finish`を使用せずに`taskwait`を使わないでください  
-`taskwait`メソッドは[taskプロセス](/learn?id=taskworkerプロセス)内で呼び出すことはできません
## taskWaitMulti()

複数の`task`非同期タスクを並行して実行し、このメソッドは[コルーチンスケジューリング](/coroutine?id=コルーチンスケジューリング)をサポートしていません。コルーチン環境では、後述の`taskCo`を使用する必要があります。

```php
Swoole\Server->taskWaitMulti(array $tasks, float $timeout = 0.5): false|array
```

  * **パラメータ**

    * `array $tasks`

      * 機能：数字でインデックスが付けられた配列でなければなりません。関連付けられたインデックス配列はサポートされていません。ベースラインは`$tasks`を反復処理し、それぞれのタスクを[Taskプロセス](/learn?id=taskworkerプロセス)に逐次送信します。
      * デフォルト値：なし
      * その他の値：なし

    * `float $timeout`

      * 機能：浮動小数点数で、単位は秒です。
      * デフォルト値：0.5秒
      * その他の値：なし

  * **戻り値**

    * タスクが完了するかタイムアウトした場合、結果の配列を返します。結果の配列では、各タスク結果の順序は`$tasks`と対応しています。例：`$tasks[2]`に対応する結果は`$result[2]`です。
    * 特定のタスクがタイムアウトした場合、他のタスクに影響を与えず、返された結果データにはタイムアウトしたタスクが含まれません。

  * **注意**

  !> -最大並行タスク数は`1024`を超えてはいけません。

  * **例**

```php
$tasks[] = mt_rand(1000, 9999); // タスク1
$tasks[] = mt_rand(1000, 9999); // タスク2
$tasks[] = mt_rand(1000, 9999); // タスク3
var_dump($tasks);

// すべてのタスクの結果を待機し、タイムアウトは10秒
$results = $server->taskWaitMulti($tasks, 10.0);

if (!isset($results[0])) {
    echo "タスク1はタイムアウトしました\n";
}
if (isset($results[1])) {
    echo "タスク2の実行結果は{$results[1]}\n";
}
if (isset($results[2])) {
    echo "タスク3の実行結果は{$results[2]}\n";
}
```
## taskCo()

[Coroutine scheduling](/coroutine?id=coroutine-scheduling) is used to concurrently execute `Tasks`, supporting the functionality of `taskWaitMulti` in a coroutine environment.

```php
Swoole\Server->taskCo(array $tasks, float $timeout = 0.5): false|array
```
  
* `$tasks` is an array of tasks. The underlying system will iterate through the array and deliver each element as a `task` to the `Task` process pool.
* `$timeout` is the timeout period, defaulting to `0.5` seconds. If all tasks are not completed within the specified time, the operation will be terminated immediately and the result will be returned.
* Upon completion or timeout of tasks, an array of results will be returned. The order of results in the array corresponds to each task in `$tasks`, for example, the result for `$tasks[2]` will be `$result[2]`.
* If a task fails to execute or times out, the corresponding item in the result array will be `false`. For example, if `$tasks[2]` fails, then the value of `$result[2]` will be `false`.

!> The maximum number of concurrent tasks must not exceed `1024`.  

  * **Scheduling Process**

    * Each task in the `$tasks` list will be randomly delivered to a `Task` worker process. After delivery, a `yield` will relinquish the current coroutine and set a timer for `$timeout` seconds.
    * In `onFinish`, the corresponding task results are collected and saved to the result array. It is then checked whether all tasks have returned results. If not, continue waiting. If yes, `resume` the corresponding coroutine's execution and clear the timeout timer.
    * If all tasks are not completed within the specified time, the timer will trigger first, causing the underlying system to clear the waiting state. Any incomplete task results will be marked as `false`, and the corresponding coroutine will be `resumed` immediately.

  * **Example**

```php
$server = new Swoole\Http\Server("127.0.0.1", 9502, SWOOLE_BASE);

$server->set([
    'worker_num'      => 1,
    'task_worker_num' => 2,
]);

$server->on('Task', function (Swoole\Server $serv, $task_id, $worker_id, $data) {
    echo "#{$serv->worker_id}\tonTask: worker_id={$worker_id}, task_id=$task_id\n";
    if ($serv->worker_id == 1) {
        sleep(1);
    }
    return $data;
});

$server->on('Request', function ($request, $response) use ($server) {
    $tasks[0] = "hello world";
    $tasks[1] = ['data' => 1234, 'code' => 200];
    $result   = $server->taskCo($tasks, 0.5);
    $response->end('Test End, Result: ' . var_export($result, true));
});

$server->start();
```
## finish()

[Task Workerプロセス](/learn?id=taskworkerプロセス)内で、投稿されたタスクが完了したことを`Worker`プロセスに通知するために使用されます。この関数は結果データを`Worker`プロセスに渡すことができます。

```php
Swoole\Server->finish(mixed $data): bool
```

  * **Parameters**

    * `mixed $data`

      * Functionality: タスク処理の結果の内容
      * Default Value: なし
      * Other Values: なし

  * **Return Value**

    * `true` を返すと操作は成功、`false` を返すと操作は失敗

  * **Tips**
    * `finish` 関数は複数回続けて呼び出すことができ、`Worker` プロセスは何度も[onFinish](/server/events?id=onfinish) イベントをトリガーします
    * [onTask](/server/events?id=ontask) コールバック関数内で `finish` 関数を呼び出した後、`return` データは引き続き[onFinish](/server/events?id=onfinish) イベントをトリガーします
    * `Server->finish` はオプションです。`Worker` プロセスがタスクの実行結果に興味を持たない場合、この関数を呼び出す必要はありません
    * [onTask](/server/events?id=ontask) コールバック関数内で `return` 文字列をすると、`finish` の呼び出しと同じです

  * **注意**

  !> `Server->finish` 関数を使用する場合は、`Server` に[onFinish](/server/events?id=onfinish) コールバック関数を設定する必要があります。この関数は [Task Workerプロセス](/learn?id=taskworkerプロセス) の[onTask](/server/events?id=ontask) コールバック内でのみ使用できます
## heartbeat()

[heartbeat_check_interval](/server/setting?id=heartbeat_check_interval)とは異なり、このメソッドはサーバーのすべての接続をアクティブに監視し、約定された時間を超過した接続を特定します。`if_close_connection`を指定した場合は、タイムアウトした接続を自動的に閉じます。指定しない場合は、接続の`fd`配列のみを返します。

```php
Swoole\Server->heartbeat(bool $ifCloseConnection = true): bool|array
```

  * **Parameters**

    * `bool $ifCloseConnection`

      * Function: タイムアウトした接続を閉じるかどうか
      * Default: true
      * Other Values: false

  * **Return Value**

    * 成功すると、閉じられた`$fd`を要素とする連続した配列が返されます
    * 失敗すると、`false`を返します

  * **Example**

```php
$closeFdArrary = $server->heartbeat();
```
## getLastError()

最新のエラーコードを取得します。ビジネスコードで、エラーコードのタイプに応じて異なるロジックを実行できます。

```php
Swoole\Server->getLastError(): int
```

  * **Return Value**

Error Code | Explanation
---|---
1001 | このエラーは、接続が`Server`側で閉じられ、コード中で既に`$server->close()`が呼び出されたが、それにもかかわらず`$server->send()`を使用してこの接続にデータを送信しようとした場合に発生します
1002 | このエラーは、接続が`Client`側で閉じられ、`Socket`が閉じられているためデータを対向先に送信できない場合に発生します
1003 | `close`が実行中であり、[onClose](/server/events?id=onclose)コールバック関数で`$server->send()`を使用してはいけない
1004 | 接続がすでに閉じられています
1005 | 接続が存在せず、渡された`$fd`が誤っている可能性があります
1007 | タイムアウトしたデータが受信されました。`TCP`接続を閉じた後、UNIXソケットのバッファ領域に一部のデータが残っている可能性があり、これらのデータは破棄されます
1008 | 送信バッファが一杯で`send`操作を実行できません。このエラーが発生すると、接続の対向先がデータを遅延して受信したため、送信バッファが一杯になりました
1202 | 送信されたデータが[server->buffer_output_size](/server/setting?id=buffer_output_size)を超えました
9007 | [dispatch_mode](/server/setting?id=dispatch_mode)=3を使用している場合にのみ発生し、現在使用可能なプロセスがないことを示します。`worker_num`プロセス数を増やすことができます
## getSocket()

このメソッドを呼び出すと、基礎となる`socket`ハンドルを取得できます。返されるオブジェクトは`sockets`リソースハンドルです。

```php
Swoole\Server->getSocket(): false|\Socket
```

!> このメソッドは、PHPの`sockets`拡張機能に依存しており、`Swoole`をコンパイルする際には`--enable-sockets`オプションを有効にする必要があります。

  * **ポートをリッスンする**

    * `listen`メソッドで追加したポートは、`Swoole\Server\Port`オブジェクトの`getSocket`メソッドを使用できます。

    ```php
    $port = $server->listen('127.0.0.1', 9502, SWOOLE_SOCK_TCP);
    $socket = $port->getSocket();
    ```

    * `socket_set_option`関数を使用して、より低レベルのいくつかの`socket`パラメータを設定できます。

    ```php
    $socket = $server->getSocket();
    if (!socket_set_option($socket, SOL_SOCKET, SO_REUSEADDR, 1)) {
        echo 'Unable to set option on socket: '. socket_strerror(socket_last_error()) . PHP_EOL;
    }
    ```

  * **マルチキャストをサポート**

    * `socket_set_option`を使用して`MCAST_JOIN_GROUP`パラメータを設定すると、`Socket`をマルチキャストに追加してネットワークマルチキャストデータパケットを受信することができます。

```php
$server = new Swoole\Server('0.0.0.0', 9905, SWOOLE_BASE, SWOOLE_SOCK_UDP);
$server->set(['worker_num' => 1]);
$socket = $server->getSocket();

$ret = socket_set_option(
    $socket,
    IPPROTO_IP,
    MCAST_JOIN_GROUP,
    array(
        'group' => '224.10.20.30', // マルチキャストアドレスを示す
        'interface' => 'eth0' // ネットワークインターフェースの名前を示す、数字または文字列の形式で、例えばeth0、wlan0
    )
);

if ($ret === false) {
    throw new RuntimeException('Unable to join multicast group');
}

$server->on('Packet', function (Swoole\Server $server, $data, $addr) {
    $server->sendto($addr['address'], $addr['port'], "Swoole: $data");
    var_dump($addr, strlen($data));
});

$server->start();
```  
## protect()

クライアント接続を保護された状態に設定し、ハートビートスレッドによって切断されないようにします。

```php
Swoole\Server->protect(int $fd, bool $is_protected = true): bool
```

  * **パラメータ**

    * `int $fd`

      * 機能: クライアント接続の `fd` を指定します
      * デフォルト値: なし
      * 他の値: なし

    * `bool $is_protected`

      * 機能: 設定する状態
      * デフォルト値: true 【保護状態を表します】
      * 他の値: false 【保護しないことを表します】

  * **戻り値**

    * `true` が返された場合は操作が成功し、`false` が返された場合は操作が失敗したことを示します
## confirm()

[enable_delay_receive](/server/setting?id=enable_delay_receive)と一緒に使用して、接続を確認します。クライアントが接続を確立した後、読み込み可能なイベントをリッスンせず、[onConnect](/server/events?id=onconnect)イベントコールバックをトリガーし、[onConnect](/server/events?id=onconnect)コールバックで`confirm`を実行すると、サーバーがクライアント接続からのデータを受信する準備が整います。

!> Swooleバージョン >= `v4.5.0` で利用可能

```php
Swoole\Server->confirm(int $fd): bool
```

  * **パラメーター**

    * `int $fd`

      * 意味：接続の一意な識別子
      * デフォルト値：なし
      * その他の値：なし

  * **戻り値**
  
    * 確認が成功したら`true`を返す
    * `$fd`に対応する接続が存在しない、閉じられている、または既にリッスンされている場合、確認に失敗したら`false`を返す

  * **用途**
  
    このメソッドは通常、サーバーを保護し、過剰なトラフィック攻撃を防ぐために使用されます。クライアント接続を受け取ったときに[onConnect](/server/events?id=onconnect)関数がトリガーされ、`IP`の出所を判断し、サーバーにデータを送信できるかどうかを判別することができます。

  * **例**
    
```php
// Serverオブジェクトを作成し、127.0.0.1:9501ポートをリッスン
$serv = new Swoole\Server("127.0.0.1", 9501); 
$serv->set([
    'enable_delay_receive' => true,
]);

//接続が入るイベントをリッスン
$serv->on('Connect', function ($serv, $fd) {  
    //この$fdをここでチェックして、問題がなければconfirm
    $serv->confirm($fd);
});

//データ受信イベントをリッスン
$serv->on('Receive', function ($serv, $fd, $reactor_id, $data) {
    $serv->send($fd, "Server: ".$data);
});

//接続が閉じられたイベントをリッスン
$serv->on('Close', function ($serv, $fd) {
    echo "Client: Close.\n";
});

//サーバーを起動
$serv->start(); 
``` 
## getWorkerId()

現在の`Worker`プロセスの`id`（プロセスの`PID`ではない）を取得します。これは[onWorkerStart](/server/events?id=onworkerstart)時の`$workerId`と同じです。

```php
Swoole\Server->getWorkerId(): int|false
```

!> Swooleのバージョン >= `v4.5.0RC1` で使用可能
```php
Swoole\Server->getWorkerPid(int $worker_id = -1): int|false
```

  * **パラメータ**

    * `int $worker_id`

      * 機能：指定したプロセスのPIDを取得します
      * デフォルト値：-1（-1は現在のプロセスを示します）
      * その他の値：なし

!> Swooleのバージョン >= `v4.5.0RC1` で利用可能
```
## getWorkerStatus()

`Worker`プロセスの状態を取得します

```php
Swoole\Server->getWorkerStatus(int $worker_id = -1): int|false
```

!> Swooleバージョン >= `v4.5.0RC1` で利用可能

  * **パラメータ**

    * `int $worker_id`

      * 機能：プロセスの状態を取得します
      * デフォルト値：-1、【-1は現在のプロセスを示します】
      * その他の値：なし

  * **返り値**
  
    * `Worker`プロセスの状態を返します。プロセスの状態値を参照してください
    * `Worker`プロセスでない場合やプロセスが存在しない場合は`false`を返します

  * **プロセスの状態値**

    定数 | 値 | 説明 | バージョン依存
    ---|---|---|---
    SWOOLE_WORKER_BUSY | 1 | ビジー | v4.5.0RC1
    SWOOLE_WORKER_IDLE | 2 | アイドル | v4.5.0RC1
    SWOOLE_WORKER_EXIT | 3 | [reload_async](/server/setting?id=reload_async)が有効な場合、同じworker_idには2つのプロセスがある可能性があります。新しいプロセスと古いプロセスがあり、古いプロセスはEXITの状態コードを取得します。 | v4.5.5
## getManagerPid()

```php
Swoole\Server->getManagerPid(): int
```

!> Swoole version >= `v4.5.0RC1` available
## getMasterPid()

```php
Swoole\Server->getMasterPid(): int
```

Swooleのバージョンが `v4.5.0RC1` 以上の場合に利用可能
## addCommand()

A custom command `command` is added

```php
Swoole\Server->addCommand(string $name, int $accepted_process_types, Callable $callback): bool
```

!> - Available since Swoole version >= `v4.8.0`         
  - This function can only be called before the server starts. If a command with the same name already exists, it will return `false`.

* **Parameters**

    * `string $name`

        * Description: Name of the `command`
        * Default: None
        * Others: None

    * `int $accepted_process_types`

      * Description: Types of processes that can accept the command. Multiple process types can be supported by using `|` to connect, for example, `SWOOLE_SERVER_COMMAND_MASTER | SWOOLE_SERVER_COMMAND_MANAGER`
      * Default: None
      * Others:
        * `SWOOLE_SERVER_COMMAND_MASTER` master process
        * `SWOOLE_SERVER_COMMAND_MANAGER` manager process
        * `SWOOLE_SERVER_COMMAND_EVENT_WORKER` worker process
        * `SWOOLE_SERVER_COMMAND_TASK_WORKER` task process

    * `callable $callback`

        * Description: Callback function with two parameters, one is the class of `Swoole\Server`, and the other is a user-defined variable. This variable is passed through the 4th parameter of `Swoole\Server::command()`.
        * Default: None
        * Others: None

* **Return Value**

    * Returns `true` if adding the custom command is successful, `false` if not.
## command()

定義されたカスタムコマンド`command`を呼び出します。

```php
Swoole\Server->command(string $name, int $process_id, int $process_type, mixed $data, bool $json_decode = true): false|string|array
```

!>Swooleバージョン`v4.8.0`以降で利用可能であり、`SWOOLE_PROCESS`および`SWOOLE_BASE`モードでは、この関数は`master`プロセスでのみ使用できます。

* **パラメータ**

    * `string $name`
    
        * 機能：`command`の名前
        * デフォルト値：なし
        * その他の値：なし

    * `int $process_id`
    
        * 機能：プロセスID
        * デフォルト値：なし
        * その他の値：なし

    * `int $process_type`
    
        * 機能：プロセスのリクエストタイプ、以下の他の値から1つだけ選択できます。
        * デフォルト値：なし
        * その他の値：
          * `SWOOLE_SERVER_COMMAND_MASTER` masterプロセス
          * `SWOOLE_SERVER_COMMAND_MANAGER` managerプロセス
          * `SWOOLE_SERVER_COMMAND_EVENT_WORKER` workerプロセス
          * `SWOOLE_SERVER_COMMAND_TASK_WORKER` taskプロセス

    * `mixed $data`
    
        * 機能：リクエストされたデータ、このデータはシリアライズできる必要があります。
        * デフォルト値：なし
        * その他の値：なし

    * `bool $json_decode`
    
        * 機能：`json_decode`を使用して解析するかどうか
        * デフォルト値：true
        * その他の値：false

* **使用例**

    ```php
    <?php
    use Swoole\Http\Server;
    use Swoole\Http\Request;
    use Swoole\Http\Response;

    $server = new Server('127.0.0.1', 9501, SWOOLE_BASE);
    $server->addCommand('test_getpid', SWOOLE_SERVER_COMMAND_MASTER | SWOOLE_SERVER_COMMAND_EVENT_WORKER,
        function ($server, $data) {
            var_dump($data);
            return json_encode(['pid' => posix_getpid()]);
        });

    $server->set([
        'log_file' => '/dev/null',
        'worker_num' => 2,
    ]);

    $server->on('start', function (Server $serv) {
        $result = $serv->command('test_getpid', 0, SWOOLE_SERVER_COMMAND_MASTER, ['type' => 'master']);
        Assert::eq($result['pid'], $serv->getMasterPid());
        $result = $serv->command('test_getpid', 1, SWOOLE_SERVER_COMMAND_EVENT_WORKER, ['type' => 'worker']);
        Assert::eq($result['pid'], $serv->getWorkerPid(1));
        $result = $serv->command('test_not_found', 1, SWOOLE_SERVER_COMMAND_EVENT_WORKER, ['type' => 'worker']);
        Assert::false($result);

        $serv->shutdown();
    });

    $server->on('request', function (Request $request, Response $response) {
    });

    $server->start();
    ```

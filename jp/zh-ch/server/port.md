# 複数ポートのリスニング

`Swoole\Server`は複数のポートをリスニングすることができ、各ポートには異なるプロトコル処理方法を設定することができます。例えば、80番ポートではHTTPプロトコルを処理し、9507番ポートではTCPプロトコルを処理します。`SSL/TLS`転送暗号化も特定のポートだけに有効にすることができます。

!> たとえば、メインサーバーがWebSocketまたはHTTPプロトコルの場合、新しくリッスンするTCPポート（`listen`の戻り値である`Swoole\Server\Port`オブジェクト、以下単にportとします）は、デフォルトでメインServerのプロトコル設定を引き継ぎます。新しいプロトコルを有効にするには、`port`オブジェクトの`set`メソッドと`on`メソッドを個別に呼び出す必要があります。
## 新しいポートをリッスンする

```php
//ポートオブジェクトを返す
$port1 = $server->listen("127.0.0.1", 9501, SWOOLE_SOCK_TCP);
$port2 = $server->listen("127.0.0.1", 9502, SWOOLE_SOCK_UDP);
$port3 = $server->listen("127.0.0.1", 9503, SWOOLE_SOCK_TCP | SWOOLE_SSL);
```
```php
//portオブジェクトのsetメソッドの呼び出し
$port1->set([
	'open_length_check' => true,
	'package_length_type' => 'N',
	'package_length_offset' => 0,
	'package_max_length' => 800000,
]);

$port3->set([
	'open_eof_split' => true,
	'package_eof' => "\r\n",
	'ssl_cert_file' => 'ssl.cert',
	'ssl_key_file' => 'ssl.key',
]);
```
## コールバック関数の設定

```php
//各ポートのコールバック関数を設定します
$port1->on('connect', function ($serv, $fd){
    echo "Client:Connect.\n";
});

$port1->on('receive', function ($serv, $fd, $reactor_id, $data) {
    $serv->send($fd, 'Swoole: '.$data);
    $serv->close($fd);
});

$port1->on('close', function ($serv, $fd) {
    echo "Client: Close.\n";
});

$port2->on('packet', function ($serv, $data, $addr) {
    var_dump($data, $addr);
});
```

!> `Swoole\Server\Port`の詳細については[こちら](/server/server_port)をクリックしてください。
## Http/WebSocket

`Swoole\Http\Server`および`Swoole\WebSocket\Server`は、継承されたサブクラスを使用して実装されているため、`Swoole\Server`インスタンスの`listen`メソッドを呼び出してHTTPサーバーまたはWebSocketサーバーを作成することはできません。

サーバーの主な機能が`RPC`であるが、単純なWeb管理インターフェイスを提供したい場合、`HTTP/WebSocket`サーバーを作成し、その後ネイティブTCPポートを`listen`することができます。
### サンプル

```php
$http_server = new Swoole\Http\Server('0.0.0.0',9998);
$http_server->set(['daemonize'=> false]);
$http_server->on('request', function ($request, $response) {
    $response->header("Content-Type", "text/html; charset=utf-8");
    $response->end("<h1>Hello Swoole. #".rand(1000, 9999)."</h1>");
});

//多监听一个TCP端口，对外开启TCP服务，并设置TCP服务器的回调
$tcp_server = $http_server->listen('0.0.0.0', 9999, SWOOLE_SOCK_TCP);
//新しくリッスンを追加したポート9999はメインサーバーの設定を引き継ぎますが、HTTPプロトコルとなります
//メインサーバーの設定を上書きするにはsetメソッドを使う必要があります
$tcp_server->set([]);
$tcp_server->on('receive', function ($server, $fd, $threadId, $data) {
    echo $data;
});

$http_server->start();
```

このコードを使用すると、外部からHTTPサービスを提供し、同時に外部からTCPサービスを提供するサーバーを構築できます。より具体的でエレガントなコードの組み合わせは、あなた自身で実装してください。
```php
$port1 = $server->listen("127.0.0.1", 9501, SWOOLE_SOCK_TCP);
$port1->set([
    'open_websocket_protocol' => true, // WebSocketプロトコルをサポートするようにこのポートを設定
]);
```

```php
$port1 = $server->listen("127.0.0.1", 9501, SWOOLE_SOCK_TCP);
$port1->set([
    'open_http_protocol' => false, // このポートでHTTPプロトコル機能を無効にする
]);
```

同様に、`open_http_protocol`、`open_http2_protocol`、`open_mqtt_protocol` パラメーターもあります。
## オプションパラメータ

- `port`リスニングポートが`set`メソッドを呼び出されていない場合、プロトコル処理オプションのリスニングポートは、親サーバーの関連設定を引き継ぎます。
- 親サーバーが`HTTP/WebSocket`サーバーである場合、プロトコルパラメータが設定されていない場合でも、リスニングポートは引き続き`HTTP`または`WebSocket`プロトコルに設定され、リスニングポートに設定された[onReceive](/server/events?id=onreceive)コールバックは実行されません。
- 親サーバーが`HTTP/WebSocket`サーバーである場合、リスニングポートが`set`を呼び出して設定された構成パラメータがあると、親サーバーのプロトコル設定がクリアされます。リスニングポートは`TCP`プロトコルになります。引き続き`HTTP/WebSocket`プロトコルを使用したい場合は、構成に`open_http_protocol => true` と `open_websocket_protocol => true` を追加する必要があります。

**`port`で`set`メソッドで設定できるパラメータは以下の通りです：**

- ソケットパラメータ：例`backlog`、`open_tcp_keepalive`、`open_tcp_nodelay`、`tcp_defer_accept`など
- プロトコル関連：例`open_length_check`、`open_eof_check`、`package_length_type`など
- SSL証明書関連：例`ssl_cert_file`、`ssl_key_file`など

詳細は[設定セクション](/server/setting)を参照してください。
## Optional Callbacks

`port`未调用`on`方法，设置回调函数的监听端口，默认使用主服务器的回调函数，`port`可以通过`on`方法设置的回调有：
### TCPサーバー

* onConnect
* onClose
* onReceive
### UDPサーバ

* onPacket
* onReceive
### HTTPサーバー

* onRequest
### WebSocket サーバー

* onMessage
* onOpen
* onHandshake

!> 異なるポートのリスナー関数でも、引き続き同じ `Worker` プロセススペースで実行されます。
```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9514, SWOOLE_BASE);

$tcp = $server->listen("0.0.0.0", 9515, SWOOLE_SOCK_TCP);
$tcp->set([]);

$server->on("open", function ($serv, $req) {
    echo "new WebSocket Client, fd={$req->fd}\n";
});

$server->on("message", function ($serv, $frame) {
    echo "receive from {$frame->fd}:{$frame->data},opcode:{$frame->opcode},fin:{$frame->finish}\n";
    $serv->push($frame->fd, "this is server OnMessage");
});

$tcp->on('receive', function ($server, $fd, $reactor_id, $data) {
    // 9514 ポートの接続を遍歴するだけでよいため、$server を使用しています
    $websocket = $server->ports[0];
    foreach ($websocket->connections as $_fd) {
        var_dump($_fd);
        if ($server->exist($_fd)) {
            $server->push($_fd, "this is server onReceive");
        }
    }
    $server->send($fd, 'receive: '.$data);
});

$server->start();
```  

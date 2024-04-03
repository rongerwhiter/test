# Swoole\WebSocket\Server

内蔵の`WebSocket`サーバーサポートを通じて、わずか数行の`PHP`コードで非同期IOのマルチプロセス`WebSocket`サーバーを書くことができます。

```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);

$server->on('open', function (Swoole\WebSocket\Server $server, $request) {
    echo "server: handshake success with fd{$request->fd}\n";
});

$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    echo "receive from {$frame->fd}:{$frame->data},opcode:{$frame->opcode},fin:{$frame->finish}\n";
    $server->push($frame->fd, "this is server");
});

$server->on('close', function ($server, $fd) {
    echo "client {$fd} closed\n";
});

$server->start();
```

* **クライアント**

  * `Chrome/Firefox/`および最新の`IE/Safari`などのブラウザには、`JS`言語の`WebSocket`クライアントが組み込まれています。
  * WeChat Mini Program開発フレームワークには、`WebSocket`クライアントが組み込まれています。
  * [非同期IO](/learn?id=同步io异步io)の`PHP`プログラムでは、[Swoole\Coroutine\Http](/coroutine_client/http_client)を`WebSocket`クライアントとして使用できます。
  * `Apache/PHP-FPM`や他の同期ブロッキングの`PHP`プログラムでは、`swoole/framework`が提供する[同期WebSocketクライアント](https://github.com/matyhtf/framework/blob/master/libs/Swoole/Client/WebSocket.php)を使用できます。
  * 非`WebSocket`クライアントは`WebSocket`サーバーと通信できません。

* **WebSocketクライアントの接続をどのように判別するか**
?> 通过使用 [下面的示例](/server/methods?id=getclientinfo) 获取连接信息，返回的数组中有一项为 [websocket_status](/websocket_server?id=连接状态)，根据此状态可以判断是否为`WebSocket`客户端。
```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);
$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    $client = $server->getClientInfo($frame->fd);
    // 或者 $client = $server->connection_info($frame->fd);
    if (isset($client['websocket_status'])) {
        echo "是websocket 连接";
    } else {
        echo "不是websocket 连接";
    }
});
```
## イベント

?> `WebSocket` サーバーは、[Swoole\Server](/server/methods) と [Swoole\Http\Server](/http_server) の基底クラスからのコールバック関数の受け取りに加えて、さらに `4` つのコールバック関数の設定を追加しました。以下の通りです：

* `onMessage` コールバック関数は必須です
* `onOpen`、`onHandShake`、そして (`Swoole5` で追加された) `onBeforeHandShakeResponse` コールバック関数は任意です
### onBeforeHandshakeResponse

!> Swoole version >= `v5.0.0` available

?> **Occurs before establishing a `WebSocket` connection. If you do not need to customize the handshake process, but still want to set some `http header` information to the response header, you can call this event.**

```php
onBeforeHandshakeResponse(Swoole\Http\Request $request, Swoole\Http\Response $response);
```
# Swoole\WebSocket\Server

内蔵の`WebSocket`サポートを使用して、数行の`PHP`コードで非同期IOのマルチプロセス`WebSocket`サーバーを作成できます。

```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);

$server->on('open', function (Swoole\WebSocket\Server $server, $request) {
    echo "server: handshake success with fd{$request->fd}\n";
});

$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    echo "receive from {$frame->fd}:{$frame->data},opcode:{$frame->opcode},fin:{$frame->finish}\n";
    $server->push($frame->fd, "this is server");
});

$server->on('close', function ($server, $fd) {
    echo "client {$fd} closed\n";
});

$server->start();
```

* **クライアント**

  * `Chrome/Firefox/`および高いバージョンの`IE/Safari`などのブラウザには、`JS`言語の`WebSocket`クライアントが組み込まれています。
  * WeChat小プログラム開発フレームワークには、`WebSocket`クライアントが組み込まれています。
  * [非同期IO](/learn?id=同期io非同期io) の`PHP`プログラムでは、[Swoole\Coroutine\Http](/coroutine_client/http_client) を`WebSocket`クライアントとして使用できます。
  * `Apache/PHP-FPM`やその他の同期ブロッキング`PHP`プログラムでは、`swoole/framework`が提供する[同期WebSocketクライアント](https://github.com/matyhtf/framework/blob/master/libs/Swoole/Client/WebSocket.php) を使用できます。
  * 非`WebSocket`クライアントは`WebSocket`サーバーと通信できません。

* **WebSocketクライアントを判断する方法**
```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);
$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    $client = $server->getClientInfo($frame->fd);
    // 或者 $client = $server->connection_info($frame->fd);
    if (isset($client['websocket_status'])) {
        echo "是websocket 连接";
    } else {
        echo "不是websocket 连接";
    }
});
```
### onHandShake

?> **ノート: **WebSocket接続が確立された後にハンドシェイクが行われます。WebSocketサーバーは自動的にハンドシェイクのプロセスを実行しますが、ユーザーが自身でハンドシェイク処理を行いたい場合は、`onHandShake`イベントコールバック関数を設定することができます。

```php
onHandShake(Swoole\Http\Request $request, Swoole\Http\Response $response);
```

* **ヒント**

  * `onHandShake`イベントコールバックはオプションです
  * `onHandShake`コールバック関数を設定すると、`onOpen`イベントが発生しなくなります。そのため、アプリケーションコードで`$server->defer`を使用して`onOpen`ロジックを呼び出す必要があります
  * `onHandShake`内で [response->status()](/http_server?id=status) を呼び出しステータスコードを`101`に設定し、[response->end()](/http_server?id=end) を呼び出して応答をする必要があります。そうしないとハンドシェイクは失敗します。
  * 組み込みのハンドシェイクプロトコルは`Sec-WebSocket-Version: 13`ですが、古いバージョンのブラウザではハンドシェイクを自前で実装する必要があります

* **注意**

!> `handshake`を自分で処理する必要がある場合にのみ、このコールバック関数を設定してください。ハンドシェイクプロセスを「カスタマイズ」する必要がない場合は、このコールバックを設定せず、デフォルトの`Swoole`ハンドシェイクを使用してください。以下は、"カスタム"ハンドシェイクイベントコールバックに必要な情報です：

```php
$server->on('handshake', function (\Swoole\Http\Request $request, \Swoole\Http\Response $response) {
    // print_r( $request->header );
    // if (特定のカスタム要件を満たさない場合は、endを返してfalseを返し、ハンドシェイクを失敗させる) {
    //    $response->end();
    //     return false;
    // }

    // WebSocketハンドシェイク接続アルゴリズムの検証
    $secWebSocketKey = $request->header['sec-websocket-key'];
```php
$patten = '#^[+/0-9A-Za-z]{21}[AQgw]==$#';
    if (0 === preg_match($patten, $secWebSocketKey) || 16 !== strlen(base64_decode($secWebSocketKey))) {
        $response->end();
        return false;
    }
    echo $request->header['sec-websocket-key'];
    $key = base64_encode(
        sha1(
            $request->header['sec-websocket-key'] . '258EAFA5-E914-47DA-95CA-C5AB0DC85B11',
            true
        )
    );

    $headers = [
        'Upgrade' => 'websocket',
        'Connection' => 'Upgrade',
        'Sec-WebSocket-Accept' => $key,
        'Sec-WebSocket-Version' => '13',
    ];

    // WebSocket connection to 'ws://127.0.0.1:9502/'
    // failed: Error during WebSocket handshake:
    // Response must not include 'Sec-WebSocket-Protocol' header if not present in request: websocket
    if (isset($request->header['sec-websocket-protocol'])) {
        $headers['Sec-WebSocket-Protocol'] = $request->header['sec-websocket-protocol'];
    }

    foreach ($headers as $key => $val) {
        $response->header($key, $val);
    }

    $response->status(101);
    $response->end();
});
```

!> 设置`onHandShake`回调函数后不会再触发`onOpen`事件，需要应用代码自行处理，可以使用`$server->defer`调用`onOpen`逻辑

```php
$server->on('handshake', function (\Swoole\Http\Request $request, \Swoole\Http\Response $response) {
    // 省略了握手内容
    $response->status(101);
    $response->end();

    global $server;
    $fd = $request->fd;
    $server->defer(function () use ($fd, $server)
    {
      echo "Client connected\n";
      $server->push($fd, "hello, welcome\n");
    });
});
```
### onOpen

?> **WebSocketクライアントがサーバーと接続し、ハンドシェイクを完了した後にこの関数がコールバックされます。**

```php
onOpen(Swoole\WebSocket\Server $server, Swoole\Http\Request $request);
```

* **ヒント**

    * `$request`は[HTTP](/http_server?id=httprequest)リクエストオブジェクトであり、クライアントからのハンドシェイクリクエスト情報が含まれています
    * `onOpen`イベント関数は、[push](/websocket_server?id=push)を使用してクライアントにデータを送信したり、[close](/server/methods?id=close)を呼び出して接続を切断したりすることができます
    * `onOpen`イベントコールバックはオプションです
### onMessage

?> **When the server receives data frames from the client, this function will be called.**

```php
onMessage(Swoole\WebSocket\Server $server, Swoole\WebSocket\Frame $frame)
```

* **Note**

  * `$frame` is an [Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe) object, containing information about the data frame sent by the client.
  * The `onMessage` callback must be set, otherwise the server will not start.
  * Sending a `ping` frame from the client will not trigger `onMessage`. The underlying layer will automatically reply with a `pong` packet. Alternatively, you can manually handle it by setting the [open_websocket_ping_frame
](/websocket_server?id=open_websocket_ping_frame) parameter.

!> `$frame->data` will be encoded in `UTF-8` if it is of text type. This is mandated by the WebSocket protocol.
### onRequest

`Swoole\WebSocket\Server`は[Swoole\Http\Server](/http_server)を継承しているため、`Http\Server`が提供するすべての`API`および設定が利用可能です。[Swoole\Http\Server](/http_server)セクションを参照してください。

- [onRequest](/http_server?id=on)コールバックが設定されている場合、`WebSocket\Server`は`HTTP`サーバーとしても機能します
- [onRequest](/http_server?id=on)コールバックが設定されていない場合、`WebSocket\Server`は`HTTP`リクエストを受け取った後に`HTTP 400`のエラーページを返します
- 接收した`HTTP`を介してすべての`WebSocket`をトリガーする場合は、スコープに注意する必要があります。手続き型の場合、「global」を使用して`Swoole\WebSocket\Server`へアクセスします。オブジェクト指向の場合は、`Swoole\WebSocket\Server`をメンバー属性に設定できます。
```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);
$server->on('open', function (Swoole\WebSocket\Server $server, $request) {
    echo "server: handshake success with fd{$request->fd}\n";
});
$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    echo "receive from {$frame->fd}:{$frame->data},opcode:{$frame->opcode},fin:{$frame->finish}\n";
    $server->push($frame->fd, "this is server");
});
$server->on('close', function ($server, $fd) {
    echo "client {$fd} closed\n";
});
$server->on('request', function (Swoole\Http\Request $request, Swoole\Http\Response $response) {
    global $server;//调用外部的server
    // $server->connections 遍历所有websocket连接用户的fd，给所有用户推送
    foreach ($server->connections as $fd) {
        // 需要先判断是否是正确的websocket连接，否则有可能会push失败
        if ($server->isEstablished($fd)) {
            $server->push($fd, $request->get['message']);
        }
    }
});
$server->start();
```
```php
クラスWebSocketServer
{
    public $server;

    public function __construct()
    {
        $this->server = new Swoole\WebSocket\Server("0.0.0.0", 9501);
        $this->server->on('open', function (Swoole\WebSocket\Server $server, $request) {
            echo "server: handshake success with fd{$request->fd}\n";
        });
        $this->server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
            echo "receive from {$frame->fd}:{$frame->data},opcode:{$frame->opcode},fin:{$frame->finish}\n";
            $server->push($frame->fd, "this is server");
        });
        $this->server->on('close', function ($ser, $fd) {
            echo "client {$fd} closed\n";
        });
        $this->server->on('request', function ($request, $response) {
            // 接收http请求从get获取message参数的值，给用户推送
            // $this->server->connections 遍历所有websocket连接用户的fd，给所有用户推送
            foreach ($this->server->connections as $fd) {
                // 需要先判断是否是正确的websocket连接，否则有可能会push失败
                if ($this->server->isEstablished($fd)) {
                    $this->server->push($fd, $request->get['message']);
                }
            }
        });
        $this->server->start();
    }
}

new WebSocketServer();
```
### onDisconnect

?> **Only triggered when a non-WebSocket connection is closed.**

!> Available in Swoole version >= `v4.7.0`

```php
onDisconnect(Swoole\WebSocket\Server $server, int $fd)
```

!> If `onDisconnect` event callback is set, it will be triggered for non-WebSocket requests or when `$response->close()` is called in [onRequest](/websocket_server?id=onrequest). However, normal termination in the [onRequest](/websocket_server?id=onrequest) event will not invoke the `onClose` or `onDisconnect` events.
## 方法

`Swoole\WebSocket\Server` は [Swoole\Server](/server/methods) のサブクラスであり、そのため`Server`のすべてのメソッドを呼び出すことができます。

`WebSocket`サーバーからクライアントにデータを送信する場合は、`Swoole\WebSocket\Server::push`メソッドを使用する必要があります。このメソッドは`WebSocket`プロトコルでのパッケージングを行います。一方、[Swoole\Server->send()](/server/methods?id=send) メソッドは元々の`TCP`送信インターフェースです。

[Swoole\WebSocket\Server->disconnect()](/websocket_server?id=disconnect) メソッドはサーバーから`WebSocket`接続をアクティブに閉じることができます。閉じる際に、[閉じるステータスコード](/websocket_server?id=websocket关闭帧状态码)（`WebSocket`プロトコルにより、使用可能なステータスコードは10進数の整数で`1000`または`4000-4999`の値を取ることができます）および閉じる理由（`utf-8`エンコード、バイト長が`125`を超えない文字列を使用）を指定することができます。指定がない場合のステータスコードは`1000`で、閉じる理由は空です。
# Swoole\WebSocket\Server

内蔵の`WebSocket`サーバーサポートを使用して、数行の`PHP`コードで非同期IOのマルチプロセス`WebSocket`サーバーを作成できます。

```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);

$server->on('open', function (Swoole\WebSocket\Server $server, $request) {
    echo "server: handshake success with fd{$request->fd}\n";
});

$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    echo "receive from {$frame->fd}:{$frame->data},opcode:{$frame->opcode},fin:{$frame->finish}\n";
    $server->push($frame->fd, "this is server");
});

$server->on('close', function ($server, $fd) {
    echo "client {$fd} closed\n";
});

$server->start();
```

* **クライアント**

  * `Chrome/Firefox/`などの高バージョンの`IE/Safari`ブラウザには`JS`言語の`WebSocket`クライアントが組み込まれています。
  * 微信小程序開発フレームワークには`WebSocket`クライアントが組み込まれています。
  * [非同期IO](/learn?id=同步io异步io) の`PHP`プログラムでは`[Swoole\Coroutine\Http](/coroutine_client/http_client)` を`WebSocket`クライアントとして使用できます。
  * `Apache/PHP-FPM`や他の同期ブロッキングの`PHP`プログラムでは、`[同步WebSocket客户端](https://github.com/matyhtf/framework/blob/master/libs/Swoole/Client/WebSocket.php)`を提供する`swoole/framework`を使用できます。
  * 非`WebSocket`クライアントは`WebSocket`サーバーと通信できません。

* **WebSocketクライアントであるかを判断する方法**
```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);
$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    $client = $server->getClientInfo($frame->fd);
    // 或者 $client = $server->connection_info($frame->fd);
    if (isset($client['websocket_status'])) {
        echo "是WebSocket 连接";
    } else {
        echo "不是WebSocket 连接";
    }
});
```
### onHandShake

?> **`WebSocket`接続確立後に行われるハンドシェイク。 `WebSocket`サーバーは自動的にハンドシェイクプロセスを行いますが、ユーザーが独自にハンドシェイク処理を行いたい場合は、`onHandShake`イベントコールバック関数を設定できます。**

```php
onHandShake(Swoole\Http\Request $request, Swoole\Http\Response $response);
```

* **ヒント**

  * `onHandShake`イベントコールバックはオプションです。
  * `onHandShake`コールバック関数を設定すると、`onOpen`イベントが起動しなくなります。アプリケーションコードで処理する必要があり、`$server->defer`を呼び出して`onOpen`ロジックを実行できます。
  * `onHandShake`内で [response->status()](/http_server?id=status) を呼び出してステータスコードを`101`に設定し、[response->end()](/http_server?id=end) を呼び出して応答する必要があります。そうしないと、ハンドシェイクが失敗します。
  * 組み込みのハンドシェイクプロトコルは`Sec-WebSocket-Version: 13`ですが、古いバージョンのブラウザでは独自でハンドシェイクする必要があります。

* **注意**

!> ハンドシェイクを独自に処理する必要がある場合にのみ、このコールバック関数を設定してください。 "カスタム"ハンドシェイクプロセスを必要としない場合は、このコールバックを設定せずに、`Swoole`のデフォルトハンドシェイクを使用してください。以下は、"カスタム" `handshake`イベントコールバック関数で必要な処理の例です：

```php
$server->on('handshake', function (\Swoole\Http\Request $request, \Swoole\Http\Response $response) {
    // print_r( $request->header );
    // if (特定のカスタム要件を満たさない場合、endを返し、falseを返してハンドシェイクを失敗させます) {
    //    $response->end();
    //     return false;
    // }

    // WebSocketハンドシェイク接続アルゴリズムの検証
    $secWebSocketKey = $request->header['sec-websocket-key'];
```php
$patten = '#^[+/0-9A-Za-z]{21}[AQgw]==$#';
if (0 === preg_match($patten, $secWebSocketKey) || 16 !== strlen(base64_decode($secWebSocketKey))) {
    $response->end();
    return false;
}
echo $request->header['sec-websocket-key'];
$key = base64_encode(
    sha1(
        $request->header['sec-websocket-key'] . '258EAFA5-E914-47DA-95CA-C5AB0DC85B11',
        true
    )
);

$headers = [
    'Upgrade' => 'websocket',
    'Connection' => 'Upgrade',
    'Sec-WebSocket-Accept' => $key,
    'Sec-WebSocket-Version' => '13',
];

// WebSocket connection to 'ws://127.0.0.1:9502/'
// failed: Error during WebSocket handshake:
// Response must not include 'Sec-WebSocket-Protocol' header if not present in request: websocket
if (isset($request->header['sec-websocket-protocol'])) {
    $headers['Sec-WebSocket-Protocol'] = $request->header['sec-websocket-protocol'];
}

foreach ($headers as $key => $val) {
    $response->header($key, $val);
}

$response->status(101);
$response->end();
});
```

!> `onHandShake`コールバック関数を設定すると`onOpen`イベントが発生しなくなります。この場合はアプリケーションコードで個別に処理する必要があり、`$server->defer`を使用して`onOpen`ロジックを呼び出すことができます。

```php
$server->on('handshake', function (\Swoole\Http\Request $request, \Swoole\Http\Response $response) {
    // Handshake content omitted
    $response->status(101);
    $response->end();

    global $server;
    $fd = $request->fd;
    $server->defer(function () use ($fd, $server)
    {
      echo "Client connected\n";
      $server->push($fd, "hello, welcome\n");
    });
});
```
### push

?> **データは`WebSocket`クライアントに接続してデータを送信します。データの最大長は`2M`を超えることはできません。**

```php
Swoole\WebSocket\Server->push(int $fd, \Swoole\WebSocket\Frame|string $data, int $opcode = WEBSOCKET_OPCODE_TEXT, bool $finish = true): bool

// v4.4.12のバージョンでは、flagsパラメータに変更されました
Swoole\WebSocket\Server->push(int $fd, \Swoole\WebSocket\Frame|string $data, int $opcode = WEBSOCKET_OPCODE_TEXT, int $flags = SWOOLE_WEBSOCKET_FLAG_FIN): bool
```

* **パラメータ** 

  * **`int $fd`**

    * **機能**：クライアント接続の`ID` 【指定された`$fd`が対応する`TCP`接続が`WebSocket`クライアントでない場合、送信に失敗します】
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`Swoole\WebSocket\Frame|string $data`**

    * **機能**：送信するデータの内容
    * **デフォルト値**：なし
    * **その他の値**：なし

  !> Swooleバージョン >= v4.2.0 では、渡される`$data`が [Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe) のオブジェクトである場合、その後続のパラメータは無視されます。

  * **`int $opcode`**

    * **機能**：送信するデータの形式を指定します 【デフォルトはテキストです。バイナリデータを送信する場合は`$opcode`パラメータを`WEBSOCKET_OPCODE_BINARY`に設定する必要があります】
    * **デフォルト値**：`WEBSOCKET_OPCODE_TEXT`
    * **その他の値**：`WEBSOCKET_OPCODE_BINARY`

  * **`bool $finish`**

    * **機能**：送信が完了したかどうか
    * **デフォルト値**：`true`
    * **その他の値**：`false`

* **戻り値**

  * 操作が成功した場合は`true`、失敗した場合は`false`
!> 自`v4.4.12`バージョンから、`finish`パラメータ（`bool`型）が`flags`パラメータ（`int`型）に変更され、`WebSocket`の圧縮をサポートしています。`finish`は`SWOOLE_WEBSOCKET_FLAG_FIN`という値に対応しており、元々の`bool`型の値は暗黙的に`int`型に変換されます。この変更は下位互換性があり影響はありません。 さらに、圧縮の`flag`は`SWOOLE_WEBSOCKET_FLAG_COMPRESS`です。

!> [BASE モード](/learn?id=base模式的限制：) では、プロセス間でのデータの `push` 送信はサポートされていません。
### 存在

?> **判断`WebSocket`クライアントが存在し、かつ`Active`状態であるかを確認します。**

!> `v4.3.0`以降、この`API`は接続の存在を確認するためだけに使用され、`isEstablished` を使って`WebSocket`接続かを判断してください。

```php
Swoole\WebSocket\Server->exist(int $fd): bool
```

* **戻り値**

  * 接続が存在し、かつ`WebSocket`のハンドシェイクが完了している場合は`true`を返します。
  * 接続が存在しないか、ハンドシェイクが完了していない場合は`false`を返します。
# Swoole\WebSocket\Server

内蔵の`WebSocket`サーバーサポートを使用して、3行の`PHP`コードで[非同期IO](/learn?id=同期io非同期io)のマルチプロセス`WebSocket`サーバーを作成できます。

```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);

$server->on('open', function (Swoole\WebSocket\Server $server, $request) {
    echo "server: handshake success with fd{$request->fd}\n";
});

$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    echo "receive from {$frame->fd}:{$frame->data},opcode:{$frame->opcode},fin:{$frame->finish}\n";
    $server->push($frame->fd, "this is server");
});

$server->on('close', function ($server, $fd) {
    echo "client {$fd} closed\n";
});

$server->start();
```

* **クライアント**

  * `Chrome/Firefox/`などの新しいバージョンの`IE/Safari`などのブラウザには、組み込みの`JS`言語の`WebSocket`クライアントがあります。
  * WeChatミニプログラム開発フレームワークには組み込みの`WebSocket`クライアントがあります。
  * [非同期IO](/learn?id=同期io非同期io)の`PHP`プログラムでは、 [Swoole\Coroutine\Http](/coroutine_client/http_client) を`WebSocket`クライアントとして使用できます。
  * `Apache/PHP-FPM`や他の同期ブロッキングの`PHP`プログラムでは、`swoole/framework`が提供する[同期WebSocketクライアント](https://github.com/matyhtf/framework/blob/master/libs/Swoole/Client/WebSocket.php)を使用できます。
  * 非`WebSocket`クライアントは`WebSocket`サーバーと通信できません。

* **WebSocketクライアントであるかどうかを判別する方法**
```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);
$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    $client = $server->getClientInfo($frame->fd);
    // 或者 $client = $server->connection_info($frame->fd);
    if (isset($client['websocket_status'])) {
        echo "是websocket 连接";
    } else {
        echo "不是websocket 连接";
    }
});
```
### onHandShake

**`WebSocket`接続の確立後にハンドシェイクが行われます。`WebSocket`サーバーは自動的にハンドシェイクのプロセスを行いますが、ユーザーが独自のハンドシェイク処理を行いたい場合は`onHandShake`イベントコールバック関数を設定してください。**

```php
onHandShake(Swoole\Http\Request $request, Swoole\Http\Response $response);
```

* **ヒント**

  * `onHandShake`イベントコールバックはオプションです
  * `onHandShake`コールバック関数を設定すると`onOpen`イベントは発生しなくなります。アプリケーションコードで個別に処理する必要があり、`$server->defer`を使用して`onOpen`ロジックを呼び出すことができます。
  * `onHandShake`内で[response->status()](/http_server?id=status)を呼び出してステータスコードを`101`に設定し、[response->end()](/http_server?id=end)を呼び出してレスポンスする必要があります。そうしないとハンドシェイクは失敗します。
  * 組み込みのハンドシェイクプロトコルは`Sec-WebSocket-Version: 13`ですが、古いバージョンのブラウザでは自分でハンドシェイクを実装する必要があります。

* **注意**

**`handshake`を独自に処理する必要がある場合にこのコールバック関数を設定してください。ハンドシェイクプロセスを「カスタマイズ」する必要がない場合は、このコールバックを設定せずに、`Swoole`のデフォルトのハンドシェイクを使用してください。以下は「カスタム」`handshake`イベントコールバック関数で必要な内容です：

```php
$server->on('handshake', function (\Swoole\Http\Request $request, \Swoole\Http\Response $response) {
    // print_r( $request->header );
    // if (I have some custom requirements that are not met, then return end output, return false and fail the handshake) {
    //    $response->end();
    //     return false;
    // }

    // WebSocket handshake connection algorithm verification
    $secWebSocketKey = $request->header['sec-websocket-key'];  
```php
$patten = '#^[+/0-9A-Za-z]{21}[AQgw]==$#';
    if (0 === preg_match($patten, $secWebSocketKey) || 16 !== strlen(base64_decode($secWebSocketKey))) {
        $response->end();
        return false;
    }
    echo $request->header['sec-websocket-key'];
    $key = base64_encode(
        sha1(
            $request->header['sec-websocket-key'] . '258EAFA5-E914-47DA-95CA-C5AB0DC85B11',
            true
        )
    );

    $headers = [
        'Upgrade' => 'websocket',
        'Connection' => 'Upgrade',
        'Sec-WebSocket-Accept' => $key,
        'Sec-WebSocket-Version' => '13',
    ];

    // WebSocket接続先：'ws://127.0.0.1:9502/'
    // 失敗：WebSocketハンドシェイク中にエラーが発生：
    // リクエストに't゙Sec-WebSocket-Protocol'ヘッダーが含まれない場合、レスポンスには含めてはいけません
    if (isset($request->header['sec-websocket-protocol'])) {
        $headers['Sec-WebSocket-Protocol'] = $request->header['sec-websocket-protocol'];
    }

    foreach ($headers as $key => $val) {
        $response->header($key, $val);
    }

    $response->status(101);
    $response->end();
});
```

!> 設定`onHandShake`コールバック関数後、`onOpen`イベントが発生しません。機能を処理するためにアプリケーションコードで自身で処理する必要があります。`$server->defer`を使用して`onOpen`ロジックを呼び出すことができます。

```php
$server->on('handshake', function (\Swoole\Http\Request $request, \Swoole\Http\Response $response) {
    // Handshakeの内容は省略しています
    $response->status(101);
    $response->end();

    global $server;
    $fd = $request->fd;
    $server->defer(function () use ($fd, $server)
    {
      echo "クライアントが接続されました\n";
      $server->push($fd, "こんにちは、ようこそ\n");
    });
});
```
### push

?> **Send data to a WebSocket client connection, with a maximum length of `2M`.**

```php
Swoole\WebSocket\Server->push(int $fd, \Swoole\WebSocket\Frame|string $data, int $opcode = WEBSOCKET_OPCODE_TEXT, bool $finish = true): bool

// From version 4.4.12, the `flags` parameter was changed
Swoole\WebSocket\Server->push(int $fd, \Swoole\WebSocket\Frame|string $data, int $opcode = WEBSOCKET_OPCODE_TEXT, int $flags = SWOOLE_WEBSOCKET_FLAG_FIN): bool
```

* **Parameters** 

  * **`int $fd`**

    * **Description**: The `ID` of the client connection 【If the specified `$fd` does not correspond to a WebSocket client TCP connection, the sending will fail】
    * **Default**: None
    * **Other values**: None

  * **`Swoole\WebSocket\Frame|string $data`**

    * **Description**: The data to be sent
    * **Default**: None
    * **Other values**: None

  !> For Swoole version >= v4.2.0, if `$data` passed is a [Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe) object, its subsequent parameters will be ignored

  * **`int $opcode`**

    * **Description**: Specifies the format of the data to be sent 【Default is text. For sending binary content, set the `$opcode` parameter to `WEBSOCKET_OPCODE_BINARY`】
    * **Default**: `WEBSOCKET_OPCODE_TEXT`
    * **Other values**: `WEBSOCKET_OPCODE_BINARY`

  * **`bool $finish`**

    * **Description**: Whether the sending is complete
    * **Default**: `true`
    * **Other values**: `false`

* **Return Value**

  * Returns `true` on success, `false` on failure
!> 自`v4.4.12`バージョンから、`finish`パラメータ（`bool`型）が`flags`パラメータ（`int`型）に変更され、`WebSocket`圧縮をサポートします。`finish`に対応する`SWOOLE_WEBSOCKET_FLAG_FIN`値は`1`であり、元の`bool`値は暗黙的に`int`型に変換されます。この変更は下位互換性があり、影響を受けません。 また、圧縮の`flag`は`SWOOLE_WEBSOCKET_FLAG_COMPRESS`です。

!> [BASE モード](/learn?id=base模式的限制：) で、プロセス間通信によるデータの  `push` 送信がサポートされていません。
### パック

?> **WebSocketメッセージをパックします。**

```php
Swoole\WebSocket\Server::pack(\Swoole\WebSocket\Frame|string $data $data, int $opcode = WEBSOCKET_OPCODE_TEXT, bool $finish = true, bool $mask = false): string

// バージョンv4.4.12では、flagsパラメータに変更されました
Swoole\WebSocket\Server::pack(\Swoole\WebSocket\Frame|string $data $data, int $opcode = WEBSOCKET_OPCODE_TEXT, int $flags = SWOOLE_WEBSOCKET_FLAG_FIN): string

Swoole\WebSocket\Frame::pack(\Swoole\WebSocket\Frame|string $data $data, int $opcode = WEBSOCKET_OPCODE_TEXT, int $flags = SWOOLE_WEBSOCKET_FLAG_FIN): string
```

* **パラメータ** 

  * **`Swoole\WebSocket\Frame|string $data $data`**

    * **機能**：メッセージの内容
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $opcode`**

    * **機能**：送信データの形式を指定します 【デフォルトはテキスト。バイナリ内容を送信する場合は、`$opcode`パラメータを`WEBSOCKET_OPCODE_BINARY`に設定する必要があります。】
    * **デフォルト値**：`WEBSOCKET_OPCODE_TEXT`
    * **その他の値**：`WEBSOCKET_OPCODE_BINARY`

  * **`bool $finish`**

    * **機能**：フレームが完了したかどうか
    * **デフォルト値**：なし
    * **その他の値**：なし

    !> `v4.4.12`以降、`finish`パラメータ（`bool`型）は`flags`パラメータ（`int`型）に変更されました。これにより`WebSocket`の圧縮をサポートし、`finish`は`SWOOLE_WEBSOCKET_FLAG_FIN`値`1`に対応します。元の`bool`型の値は暗黙的に`int`型に変換されます。この変更は下位互換性があります。

  * **`bool $mask`**

    * **機能**：マスクを設定するかどうか【`v4.4.12`ではこのパラメータが削除されました】
    * **デフォルト値**：なし
* **其它值**：無

* **返り値**

  * `WebSocket`データをパッケージ化された状態で返し、[send()](/server/methods?id=send)メソッドを用いて`Swoole\Server`の基本クラスに送信できます

* **例**

```php
$ws = new Swoole\Server('127.0.0.1', 9501 , SWOOLE_BASE);

$ws->set(array(
    'log_file' => '/dev/null'
));

$ws->on('WorkerStart', function (\Swoole\Server $serv) {
});

$ws->on('receive', function ($serv, $fd, $threadId, $data) {
    $sendData = "HTTP/1.1 101 Switching Protocols\r\n";
    $sendData .= "Upgrade: websocket\r\nConnection: Upgrade\r\nSec-WebSocket-Accept: IFpdKwYy9wdo4gTldFLHFh3xQE0=\r\n";
    $sendData .= "Sec-WebSocket-Version: 13\r\nServer: swoole-http-server\r\n\r\n";
    $sendData .= Swoole\WebSocket\Server::pack("hello world\n");
    $serv->send($fd, $sendData);
});

$ws->start();
```
### アンパック

?> **WebSocket データフレームを解析します。**

```php
Swoole\WebSocket\Server::unpack(string $data): Swoole\WebSocket\Frame|false;
```

* **パラメータ**

  * **`string $data`**

    * **機能**： メッセージの内容
    * **デフォルト値**： なし
    * **その他の値**： なし

* **戻り値**

  * 解析に失敗した場合は`false`を、成功した場合は[Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe)オブジェクトを返します
### 切断

?> **主动给`WebSocket`クライアントに閉じるフレームを送信し、接続を閉じます。**

!> Swooleバージョン >= `v4.0.3` で利用可能

```php
Swoole\WebSocket\Server->disconnect(int $fd, int $code = SWOOLE_WEBSOCKET_CLOSE_NORMAL, string $reason = ''): bool
```

* **引数** 

  * **`int $fd`**

    * **機能**：クライアント接続の`ID`【指定した`$fd`が対応する`TCP`接続が`WebSocket`クライアントでない場合、送信に失敗します】
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $code`**

    * **機能**：接続を閉じる状態コード【`RFC6455`に基づき、アプリケーションが接続を閉じる場合の状態コードは、`1000`または`4000-4999`の範囲を取ることができます】
    * **デフォルト値**：`SWOOLE_WEBSOCKET_CLOSE_NORMAL`
    * **その他の値**：なし

  * **`string $reason`**

    * **機能**：接続を閉じる理由【`utf-8`形式の文字列で、バイト長が`125`を超えない】
    * **デフォルト値**：なし
    * **その他の値**：なし

* **戻り値**

  * 送信が成功すると`true`を返し、送信が失敗した場合や状態コードが不正な場合は`false`を返します。
### isEstablished

?> **WebSocket クライアント接続が有効かどうかを確認します。**

?> この関数は `exist` メソッドとは異なり、`exist` メソッドは単に `TCP` 接続であるかどうかを判断しますが、完全なハンドシェイクが行われた `WebSocket` クライアントであるかどうかを判断することはできません。

```php
Swoole\WebSocket\Server->isEstablished(int $fd): bool
```

* **Parameters** 

  * **`int $fd`**

    * **説明**：クライアント接続の `ID`【指定された `$fd` が `TCP` 接続としてではなく `WebSocket` クライアントとして成立していない場合、失敗する可能性があります】
    * **デフォルト値**：なし
    * **その他の値**：なし

* **Return Value**

  * 有効な接続の場合は`true`を返し、そうでない場合は `false` を返します
## WebSocket データフレームクラス
### Swoole\WebSocket\Frame

?> In version `v4.2.0`, support for sending [Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe) objects from the server and client was added.  
In version `v4.4.12`, the `flags` property was added to support `WebSocket` compressed frames, and a new subclass [Swoole\WebSocket\CloseFrame](/websocket_server?id=swoolewebsocketcloseframe) was added.

A regular `frame` object has the following properties

Constants | Description 
---|--- 
fd | The `socket id` of the client, needed when using `$server->push` to push data   
data | Data content, can be text or binary data, can be identified by the value of `opcode`  
opcode | [Data frame type](/websocket_server?id=frame-type) of WebSocket, refer to the WebSocket protocol standard document   
finish | Indicates whether the data frame is complete, a WebSocket request may be divided into multiple data frames for transmission (automatic frame merging has been implemented at the underlying level, so there is no longer need to worry about receiving incomplete data frames)  

This class comes with [Swoole\WebSocket\Frame::pack()](/websocket_server?id=pack) and [Swoole\WebSocket\Frame::unpack()](/websocket_server?id=unpack), used for packing and unpacking `websocket` messages, with the same parameters as `Swoole\WebSocket\Server::pack()` and `Swoole\WebSocket\Server::unpack()` respectively.
### Swoole\WebSocket\CloseFrame

通常的「关闭帧 close frame」对象具有以下属性

常量 | 说明
---|---
opcode |  `WebSocket` 的[データフレームタイプ](/websocket_server?id=データフレームタイプ)，`WebSocket` プロトコルの標準文書を参照することができます
code | `WebSocket` の[クローズフレームステータスコード](/websocket_server?id=WebSocket断开状态码)，[WebSocketプロトコルで定義されたエラーコード](https://developer.mozilla.org/zh-CN/docs/Web/API/CloseEvent) を参照してください
reason | クローズ理由、明示的に指定されていない場合は空です

`close frame`をサーバーで受信する必要がある場合は、`$server->set`で[open_websocket_close_frame](/websocket_server?id=open_websocket_close_frame)パラメータを有効にする必要があります。
## Constants
### データフレームのタイプ

定数 | 対応値 | 説明
---|---|---
WEBSOCKET_OPCODE_TEXT | 0x1 | UTF-8テキスト文字データ
WEBSOCKET_OPCODE_BINARY | 0x2 | バイナリデータ
WEBSOCKET_OPCODE_CLOSE | 0x8 | クローズフレームタイプデータ
WEBSOCKET_OPCODE_PING | 0x9 | pingタイプデータ
WEBSOCKET_OPCODE_PONG | 0xa | pongタイプデータ
### 連続ステータス

定数 | 対応値 | 説明
---|---|---
WEBSOCKET_STATUS_CONNECTION | 1 | ハンドシェイクを待っている接続
WEBSOCKET_STATUS_HANDSHAKE | 2 | ハンドシェイク中
WEBSOCKET_STATUS_ACTIVE | 3 | ハンドシェイク成功、ブラウザからのデータフレームを待機中
WEBSOCKET_STATUS_CLOSING | 4 | 接続を閉じるハンドシェイクが進行中で、まもなく閉じます
```python
WEBSOCKET_CLOSE_NORMAL = 1000
WEBSOCKET_CLOSE_GOING_AWAY = 1001
WEBSOCKET_CLOSE_PROTOCOL_ERROR = 1002
WEBSOCKET_CLOSE_DATA_ERROR = 1003
WEBSOCKET_CLOSE_STATUS_ERROR = 1005
WEBSOCKET_CLOSE_ABNORMAL = 1006
WEBSOCKET_CLOSE_MESSAGE_ERROR = 1007
WEBSOCKET_CLOSE_POLICY_ERROR = 1008
WEBSOCKET_CLOSE_MESSAGE_TOO_BIG = 1009
WEBSOCKET_CLOSE_EXTENSION_MISSING = 1010
WEBSOCKET_CLOSE_SERVER_ERROR = 1011
WEBSOCKET_CLOSE_TLS = 1015
```

WebSocket关闭フレームのステータスコード

定数 | 値 | 説明
---|---|---
WEBSOCKET_CLOSE_NORMAL | 1000 | 正常な閉じる、接続が完了しました
WEBSOCKET_CLOSE_GOING_AWAY | 1001 | サーバーが切断しました
WEBSOCKET_CLOSE_PROTOCOL_ERROR | 1002 | プロトコルエラー、接続が中断されました
WEBSOCKET_CLOSE_DATA_ERROR | 1003 | データエラー、テキストデータが必要な場合にバイナリデータが受信されたなど
WEBSOCKET_CLOSE_STATUS_ERROR | 1005 | 予期しない状態コードを受信しなかったことを示します
WEBSOCKET_CLOSE_ABNORMAL | 1006 | 閉じるフレームが送信されていない
WEBSOCKET_CLOSE_MESSAGE_ERROR | 1007 | 形式に適合しないデータを受信したため、接続が切断されました（テキストメッセージに非UTF-8データが含まれるなど）
WEBSOCKET_CLOSE_POLICY_ERROR | 1008 | 約束に違反するデータを受信したため、接続が切断されました。これは1003および1009の状態コードが適していない場合に使用される汎用コードです
WEBSOCKET_CLOSE_MESSAGE_TOO_BIG | 1009 | 大きすぎるデータフレームを受信したため、接続が切断されました
WEBSOCKET_CLOSE_EXTENSION_MISSING | 1010 | クライアントが1つ以上の拡張をサーバーが処理しなかったため、クライアントが接続を切断しました
WEBSOCKET_CLOSE_SERVER_ERROR | 1011 | クライアントが予期しない状況に遭遇し、リクエストを完了できなかったため、サーバーが接続を切断しました
WEBSOCKET_CLOSE_TLS | 1015 | 予約されています。 接続がTLSハンドシェイクを完了できなかったため（たとえば、サーバー証明書を検証できない場合）、閉じられました
```
## オプション

?> `Swoole\WebSocket\Server`は`Server`クラスのサブクラスであり、[Swoole\WebSocker\Server::set()](/server/methods?id=set)メソッドを使用して設定オプションを渡し、特定のパラメータを設定できます。
### websocket_subprotocol

?> **Setting the `WebSocket` subprotocol.**

?> After setting this, the `Sec-WebSocket-Protocol: {$websocket_subprotocol}` will be added to the handshake response HTTP header. Please refer to relevant `RFC` documents for specific usage of the `WebSocket` protocol.

```php
$server->set([
    'websocket_subprotocol' => 'chat',
]);
```
### open_websocket_close_frame

?> **`onMessage`コールバックで`WebSocket`プロトコルのクローズフレーム（`opcode`が`0x08`のフレーム）を受信するように設定します。デフォルトは`false`です。**

?> このオプションを有効にすると、`Swoole\WebSocket\Server`の`onMessage`コールバックでクライアントまたはサーバーから送信されたクローズフレームを受信することができます。開発者は、クローズフレームに対して独自の処理を行うことができます。

```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);
$server->set(array("open_websocket_close_frame" => true));
$server->on('open', function (Swoole\WebSocket\Server $server, $request) {
});

$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    if ($frame->opcode == 0x08) {
        echo "Close frame received: Code {$frame->code} Reason {$frame->reason}\n";
    } else {
        echo "Message received: {$frame->data}\n";
    }
});

$server->on('close', function ($server, $fd) {
});

$server->start();
```
### open_websocket_ping_frame

?> **`onMessage`コールバックで`WebSocket`プロトコルの`Ping`フレーム（`opcode`が`0x09`のフレーム）を受信するようにするかどうかを設定します。デフォルトは`false`です。**

?> この設定を有効にすると、`Swoole\WebSocket\Server`の`onMessage`コールバックでクライアントまたはサーバーから送信された`Ping`フレームを受信でき、開発者はそれに対して処理を行うことができます。

!> Swooleのバージョン >= `v4.5.4` で利用可能です

```php
$server->set([
    'open_websocket_ping_frame' => true,
]);
```

!> 値を`false`に設定すると、内部で自動的に`Pong`フレームが返信されますが、`true`に設定した場合は開発者が自ら`Pong`フレームを返信する必要があります。

* **例**

```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);
$server->set(array("open_websocket_ping_frame" => true));
$server->on('open', function (Swoole\WebSocket\Server $server, $request) {
});

$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    if ($frame->opcode == 0x09) {
        echo "Ping frame received: Code {$frame->opcode}\n";
        // Pong フレームを返信
        $pongFrame = new Swoole\WebSocket\Frame;
        $pongFrame->opcode = WEBSOCKET_OPCODE_PONG;
        $server->push($frame->fd, $pongFrame);
    } else {
        echo "Message received: {$frame->data}\n";
    }
});

$server->on('close', function ($server, $fd) {
});

$server->start();
```
### open_websocket_pong_frame

?> **`WebSocket`プロトコルで`Pong`フレーム（`opcode`が`0x0A`のフレーム）を`onMessage`コールバックで受信するかどうかを有効にします。デフォルトは`false`です。**

?> 有効にすると、`Swoole\WebSocket\Server`の`onMessage`コールバックでクライアントまたはサーバーから送信された`Pong`フレームを受信できます。開発者はこれを自分で処理することができます。

!> Swooleのバージョン >= `v4.5.4` で利用可能

```php
$server->set([
    'open_websocket_pong_frame' => true,
]);
```

* **例**

```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);
$server->set(array("open_websocket_pong_frame" => true));
$server->on('open', function (Swoole\WebSocket\Server $server, $request) {
});

$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    if ($frame->opcode == 0xa) {
        echo "Pong frame received: Code {$frame->opcode}\n";
    } else {
        echo "Message received: {$frame->data}\n";
    }
});

$server->on('close', function ($server, $fd) {
});

$server->start();
```
### websocket_compression

**データ圧縮を有効にする**

`true`に設定すると、フレームを`zlib`で圧縮することが許可されます。具体的に圧縮されるかどうかは、クライアントが圧縮を処理できるかどうかによります（ハンドシェイク情報に基づく、`RFC-7692`を参照）。 実際に特定のフレームを圧縮するには、`flags`パラメータの`SWOOLE_WEBSOCKET_FLAG_COMPRESS`を組み合わせる必要があります。具体的な使用方法は[こちらを参照](/websocket_server?id=websocket帧压缩-（rfc-7692）)

!> Swooleバージョン >= `v4.4.12` で利用可能
## その他

!> 関連するサンプルコードは、[WebSocket ユニットテスト](https://github.com/swoole/swoole-src/tree/master/tests/swoole_websocket_server) で見つけることができます。
### WebSocket frame compression (RFC-7692)

?> First you need to configure `'websocket_compression' => true` to enable compression (will exchange compression support information with the peer during `WebSocket` handshake), then you can use the `flag SWOOLE_WEBSOCKET_FLAG_COMPRESS` to compress a specific frame.
#### サンプル

* **サーバー**

```php
use Swoole\WebSocket\Frame;
use Swoole\WebSocket\Server;

$server = new Server('127.0.0.1', 9501);
$server->set(['websocket_compression' => true]);
$server->on('message', function (Server $server, Frame $frame) {
    $server->push(
        $frame->fd,
        'Hello Swoole',
        SWOOLE_WEBSOCKET_OPCODE_TEXT,
        SWOOLE_WEBSOCKET_FLAG_FIN | SWOOLE_WEBSOCKET_FLAG_COMPRESS
    );
    // $server->push($frame->fd, $frame); // サーバーはクライアントのフレームオブジェクトをそのまま転送することもできます
});
$server->start();
```

* **クライアント**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $cli = new Client('127.0.0.1', 9501);
    $cli->set(['websocket_compression' => true]);
    $cli->upgrade('/');
    $cli->push(
        'Hello Swoole',
        SWOOLE_WEBSOCKET_OPCODE_TEXT,
        SWOOLE_WEBSOCKET_FLAG_FIN | SWOOLE_WEBSOCKET_FLAG_COMPRESS
    );
});
```
### Pingフレームの送信

?> WebSocketは長期の接続であるため、一定時間通信がないと接続が切断される可能性があります。その場合、ハートビートメカニズムが必要です。WebSocketプロトコルには、PingフレームとPongフレームが含まれており、定期的にPingフレームを送信して長期の接続を維持できます。
#### サンプル

* **サーバー**

```php
use Swoole\WebSocket\Frame;
use Swoole\WebSocket\Server;

$server = new Server('127.0.0.1', 9501);
$server->on('message', function (Server $server, Frame $frame) {
    $pingFrame = new Frame;
    $pingFrame->opcode = WEBSOCKET_OPCODE_PING;
    $server->push($frame->fd, $pingFrame);
});
$server->start();
```

* **クライアント**

```php
use Swoole\WebSocket\Frame;
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $cli = new Client('127.0.0.1', 9501);
    $cli->upgrade('/');
    $pingFrame = new Frame;
    $pingFrame->opcode = WEBSOCKET_OPCODE_PING;
    // PINGの送信
    $cli->push($pingFrame);
    
    // PONGの受信
    $pongFrame = $cli->recv();
    var_dump($pongFrame->opcode === WEBSOCKET_OPCODE_PONG);
});
```

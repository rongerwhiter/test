# WebSocketサーバー

## プログラムコード

以下のコードをwebsocketServer.phpに書き込んでください。

```php
// WebSocket Serverオブジェクトを作成し、0.0.0.0:9502ポートをリッスンします。
$ws = new Swoole\WebSocket\Server('0.0.0.0', 9502);

// WebSocket接続が開かれたときのイベントをリッスンします。
$ws->on('Open', function ($ws, $request) {
    $ws->push($request->fd, "hello, welcome\n");
});

// WebSocketメッセージイベントをリッスンします。
$ws->on('Message', function ($ws, $frame) {
    echo "Message: {$frame->data}\n";
    $ws->push($frame->fd, "server: {$frame->data}");
});

// WebSocket接続が閉じられたときのイベントをリッスンします。
$ws->on('Close', function ($ws, $fd) {
    echo "client-{$fd} is closed\n";
});

$ws->start();
```

* クライアントがサーバーにメッセージを送信すると、サーバーは`onMessage`イベントコールバックをトリガーします。
* サーバー側は`$server->push()`を使用して特定のクライアント（$fd識別子を使用）にメッセージを送信できます。

## プログラムの実行

```shell
php websocketServer.php
```

Chromeブラウザを使用してテストできます。使用するJavaScriptコードは次のとおりです：

```javascript
var wsServer = 'ws://127.0.0.1:9502';
var websocket = new WebSocket(wsServer);
websocket.onopen = function (evt) {
	console.log("Connected to WebSocket server.");
};

websocket.onclose = function (evt) {
	console.log("Disconnected");
};

websocket.onmessage = function (evt) {
	console.log('Retrieved data from server: ' + evt.data);
};

websocket.onerror = function (evt, e) {
	console.log('Error occured: ' + evt.data);
};
```

## Comet

WebSocketサーバーは、WebSocket機能だけでなく、実際にはHTTPロングポーリングも処理できます。[onRequest](/http_server?id=on)イベントリスナーを追加するだけで、CometソリューションのHTTPロングポーリングを実現できます。

!> 詳細な使用方法については、[Swoole\WebSocket](/websocket_server)を参照してください。

# WebSocketサーバー

?> 完全なコルーチン化されたWebSocketサーバーの実装であり、[Coroutine\Http\Server](/coroutine/http_server)から継承されています。WebSocketプロトコルをサポートしており、ここではその差異についてのみ述べます。

!> この章はv4.4.13以降で利用可能です。
```php
use Swoole\Http\Request;
use Swoole\Http\Response;
use Swoole\WebSocket\CloseFrame;
use Swoole\Coroutine\Http\Server;
use function Swoole\Coroutine\run;

run(function () {
    $server = new Server('127.0.0.1', 9502, false);
    $server->handle('/websocket', function (Request $request, Response $ws) {
        $ws->upgrade();
        while (true) {
            $frame = $ws->recv();
            if ($frame === '') {
                $ws->close();
                break;
            } else if ($frame === false) {
                echo 'errorCode: ' . swoole_last_error() . "\n";
                $ws->close();
                break;
            } else {
                if ($frame->data == 'close' || get_class($frame) === CloseFrame::class) {
                    $ws->close();
                    break;
                }
                $ws->push("Hello {$frame->data}!");
                $ws->push("How are you, {$frame->data}?");
            }
        }
    });

    $server->handle('/', function (Request $request, Response $response) {
        $response->end(<<<HTML
    <h1>Swoole WebSocket Server</h1>
    <script>
var wsServer = 'ws://127.0.0.1:9502/websocket';
var websocket = new WebSocket(wsServer);
websocket.onopen = function (evt) {
    console.log("Connected to WebSocket server.");
    websocket.send('hello');
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
</script>
HTML
        );
    });

    $server->start();
});
```
```php
use Swoole\Http\Request;
use Swoole\Http\Response;
use Swoole\WebSocket\CloseFrame;
use Swoole\Coroutine\Http\Server;
use function Swoole\Coroutine\run;

run(function () {
    $server = new Server('127.0.0.1', 9502, false);
    $server->handle('/websocket', function (Request $request, Response $ws) {
        $ws->upgrade();
        global $wsObjects;
        $objectId = spl_object_id($ws);
        $wsObjects[$objectId] = $ws;
        while (true) {
            $frame = $ws->recv();
            if ($frame === '') {
                unset($wsObjects[$objectId]);
                $ws->close();
                break;
            } else if ($frame === false) {
                echo 'エラーコード：' . swoole_last_error() . "\n";
                $ws->close();
                break;
            } else {
                if ($frame->data == 'close' || get_class($frame) === CloseFrame::class) {
                    unset($wsObjects[$objectId]);
                    $ws->close();
                    break;
                }
                foreach ($wsObjects as $obj) {
                    $obj->push("サーバー：{$frame->data}");
                }
            }
        }
    });
    $server->start();
});
```
## 処理フロー

* `$ws->upgrade()`：クライアントに`WebSocket`ハンドシェイクメッセージを送信
* `while(true)` ループを使用してメッセージの受信と送信を処理
* `$ws->recv()`：`WebSocket`メッセージフレームを受信
* `$ws->push()`：対向にデータフレームを送信
* `$ws->close()`：接続を閉じる

!> `$ws`は`Swoole\Http\Response`オブジェクトであり、各メソッドの使用方法については後述を参照してください。
Translate to Japanese:

## Methods
### upgrade()

Send a success message for the `WebSocket` handshake.

!> Do not use this method in servers with [asynchronous style](/http_server).

```php
Swoole\Http\Response->upgrade(): bool
```
### recv()

`WebSocket`メッセージを受信します。

!> このメソッドは[非同期スタイル](/http_server)のサーバーでは使用しないでください。`recv`メソッドを呼び出すと、現在のコルーチンが[サスペンド](/coroutine?id=協調スケジューリング)され、データが到着するのを待ってからコルーチンの実行が再開されます。

```php
Swoole\Http\Response->recv(float $timeout = 0): Swoole\WebSocket\Frame | false | string
```

* **戻り値**

  * メッセージを正常に受信した場合、`Swoole\WebSocket\Frame`オブジェクトが返されます。詳細は[Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe)を参照してください。
  * 失敗した場合は`false`が返され、エラーコードを取得するには [swoole_last_error()](/functions?id=swoole_last_error) を使用してください。
  * 接続が閉じられた場合は空の文字列が返されます。
  * 戻り値の処理については、[ブロードキャスト例](/coroutine/ws_server?id=群発例)を参照してください。
### push()

`WebSocket`データフレームを送信します。

!> このメソッドは[非同期スタイル](/http_server)のサーバーで使用しないでください。大きなデータパケットを送信する際には書き込み可能を監視する必要があり、それにより複数の[コルーチンスイッチ](/coroutine?id=コルーチンスケジューリング)が発生します。

```php
Swoole\Http\Response->push(string|object $data, int $opcode = WEBSOCKET_OPCODE_TEXT, bool $finish = true): bool
```

* **Parameters**

  !> `$data`に[Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe)オブジェクトが渡された場合、その後のパラメータは無視され、様々なフレームタイプを送信することができます。

  * **`string|object $data`**

    * **Description**: 送信するコンテンツ
    * **Default value**: None
    * **Other values**: None

  * **`int $opcode`**

    * **Description**: 送信するデータコンテンツの形式を指定します 【デフォルトはテキストです。バイナリコンテンツを送信する場合は`$opcode`パラメータを`WEBSOCKET_OPCODE_BINARY`に設定する必要があります】
    * **Default value**: `WEBSOCKET_OPCODE_TEXT`
    * **Other values**: `WEBSOCKET_OPCODE_BINARY`

  * **`bool $finish`**

    * **Description**: 送信が完了したかどうか
    * **Default value**: `true`
    * **Other values**: `false`
### close()

`WebSocket`接続を閉じます。

!> このメソッドは非同期スタイルのサーバーで使用しないでください。古いバージョン（v4.4.15以前）では`Warning`を誤って報告する可能性がありますが、無視して構いません。

```php
Swoole\Http\Response->close(): bool
```

# コルーチンHTTP/WebSocketクライアント

コルーチン版の`HTTP`クライアントは、純粋な`C`で記述されており、サードパーティの拡張ライブラリに依存せず、非常に高いパフォーマンスを持っています。

* `Http-Chunk`、`Keep-Alive`機能のサポート、`form-data`形式のサポート
* `HTTP`プロトコルバージョンは`HTTP/1.1`
* `WebSocket`クライアントにアップグレード可能
* `gzip`圧縮形式のサポートには`zlib`ライブラリが必要です
* クライアントは基本的な機能のみを実装しており、実際のプロジェクトでは [Saber](https://github.com/swlib/saber) を使用することをお勧めします。
## Properties
### errCode

エラーの状態コード。 `connect/send/recv/close` が失敗したりタイムアウトした場合、 `Swoole\Coroutine\Http\Client->errCode` の値が自動的に設定されます。

```php
Swoole\Coroutine\Http\Client->errCode: int
```

`errCode` の値は `Linux errno` に等しいです。 `socket_strerror` を使用してエラーコードをエラーメッセージに変換できます。

```php
// もしconnect refuseが起きた場合、エラーコードは111
// もしタイムアウトが起きた場合、エラーコードは110
echo socket_strerror($client->errCode);
```

!> 参考：[Linux エラーコードのリスト](/other/errno?id=linux)
### 身体

前回のリクエストのレスポンスボディを格納します。

```php
Swoole\Coroutine\Http\Client->body: string
```

  * **例**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $cli = new Client('httpbin.org', 80);
    $cli->get('/get');
    echo $cli->body;
    $cli->close();
});
```
### statusCode

HTTPステータスコード、例えば200、404などです。ステータスコードが負数の場合、接続に問題があることを示します。[詳細を見る](/coroutine_client/http_client?id=getstatuscode)

```php
Swoole\Coroutine\Http\Client->statusCode: int
```
This is a code block and it should not be translated.

```
print("Hello, World!")
```

This is another code block and it should also remain unchanged.

## 結果
### __construct()

コンストラクタ。

```php
Swoole\Coroutine\Http\Client::__construct(string $host, int $port, bool $ssl = false);
```

  * **パラメータ** 

    * **`string $host`**
      * **機能**：ターゲットサーバーのホストアドレス【IPアドレスまたはドメイン名を指定します。自動的にドメイン名解決が行われます。ローカルUNIXソケットの場合は、`unix://tmp/your_file.sock`のような形式で入力してください。ドメイン名を指定する場合は、`http://`または`https://`のプロトコルは必要ありません】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $port`**
      * **機能**：ターゲットサーバーのポート番号
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`bool $ssl`**
      * **機能**：`SSL/TLS`トンネル暗号化を有効にするかどうか。ターゲットサーバーがhttpsの場合は、`$ssl`パラメータを`true`に設定する必要があります。
      * **デフォルト値**：`false`
      * **その他の値**：なし

  * **例**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client('127.0.0.1', 80);
    $client->setHeaders([
        'Host' => 'localhost',
        'User-Agent' => 'Chrome/49.0.2587.3',
        'Accept' => 'text/html,application/xhtml+xml,application/xml',
        'Accept-Encoding' => 'gzip',
    ]);
    $client->set(['timeout' => 1]);
    $client->get('/index.php');
    echo $client->body;
    $client->close();
});
```
### set()

クライアントのパラメータを設定します。

```php
Swoole\Coroutine\Http\Client->set(array $options);
```

このメソッドは、`Swoole\Client->set` が受け入れるパラメータと完全に同じです。詳細は [Swoole\Client->set](/client?id=set) メソッドのドキュメントを参照してください。

`Swoole\Coroutine\Http\Client` には、`HTTP`および`WebSocket`クライアントを制御するためのいくつかの追加オプションがあります。
#### 额外选项
##### タイムアウト制御

`timeout`オプションを設定して、HTTPリクエストのタイムアウト検出を有効にします。単位は秒で、最小粒度はミリ秒までサポートされます。

```php
$http->set(['timeout' => 3.0]);
```

- 接続タイムアウトまたはサーバーが接続を閉じた場合、`statusCode`は`-1`に設定されます。
- 指定された時間内にサーバーが応答を返さない場合、リクエストはタイムアウトし、`statusCode`は`-2`に設定されます。
- リクエストがタイムアウトした後、下層では自動的に接続が切断されます。
- [クライアントのタイムアウト規則](/coroutine_client/init?id=超时规则)を参照してください。
##### keep_alive

`keep_alive`オプションを設定して、HTTP長い接続を有効または無効にします。

```php
$http->set(['keep_alive' => false]);
```
##### websocket_mask

> Due to RFC specifications, this configuration is enabled by default starting from v4.4.0, but it may lead to performance degradation. If the server does not require it, you can set it to false to disable.

WebSocketClient will enable or disable masking. By default, it is enabled. When enabled, it will use masking to transform the data sent by the WebSocket client.

```php
$http->set(['websocket_mask' => false]);
```
##### websocket_compression

> Requires `v4.4.12` or higher

When set to `true`, it **enables** zlib compression for frames, but whether compression actually occurs depends on whether the server can handle compression (determined by handshake information, see `RFC-7692`).

To actually compress a specific frame, you need to use the `SWOOLE_WEBSOCKET_FLAG_COMPRESS` flag parameter. For details on how to use it, refer to [this section](/websocket_server?id=websocket-frame-compression-rfc-7692).

```php
$http->set(['websocket_compression' => true]);
```
### setMethod()

設定リクエストのメソッド。このメソッドは現在のリクエストにのみ有効であり、リクエストを送信した後はメソッドの設定はすぐにクリアされます。

```php
Swoole\Coroutine\Http\Client->setMethod(string $method): void
```

  * **パラメータ** 

    * **`string $method`**
      * **機能**：メソッドを設定します 
      * **デフォルト値**：なし
      * **他の値**：なし

      !> `$method` は `HTTP` 標準に従ったメソッド名でなければならず、誤った `$method` を設定すると `HTTP` サーバーによってリクエストが拒否される可能性があります。

  * **例**

```php
$http->setMethod("PUT");
```
### setHeaders()

HTTPリクエストヘッダーを設定します。

```php
Swoole\Coroutine\Http\Client->setHeaders(array $headers): void
```

  * **パラメータ** 

    * **`array $headers`**
      * **機能**：リクエストヘッダーを設定します 【キーと値のペアに対応する配列でなければなりません。内部では`$key`:`$value`形式の`HTTP`標準ヘッダー形式に自動変換されます】
      * **デフォルト値**：なし
      * **その他の値**：なし

!> `setHeaders`で設定された`HTTP`ヘッダーは、`Coroutine\Http\Client`オブジェクトの寿命中の各リクエストで永続的に有効です。`setHeaders`を再度呼び出すと、前回の設定が上書きされます。
### setCookies()

`Cookie`を設定し、値は`urlencode`エンコードされます。元の情報を保持したい場合は、`setHeaders`を使用して`Cookie`ヘッダーを設定してください。

```php
Swoole\Coroutine\Http\Client->setCookies(array $cookies): void
```

  * **Parameters** 

    * **`array $cookies`**
      * **Description**: Set `COOKIE` 【必ずキーバリューペア形式の配列である必要があります】
      * **Default**: None
      * **Other values**: None

!> -`COOKIE`を設定すると、クライアントオブジェクトが有効な間は保存され続けます  
-サーバー側が設定した`COOKIE`は`cookies`配列にマージされ、現在の`HTTP`クライアントの`COOKIE`情報を`$client->cookies`プロパティで取得できます  
-`setCookies`メソッドを繰り返し呼び出すと、現在の`Cookies`状態が上書きされ、これにより以前にサーバー側で設定された`COOKIE`や以前に主動的に設定された`COOKIE`が破棄される可能性があります
### setData()

HTTPリクエストのボディ部分を設定します。

```php
Swoole\Coroutine\Http\Client->setData(string|array $data): void
```

  * **パラメータ** 

    * **`string|array $data`**
      * **説明**：リクエストのボディ部分を設定します
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **ヒント**

    * `$data`を設定した場合、かつ`$method`が設定されていない場合、内部で自動的にPOSTに設定されます
    * `$data`が配列であり`Content-Type`が`urlencoded`形式の場合、内部で自動的に`http_build_query`が実行されます
    * `addFile`または`addData`を使用して`form-data`形式が有効になった場合、`$data`が文字列である場合は無視されます（形式が異なるため）、しかし配列である場合は、内部で`form-data`形式に配列内のフィールドが追加されます
### addFile()

POSTファイルを追加します。

!> `addFile`を使用すると、`POST`リクエストの`Content-Type`が自動的に`form-data`に変更されます。`addFile`は`sendfile`をベースにしており、超大容量ファイルの非同期送信をサポートします。

```php
Swoole\Coroutine\Http\Client->addFile(string $path, string $name, string $mimeType = null, string $filename = null, int $offset = 0, int $length = 0): void
```

  * **Parameters**

    * **`string $path`**
      * **Description**: ファイルのパス【必須、存在しないファイルや空のファイルは指定できません】
      * **Default**: なし
      * **Other Values**: なし

    * **`string $name`**
      * **Description**: フォームの名前【必須、`FILES`パラメータ内のキーです】
      * **Default**: なし
      * **Other Values**: なし

    * **`string $mimeType`**
      * **Description**: ファイルの`MIME`タイプ【オプション、ファイルの拡張子に基づいて自動的に推測されます】
      * **Default**: なし
      * **Other Values**: なし

    * **`string $filename`**
      * **Description**: ファイル名【オプション】
      * **Default**: `basename($path)`
      * **Other Values**: なし

    * **`int $offset`**
      * **Description**: ファイルのオフセット【オプション、ファイルの一部からデータを転送することができます。これは断点継続をサポートするための機能です。】
      * **Default**: なし
      * **Other Values**: なし

    * **`int $length`**
      * **Description**: データのサイズを送信します【オプション】
      * **Default**: ファイル全体のサイズがデフォルト値です
      * **Other Values**: なし

  * **Example**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $cli = new Client('httpbin.org', 80);
    $cli->setHeaders([
        'Host' => 'httpbin.org'
    ]);
    $cli->set(['timeout' => -1]);
    $cli->addFile(__FILE__, 'file1', 'text/plain');
    $cli->post('/post', ['foo' => 'bar']);
    echo $cli->body;
    $cli->close();
});
```
### addData()

upload file content by string.

!> `addData` is available in version `v4.1.0` and above

```php
Swoole\Coroutine\Http\Client->addData(string $data, string $name, string $mimeType = null, string $filename = null): void
```

  * **Parameters** 

    * **`string $data`**
      * **Function**：data content【required parameter, the maximum length should not exceed [buffer_output_size](/server/setting?id=buffer_output_size)】
      * **Default**：none
      * **Other values**: none

    * **`string $name`**
      * **Function**：name of the form【required parameter, `key` in `$_FILES` parameters】
      * **Default**：none
      * **Other values**: none

    * **`string $mimeType`**
      * **Function**：file's `MIME` type【optional parameter, default is `application/octet-stream`】
      * **Default**：none
      * **Other values**: none

    * **`string $filename`**
      * **Function**：file name【optional parameter, default is `$name`】
      * **Default**：none
      * **Other values**: none

  * **Examples**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client('httpbin.org', 80);
    $client->setHeaders([
        'Host' => 'httpbin.org'
    ]);
    $client->set(['timeout' => -1]);
    $client->addData(Co::readFile(__FILE__), 'file1', 'text/plain');
    $client->post('/post', ['foo' => 'bar']);
    echo $client->body;
    $client->close();
});
``` 

上記のコードは、`addData`メソッドを使用してファイルをアップロードする方法を示しています。`addData`はファイルの内容を文字列として受け取り、ファイルの名前、MIMEタイプなどの情報を指定することができます。
### get()

GETリクエストを送信します。

```php
Swoole\Coroutine\Http\Client->get(string $path): void
```

  * **Parameters** 

    * **`string $path`**
      * **Description**：`URL`パスを設定します【例：`/index.html`、ここでは`http://domain`を渡すことはできません】
      * **Default**：None
      * **Other values**：None

  * **Example**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client('127.0.0.1', 80);
    $client->setHeaders([
        'Host' => 'localhost',
        'User-Agent' => 'Chrome/49.0.2587.3',
        'Accept' => 'text/html,application/xhtml+xml,application/xml',
        'Accept-Encoding' => 'gzip',
    ]);
    $client->get('/index.php');
    echo $client->body;
    $client->close();
});
```

!> `get`を使用すると、`setMethod`で設定したリクエスト方法が無視され、強制的に`GET`が使用されます。
### post()

POSTリクエストを送信します。

```php
Swoole\Coroutine\Http\Client->post(string $path, mixed $data): void
```

  * **パラメータ** 

    * **`string $path`**
      * **機能**：`URL`のパスを設定します【例：`/index.html`、ここに`http://domain`を渡すことはできません】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`mixed $data`**
      * **機能**：リクエストのボディデータ
      * **デフォルト値**：なし
      * **その他の値**：なし

      !> `$data`が配列の場合、下部では`x-www-form-urlencoded`形式の`POST`コンテンツに自動的にパッケージ化され、`Content-Type`が`application/x-www-form-urlencoded`に設定されます

  * **注意**

    !> `post`を使用すると、設定されたリクエストメソッド`setMethod`が無視され、強制的に`POST`が使用されます

  * **例**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client('127.0.0.1', 80);
    $client->post('/post.php', array('a' => '123', 'b' => '456'));
    echo $client->body;
    $client->close();
});
```  
### upgrade()

`WebSocket`へとアップグレードします。

```php
Swoole\Coroutine\Http\Client->upgrade(string $path): bool
```

  * **パラメータ** 

    * **`string $path`**
      * **機能**：URLのパスを設定します【例：`/`、ここには`http://domain`を渡すことはできません】
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **ヒント**

    * 一部の状況では、`upgrade`が`true`を返却しても、サーバーが`HTTP`ステータスコードを`101`に設定しておらず、代わりに`200`または`403`を返している場合があります。これはサーバーがハンドシェイクリクエストを拒否したことを意味します。
    * `WebSocket`のハンドシェイクが成功した後、`push`メソッドを使用してサーバーにメッセージを送信したり、`recv`を呼び出してメッセージを受信したりすることができます。
    * `upgrade`によって、[協力スケジューリング](/coroutine?id=协程调度)が1回発生します。

  * **例**

```php
use Swoole\Coroutine;
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client('127.0.0.1', 9501);
    $ret = $client->upgrade('/');
    if ($ret) {
        while(true) {
            $client->push('hello');
            var_dump($client->recv());
            Coroutine::sleep(0.1);
        }
    }
});
```
### push()

`WebSocket`サーバーにメッセージをプッシュします。

!> `push`メソッドは`upgrade`が成功した後にのみ実行できます。`push`メソッドは[コルーチンスケジューリング](/coroutine?id=协程调度)を発生させず、送信バッファに書き込んだ後、すぐに返ります。

```php
Swoole\Coroutine\Http\Client->push(mixed $data, int $opcode = WEBSOCKET_OPCODE_TEXT, bool $finish = true): bool
```

  * **Parameters** 

    * **`mixed $data`**
      * **Description**: The data content to send 【defaults to `UTF-8` text format, for other encoding or binary data, use `WEBSOCKET_OPCODE_BINARY`】
      * **Default value**: None
      * **Other values**: None

      !> Swoole version >= v4.2.0 `$data` can be a [Swoole\WebSocket\Frame](/websocket_server?id=swoolewebsocketframe) object, supporting various frame types

    * **`int $opcode`**
      * **Description**: Operation type
      * **Default value**: `WEBSOCKET_OPCODE_TEXT`
      * **Other values**: None

      !> `$opcode` must be a valid `WebSocket OPCode`, otherwise, it will fail and print the error message `opcode max 10`

    * **`int|bool $finish`**
      * **Description**: Operation type
      * **Default value**: `SWOOLE_WEBSOCKET_FLAG_FIN`
      * **Other values**: None

      !> Since version `v4.4.12`, the `finish` parameter (boolean type) has been changed to `flags` (integer type) to support `WebSocket` compression. `finish` corresponds to the value `SWOOLE_WEBSOCKET_FLAG_FIN` with `1`, the original boolean value will be implicitly converted to an integer. This change is backward compatible. In addition, the compression `flag` is `SWOOLE_WEBSOCKET_FLAG_COMPRESS`.

  * **Return Value**

    * If sending is successful, it returns `true`.
    * If the connection does not exist, is closed, or the `WebSocket` is not completed, it returns `false`.

  * **Error Codes**

Error Code | Description
---|---
8502 | Invalid OPCode
8503 | Not connected to the server or connection is closed
8504 | Handshake failed
```php
use Swoole\Coroutine;
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client('127.0.0.1', 9501);
    $ret = $client->upgrade('/');
    if ($ret) {
        while(true) {
            $client->push('hello');
            var_dump($client->recv());
            Coroutine::sleep(0.1);
        }
    }
});
```
### download()

ファイルをHTTPでダウンロードします。

!> downloadメソッドは、データを受信した後にそれをディスクに書き込みます。つまり、HTTP Bodyをメモリ内で結合するのではなく、ファイルごとに書き込むことで、少量のメモリで非常に大きなファイルのダウンロードが可能です。

```php
Swoole\Coroutine\Http\Client->download(string $path, string $filename,  int $offset = 0): bool
```

  * **Parameters** 

    * **`string $path`**
      * **Purpose**: `URL`のパスを設定します
      * **Default**: None
      * **Other values**: None

    * **`string $filename`**
      * **Purpose**: ダウンロードコンテンツを書き込むファイルのパスを指定します【自動的に`downloadFile`属性に書き込まれます】
      * **Default**: None
      * **Other values**: None

    * **`int $offset`**
      * **Purpose**: 書き込むファイルのオフセットを指定します【これは一時停止と再開をサポートするためのオプションで、`HTTP`ヘッダー`Range:bytes=$offset`と一緒に使用することができます】
      * **Default**: None
      * **Other values**: None

      !> `$offset`が`0`の場合、ファイルが既に存在する場合、そのファイルは自動的にクリアされます

  * **Return value**

    * 成功した場合は`true`を返します
    * ファイルのオープンに失敗したか、`fseek()`でファイルのシークに失敗した場合は`false`を返します

  * **Example**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $host = 'cdn.jsdelivr.net';
    $client = new Client($host, 443, true);
    $client->set(['timeout' => -1]);
    $client->setHeaders([
        'Host' => $host,
        'User-Agent' => 'Chrome/49.0.2587.3',
        'Accept' => '*',
        'Accept-Encoding' => 'gzip'
    ]);
    $client->download('/gh/swoole/swoole-src/mascot.png', __DIR__ . '/logo.png');
});
```
### getCookies()

`HTTP`レスポンスの`cookie`コンテンツを取得します。

```php
Swoole\Coroutine\Http\Client->getCookies(): array|false
```

!> Cookie情報は`urldecode`によってデコードされます。元のCookie情報を取得するには、下記の方法で解析してください。
```php
var_dump($client->set_cookie_headers);
````

上記のコードは、変数`$client`に含まれる`set_cookie_headers`プロパティの値を表示するものです。これは、重複するCookieヘッダー情報を含む場合やオリジナルのCookieヘッダー情報を取得する場合に役立ちます。
### getHeaders()

`HTTP`応答のヘッダー情報を返します。

```php
Swoole\Coroutine\Http\Client->getHeaders(): array|false
```
### getStatusCode()

`HTTP`レスポンスのステータスコードを取得します。

```php
Swoole\Coroutine\Http\Client->getStatusCode(): int|false
```

  * **ヒント**

    * **ステータスコードが負数の場合、接続に問題が発生していることを示します。**

ステータスコード | v4.2.10 以降の対応する定数 | 説明
---|---|---
-1 | SWOOLE_HTTP_CLIENT_ESTATUS_CONNECT_FAILED | 接続がタイムアウトしました。サーバーがポートをリッスンしていないか、ネットワークに問題が発生しています。具体的なネットワークエラーコードを$errCodeで取得できます
-2 | SWOOLE_HTTP_CLIENT_ESTATUS_REQUEST_TIMEOUT | リクエストがタイムアウトしました。サーバーは指定されたtimeout時間内に応答せずにいます。
-3 | SWOOLE_HTTP_CLIENT_ESTATUS_SERVER_RESET | クライアントリクエストを送信した後、サーバーが強制的に接続を切断しました。
-4 | SWOOLE_HTTP_CLIENT_ESTATUS_SEND_FAILED | クライアントが送信に失敗しました（この定数はSwooleバージョン>= `v4.5.9`で利用可能ですが、それ以前のバージョンではステータスコードを使用してください）
### getBody()

`HTTP`レスポンスのボディコンテンツを取得します。

```php
Swoole\Coroutine\Http\Client->getBody(): string|false
```
### close()

接続を閉じます。

```php
Swoole\Coroutine\Http\Client->close(): bool
```

!> `close`した後、`get`、`post`などのメソッドを再度要求すると、Swooleが自動的にサーバーに再接続します。
### execute()

`HTTP` request method at a lower level, requires calling interfaces such as [setMethod](/coroutine_client/http_client?id=setmethod) and [setData](/coroutine_client/http_client?id=setdata) in the code to set the request method and data.

```php
Swoole\Coroutine\Http\Client->execute(string $path): bool
```

* **Example**

```php
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $httpClient = new Client('httpbin.org', 80);
    $httpClient->setMethod('POST');
    $httpClient->setData('swoole');
    $status = $httpClient->execute('/post');
    var_dump($status);
    var_dump($httpClient->getBody());
});
```
## 関数

`Coroutine\Http\Client` の使用を簡単にするために、3つの関数が追加されました:

!> Swooleバージョン >= `v4.6.4` で利用可能
```php
function request(string $url, string $method, $data = null, array $options = null, array $headers = null, array $cookies = null)
```

この関数は、指定されたリクエスト方法でリクエストを送信します。
```php
function post(string $url, $data, array $options = null, array $headers = null, array $cookies = null)
```
```php
function get(string $url, array $options = null, array $headers = null, array $cookies = null)
```  
### 使用例

```php
use function Swoole\Coroutine\go;
use function Swoole\Coroutine\run;
use function Swoole\Coroutine\Http\get;
use function Swoole\Coroutine\Http\post;
use function Swoole\Coroutine\Http\request;

run(function () {
    go(function () {
        $data = get('http://httpbin.org/get?hello=world');
        $body = json_decode($data->getBody());
        assert($body->headers->Host === 'httpbin.org');
        assert($body->args->hello === 'world');
    });
    go(function () {
        $random_data = base64_encode(random_bytes(128));
        $data = post('http://httpbin.org/post?hello=world', ['random_data' => $random_data]);
        $body = json_decode($data->getBody());
        assert($body->headers->Host === 'httpbin.org');
        assert($body->args->hello === 'world');
        assert($body->form->random_data === $random_data);
    });
});
```

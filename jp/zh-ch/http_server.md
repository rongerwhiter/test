# Http\Server

?> `Http\Server`は[Server](/server/init)から継承されているため、`Server`が提供するすべての`API`や設定を使用でき、プロセスモデルも同じです。[Server](/server/init)セクションを参照してください。

組み込みの`HTTP`サーバーサポートを使用して、わずか数行のコードで高並列、高性能、[非同期IO](/learn?id=同期io异步io)のマルチプロセス`HTTP`サーバーを記述できます。

```php
$http = new Swoole\Http\Server("127.0.0.1", 9501);
$http->on('request', function ($request, $response) {
    $response->end("<h1>Hello Swoole. #".rand(1000, 9999)."</h1>");
});
$http->start();
```

`Apache bench`ツールを使用して負荷テストを行うと、通常のPCマシンである`Inter Core-I5 4コア + 8Gメモリ`上で、`Http\Server`はほぼ`11万QPS`に達することができます。

これは`PHP-FPM`、`Golang`、`Node.js`の組み込みの`Http`サーバーをはるかに上回り、性能はほぼ`Nginx`の静的ファイル処理に匹敵します。

```shell
ab -c 200 -n 200000 -k http://127.0.0.1:9501/
```

* **HTTP2プロトコルの使用**

  * `SSL`を使用した`HTTP2`プロトコルを使用するには、`openssl`をインストールする必要があり、高いバージョンの`openssl`が`TLS1.2`、`ALPN`、`NPN`をサポートしている必要があります。
  * コンパイル時には[--enable-http2](/environment?id=编译选项)を有効にする必要があります。
  * Swoole5以降は、デフォルトでhttp2プロトコルが有効になっています。

```shell
./configure --enable-openssl --enable-http2
```

`HTTP`サーバーの[open_http2_protocol](/http_server?id=open_http2_protocol)を`true`に設定します。

```php
$server = new Swoole\Http\Server("127.0.0.1", 9501, SWOOLE_PROCESS, SWOOLE_SOCK_TCP | SWOOLE_SSL);
$server->set([
    'ssl_cert_file' => $ssl_dir . '/ssl.crt',
    'ssl_key_file' => $ssl_dir . '/ssl.key',
    'open_http2_protocol' => true,
]);
```

* **Nginx + Swooleの設定**

!> `Http\Server`は`HTTP`プロトコルのサポートが完全ではないため、アプリケーションサーバーとしてのみ使用し、ダイナミックリクエストを処理するために使用し、フロントエンドに`Nginx`をプロキシとして追加することをお勧めします。

```nginx
server {
    listen 80;
    server_name swoole.test;

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_pass http://127.0.0.1:9501;
    }
}
```

?> クライアントの実際の`IP`を取得するには、`$request->header['x-real-ip']`を読み取ることができます。
## Methods

```python
def compute_average(lst):
    total = sum(lst)
    n = len(lst)
    average = total / n
    return average
```

このコードは、リスト内の数値の平均値を計算する関数です。
### on()

?> **Register event callback functions.**

?> Same as [server-side callbacks](/server/events), the difference is:

  * `Http\Server->on` does not accept setting of [onConnect](/server/events?id=onconnect)/[onReceive](/server/events?id=onreceive) callbacks
  * `Http\Server->on` additionally accepts a new event type `onRequest`, the client-requested request is executed in the `Request` event

```php
$http_server->on('request', function(\Swoole\Http\Request $request, \Swoole\Http\Response $response) {
     $response->end("<h1>hello swoole</h1>");
});
```

After receiving a complete HTTP request, this function will be called back. The callback function has `2` parameters:

* [Swoole\Http\Request](/http_server?id=httpRequest), `HTTP` request information object, including `header/get/post/cookie` and other relevant information.
* [Swoole\Http\Response](/http_server?id=httpResponse), `HTTP` response object that supports `cookie/header/status` and other `HTTP` operations.

!> When the [onRequest](/http_server?id=on) callback function returns, the `$request` and `$response` objects will be destroyed.
### start()

?> **Start the HTTP server**

?> After starting, it begins to listen on the port and receive new `HTTP` requests.

```php
Swoole\Http\Server->start();
```
## Swoole\Http\Request

`HTTP`リクエストオブジェクトは、`GET`、`POST`、`COOKIE`、`Header`などの`HTTP`クライアントリクエストに関連する情報を保存します。

!> `Http\Request`オブジェクトを参照する際に`&`を使用しないでください
### ヘッダー

?> **`HTTP`リクエストのヘッダー情報。配列の形式であり、すべての`key`は小文字で表されます。**

```php
Swoole\Http\Request->header: array
```

* **例**

```php
echo $request->header['host'];
echo $request->header['accept-language'];
```
### サーバー

**`HTTP`リクエストに関連するサーバーの情報。**

`PHP`の`$_SERVER`配列に相当します。リクエストのメソッド、`URL`パス、クライアントの`IP`などの情報が含まれます。

```php
Swoole\Http\Request->server: array
```

配列の`key`はすべて小文字であり、`PHP`の`$_SERVER`配列と一致しています。

* **例**

```php
echo $request->server['request_time'];
```

key | 説明
---|---
query_string | `GET`パラメーターのリクエスト、例：`id=1&cid=2`。`GET`パラメーターがない場合、この項目は存在しません。
request_method | リクエストメソッド、`GET/POST`など
request_uri | `GET`パラメーターのないアクセス先アドレス、例：`/favicon.ico`
path_info | `request_uri`と同じ
request_time | `request_time`は`Worker`で設定されます。[SWOOLE_PROCESS](/learn?id=swoole_process)モードでは`dispatch`プロセスが存在するため、実際のパケットの受信時間とは乖離する可能性があります。特にサーバーの処理能力を超えるリクエストがある場合、`request_time`は実際のパケット受信時間よりも大幅に遅れる場合があります。正確な受信時間を取得するには、`$server->getClientInfo`メソッドを使用して`last_time`を取得できます。
request_time_float | リクエストの開始時間のタイムスタンプ、マイクロ秒単位、`float`型、例：`1576220199.2725`
server_protocol | サーバーのプロトコルバージョン、`HTTP`は：`HTTP/1.0`または`HTTP/1.1`、`HTTP2`は：`HTTP/2`
server_port | サーバーがリッスンしているポート
remote_port | クライアントのポート
remote_addr | クライアントの`IP`アドレス
master_time | 最後に通信した時間
### get

?> **`HTTP`リクエストの`GET`パラメータは、`PHP`の`$_GET`と同等で、配列形式です。**

```php
Swoole\Http\Request->get: array
```

* **例**

```php
// 例：index.php?hello=123
echo $request->get['hello'];
// すべてのGETパラメータを取得
var_dump($request->get);
```

* **注意**

!> `HASH`攻撃を防ぐために、`GET`パラメータの最大数は`128`を超えないようにしてください。
### 投稿

?> **POSTリクエストのパラメータは、配列形式であります**

```php
Swoole\Http\Request->post: array
```

* **例**

```php
echo $request->post['hello'];
```

* **注意**

!> - `POST`および`Header`の合計サイズは、[package_max_length](/server/setting?id=package_max_length)の設定を超えてはいけません。それ以外の場合、悪意のあるリクエストと見なされます
- `POST`パラメータの最大数は128を超えてはいけません
### クッキー

?> **`HTTP`リクエストに含まれる`COOKIE`情報は、キーと値の配列形式です。**

```php
Swoole\Http\Request->cookie: array
```

* **例**

```php
echo $request->cookie['username'];
```
### ファイル

**アップロードされたファイルの情報。**

'form'名が 'key' の2次元配列形式です。`PHP` の `$_FILES` と同じです。ファイルの最大サイズは [package_max_length](/server/setting?id=package_max_length) の値を超えてはいけません。Swoole はメッセージを解析する際にメモリを使用するため、メッセージが大きいほどメモリ使用量も多くなります。そのため、`Swoole\Http\Server` を使用して大きなファイルアップロードを処理するか、ユーザーが途中からアップロードを続行する機能を設計することは避けてください。

```php
Swoole\Http\Request->files: array
```

* **例**

```php
Array
(
    [name] => facepalm.jpg // ブラウザからのアップロード時に渡されたファイル名
    [type] => image/jpeg // MIME タイプ
    [tmp_name] => /tmp/swoole.upfile.n3FmFr // アップロードされた一時ファイル、ファイル名は/tmp/swoole.upfileで始まります
    [error] => 0
    [size] => 15476 // ファイルのサイズ
)
```

* **注意**

**`Swoole\Http\Request` オブジェクトが破棄されると、アップロードされた一時ファイルは自動的に削除されます。**
### getContent()

!> Swoole version >= `v4.5.0` で使用可能であり、古いバージョンでは`rawContent`という別名が使えます（この別名は永続的に残り、下位互換を保ちます）

?> **`POST`リクエストの元のボディを取得します。**

?> `application/x-www-form-urlencoded`形式以外のHTTP`POST`リクエストに使用されます。元の`POST`データを返します。この関数は`PHP`の`fopen('php://input')`と同等です。

```php
Swoole\Http\Request->getContent(): string|false
```

  * **戻り値**

    * 成功した場合はメッセージが返され、コンテキスト接続が存在しない場合は`false`が返されます

!> 一部のケースではサーバーはHTTP `POST`リクエストパラメータを解析する必要がないため、[http_parse_post](/http_server?id=http_parse_post)設定を使用して`POST`データの解析を無効にできます。
### getData()

?> **Get the complete original `Http` request message, note that it cannot be used under `Http2`. Includes `Http Header` and `Http Body`**

```php
Swoole\Http\Request->getData(): string|false
```

  * **Return Value**

    * Returns the message if successful, returns `false` if the context connection does not exist or in `Http2` mode.
### create()

?> **`Swoole\Http\Request`オブジェクトを作成します。**

!> Swooleのバージョン >= `v4.6.0` が必要です

```php
Swoole\Http\Request->create(array $options): Swoole\Http\Request
```

  * **パラメータ**

    * **`array $options`**
      * **機能**：オプションの配列で、`Request`オブジェクトの設定を指定します

| パラメータ                                                | デフォルト値 | 説明                                                                |
| -------------------------------------------------------- | ------------ | ------------------------------------------------------------------ |
| [parse_cookie](/http_server?id=http_parse_cookie)        | true         | `Cookie`を解析するかどうかを設定する                             |
| [parse_body](/http_server?id=http_parse_post)            | true         | `Http Body`を解析するかどうかを設定する                          |
| [parse_files](/http_server?id=http_parse_files)          | true         | アップロードファイルの解析を有効にするかどうかを設定する         |
| enable_compression                                       | true         | サーバーが圧縮をサポートしていない場合、デフォルト値はfalse   | 圧縮を有効にするかを設定します                                    |
| compression_level                                        | 1            | 圧縮レベルを設定します。レベルは1から9までで、高いレベルほど圧縮後のサイズは小さくなりますが、CPU負荷も増加します |
| upload_tmp_dir                                           | /tmp         | 一時ファイルの保存場所を設定します。ファイルのアップロードに使用します |

  * **戻り値**

    * `Swoole\Http\Request`オブジェクトを返します

* **例**
```php
Swoole\Http\Request::create([
    'parse_cookie' => true,
    'parse_body' => true,
    'parse_files' => true,
    'enable_compression' => true,
    'compression_level' => 1,
    'upload_tmp_dir' => '/tmp',
]);
```
### parse()

?> **解析`HTTP`リクエストデータパケットを行い、成功した場合は解析したデータパケットの長さを返します。**

!> Swooleバージョン >= `v4.6.0` で利用可能

```php
Swoole\Http\Request->parse(string $data): int|false
```

  * **パラメータ**

    * **`string $data`**
      * 解析するパケット

  * **戻り値**

    * 解析に成功した場合は解析されたパケットの長さを返し、接続コンテキストが存在しないか、コンテキストが終了している場合は`false`を返します
### isCompleted()

?> **Get whether the current `HTTP` request data packet has reached the end.**

!> Available from Swoole version >= `v4.6.0`

```php
Swoole\Http\Request->isCompleted(): bool
```

  * **Return value**

    * `true` means it has reached the end, `false` means the connection context has ended or not reached the end.

* **Example**

```php
use Swoole\Http\Request;

$data = "GET /index.html?hello=world&test=2123 HTTP/1.1\r\n";
$data .= "Host: 127.0.0.1\r\n";
$data .= "Connection: keep-alive\r\n";
$data .= "Pragma: no-cache\r\n";
$data .= "Cache-Control: no-cache\r\n";
$data .= "Upgrade-Insecure-Requests: \r\n";
$data .= "User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.75 Safari/537.36\r\n";
$data .= "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9\r\n";
$data .= "Accept-Encoding: gzip, deflate, br\r\n";
$data .= "Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,zh-TW;q=0.7,ja;q=0.6\r\n";
$data .= "Cookie: env=pretest; phpsessid=fcccs2af8673a2f343a61a96551c8523d79ea; username=hantianfeng\r\n";

/** @var Request $req */
$req = Request::create(['parse_cookie' => false]);
var_dump($req);

var_dump($req->isCompleted());
var_dump($req->parse($data));

var_dump($req->parse("\r\n"));
var_dump($req->isCompleted());

var_dump($req);
// Since cookie parsing is disabled, it will be null
var_dump($req->cookie);
```
### getMethod()

?> **現在の`HTTP`リクエストのリクエスト方法を取得します。**

!> Swooleバージョン >= `v4.6.2` で利用可能

```php
Swoole\Http\Request->getMethod(): string|false
```

  * **返り値**

    * 成功時はリクエスト方法の大文字表記、`false`はコンテキストが存在しないことを示す

```php
var_dump($request->server['request_method']);
var_dump($request->getMethod());
```
## Swoole\Http\Response

`HTTP`応答オブジェクトは、このオブジェクトのメソッドを呼び出すことで`HTTP`応答を送信できます。

?> `Response`オブジェクトが破棄されたときに、`end`メソッドを使用して`HTTP`応答を送信しない場合、内部で自動的に`end("")`が実行されます。

!> `Http\Response`オブジェクトを`&`記号で参照しないでください。
### header()：id=setheader

?> **HTTPレスポンスのヘッダー情報を設定します**【別名`setHeader`】

```php
Swoole\Http\Response->header(string $key, string $value, bool $format = true): bool;
```

* **パラメータ** 

  * **`string $key`**
    * **機能**：`HTTP`ヘッダーの`キー`
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`string $value`**
    * **機能**：`HTTP`ヘッダーの`値`
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`bool $format`**
    * **機能**：`キー`を`HTTP`の規約に従ってフォーマットするかどうか【デフォルトは`true`で自動的にフォーマットされます】
    * **デフォルト値**：`true`
    * **その他の値**：なし

* **戻り値** 

  * 設定に失敗した場合は`false`を返す
  * 設定に成功した場合は`true`を返す
* **注意事項**

   -`header`の設定は`end`メソッドの前で行う必要があります
   -`$key`はHTTPの規約に完全に準拠している必要があり、各単語の先頭文字が大文字である必要があり、中国語、アンダースコア、またはその他の特殊文字を含んではいけません  
   -`$value`は必ず入力する必要があります  
   -`$ucwords`を`true`に設定すると、内部で`$key`が自動的に規約に従ってフォーマットされます  
   -同じ`$key`のHTTPヘッダーを繰り返し設定すると、最後に設定したものが使用されます  
   -クライアントが`Accept-Encoding`を設定している場合、サーバーは`Content-Length`応答を設定できません。このような場合、`Swoole`は`Content-Length`の値を無視し、警告を出力します  
   -`Content-Length`を設定した場合、応答では`Swoole\Http\Response::write()`を呼び出すことはできません。`Swoole`はこのような場合、`Content-Length`の値を無視し、警告を出力します

!> Swoole バージョンが`v4.6.0`以上の場合、同じ`$key`のHTTPヘッダーを繰り返し設定でき、`$value`は`array`、`object`、`int`、`float`などのさまざまなタイプをサポートし、内部で`toString`変換が行われ、末尾のスペースや改行が削除されます。

* **例**

```php
$response->header('content-type', 'image/jpeg', true);

$response->header('Content-Length', '100002 ');
$response->header('Test-Value', [
    "a\r\n",
    'd5678',
    "e  \n ",
    null,
    5678,
    3.1415926,
]);
$response->header('Foo', new SplFileInfo('bar'));
```
### trailer()

?> **`Header`情報を`HTTP`レスポンスの末尾に追加します。 `HTTP2`でのみ使用可能であり、メッセージ完全性チェック、デジタル署名などに使用されます。**

```php
Swoole\Http\Response->trailer(string $key, string $value): bool;
```

* **パラメータ**

  * **`string $key`**
    * **機能**：`HTTP`ヘッダの`Key`
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`string $value`**
    * **機能**：`HTTP`ヘッダの`value`
    * **デフォルト値**：なし
    * **その他の値**：なし

* **戻り値**

  * 設定に失敗した場合、`false`を返す
  * 設定に成功した場合、`true`を返す

* **注意**

  !> 同じ`$key`を持つ`Http`ヘッダを複数回設定すると、最後のものが適用されます。

* **例**

```php
$response->trailer('grpc-status', 0);
$response->trailer('grpc-message', '');
```
### cookie()

?> **HTTPレスポンスのcookie情報を設定します。`setCookie`の別名です。このメソッドの引数は`PHP`の`setcookie`と同じです。**

```php
Swoole\Http\Response->cookie(string $key, string $value = '', int $expire = 0 , string $path = '/', string $domain  = '', bool $secure = false , bool $httponly = false, string $samesite = '', string $priority = ''): bool;
```

  * **Parameters** 

    * **`string $key`**
      * **Description**: CookieのKey
      * **Default**: なし
      * **Other values**: なし

    * **`string $value`**
      * **Description**: Cookieのvalue
      * **Default**: なし
      * **Other values**: なし
  
    * **`int $expire`**
      * **Description**: Cookieの有効期限
      * **Default**: 0, 期限なし
      * **Other values**: なし

    * **`string $path`**
      * **Description**: Cookieのサーバーパスを指定します。
      * **Default**: /
      * **Other values**: なし

    * **`string $domain`**
      * **Description**: Cookieのドメインを指定します
      * **Default**: ''
      * **Other values**: なし

    * **`bool $secure`**
      * **Description**: 安全なHTTPS接続を介してCookieを転送するかどうかを指定します
      * **Default**: ''
      * **Other values**: なし

    * **`bool $httponly`**
      * **Description**: JavaScriptがHttpOnly属性を持つCookieにアクセスするのを許可するかどうか、`true`は許可しないことを示し、`false`は許可することを示します
      * **Default**: false
      * **Other values**: なし

    * **`string $samesite`**
      * **Description**: サードパーティCookieを制限してセキュリティリスクを減らします。`Strict`、`Lax`、`None`を選択できます
      * **Default**: ''
      * **Other values**: なし

    * **`string $priority`**
      * **Description**: Cookieの優先度、Cookieの数が規定値を超えると、優先度が低いものが先に削除されます。`Low`、`Medium`、`High`を選択できます
      * **Default**: ''
      * **Other values**: なし
  
  * **Return Value** 

    * 設定に失敗した場合は`false`を返します
    * 設定に成功した場合は`true`を返します

* **注意**

  !> - `cookie`の設定は[end](/http_server?id=end)メソッドの前に行う必要があります  
  - `$samesite` パラメータは `v4.4.6` バージョンからサポートされ、`$priority` パラメータは `v4.5.8` バージョンからサポートされます  
  - `Swoole`は`$value`を自動的に`urlencode`エンコードします。`rawCookie()`メソッドを使用して`$value`のエンコード処理を無効にできます  
  - `Swoole`は同じ`$key`の複数の`COOKIE`を設定することを許可します
### rawCookie()

?> **Set the `cookie` information of the `HTTP` response**

!> The parameters of `rawCookie()` are the same as `cookie()` mentioned earlier, except that no encoding is applied.
### status()

?> **Send `Http` status code. Alias `setStatusCode()`**

```php
Swoole\Http\Response->status(int $http_status_code, string $reason = ''): bool
```

* **Parameters** 

  * **`int $http_status_code`**
    * **Description**: Set `HttpCode`
    * **Default**: None
    * **Other values**: None

  * **`string $reason`**
    * **Description**: Status code reason
    * **Default**: ''
    * **Other values**: None

  * **Return Value** 

    * Return `false` if failed to set
    * Return `true` if set successfully

* **Note**

  * If only the first parameter `$http_status_code` is passed, it must be a valid `HttpCode` such as `200`, `502`, `301`, `404`, etc., otherwise, it will be set to `200` status code
  * If the second parameter `$reason` is set, `$http_status_code` can be any numerical value, including undefined `HttpCode` like `499`
  * The `status` method must be executed before calling [$response->end()](/http_server?id=end)
### gzip()

!> This method has been deprecated since `4.1.0`, please refer to the [http_compression](/http_server?id=http_compression) section; the `http_compression` configuration option should be used instead of the `gzip` method in newer versions.  
The main reason is that the `gzip()` method does not check the `Accept-Encoding` header sent by the client browser. Using `gzip` forcefully when the client does not support gzip compression can cause the client to be unable to decompress.  
The new `http_compression` configuration option will automatically choose whether to compress based on the client's `Accept-Encoding` header and will automatically choose the best compression algorithm.

?> **Enable `Http GZIP` compression. Compression can reduce the size of `HTML` content, effectively save network bandwidth, and improve response time. You must execute `gzip` before sending content in `write/end`, otherwise an error will be thrown.**
```php
Swoole\Http\Response->gzip(int $level = 1);
```

* **Parameters** 
   
     * **`int $level`**
       * **Description**: Compression level, the higher the level, the smaller the compressed size, but it consumes more `CPU`.
       * **Default**: 1
       * **Other Values**: `1-9`

!> After calling the `gzip` method, the underlying system will automatically add the `Http` encoding header, so there is no need to set related `Http` headers in the PHP code; images in `jpg/png/gif` format have already been compressed and do not need to be compressed again.

!> The `gzip` function depends on the `zlib` library. When compiling swoole, the underlying system will check if `zlib` exists in the system. If not, the `gzip` method will be unavailable. You can install the `zlib` library using `yum` or `apt-get`:

```shell
sudo apt-get install libz-dev
```
### redirect()

?> **`Http`リダイレクトを送信します。このメソッドを呼び出すと、自動的に`end`が送信されてレスポンスが終了します。**

```php
Swoole\Http\Response->redirect(string $url, int $http_code = 302): bool
```

  * **Parameters** 
    * **`string $url`**
      * **Functionality**: The new address to redirect to, sent as a `Location` header
      * **Default value**: None
      * **Other values**: None

    * **`int $http_code`**
      * **Functionality**: Status code [default is `302` for temporary redirect, pass `301` for permanent redirect]
      * **Default value**: `302`
      * **Other values**: None

  * **Return Value** 
    * If the call is successful, it returns `true`; if the call fails or the connection context does not exist, it returns `false`

* **Example**

```php
$http = new Swoole\Http\Server("0.0.0.0", 9501, SWOOLE_BASE);

$http->on('request', function ($req, Swoole\Http\Response $resp) {
    $resp->redirect("http://www.baidu.com/", 301);
});

$http->start();
```
### write()

?> **Enable `Http Chunk` to send response content to the browser in segments.**

?> You can refer to the `Http` protocol standard document for more information on `Http Chunk`.

```php
Swoole\Http\Response->write(string $data): bool
```

  * **Parameters**
  
    * **`string $data`**
      * **Purpose**: The content data to be sent [with a maximum length of `2M`, controlled by the [buffer_output_size](/server/setting?id=buffer_output_size) configuration].
      * **Default value**: None
      * **Other values**: None

  * **Return Value**
  
    * Returns `true` if the call is successful; returns `false` if the call fails or the connection context does not exist.

* **Tips**

  * After sending data in segments using `write`, the [end](/http_server?id=end) method will not accept any parameters. Calling `end` will simply send a `Chunk` with a length of `0` to indicate that the data transfer is complete.
  * If you set `Content-Length` using the Swoole\Http\Response::header() method and then call this method, `Swoole` will ignore the `Content-Length` setting and throw a warning.
  * This function cannot be used with `Http2`, otherwise a warning will be thrown.
  * If the client supports response compression, `Swoole\Http\Response::write()` will forcefully disable compression.
### sendfile()

?> **ブラウザにファイルを送信する。**

```php
Swoole\Http\Response->sendfile(string $filename, int $offset = 0, int $length = 0): bool
```

  * **パラメータ** 

    * **`string $filename`**
      * **機能**: 送信するファイル名【ファイルが存在しないかアクセス権がない場合、`sendfile` は失敗します。】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $offset`**
      * **機能**: アップロードファイルのオフセット【ファイルの中間部分からデータを転送することができます。この機能はレジュームをサポートするために使われます。】
      * **デフォルト値**: `0`
      * **その他の値**: なし

    * **`int $length`**
      * **機能**: データの送信サイズ
      * **デフォルト値**: ファイルのサイズ
      * **その他の値**: なし

  * **戻り値** 

      * 呼び出し成功：`true` を返し、呼び出し失敗または接続コンテキストが存在しない場合：`false` を返します。

* **ヒント**

  * ファイルのMIME形式を自動的に推測できないため、アプリケーションコードで`Content-Type`を指定する必要があります。
  * `sendfile` を呼び出す前に、`write` メソッドを使用して `Http-Chunk` を送信してはいけません。
  * `sendfile` を呼び出した後、内部で `end` が自動的に実行されます。
  * `gzip` 圧縮は `sendfile` に対応していません。

* **例**

```php
$response->header('Content-Type', 'image/jpeg');
$response->sendfile(__DIR__.$request->server['request_uri']);
```
### end()

?> **Send the `Http` response body and end the request handling.**

```php
Swoole\Http\Response->end(string $html): bool
```

  * **Parameters** 
  
    * **`string $html`**
      * **Function**：The content to send
      * **Default value**：none
      * **Other values**：none

  * **Return Value** 

    * If the call is successful, return `true`, if the call fails or the connection context does not exist, return `false`

* **Tips**

  * `end` can only be called once. If you need to send data to the client multiple times, please use the [write](/http_server?id=write) method
  * If the client has enabled [KeepAlive](/coroutine_client/http_client?id=keep_alive), the connection will be maintained, and the server will wait for the next request
  * If the client has not enabled `KeepAlive`, the server will disconnect the connection
  * The content to be sent by `end` is limited by [output_buffer_size](/server/setting?id=buffer_output_size), which is `2M` by default. If it exceeds this limit, the response will fail and the following error will be thrown:

!> Solution: Use [sendfile](/http_server?id=sendfile), [write](/http_server?id=write), or adjust [output_buffer_size](/server/setting?id=buffer_output_size)

```bash
WARNING finish (ERRNO 1203): The length of data [262144] exceeds the output buffer size[131072], please use the sendfile, chunked transfer mode or adjust the output_buffer_size
```
### detach()

?> **分離レスポンスオブジェクト。** このメソッドを使うと、`$response`オブジェクトが破壊されても自動的に[end](/http_server?id=httpresponse)されません。[Http\Response::create](/http_server?id=create) と [Server->send](/server/methods?id=send) と一緒に使用します。

```php
Swoole\Http\Response->detach(): bool
```

  * **返り値** 

    * 成功した場合は`true`を返し、失敗した場合や接続コンテキストが存在しない場合は`false`を返します

* **例** 

  * **クロスプロセスレスポンス**

  ?> 一部の場合には、[Task ワーカー](/learn?id=taskworkerプロセス)内でクライアントに応答する必要があります。この時に`detach`を使用して`$response`オブジェクトを独立させることができます。[Task ワーカー](/learn?id=taskworkerプロセス)で`$response`を再構築し、HTTP リクエストを応答できます。

  ```php
  $http = new Swoole\Http\Server("0.0.0.0", 9501);

  $http->set(['task_worker_num' => 1, 'worker_num' => 1]);

  $http->on('request', function ($req, Swoole\Http\Response $resp) use ($http) {
      $resp->detach();
      $http->task(strval($resp->fd));
  });

  $http->on('finish', function () {
      echo "task finish";
  });

  $http->on('task', function ($serv, $task_id, $worker_id, $data) {
      var_dump($data);
      $resp = Swoole\Http\Response::create($data);
      $resp->end("in task");
      echo "async task\n";
  });

  $http->start();
  ```

  * **任意のコンテンツを送信**

  ?> 特定のシーンでは、クライアントに特別な応答コンテンツを送信する必要があるかもしれません。 `Http\Response` オブジェクトの `end` メソッドでは要求を満たすことができない場合、`detach` を使用してレスポンスオブジェクトを切り離し、自前で HTTP プロトコルの応答データを組み立て、`Server->send` を使用してデータを送信できます。

  ```php
  $http = new Swoole\Http\Server("0.0.0.0", 9501);

  $http->on('request', function ($req, Swoole\Http\Response $resp) use ($http) {
      $resp->detach();
      $http->send($resp->fd, "HTTP/1.1 200 OK\r\nServer: server\r\n\r\nHello World\n");
  });

  $http->start();
  ```
### create()

?> **Create a new `Swoole\Http\Response` object.**

!> Before using this method, make sure to call the `detach` method to detach the old `$response` object, otherwise it may cause sending response content twice for the same request.

```php
Swoole\Http\Response::create(object|array|int $server = -1, int $fd = -1): Swoole\Http\Response
```

  * **Parameters** 

    * **`int $server`**
      * **Function**: `Swoole\Server` or `Swoole\Coroutine\Socket` object, an array (which can only have two parameters - the first one being `Swoole\Server` object, the second being `Swoole\Http\Request` object), or a file descriptor
      * **Default**: -1
      * **Other values**: N/A

    * **`int $fd`**
      * **Function**: File descriptor. If the `$server` parameter is a `Swoole\Server` object, `$fd` is mandatory
      * **Default**: -1
      * **Other values**: N/A

  * **Return Value** 

    * Returns a new `Swoole\Http\Response` object if successful, false if failed

* **Example**

```php
$http = new Swoole\Http\Server('0.0.0.0', 9501);

$http->on('request', function ($req, Swoole\Http\Response $resp) use ($http) {
    $resp->detach();
    // Example 1
    $resp2 = Swoole\Http\Response::create($req->fd);
    // Example 2
    $resp2 = Swoole\Http\Response::create($http, $req->fd);
    // Example 3
    $resp2 = Swoole\Http\Response::create([$http, $req]);
    // Example 4
    $socket = new Swoole\Coroutine\Socket(AF_INET, SOCK_STREAM, IPPROTO_IP);
    $socket->connect('127.0.0.1', 9501)
    $resp2 = Swoole\Http\Response::create($socket);
    $resp2->end("hello world");
});

$http->start();
```
### isWritable()

?> **`Swoole\Http\Response`オブジェクトが終了(`end`)または分離(`detach`)されたかどうかを判断します。**

```php
Swoole\Http\Response->isWritable(): bool
```

  * **戻り値** 

    * `Swoole\Http\Response`オブジェクトが終了していない場合や分離していない場合は`true`を返し、そうでない場合は`false`を返します。


!> Swooleバージョン >= `v4.6.0` で利用可能

* **例**

```php
use Swoole\Http\Server;
use Swoole\Http\Request;
use Swoole\Http\Response;

$http = new Server('0.0.0.0', 9501);

$http->on('request', function (Request $req, Response $resp) {
    var_dump($resp->isWritable()); // true
    $resp->end('hello');
    var_dump($resp->isWritable()); // false
    $resp->setStatusCode(403); // http response is unavailable (maybe it has been ended or detached)
});

$http->start();
```
このセクションは配置オプションについて説明しています。
### http_parse_cookie

?> **Disable `Cookie` parsing for the `Swoole\Http\Request` object, keeping the raw `Cookies` information untouched in the `header`. Enabled by default.**

```php
$server->set([
    'http_parse_cookie' => false,
]);
```
### http_parse_post

?> **Configure the `Swoole\Http\Request` object to toggle POST message parsing, enabled by default**

* When set to `true`, automatically parse the request body of `Content-Type x-www-form-urlencoded` to the `POST` array.
* When set to `false`, POST parsing will be turned off.

```php
$server->set([
    'http_parse_post' => false,
]);
```
### http_parse_files

?> **Setting for parsing upload files for `Swoole\Http\Request` object. Enabled by default**

```php
$server->set([
    'http_parse_files' => false,
]);
``` 

日本語:
### http_parse_files

?> **`Swoole\Http\Request`オブジェクトのアップロードファイルを解析するための設定。デフォルトで有効**

```php
$server->set([
    'http_parse_files' => false,
]);
```
### http_compression

?> **`Swoole\Http\Response`オブジェクト向けの設定、圧縮を有効にします。デフォルトでは有効です。**

!> - `http-chunk`はセグメントごとの個別の圧縮をサポートしていないため、[write](/http_server?id=write)メソッドを使用すると圧縮が強制的にオフになります。  
- `http_compression`は `v4.1.0` 以降で利用可能

```php
$server->set([
    'http_compression' => false,
]);
```

現在、`gzip`、`br`、`deflate`の3つの圧縮形式がサポートされており、基本的にブラウザクライアントが送信する`Accept-Encoding`ヘッダに基づいて自動的に圧縮方法が選択されます（圧縮アルゴリズムの優先順位：`br` > `gzip` > `deflate`）。

**依存関係：**

`gzip`と`deflate`は、`zlib`ライブラリに依存しており、`Swoole`をコンパイルする際にシステムに`zlib`が存在するかどうかを検出します。

`yum`や `apt-get` を使用して`zlib`ライブラリをインストールできます：

```shell
sudo apt-get install libz-dev
```

`br`圧縮形式は、`google`の `brotli`ライブラリに依存しており、`install brotli on linux` と検索してインストール方法を調べてください。`Swoole`をコンパイルする際にシステムに`brotli`が存在するかどうかを検出します。
### http_compression_level / compression_level / http_gzip_level

?> **`Swoole\Http\Response`オブジェクトに対する圧縮レベルの設定**
  
!> `$level` 圧縮レベル、範囲は`1-9`で、レベルが高いほど圧縮後のサイズが小さくなりますが、`CPU`負荷が増えます。デフォルトは`1`で、最大は`9`
### http_compression_min_length / compression_min_length

?> **`Swoole\Http\Response`オブジェクトの設定に関する、圧縮を有効にするための最小バイト数を設定します。このオプション値を超えると、圧縮が有効になります。デフォルトは20バイトです。**

!> Swoole version >= `v4.6.3` で利用可能

```php
$server->set([
    'compression_min_length' => 128,
]);
```
### upload_tmp_dir

?> **Set the temporary directory for uploaded files. The directory must not exceed `220` bytes in length.**

```php
$server->set([
    'upload_tmp_dir' => '/data/uploadfiles/',
]);
``` 

翻訳：  
**アップロードされたファイルの一時ディレクトリを設定します。ディレクトリの長さは`220`バイトを超えてはいけません。**
### upload_max_filesize

?> **設定アップロードファイルの最大サイズ**

```php
$server->set([
    'upload_max_filesize' => 5 * 1024,
]);
```
### enable_static_handler（静的ファイルハンドラーを有効にする）

静的ファイルリクエストの処理機能を有効にします。`document_root`と一緒に使用する必要があります。デフォルトは`false`です。
### http_autoindex

`http autoindex`機能を有効にする。デフォルトでは無効です。
### http_index_files

配合`http_autoindex`使用，指定需要被索引的文件列表

```php
$server->set([
    'document_root' => '/data/webroot/example.com',
    'enable_static_handler' => true,
    'http_autoindex' => true,
    'http_index_files' => ['indesx.html', 'index.txt'],
]);
```
### http_compression_types / compression_types

?> **`Swoole\Http\Response`オブジェクトに対する設定で、圧縮するレスポンスタイプを指定します**

```php
$server->set([
        'http_compression_types' => [
            'text/html',
            'application/json'
        ],
    ]);
```

!> Swooleバージョン >= `v4.8.12` で利用可能
### static_handler_locations

?> **静的ハンドラーの場所を設定します。タイプは配列で、デフォルトは無効です。**

!> `Swoole`バージョン >= `v4.4.0` で利用可能

```php
$server->set([
    'static_handler_locations' => ['/static', '/app/images'],
]);
```

* `Nginx`の`location`ディレクティブに似ており、1つまたは複数のパスを静的なパスとして指定できます。指定されたパス内でのみ、静的ファイルハンドラーが有効になります。それ以外の場合は、動的リクエストと見なされます。
* `location`項目は`/`で始まる必要があります。
* `/app/images`のような複数のレベルのパスもサポートされています。
* `static_handler_locations`を有効にした場合、リクエストに対応するファイルが存在しない場合、直接404エラーが返されます。
### open_http2_protocol

?> **Enable `HTTP2` protocol parsing** 【Default value: `false`】

!> Requires compiling with the [--enable-http2](/environment?id=compile-options) option, `Swoole5` starts compiling http2 by default.
### document_root

?> **静的ファイルのルートディレクトリを設定し、`enable_static_handler`と共に使用します。** 

!> この機能はかなりシンプルですので、直接公開ネットワーク環境で使用しないでください

```php
$server->set([
    'document_root' => '/data/webroot/example.com', // v4.4.0以下のバージョンでは、ここは絶対パスである必要があります
    'enable_static_handler' => true,
]);
```

* `document_root`を設定し、`enable_static_handler`を`true`に設定した後、下位レベルで`Http`リクエストを受け取ると、まずdocument_rootのパスにリクエストされたファイルが存在するかどうかをチェックし、存在する場合はファイルの内容を直接クライアントに送信し、[onRequest](/http_server?id=on)コールバックを発生させません。
* 静的ファイル処理機能を使用する場合、動的PHPコードと静的ファイルを隔離し、静的ファイルを特定のディレクトリに配置する必要があります。
### max_concurrency

?> **This limits the maximum number of concurrent requests for the `HTTP1/2` service. If exceeded, a `503` error will be returned. The default value is 4294967295, which is the maximum value of an unsigned int.**

```php
$server->set([
    'max_concurrency' => 1000,
]);
```
### worker_max_concurrency

?> **ワーカープロセスは一度に多くのリクエストを受け取るため、過度の負荷を避けるために、`worker`プロセスのリクエスト実行数を制限するために`worker_max_concurrency`を設定できます。この値を超えると、余分なリクエストはキューに一時的に保存されます。デフォルト値は4294967295であり、これは符号なし整数の最大値です。`worker_max_concurrency`が設定されていない場合、`max_concurrency`が設定されている場合は、自動的に`worker_max_concurrency`が`max_concurrency`と等しくなります。**

```php
$server->set([
    'worker_max_concurrency' => 1000,
]);
```

!> Swooleバージョン >= `v5.0.0` で利用可能
### http2_header_table_size

?> Define the maximum `header table` size for HTTP/2 network connections.

```php
$server->set([
  'http2_header_table_size' => 0x1
])
``` 

### http2_header_table_size

?>HTTP/2ネットワーク接続の最大`header table`サイズを定義します。

```php
$server->set([
  'http2_header_table_size' => 0x1
])
```
### http2_enable_push

?> この設定は、HTTP2プッシュを有効または無効にするために使用されます。

```php
$server->set([
  'http2_enable_push' => 0x2
])
```
### http2_max_concurrent_streams

?> Set the maximum number of multiplexed streams to be accepted in each HTTP/2 network connection.

```php
$server->set([
  'http2_max_concurrent_streams' => 0x3
])
```
### http2_init_window_size

?> HTTP/2のフロー制御ウィンドウの初期サイズを設定します。

```php
$server->set([
  'http2_init_window_size' => 0x4
])
```
### http2_max_frame_size

?> 単一のHTTP/2プロトコルフレームで送信する本文の最大サイズを設定します。

```php
$server->set([
  'http2_max_frame_size' => 0x5
])
```  
### http2_max_header_list_size

?> Setting for the maximum size of the headers that can be sent in a request on an HTTP/2 stream.

```php
$server->set([
  'http2_max_header_list_size' => 0x6
])
``` 

外部ライブラリによると、 `http2_max_header_list_size` を使用して HTTP/2 コネクション上のリクエストで送信できるヘッダーの最大サイズを設定することができます。 上記の例は、16進数の値 `0x6` を使用して最大サイズを設定しています。

```python
# Coroutine\Http2\Client
```

協力Http2クライアント
```php
use Swoole\Http2\Request;
use Swoole\Coroutine\Http2\Client;
use function Swoole\Coroutine\run;

run(function () {
    $domain = 'www.zhihu.com';
    $cli = new Client($domain, 443, true);
    $cli->set([
        'timeout' => -1,
        'ssl_host_name' => $domain
    ]);
    $cli->connect();
    $req = new Request();
    $req->method = 'POST';
    $req->path = '/api/v4/answers/300000000/voters';
    $req->headers = [
        'host' => $domain,
        'user-agent' => 'Chrome/49.0.2587.3',
        'accept' => 'text/html,application/xhtml+xml,application/xml',
        'accept-encoding' => 'gzip'
    ];
    $req->data = '{"type":"up"}';
    $cli->send($req);
    $response = $cli->recv();
    var_dump(assert(json_decode($response->data)->error->code === 10002));
});
```
```python
# This is a code block and should not be translated
def method():
    print("This is a method")
```

### __construct()

构造方法。

```php
Swoole\Coroutine\Http2\Client::__construct(string $host, int $port, bool $open_ssl = false): void
```

  * **参数** 

    * **`string $host`**
      * **功能**：目标主机的IP地址【`$host`如果为域名底层需要进行一次`DNS`查询】
      * **默认值**：无
      * **其它值**：无

    * **`int $port`**
      * **功能**：目标端口【`Http`一般为`80`端口，`Https`一般为`443`端口】
      * **默认值**：无
      * **其它值**：无

    * **`bool $open_ssl`**
      * **功能**：是否开启`TLS/SSL`隧道加密 【`https`网站必须设置为`true`】
      * **默认值**：`false`
      * **其它值**：`true`

  * **注意**

    !> -如果你需要请求外网URL请修改`timeout`为更大的数值，参考[客户端超时规则](/coroutine_client/init?id=超时规则)  
    -`$ssl`需要依赖`openssl`，必须在编译`Swoole`时启用[--enable-openssl](/environment?id=编译选项)
### set()

[Swoole\Client::set](/client?id=configuration)の詳細な構成項目については、そちらを参照してください。

```php
Swoole\Coroutine\Http2\Client->set(array $options): void
```
### connect()

目標サーバーに接続します。このメソッドには引数がありません。

!> `connect`を呼び出すと、[coroutine schedule](/coroutine?id=coroutine-schedule)が自動的に行われ、接続が成功するか失敗するかに関わらず、`connect`が戻ります。接続が確立された後は、`send`メソッドを使用してサーバーにリクエストを送信できます。

```php
Swoole\Coroutine\Http2\Client->connect(): bool
```

  * **Return Value**

    * 接続に成功した場合は`true`を返す
    * 接続に失敗した場合は`false`を返し、`errCode`プロパティでエラーコードを取得してください
### stats()

流の状態を取得します。

```php
Swoole\Coroutine\Http2\Client->stats([$key]): array|bool
```

  * **例**

```php
var_dump($client->stats(), $client->stats()['local_settings'], $client->stats('local_settings'));
```
### isStreamExist()

指定されたストリームが存在するかどうかを判別します。

```php
Swoole\Coroutine\Http2\Client->isStreamExist(int $stream_id): bool
```
### send()

サーバーにリクエストを送信し、バックエンドは自動的に`Http2`の`stream`を確立します。複数のリクエストを同時に送信できます。

```php
Swoole\Coroutine\Http2\Client->send(Swoole\Http2\Request $request): int|false
```

  * **パラメータ** 

    * **`Swoole\Http2\Request $request`**
      * **役割**：Swoole\Http2\Request オブジェクトを送信します
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **戻り値**

    * 成功した場合、流の番号を返します。番号は`1`から始まる奇数です。
    * 失敗した場合は`false`を返します。

  * **ポイント**

    * **Requestオブジェクト**

      !> `Swoole\Http2\Request`オブジェクトにはメソッドがありません。リクエスト関連の情報を書き込むためにオブジェクトのプロパティを設定します。

      * `headers` 配列、HTTPヘッダー
      * `method` 文字列、リクエストメソッドを設定します。例：`GET`、`POST`
      * `path` 文字列、URLのパスを設定します。例：`/index.php?a=1&b=2`、開始を`/`で始める必要があります
      * `cookies` 配列、`COOKIES`を設定します
      * `data` リクエストの本文を設定します。文字列の場合は直接`RAW form-data`として送信されます
      * `data` が配列の場合、バックエンドは自動的に`x-www-form-urlencoded`形式の`POST`コンテンツにパッケージ化し、`Content-Type`を`application/x-www-form-urlencoded`に設定します
      * `pipeline` ブール値、`true`に設定すると、`$request`を送信した後、`stream`を閉じずに続けてデータを書き込むことができます

    * **pipeline**

      * デフォルトでは、`send`メソッドはリクエストを送信した後、現在の`Http2 Stream`を終了します。`pipeline`を有効にすると、バックエンドは`stream`を維持し、複数回`write`メソッドを呼び出してデータフレームをサーバーに送信できます。詳細は`write`メソッドを参照してください。
### write()

More data frames can be sent to the server by calling write multiple times to write data frames to the same stream.

```php
Swoole\Coroutine\Http2\Client->write(int $streamId, mixed $data, bool $end = false): bool
```

  * **Parameters** 

    * **`int $streamId`**
      * **Description**: Stream number returned by the `send` method
      * **Default**: None
      * **Other values**: None

    * **`mixed $data`**
      * **Description**: Content of the data frame, can be a string or an array
      * **Default**: None
      * **Other values**: None

    * **`bool $end`**
      * **Description**: Whether to close the stream
      * **Default**: `false`
      * **Other values**: `true`

  * **Usage example**

```php
use Swoole\Http2\Request;
use Swoole\Coroutine\Http2\Client;
use function Swoole\Coroutine\run;

run(function () {
    $cli = new Client('127.0.0.1', 9518);
    $cli->set(['timeout' => 1]);
    var_dump($cli->connect());

    $req3 = new Request();
    $req3->path = "/index.php";
    $req3->headers = [
        'host' => "localhost",
        "user-agent" => 'Chrome/49.0.2587.3',
        'accept' => 'text/html,application/xhtml+xml,application/xml',
        'accept-encoding' => 'gzip',
    ];
    $req3->pipeline = true;
    $req3->method = "POST";
    $streamId = $cli->send($req3);
    $cli->write($streamId, ['int' => rand(1000, 9999)]);
    $cli->write($streamId, ['int' => rand(1000, 9999)]);
    //end stream
    $cli->write($streamId, ['int' => rand(1000, 9999), 'end' => true], true);
    var_dump($cli->recv());
    $cli->close();
});
```

!> If you want to use `write` to send data frames in segments, you must set `$request->pipeline` to `true` when sending the `send` request.  
Once a data frame with `end` set to `true` is sent, the stream will be closed, and you cannot call `write` to send data to this stream anymore.
### recv()

リクエストを受信します。

!> このメソッドを呼び出すと[コルーチンスケジューリング](/coroutine?id=コルーチンスケジューリング)が発生します。

```php
Swoole\Coroutine\Http2\Client->recv(float $timeout): Swoole\Http2\Response;
```

  * **引数** 

    * **`float $timeout`**
      * **機能**：タイムアウト時間を設定します。[クライアントタイムアウトルール](/coroutine_client/init?id=タイムアウトルール)を参照してください。
      * **値の単位**：秒【浮動小数点数をサポートします。例：`1.5`は`1秒`+`500ミリ秒`を表します】
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **戻り値**

成功した場合は Swoole\Http2\Response オブジェクトが返されます。

```php
/**@var $resp Swoole\Http2\Response */
var_dump($resp->statusCode); // サーバーが送信したHTTPステータスコード。例：200、502など
var_dump($resp->headers); // サーバーが送信したヘッダー情報
var_dump($resp->cookies); // サーバーが設定したCOOKIE情報
var_dump($resp->set_cookie_headers); // サーバー側が返した生のCOOKIE情報。domainやpathなどが含まれます
var_dump($resp->data); // サーバーが送信したレスポンスボディ
```

!> Swoole バージョン < [v4.0.4](/version/bc?id=_404) の場合、`data` プロパティは `body` プロパティです。Swoole バージョン < [v4.0.3](/version/bc?id=_403) の場合、`headers` および `cookies` は単数形式です。
### read()

`recv()`関数と基本的に同じですが、`pipeline`型のレスポンスの場合、`read`関数は複数回呼び出すことができます。各呼び出しでは一部のデータを受信し、メモリを節約したり、プッシュメッセージをできるだけ早く受け取ることができます。一方、`recv`関数は常に完全なレスポンスを一つのフレームにまとめてから返します。

?> このメソッドを呼び出すと[コルーチンスケジューリング](/coroutine?id=コルーチンスケジューリング)が発生します。

```php
Swoole\Coroutine\Http2\Client->read(float $timeout): Swoole\Http2\Response;
```

  * **パラメータ** 

    * **`float $timeout`**
      * **機能**：タイムアウト時間を設定します。[クライアントのタイムアウト規則](/coroutine_client/init?id=タイムアウト規則)を参照してください。
      * **値の単位**：秒【浮動小数点数もサポートされます。例：`1.5` は `1 秒` `500 ミリ秒` を意味します】
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **戻り値**

    成功した場合は Swoole\Http2\Response オブジェクトが返されます。
### goaway()

GOAWAYフレームは、接続の終了を開始したり、重大なエラー状態の信号を発信するために使用されます。

```php
Swoole\Coroutine\Http2\Client->goaway(int $error_code = SWOOLE_HTTP2_ERROR_NO_ERROR, string $debug_data): bool
```
### ping()

PINGフレームは、送信側からの最小往復時間を測定し、アイドル接続が依然として有効かどうかを確認するための仕組みです。

```php
Swoole\Coroutine\Http2\Client->ping(): bool
```
### close()

接続を閉じます。

```php
Swoole\Coroutine\Http2\Client->close(): bool
```

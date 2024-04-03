# HTTPサーバー

?> 完全なコルーチン化されたHTTPサーバーの実装、`Co\Http\Server`はHTTPパースのパフォーマンスのためにC++で書かれており、したがってPHPで書かれた[Co\Server](/coroutine/server)のサブクラスではありません。

[Http\Server](/http_server) との違い：

- ランタイムで動的に作成、破棄できる
- 接続の処理は個別のサブコルーチンで行われ、クライアント接続の`Connect`、`Request`、`Response`、`Close`は完全に直列されている

!> `v4.4.0`以上が必要

!> コンパイル時に[HTTP2を有効にした場合](/environment?id=编译选项)、デフォルトでHTTP2プロトコルサポートが有効になり、`Swoole\Http\Server`のように[open_http2_protocol](/http_server?id=open_http2_protocol)の設定は不要です（注：**v4.4.16未満のバージョンには既知のHTTP2サポートのバグがあります。アップグレード後にご使用ください**）

## 短縮名

`Co\Http\Server`の短縮名を使用できます。

## メソッド

### __construct()

```php
Swoole\Coroutine\Http\Server::__construct(string $host, int $port = 0, bool $ssl = false, bool $reuse_port = false);
```

  * **パラメータ** 

    * **`string $host`**
      * **役割**：リッスンするIPアドレス【ローカルのUNIXソケットの場合は、`unix://tmp/your_file.sock`のような形式で記入してください】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $port`**
      * **役割**：リッスンするポート 
      * **デフォルト値**：0（空いているポートをランダムでリッスン）
      * **その他の値**：0～65535

    * **`bool $ssl`**
      * **役割**：`SSL/TLS`暗号化トンネルを有効にするかどうか
      * **デフォルト値**：false
      * **その他の値**：true
      
    * **`bool $reuse_port`**
      * **役割**：ポート再利用機能を有効にするかどうか、有効にすると複数のサービスが1つのポートを共有できます
      * **デフォルト値**：false
      * **その他の値**：true

### handle()

指定された`$pattern`のパスのHTTPリクエストを処理するコールバック関数を登録します。

```php
Swoole\Coroutine\Http\Server->handle(string $pattern, callable $fn): void
```

!> 必ず [Server::start](/coroutine/server?id=start) の前に処理関数を設定してください

  * **パラメータ** 

    * **`string $pattern`**
      * **役割**：`URL`パスを設定します【例：`/index.html`、ここでは`http://domain`は渡すことができません】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`callable $fn`**
      * **役割**：処理関数、使い方は`Swoole\Http\Server`の[OnRequest](/http_server?id=on)コールバックを参照してください。ここでは省略します。
      * **デフォルト値**：なし
      * **その他の値**：なし      

      例：

      ```php
      function callback(Swoole\Http\Request $req, Swoole\Http\Response $resp) {
          $resp->end("hello world");
      }
      ```

  * **ヒント**

    * サーバーは`Accept`（接続確立）後、自動的にコルーチンを作成し、`HTTP`リクエストを受け入れます
    * `$fn`は新しいサブコルーチンのスペースで実行されるため、関数内で再度コルーチンを作成する必要はありません
    * クライアントが[KeepAlive](/coroutine_client/http_client?id=keep_alive)をサポートしている場合、サブコルーチンは新しいリクエストを続けて受け入れ、終了しません
    * クライアントが`KeepAlive`をサポートしていない場合、サブコルーチンはリクエストの受け入れを停止し、接続を閉じて終了します

  * **注意**

    !> -同じパスの`$pattern`を設定した場合、新しい設定が古い設定を上書きします；  
    -ルートパスの処理関数が設定されておらず、リクエストのパスに一致する`$pattern`が見つからない場合、Swooleは`404`エラーを返します；  
    -`$pattern`は文字列マッチングのメソッドを使用し、ワイルドカードと正規表現をサポートせず、大文字小文字を区別せず、マッチングアルゴリズムはプレフィックスマッチングです。例えば：URLが`/test111`の場合、`/test`のルールにマッチします。マッチした時点でマッチして後続の構成を無視します；  
    -推奨：ルートパスの処理関数を設定し、コールバック関数内で`$request->server['request_uri']`を使用してリクエストのルーティングを行うことを推奨します。

### start()

?> **サーバーを起動します。** 

```php
Swoole\Coroutine\Http\Server->start();
```

### shutdown()

?> **サーバーを終了します。** 

```php
Swoole\Coroutine\Http\Server->shutdown();
```

## 完全な例

```php
use Swoole\Coroutine\Http\Server;
use function Swoole\Coroutine\run;

run(function () {
    $server = new Server('127.0.0.1', 9502, false);
    $server->handle('/', function ($request, $response) {
        $response->end("<h1>Index</h1>");
    });
    $server->handle('/test', function ($request, $response) {
        $response->end("<h1>Test</h1>");
    });
    $server->handle('/stop', function ($request, $response) use ($server) {
        $response->end("<h1>Stop</h1>");
        $server->shutdown();
    });
    $server->start();
});
```

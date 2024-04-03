# 協力FastCGIクライアント

PHP-FPMは高効率なバイナリプロトコルである`FastCGIプロトコル`を使用して通信します。FastCGIクライアントを使用すると、PHP-FPMサービスと直接やり取りすることができ、HTTPリバースプロキシを介する必要がありません。

[PHPのソースディレクトリ](https://github.com/swoole/library/blob/master/src/core/Coroutine/FastCGI)
## 簡単な使用例

[他のサンプルコード](https://github.com/swoole/library/tree/master/examples/fastcgi)

!> 以下のサンプルコードは、コルーチン内で呼び出す必要があります。
```php
#greeter.php
echo 'Hello ' . ($_POST['who'] ?? 'World');
```

```php
echo \Swoole\Coroutine\FastCGI\Client::call(
    '127.0.0.1:9000', // FPM listening address, it can also be a unix socket address like unix:/tmp/php-cgi.sock
    '/tmp/greeter.php', // The entry file you want to execute
    ['who' => 'Swoole'] // Additional POST information
);
```
```php
try {
    $client = new \Swoole\Coroutine\FastCGI\Client('127.0.0.1:9000', 9000);
    $request = (new \Swoole\FastCGI\HttpRequest())
        ->withScriptFilename(__DIR__ . '/greeter.php')
        ->withMethod('POST')
        ->withBody(['who' => 'Swoole']);
    $response = $client->execute($request);
    echo "Result: {$response->getBody()}\n";
} catch (\Swoole\Coroutine\FastCGI\Client\Exception $exception) {
    echo "Error: {$exception->getMessage()}\n";
}
```
```php
#var.php
var_dump($_SERVER);
var_dump($_GET);
var_dump($_POST);
```

```php
try {
    $client = new \Swoole\Coroutine\FastCGI\Client('127.0.0.1', 9000);
    $request = (new \Swoole\FastCGI\HttpRequest())
        ->withDocumentRoot(__DIR__)
        ->withScriptFilename(__DIR__ . '/var.php')
        ->withScriptName('var.php')
        ->withMethod('POST')
        ->withUri('/var?foo=bar&bar=char')
        ->withHeader('X-Foo', 'bar')
        ->withHeader('X-Bar', 'char')
        ->withBody(['foo' => 'bar', 'bar' => 'char']);
    $response = $client->execute($request);
    echo "Result: \n{$response->getBody()}";
} catch (\Swoole\Coroutine\FastCGI\Client\Exception $exception) {
    echo "Error: {$exception->getMessage()}\n";
}
```
### WordPressのワンクリックプロキシ

!> この使用方法はプロダクションで意味がなく、プロダクション中にはproxyが古いAPIエンドポイントの一部のHTTPリクエストを古いFPMサービスにプロキシするために使用されます（サイト全体をプロキシするのではありません）

```php
use Swoole\Constant;
use Swoole\Coroutine\FastCGI\Proxy;
use Swoole\Http\Request;
use Swoole\Http\Response;
use Swoole\Http\Server;

$documentRoot = '/var/www/html'; # WordPressプロジェクトのルートディレクトリ
$server = new Server('0.0.0.0', 80, SWOOLE_BASE); # ここでポートはWordPressの設定と一致する必要があります。通常、特定のポートを指定しないで、ポート80です
$server->set([
    Constant::OPTION_WORKER_NUM => swoole_cpu_num() * 2,
    Constant::OPTION_HTTP_PARSE_COOKIE => false,
    Constant::OPTION_HTTP_PARSE_POST => false,
    Constant::OPTION_DOCUMENT_ROOT => $documentRoot,
    Constant::OPTION_ENABLE_STATIC_HANDLER => true,
    Constant::OPTION_STATIC_HANDLER_LOCATIONS => ['/wp-admin', '/wp-content', '/wp-includes'], # 静的リソースパス
]);
$proxy = new Proxy('127.0.0.1:9000', $documentRoot); # プロキシオブジェクトを作成
$server->on('request', function (Request $request, Response $response) use ($proxy) {
    $proxy->pass($request, $response); # リクエストをワンクリックでプロキシする
});
$server->start();
```
### 概要

このセクションでは、特定の方法や手順について説明します。

### ステップ

1. 最初のステップ
2. 2 番目のステップ
3. 3 番目のステップ

```python
# ここにコードを入力
def function():
    return None
```
### call

Static method, creates a new client connection directly, sends a request to the FPM server, and receives the response body.

!> FPM only supports short connections, so in usual cases, creating persistent objects does not make much sense.

```php
Swoole\Coroutine\FastCGI\Client::call(string $url, string $path, $data = '', float $timeout = -1): string
```

  * **Parameters** 

    * **`string $url`**
      * **Function**: FPM listening address [e.g. `127.0.0.1:9000`, `unix:/tmp/php-cgi.sock`, etc.]
      * **Default**: None
      * **Other Values**: None

    * **`string $path`**
      * **Function**: Entry file to be executed
      * **Default**: None
      * **Other Values**: None

    * **`$data`**
      * **Function**: Additional request data
      * **Default**: None
      * **Other Values**: None

    * **`float $timeout`**
      * **Function**: Set timeout period [default is -1 indicating never timeout]
      * **Unit**: Seconds [supports float, e.g. 1.5 is 1s+500ms]
      * **Default**: `-1`
      * **Other Values**: None

  * **Return Value** 

    * Returns the body content of the server response
    * Throws `Swoole\Coroutine\FastCGI\Client\Exception` exception in case of an error
### __construct

クライアントオブジェクトのコンストラクタメソッド、ターゲットFPMサーバーを指定します

```php
Swoole\Coroutine\FastCGI\Client::__construct(string $host, int $port = 0)
```

  * **パラメータ**

    * **`string $host`**
      * **機能**：ターゲットサーバーのアドレス【例：`127.0.0.1`、`unix://tmp/php-fpm.sock`など】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $port`**
      * **機能**：ターゲットサーバーのポート番号【ターゲットアドレスがUNIXソケットの場合は不要】
      * **デフォルト値**：なし
      * **その他の値**：なし
### 実行

リクエストを実行し、レスポンスを返します

```php
Swoole\Coroutine\FastCGI\Client->execute(Request $request, float $timeout = -1): Response
```

  * **パラメーター** 

    * **`Swoole\FastCGI\Request|Swoole\FastCGI\HttpRequest $request`**
      * **機能**：リクエスト情報を含むオブジェクト。通常はHTTPリクエストを模倣するために`Swoole\FastCGI\HttpRequest`が使用されます。特別な要件がある場合にのみ、FPMプロトコルの元のリクエストクラス`Swoole\FastCGI\Request`が使用されます。
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`float $timeout`**
      * **機能**：タイムアウト時間を設定します【デフォルトは`-1`で、無制限を表します】
      * **値の単位**：秒【浮動小数点数がサポートされ、例えば、`1.5`は`1秒`+`500ミリ秒`を表します】
      * **デフォルト値**：`-1`
      * **その他の値**：なし

  * **戻り値** 

    * リクエストオブジェクトのタイプに応じたResponseオブジェクトが返されます。例えば、`Swoole\FastCGI\HttpRequest`は`Swoole\FastCGI\HttpResponse object`が返され、FPMサーバーの応答情報を含みます。
    * エラーが発生した場合は`Swoole\Coroutine\FastCGI\Client\Exception`例外がスローされます
## 関連するリクエスト/レスポンスクラス

ライブラリはPSRの依存関係の実装と拡張を読み込むことができず、拡張の読み込みは常にPHPコードの実行前に行われるため、関連するリクエストおよびレスポンスオブジェクトはPSRインターフェースを継承していませんが、可能な限りPSRスタイルで実装しています。

HTTPリクエストとレスポンスを模倣するFastCGIクラスのソースコードは以下の場所にあります。非常にシンプルで、コードはドキュメントそのものです。

[Swoole\FastCGI\HttpRequest](https://github.com/swoole/library/blob/master/src/core/FastCGI/HttpRequest.php)
[Swoole\FastCGI\HttpResponse](https://github.com/swoole/library/blob/master/src/core/FastCGI/HttpResponse.php)

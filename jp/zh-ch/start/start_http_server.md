# HTTP サーバー

## プログラムコード

以下のコードを`httpServer.php`に書き込んでください。

```php
$http = new Swoole\Http\Server('0.0.0.0', 9501);

$http->on('Request', function ($request, $response) {
    $response->header('Content-Type', 'text/html; charset=utf-8');
    $response->end('<h1>Hello Swoole. #' . rand(1000, 9999) . '</h1>');
});

$http->start();
```

`HTTP`サーバーはリクエストとレスポンスにのみ関心があるため、[onRequest](/http_server?id=on) イベントの監視が必要です。新しい`HTTP`リクエストが発生すると、このイベントがトリガーされます。イベントコールバック関数には`2`つのパラメータがあります。1つは`$request`オブジェクトで、リクエストの関連情報（GET/POSTリクエストデータなど）が含まれています。

もう1つは`response`オブジェクトで、`response`オブジェクトを操作して`request`に対する応答を行うことができます。`$response->end()`メソッドを使用すると、HTMLコンテンツを出力してリクエストを終了します。

* `0.0.0.0`はすべての`IP`アドレスをリッスンすることを意味し、1台のサーバーには複数の`IP`が存在する可能性があります。例えば、`127.0.0.1`はローカルホストIP、`192.168.1.100`はローカルネットワークIP、`210.127.20.2`はグローバルIPです。ここでは、1つのIPを独自に指定してリッスンすることも可能です。
* `9501`はリッスンするポート番号で、プログラムが使用中の場合は致命的なエラーが発生して処理が中断されます。

## サービスの起動

```shell
php httpServer.php
```
* ブラウザで`http://127.0.0.1:9501`にアクセスしてプログラムの結果を確認できます。
* Apacheの`ab`ツールを使用してサーバーに負荷テストを行うこともできます。

## Chrome による二重リクエストの問題

`Chrome`ブラウザでサーバーにアクセスすると、追加のリクエスト`/favicon.ico`が生成されます。これに対処するために、コード内で`404`エラーへの応答が可能です。

```php
$http->on('Request', function ($request, $response) {
	if ($request->server['path_info'] == '/favicon.ico' || $request->server['request_uri'] == '/favicon.ico') {
        $response->end();
        return;
	}
    var_dump($request->get, $request->post);
    $response->header('Content-Type', 'text/html; charset=utf-8');
    $response->end('<h1>Hello Swoole. #' . rand(1000, 9999) . '</h1>');
});
```

## URL ルーティング

アプリケーションは`$request->server['request_uri']`に基づいてルーティングを実現できます。例えば：`http://127.0.0.1:9501/test/index/?a=1`のような`URL`ルーティングを次のように実装できます。

```php
$http->on('Request', function ($request, $response) {
    list($controller, $action) = explode('/', trim($request->server['request_uri'], '/'));
	// $controller、$actionに基づいて異なるコントローラークラスとメソッドにマップする。
	(new $controller)->$action($request, $response);
});
```

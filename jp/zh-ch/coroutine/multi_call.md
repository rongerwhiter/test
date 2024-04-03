# 並行呼び出し

[//]: # (
ここではsetDefer機能を削除しましたが、setDeferをサポートするクライアントはすべてワンクリックコルーチン化を推奨しています。
)

`サブコルーチン（go）` + `チャネル（channel）`を使用して並列リクエストを実現します。

!> まず[概要](/coroutine)を見て、基本的なコルーチンの概念を理解してからこのセクションを見ることをお勧めします。

### 実装原理

* `onRequest`内で2つの`HTTP`リクエストを並行して行う必要があります。`go`関数を使用して`2`つのサブコルーチンを作成し、複数の`URL`に並行してリクエストします
* `チャネル（channel）`を作成し、`use`クロージャ参照構文を使用して、サブコルーチンに渡す
* メインコルーチンは`chan->pop`をループし、サブコルーチンのタスク完了を待ち、`yield`して中断状態に入ります
* 並行して動作する2つのサブコルーチンのうちの1つがリクエストを完了すると、データをメインコルーチンにプッシュするために`chan->push`を呼び出します
* サブコルーチンは`URL`リクエストを完了すると終了し、メインコルーチンは中断状態から復帰して、`$resp->end`を呼び出して応答結果を送信します

### 使用例

```php
$serv = new Swoole\Http\Server("127.0.0.1", 9503, SWOOLE_BASE);

$serv->on('request', function ($req, $resp) {
	$chan = new Channel(2);
	go(function () use ($chan) {
		$cli = new Swoole\Coroutine\Http\Client('www.qq.com', 80);
			$cli->set(['timeout' => 10]);
			$cli->setHeaders([
			'Host' => "www.qq.com",
			"User-Agent" => 'Chrome/49.0.2587.3',
			'Accept' => 'text/html,application/xhtml+xml,application/xml',
			'Accept-Encoding' => 'gzip',
		]);
		$ret = $cli->get('/');
		$chan->push(['www.qq.com' => $cli->body]);
	});

	go(function () use ($chan) {
		$cli = new Swoole\Coroutine\Http\Client('www.163.com', 80);
		$cli->set(['timeout' => 10]);
		$cli->setHeaders([
			'Host' => "www.163.com",
			"User-Agent" => 'Chrome/49.0.2587.3',
			'Accept' => 'text/html,application/xhtml+xml,application/xml',
			'Accept-Encoding' => 'gzip',
		]);
		$ret = $cli->get('/');
		$chan->push(['www.163.com' => $cli->body]);
	});
	
	$result = [];
	for ($i = 0; $i < 2; $i++)
	{
		$result += $chan->pop();
	}
	$resp->end(json_encode($result));
});
$serv->start();
```

!> `Swoole`が提供する[WaitGroup](/coroutine/wait_group)機能を使用すると、もっと簡単になります。

# Coroutine\WaitGroup

`Swoole4`では[Channel](/coroutine/channel)を使用して、コルーチン間の通信、依存関係管理、コルーチン同期を実現できます。[Channel](/coroutine/channel)をベースに、`Golang`の`sync.WaitGroup`機能を簡単に実装することができます。

## 実装コード

> この機能はPHPで記述されたものであり、C/C++のコードではありません。実装ソースコードは[Library](https://github.com/swoole/library/blob/master/src/core/Coroutine/WaitGroup.php)にあります

- `add`メソッドはカウントを増やす
- `done`はタスクが完了したことを示す
- `wait`はすべてのタスクの完了を待って現在のコルーチンの実行を再開する
- `WaitGroup`オブジェクトは再利用することができ、`add`、`done`、`wait`の後に再度使用できます

## 使用例

```php
<?php
use Swoole\Coroutine;
use Swoole\Coroutine\WaitGroup;
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function () {
    $wg = new WaitGroup();
    $result = [];

    $wg->add();
    //最初のコルーチンを起動
    Coroutine::create(function () use ($wg, &$result) {
        //淘宝のホームページへのリクエストを行うコルーチンクライアントを起動
        $cli = new Client('www.taobao.com', 443, true);
        $cli->setHeaders([
            'Host' => 'www.taobao.com',
            'User-Agent' => 'Chrome/49.0.2587.3',
            'Accept' => 'text/html,application/xhtml+xml,application/xml',
            'Accept-Encoding' => 'gzip',
        ]);
        $cli->set(['timeout' => 1]);
        $cli->get('/index.php');

        $result['taobao'] = $cli->body;
        $cli->close();

        $wg->done();
    });

    $wg->add();
    //2番目のコルーチンを起動
    Coroutine::create(function () use ($wg, &$result) {
        //百度のホームページへのリクエストを行うコルーチンクライアントを起動
        $cli = new Client('www.baidu.com', 443, true);
        $cli->setHeaders([
            'Host' => 'www.baidu.com',
            'User-Agent' => 'Chrome/49.0.2587.3',
            'Accept' => 'text/html,application/xhtml+xml,application/xml',
            'Accept-Encoding' => 'gzip',
        ]);
        $cli->set(['timeout' => 1]);
        $cli->get('/index.php');

        $result['baidu'] = $cli->body;
        $cli->close();

        $wg->done();
    });

    //すべてのタスクが完了するのを待って現在のコルーチンを一時停止
    $wg->wait();
    //ここで $result には2つのタスクの実行結果が含まれています
    var_dump($result);
});
```

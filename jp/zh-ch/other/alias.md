# 関数の別名の要約

## コルーチンの短縮名

コルーチン関連の`API`の名前を短く書くことができます。`php.ini`を修正して`swoole.use_shortname=On/Off`を設定して、ショートネームの使用をオン/オフに切り替えることができます。デフォルトはオンです。

全ての `Swoole\Coroutine` の接頭辞が `Co` にマッピングされます。他にも以下のようなマッピングがあります：

### コルーチンの作成

```php
//Swoole\Coroutine::create は go 関数と同等です
go(function () {
	Co::sleep(0.5);
	echo 'hello';
});
go('test');
go([$object, 'method']);
```

### チャネル操作

```php
//Coroutine\Channel は chan に簡略化できます
$c = new chan(1);
$c->push($data);
$c->pop();
```

### 遅延実行

```php
//Swoole\Coroutine::defer は defer を直接使用できます
defer(function () use ($db) {
    $db->close();
});
```

## メソッドの短縮名

!> 以下の方法では、`go` と `defer` は Swoole バージョン >= `v4.6.3` で利用可能です。

```php
use function Swoole\Coroutine\go;
use function Swoole\Coroutine\run;
use function Swoole\Coroutine\defer;

run(function () {
    defer(function () {
        echo "co1 end\n";
    });
    sleep(1);
    go(function () {
        usleep(100000);
        defer(function () {
            echo "co2 end\n";
        });
        echo "co2\n";
    });
    echo "co1\n";
});
```

## コルーチンシステム API

`4.4.4` バージョンでは、システム操作に関連するコルーチン`API`は `Swoole\Coroutine` クラスから `Swoole\Coroutine\System` クラスに移行されました。これは新しいモジュールとして独立しています。下位互換性を維持するために、コルーチンクラスの別名メソッドは依然として維持されています。

* 例：`Swoole\Coroutine::sleep` は `Swoole\Coroutine\System::sleep` に対応しています
* 例：`Swoole\Coroutine::fgets` は `Swoole\Coroutine\System::fgets` に対応しています

## クラスの短縮エイリアスマッピング

!> 名前空間スタイルを使用することをお勧めします。

| アンダースコアクラススタイル | 名前空間スタイル            |
| --------------------------- | --------------------------- |
| swoole_server               | Swoole\Server               |
| swoole_client               | Swoole\Client               |
| swoole_process              | Swoole\Process              |
| swoole_timer                | Swoole\Timer                |
| swoole_table                | Swoole\Table                |
| swoole_lock                 | Swoole\Lock                 |
| swoole_atomic               | Swoole\Atomic               |
| swoole_atomic_long          | Swoole\Atomic\Long          |
| swoole_buffer               | Swoole\Buffer               |
| swoole_redis                | Swoole\Redis                |
| swoole_error                | Swoole\Error                |
| swoole_event                | Swoole\Event                |
| swoole_http_server          | Swoole\Http\Server          |
| swoole_http_client          | Swoole\Http\Client          |
| swoole_http_request         | Swoole\Http\Request         |
| swoole_http_response        | Swoole\Http\Response        |
| swoole_websocket_server     | Swoole\WebSocket\Server     |
| swoole_connection_iterator  | Swoole\Connection\Iterator  |
| swoole_exception            | Swoole\Exception            |
| swoole_http2_request        | Swoole\Http2\Request        |
| swoole_http2_response       | Swoole\Http2\Response       |
| swoole_process_pool         | Swoole\Process\Pool         |
| swoole_redis_server         | Swoole\Redis\Server         |
| swoole_runtime              | Swoole\Runtime              |
| swoole_server_port          | Swoole\Server\Port          |
| swoole_server_task          | Swoole\Server\Task          |
| swoole_table_row            | Swoole\Table\Row            |
| swoole_timer_iterator       | Swoole\Timer\Iterator       |
| swoole_websocket_closeframe | Swoole\Websocket\Closeframe |
| swoole_websocket_frame      | Swoole\Websocket\Frame      |

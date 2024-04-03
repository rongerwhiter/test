# サーバー側（コルーチンスタイル） <!-- {docsify-ignore-all} -->

`Swoole\Coroutine\Server` と[非同期スタイル](/server/init)のサーバーとの違いは、`Swoole\Coroutine\Server` が完全にコルーチン化されたサーバーであることです。参考：[完全な例](/coroutine/server?id=完全示例)。
## 利点：

- イベントコールバック関数の設定が不要です。接続の確立、データの受信、データの送信、接続の終了はすべて順番に行われ、[非同期スタイル](/server/init) の並行性の問題が発生しません。例えば：

```php
$serv = new Swoole\Server("127.0.0.1", 9501);

//接続イベントをリッスン
$serv->on('Connect', function ($serv, $fd) {
    $redis = new Redis();
    $redis->connect("127.0.0.1",6379);//ここでOnConnectのコルーチンが一時停止
    Co::sleep(5);//ここでconnectが遅い状況を模擬するsleep
    $redis->set($fd,"fd $fd connected");
});

//データ受信イベントをリッスン
$serv->on('Receive', function ($serv, $fd, $reactor_id, $data) {
    $redis = new Redis();
    $redis->connect("127.0.0.1",6379);//ここでonReceiveのコルーチンが一時停止
    var_dump($redis->get($fd));//可能性としてonReceiveの単位のredis接続が最初に確立され、上のsetがまだ実行されていないため、ここでgetはfalseになり、論理エラーが発生する可能性があります。
});

//接続クローズイベントをリッスン
$serv->on('Close', function ($serv, $fd) {
    echo "Client: Close.\n";
});

//サーバーを起動
$serv->start();
```

上記の`非同期スタイル`のサーバーでは、イベントの順序を保証できません。つまり、`onConnect`が終了してから`onReceive`に入ることを保証できません。なぜなら、コルーチンを有効にした状態では、`onConnect`や`onReceive`コールバックはどちらも自動的にコルーチンを作成し、IOに遭遇すると[コルーチンスケジュール](/coroutine?id=コルーチンスケジュール)が発生し、非同期スタイルではスケジュールの順序が保証されないため、コルーチンスタイルのサーバーにはこの問題がありません。
- サービスを動的にオンおよびオフにでき、非同期スタイルのサービスは`start()`が呼び出された後は何も実行できず、一方、コルーチンスタイルのサービスは動的にオンオフできます。
## 欠点：

- 協調風格のサービスは複数のプロセスを自動的に作成しませんので、[Process\Pool](/process/process_pool) モジュールを使用して複数のコアを活用する必要があります。
- 協調風格のサービスは実際には[Co\Socket](/coroutine_client/socket) モジュールのラッパーであり、したがって協調風格を使用するには、ソケットプログラミングの経験が必要です。
- 現在、ラッピングのレベルが非同期スタイルのサーバーほど高くないため、一部の機能は手動で実装する必要があります。たとえば、`reload` 機能を使用するには、信号をリッスンしてロジックを実行する必要があります。

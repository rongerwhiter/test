Sure, what is the issue you are facing?
## Swoole性能はどうですか

> QPS比較

Nginxの静的ページ、GolangのHTTPプログラム、PHP7+SwooleのHTTPプログラムを、Apache Benchツール（ab）を使用してストレステストしました。 同じマシンで、並行して100回100万回のHTTPリクエストを行うベンチマークテストで、QPSの比較は次のようになります：

| ソフトウェア | QPS | ソフトウェアバージョン |
| --- | --- | --- |
| Nginx | 164489.92 | nginx/1.4.6 (Ubuntu) |
| Golang | 166838.68 | go version go1.5.2 linux/amd64 |
| PHP7+Swoole | 287104.12 | Swoole-1.7.22-alpha |
| Nginx-1.9.9 | 245058.70 | nginx/1.9.9 |

!> 注：Nginx-1.9.9のテストでは、access_logを無効にし、open_file_cacheを使用して静的ファイルをメモリにキャッシュしています

> テスト環境

* CPU：Intel® Core™ i5-4590 CPU @ 3.30GHz × 4
* メモリ：16G
* ディスク：128G SSD
* OS：Ubuntu14.04（Linux 3.16.0-55-generic）

> テスト方法

```shell
ab -c 100 -n 1000000 -k http://127.0.0.1:8080/
```

> VHOST設定

```nginx
server {
    listen 80 default_server;
    root /data/webroot;
    index index.html;
}
```

> テストページ

```html
<h1>Hello World!</h1>
```

> プロセス数

Nginxは4つのWorkerプロセスを起動しています
```shell
htf@htf-All-Series:~/soft/php-7.0.0$ ps aux|grep nginx
root      1221  0.0  0.0  86300  3304 ?        Ss   12月07   0:00 nginx: master process /usr/sbin/nginx
www-data  1222  0.0  0.0  87316  5440 ?        S    12月07   0:44 nginx: worker process
www-data  1223  0.0  0.0  87184  5388 ?        S    12月07   0:36 nginx: worker process
www-data  1224  0.0  0.0  87000  5520 ?        S    12月07   0:40 nginx: worker process
www-data  1225  0.0  0.0  87524  5516 ?        S    12月07   0:45 nginx: worker process
```

> Golang

テストコード

```go
package main

import (
    "log"
    "net/http"
    "runtime"
)

func main() {
    runtime.GOMAXPROCS(runtime.NumCPU() - 1)

    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Header().Add("Last-Modified", "Thu, 18 Jun 2015 10:24:27 GMT")
        w.Header().Add("Accept-Ranges", "bytes")
        w.Header().Add("E-Tag", "55829c5b-17")
        w.Header().Add("Server", "golang-http-server")
        w.Write([]byte("<h1>\nHello world!\n</h1>\n"))
    })

    log.Printf("Go http Server listen on :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

> PHP7+Swoole

PHP7は`OPcache`アクセラレータを有効にしています。

テストコード

```php
$http = new Swoole\Http\Server("127.0.0.1", 9501, SWOOLE_BASE);

$http->set([
    'worker_num' => 4,
]);

$http->on('request', function ($request, Swoole\Http\Server $response) {
    $response->header('Last-Modified', 'Thu, 18 Jun 2015 10:24:27 GMT');
    $response->header('E-Tag', '55829c5b-17');
    $response->header('Accept-Ranges', 'bytes');    
    $response->end("<h1>\nHello Swoole.\n</h1>");
});

$http->start();
```

> **全世界のWebフレームワーク権威パフォーマンステスト Techempower Web Framework Benchmarks**

最新のベンチマーク結果はこちら: [techempower](https://www.techempower.com/benchmarks/#section=test&runid=9d5522a6-2917-467a-9d7a-8c0f6a8ed790)

Swooleは**動的言語の中でトップ**

データベースIO操作のテストは、特別な最適化を行わずに基本的なビジネスコードを使用しています

**すべての静的言語フレームワークを凌駕する性能（PostgreSQLの代わりにMySQLを使用）**
## Swoole如何维持TCP长连接

TCP長接続を維持するための2組の設定があります：[tcp_keepalive](/server/setting?id=open_tcp_keepalive) と [heartbeat](/server/setting?id=heartbeat_check_interval)。使用方法や注意事項については、[Swoole公式ビデオチュートリアル](https://course.swoole-cloud.com/course-video/10)を参照してください。
日常の開発中、PHPコードを変更するとサービスを再起動してコードを有効にする必要があることがよくあります。忙しいバックエンドサーバーは常にリクエストを処理しており、管理者がプロセスを`kill`してサーバープログラムを終了/再起動すると、ちょうどコードが半分実行されている可能性があり、ビジネスロジックの完全性を確保できません。

`Swoole`は柔軟な終了/再起動メカニズムを提供しており、管理者は`Server`に特定のシグナルを送信したり`reload`メソッドを呼び出すだけで、ワーカープロセスを終了して再起動させることができます。詳細は[reload()](/server/methods?id=reload)を参照してください。

ただし、次の点に注意する必要があります：

まず、新しく変更されたコードは`OnWorkerStart`イベントで再度ロードする必要があります。つまり、あるクラスが`OnWorkerStart`より前にcomposerのautoloadを介して読み込まれた場合には効果がありません。

次に、`reload`はこれら2つのパラメータ[max_wait_time](/server/setting?id=max_wait_time)および[reload_async](/server/setting?id=reload_async)と一緒に使用する必要があり、これら2つのパラメータを設定すると`非同期安全な再起動`が実現できます。

この機能がない場合、Workerプロセスは再起信号を受け取るか、[max_request](/server/setting?id=max_request)に達した時にサービスを即座に停止します。その時点で`Worker`プロセス内にはまだイベントリスンがあるかもしれませんが、これらの非同期タスクは破棄されます。上記のパラメータを設定すると、まず新しい`Worker`を作成し、古い`Worker`はすべてのイベントを完了した後に自己終了します、つまり`reload_async`です。

もし古い`Worker`が終了しない場合、下位層には一定の時間内( [max_wait_time](/server/setting?id=max_wait_time)秒)に古い`Worker`が終了しない場合、下位層が強制的に終了し、[WARNING](/question/use?id=forced-to-terminate)エラーが発生します。

例：

```php
<?php
$serv = new Swoole\Server('0.0.0.0', 9501, SWOOLE_PROCESS);
$serv->set(array(
    'worker_num' => 1,
    'max_wait_time' => 60,
    'reload_async' => true,
));
$serv->on('receive', function (Swoole\Server $serv, $fd, $reactor_id, $data) {

    echo "[#" . $serv->worker_id . "]\tClient[$fd] receive data: $data\n";
    
    Swoole\Timer::tick(5000, function () {
        echo 'tick';
    });
});

$serv->start();
```

上記のコードの場合、`reload_async`がないと、`onReceive`で作成されたタイマーが失われ、タイマーのコールバック関数を処理する機会がありません。
### プロセス終了イベント

非同期リスタート機能をサポートするために、下位レベルに新しい[onWorkerExit](/server/events?id=onWorkerExit)イベントが追加されました。古い`Worker`が終了する直前に、`onWorkerExit`イベントがトリガーされ、このイベントのコールバック関数内では、アプリケーションレイヤーが特定の長期接続`Socket`をクリーンアップし、fdがなくなるか[max_wait_time](/server/setting?id=max_wait_time)でプロセスを終了させることができます。

```php
$serv->on('WorkerExit', function (Swoole\Server $serv, $worker_id) {
    $redisState = $serv->redis->getState();
    if ($redisState == Swoole\Redis::STATE_READY or $redisState == Swoole\Redis::STATE_SUBSCRIBE)
    {
        $serv->redis->close();
    }
});
```

同時に、[Swoole Plus](https://www.swoole.com/swoole_plus)でファイル変更を自動的に検出する機能が追加されました。手動で再読み込みしたりシグナルを送ったりしなくても、ファイルが変更されたらworkerが自動的に再起動します。
## send完了close直接するのはなぜ安全でないんですか

sendが完了した後すぐにcloseするのは安全でないです、サーバーサイドでもクライアントサイドでも同じです。

send操作が成功するだけは、データが操作システムのソケットバッファに書き込まれることを意味し、相手が実際にデータを受け取ったかどうかを示しません。操作システムが送信に成功したかどうか、相手サーバーがデータを受け取ったか、サーバーサイドのプログラムが処理したかどうかは、確実に保証できません。

> close後のロジックについては、以下のlinger設定をご覧ください

このロジックは、電話でのコミュニケーションと同じです。AがBに何かを伝えて、Aが言い終わったら電話を切ります。その後、Bが聞いたかどうかは、Aは分かりません。Aが話し終わって、Bが了解したら、Bが電話を切れば、完全に安全です。

linger設定

ソケットはclose時、バッファにデータが残っている場合、操作システムは`linger`設定に基づいて処理方法を決定します

```c
struct linger
{
     int l_onoff;
     int l_linger;
};
```

- l_onoff = 0の場合、close時に即時にリターンされ、未送信のデータを送信してからリソースを解放し、つまり優雅に終了します。
- l_onoff != 0かつl_linger = 0の場合、close時に即時にリターンされますが、未送信のデータを送信せず、RSTパケットを使用してソケットディスクリプタを強制的にクローズします、つまり強制的に終了します。
- l_onoff != 0かつl_linger > 0の場合、close時に即時にリターンされず、カーネルは一定時間を遅延させます。この時間はl_lingerの値によって決まります。タイムアウトする前に、送信されていないデータ（FINパケットを含む）が送信され、他端から確認を受けると、closeは正常にリターンし、ソケットディスクリプタは優雅に終了します。そうでなければcloseは直ちにエラーを返し、送信されていないデータが失われ、ソケットディスクリプタが強制終了します。ソケットディスクリプタがノンブロッキングモードに設定されている場合、closeは直ちに値を返します。
## クライアントはすでに他のコルーチンにバインドされています

`TCP`接続に関して、Swooleの内部では1つのコルーチンが読み取り操作を行い、別のコルーチンが書き込み操作を同時に行うことが許可されています。つまり、1つのTCPに対して複数のコルーチンが読み取り/書き込み操作を行うことができません。このため、内部でバインディングエラーが発生します：

```shell
Fatal error: Uncaught Swoole\Error: Socket#6 has already been bound to another coroutine#2, reading or writing of the same socket in coroutine#3 at the same time is not allowed 
```

再現するためのコード：

```php
use Swoole\Coroutine;
use Swoole\Coroutine\Http\Client;
use function Swoole\Coroutine\run;

run(function() {
    $cli = new Client('www.xinhuanet.com', 80);
    Coroutine::create(function () use ($cli) {
        $cli->get('/');
    });
    Coroutine::create(function () use ($cli) {
        $cli->get('/');
    });
});
```

解決策はこちらを参照してください：https://wenda.swoole.com/detail/107474

!> この制限はすべてのマルチコルーチン環境で有効であり、最も一般的なのは[onReceive](/server/events?id=onreceive)などのコールバック関数で1つのTCP接続を共有する場合です。なぜなら、このようなコールバック関数は自動的にコルーチンを作成するからです。そして、コネクションプールが必要な場合はどうすればよいのでしょうか？ `Swoole` には組み込みの[コネクションプール](/coroutine/conn_pool)があるため、直接使用するか、手動で`channel`を使用してコネクションプールをラップすることができます。
```bash
PHP Fatal error: Uncaught Error: Call to undefined function Co\run()

PHP Fatal error: Uncaught Error: Call to undefined function go()
```

あなたの`Swoole`の拡張機能のバージョンが`v4.4.0`未満であるか、[コルーチンの短い名前](/other/alias?id=协程短名称)を手動で無効にしている可能性があります。

以下の解決策を提供します。

- もしバージョンが古い場合、拡張機能のバージョンを`>= v4.4.0`にアップグレードするか、`go`キーワードを使用して`Co\run`を置き換えてコルーチンを作成してください。
- コルーチンの短い名前が無効になっている場合は、[こちら](/other/alias?id=协程短名称)を有効にしてください。
- `Co\run`または`go`の代わりに[Coroutine::create](/coroutine/coroutine?id=create)メソッドを使用してコルーチンを作成してください。
- 完全な名前を使用する：`Swoole\Coroutine\run`；
## 1 つのRedisまたはMySQL接続を共有できますか

絶対にできません。各プロセスは独自の `Redis`、`MySQL`、`PDO` 接続を作成する必要があります。他のストレージクライアントも同様です。なぜなら、1 つの接続を共有すると、返される結果がどのプロセスで処理されるかが保証されないためです。接続を保持しているプロセスは理論的にその接続を読み書きすることができるため、データが混乱します。

**したがって、複数のプロセス間では接続を共有してはいけません**

- [Swoole\Server](/server/init)では、[onWorkerStart](/server/events?id=onworkerstart)内で接続オブジェクトを作成する必要があります。
- [Swoole\Process](/process/process)では、[Swoole\Process->start](/process/process?id=start)後、子プロセスのコールバック関数内で接続オブジェクトを作成する必要があります。
- この問題は `pcntl_fork` を使用したプログラムにも同様に適用されます。

例：

```php
$server = new Swoole\Server('0.0.0.0', 9502);

// Redis/MySQL接続はonWorkerStartコールバック内で作成する必要があります
$server->on('workerstart', function($server, $id) {
    $redis = new Redis();
	$redis->connect('127.0.0.1', 6379);
	$server->redis = $redis;
});

$server->on('receive', function (Swoole\Server $server, $fd, $reactor_id, $data) {	
	$value = $server->redis->get("key");
	$server->send($fd, "Swoole: ".$value);
});

$server->start();
```
## 接続が閉じられた問題

以下の通知が表示されています

```bash
NOTICE swFactoryProcess_finish (ERRNO 1004): send 165 byte failed, because connection[fd=123] is closed

NOTICE swFactoryProcess_finish (ERROR 1005): connection[fd=123] does not exists
```

サーバーの応答時に、クライアントが接続を切断したためです

一般的には:

* ブラウザがページを繰り返しリフレッシュし続ける（まだ読み込み中にリフレッシュ）
* ab によるテストが途中でキャンセルされる
* 時間に基づいた wrk によるテスト（時間が来ると完了しなかったリクエストがキャンセルされる）

これらの状況はすべて正常な現象であり、無視して問題ありません。そのため、このエラーのレベルはNOTICEです

その他の理由で大量の接続が断たれる場合のみ、注意が必要です

```bash
WARNING swWorker_discard_data (ERRNO 1007): [2] received the wrong data[21 bytes] from socket#75

WARNING Worker_discard_data (ERRNO 1007): [2] ignore data[5 bytes] received from session#2
```

同様に、このエラーも接続がすでに閉じられていることを示しており、受信したデータは破棄されます。[discard_timeout_request](/server/setting?id=discard_timeout_request)を参照してください.
## connected属性和接続状態の不整合

4.xコルーチンバージョン以降、`connected`属性はもはやリアルタイムで更新されません。[isConnect](/client?id=isconnected)メソッドはもはや信頼性がありません。
### 原因

協調の目標は、同期ブロッキングプログラミングモデルと一致することです。 同期ブロッキングモデルでは、リアルタイムで接続の状態を更新する概念はありません。PDO、curlなど、接続の概念がなく、IO操作中にエラーが発生したり例外がスローされることで接続が切断されたことがわかります。

Swooleの内部で一般的な方法は、IOエラーが発生した場合にfalseを返し（または空のコンテンツで接続が切断されたことを示し）、クライアントオブジェクトに対応するエラーコードとエラーメッセージを設定することです。
### 注意

以前の非同期バージョンは`connected`プロパティの"リアルタイム"更新をサポートしていましたが、実際には信頼性がありません。接続は確認した後すぐに切断される可能性があります。
## "Connection refused" とは何ですか

telnet 127.0.0.1 9501 を実行した際に "Connection refused" エラーが発生すると、サーバーがこのポートをリッスンしていないことを意味します。

* プログラムが正常に実行されているか確認してください: `ps aux`
* ポートがリッスンされているか確認してください: `netstat -lp`
* ネットワーク通信過程が正常かどうか確認してください: `tcpdump`、`traceroute`
## リソースが一時的に利用できません [11]

クライアントのswoole_clientが`recv`中に次のエラーを返します。

```shell
swoole_client::recv(): recv() failed. Error: Resource temporarily unavailable [11]
```

このエラーは、サーバー側が指定された時間内にデータを返さず、受信タイムアウトしたことを示しています。

- ネットワーク通信プロセスを確認するためにtcpdumpを使用して、サーバーがデータを送信しているかどうかを確認します。
- サーバーの`$serv->send`関数がtrueを返すかどうかを確認する必要があります。
- インターネット経由で通信する場合は、swoole_clientのタイムアウト時間を増やす必要があります。
## ワーカーの終了タイムアウト、強制終了：id=forced-to-terminate

次のようなエラーメッセージが見つかりました：

```bash
WARNING swWorker_reactor_try_to_exit (ERRNO 9012): worker exit timeout, forced to terminate
```

これは、指定された時間内（[max_wait_time](/server/setting?id=max_wait_time)秒）にこのワーカーが終了しなかった場合、Swooleの基盤がこのプロセスを強制終了することを意味します。

次のコードを使用して再現できます：

```php
use Swoole\Timer;

$server = new Swoole\Server('127.0.0.1', 9501);
$server->set(
    [
        'reload_async' => true,
        'max_wait_time' => 4,
    ]
);

$server->on('workerStart', function (Swoole\Server $server, int $wid) {
    if ($wid === 0) {
        Timer::tick(5000, function () {
            echo 'tick';
        });
        Timer::after(500, function () use ($server) {
            $server->shutdown();
        });
    }
});

$server->on('receive', function () {

});

$server->start();
```
以下のようなエラーメッセージが見つかりました：

```bash
WARNING swSignalfd_onSignal (ERRNO 707): Unable to find callback function for signal Broken pipe: 13
```

このエラーメッセージは、切断された接続にデータを送信しようとしたことを示しています。通常、送信の戻り値を確認せずに失敗した場合でも送信を続けるためです。
## 学習に必要な基本知識

### 多プロセス/マルチスレッド

- `Linux`オペレーティングシステムのプロセスとスレッドの概念を理解する
- `Linux`のプロセス/スレッドの切り替えスケジューリングの基本を理解する
- プロセス間通信の基本を理解する、例えばパイプ、`UnixSocket`、メッセージキュー、共有メモリ
### ソケット

* `accept/connect`、`send/recv`、`close`、`listen`、`bind`などの基本的な`ソケット`操作についての理解
* 受信バッファ、送信バッファ、ブロッキング/ノンブロッキング、タイムアウトなどの概念についての理解
### IOマルチプレクシング

* `select`/`poll`/`epoll`を理解する
* `select`/`epoll`をベースとしたイベントループ、`Reactor`モデルの実装を理解する
* リード可能イベント、ライト可能イベントを理解する
### TCP/IPネットワークプロトコル

* `TCP/IP`プロトコルの理解
* `TCP`、`UDP`転送プロトコルの理解
### デバッグツール

* [gdb](/other/tools?id=gdb) を使用して `Linux` プログラムをデバッグする
* [strace](/other/tools?id=strace) を使用してプロセスのシステムコールをトレースする
* [tcpdump](/other/tools?id=tcpdump) を使用してネットワーク通信をトレースする
* ps、[lsof](/other/tools?id=lsof)、top、vmstat、netstat、sar、ss などの他の `Linux` システムツール
## Swoole\Curl\Handler クラスのオブジェクトを int に変換できません

[SWOOLE_HOOK_CURL](/runtime?id=swoole_hook_curl) を使用する際に、次のエラーが発生しました：

```bash
PHP Notice:  Object of class Swoole\Curl\Handler could not be converted to int

PHP Warning: curl_multi_add_handle() expects parameter 2 to be resource, object given
```

この問題の原因は、フック後の curl がリソース型ではなくオブジェクト型になっているため、int 型に変換できないことです。

!> `int` の問題は、SDK 開発者に修正を依頼することをお勧めします。PHP8 では curl がリソース型ではなくオブジェクト型になっています。

問題を解決する方法は3つあります：

1. [SWOOLE_HOOK_CURL](/runtime?id=swoole_hook_curl) を無効にする。ただし、[v4.5.4](/version/log?id=v454) 以降、[SWOOLE_HOOK_ALL](/runtime?id=swoole_hook_all) はデフォルトで [SWOOLE_HOOK_CURL](/runtime?id=swoole_hook_curl) を含んでいます。[SWOOLE_HOOK_ALL ^ SWOOLE_HOOK_CURL] を設定して、[SWOOLE_HOOK_CURL](/runtime?id=swoole_hook_curl) を無効にできます。

2. Guzzle のSDK を使用し、Handler を置換して非同期処理を実現します。

3. Swoole `v4.6.0` 以降、[SWOOLE_HOOK_NATIVE_CURL](/runtime?id=swoole_hook_native_curl) を使用することで、[SWOOLE_HOOK_CURL](/runtime?id=swoole_hook_curl) を代替できます。
## 一括して协調可能で、かつ Guzzle 7.0+ を使用した場合、リクエストを発行すると結果が直接端末に出力されます :id=hook_guzzle

再現するコードは以下の通りです

```php
// composer require guzzlehttp/guzzle
include __DIR__ . '/vendor/autoload.php';

use GuzzleHttp\Client;
use Swoole\Coroutine;

// v4.5.4 以前のバージョン
//Coroutine::set(['hook_flags' => SWOOLE_HOOK_ALL | SWOOLE_HOOK_CURL]);
Coroutine::set(['hook_flags' => SWOOLE_HOOK_ALL]);
Coroutine\run(function () {
    $client = new Client();
    $url = 'http://baidu.com';
    $res = $client->request('GET', $url);
    var_dump($res->getBody()->getContents());
});

// リクエスト結果が直接出力され、印刷されません
//<html>
//<meta http-equiv="refresh" content="0;url=http://www.baidu.com/">
//</html>
//string(0) ""
```

!> 解決方法は前述の問題と同じです。ただし、この問題は Swoole バージョン`v4.5.8`以上で修正されています。
## エラー: 利用可能なバッファがありません[55]

このエラーを無視してもかまいません。このエラーは、[socket_buffer_size](/server/setting?id=socket_buffer_size) オプションが大きすぎて一部のシステムが受け入れていないためであり、プログラムの実行には影響しません。
## GET/POSTリクエストの最大サイズ
### GETリクエストの最大8192

GETリクエストは1つのHttpヘッダしか持たず、Swooleの内部では固定サイズの8Kメモリバッファが使用され、変更できません。正しいHttpリクエストでない場合、エラーが発生します。内部で以下のエラーが発生します：

```bash
WARN swReactorThread_onReceive_http_request: http header is too long.
```
### POSTファイルのアップロード

最大サイズは [package_max_length](/server/setting?id=package_max_length) 設定値の制限を受けます。デフォルトでは2Mで、新しい値を渡すには [Server->set](/server/methods?id=set) を呼び出してサイズを変更できます。Swooleの下層は完全なメモリベースですので、大きすぎる値を設定すると多くの並行リクエストがサーバーリソースを枯渇させる可能性があります。

計算方法：`最大メモリ使用量` = `最大並行リクエスト数` * `package_max_length`

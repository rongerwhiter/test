# Basic knowledge
## コールバック関数の設定方法

* **無名関数**

```php
$server->on('Request', function ($req, $resp) use ($a, $b, $c) {
    echo "hello world";
});
```
!> `use` を使って無名関数にパラメータを渡すことができます

* **クラスの静的メソッド**

```php
class A
{
    static function test($req, $resp)
    {
        echo "hello world";
    }
}
$server->on('Request', 'A::Test');
$server->on('Request', array('A', 'Test'));
```
!> 対応する静的メソッドは`public`である必要があります

* **関数**

```php
function my_onRequest($req, $resp)
{
    echo "hello world";
}
$server->on('Request', 'my_onRequest');
```

* **オブジェクトのメソッド**

```php
class A
{
    function test($req, $resp)
    {
        echo "hello world";
    }
}

$object = new A();
$server->on('Request', array($object, 'test'));
```

!> 対応するメソッドは`public`である必要があります
## 同期IO / 非同期IO

すべてのビジネスコードは `Swoole4+` で同期形式です（`Swoole1.x` 時代は非同期形式をサポートしていましたが、現在は非同期クライアントは削除され、コルーチンクライアントを使用して対応できます）、心配する必要が全くありませんし、人間の思考に合っていますが、同期形式の場合、低レイヤーには `同期IO / 非同期IO` の違いがあるかもしれません。

同期IO / 非同期IO のいずれであっても、`Swoole/Server` は多数の `TCP` クライアント接続を維持できます（[SWOOLE_PROCESS モード](/learn?id=swoole_process)を参照）。あなたのサービスがブロックING しているかどうかを設定するための特定のパラメータは不要です、それはあなたのコードが中に同期IO操作が含まれるかによります。

**同期IO とは何か：**
 
簡単な例として、`MySQL->query` を実行すると、このプロセスは何もせず、MySQL から結果を待ちます。結果が返された後にコードを再開します。つまり、同期IO サービスの並行性は非常に低いです。

**どのようなコードが同期IO であるか：**

- [ワンキーコルーチン化](/runtime)が有効になっていない場合、コード内のほとんどのIO操作は同期IOです。コルーチン化をすると、非同期IO に変わり、プロセスがそこで待つことなく進行します。[コルーチンスケジュール](/coroutine?id=コルーチンスケジュール)を参照。
- 一部の `IO` はワンクリックでコルーチン化できず、同期IO を非同期IOに変換することができません。例えば `MongoDB`（`Swoole` がこの問題を解決すると信じています）、コードを書く際に気を付ける必要があります。
!> [協力](/coroutine) は並行性を向上させるために設計されており、私のアプリケーションが高い並行性を必要としないか、または特定の非同期化できないIO操作（たとえば、前述のMongoDBなど）を行う必要がある場合は、[コルーチンを有効にする](/runtime)ことなく、[enable_coroutine](/server/setting?id=enable_coroutine)を無効にして、いくつかの`Worker`プロセスを増やすことで `FPM / Apache` と同じモデルにすることができます。`Swoole` が[永続プロセス](https://course.swoole-cloud.com/course-video/80)であるため、同期IOの性能も大幅に向上し、実際のアプリケーションでは多くの企業がこの方法を採用しています。
## 同期IO / 非同期IO

`Swoole4+`の下、すべてのビジネスコードは同期的な書き方です（`Swoole1.x`時代に非同期の書き方をサポートしていましたが、今では非同期クライアントが削除され、その要求は完全にコルーチンクライアントで実現できるようになりました）。心の負担はまったくなく、人間の思考習慣に合っており、同期的な書き方は変わらずに、底層での実装は`同期IO / 非同期IO`の違いがあるかもしれません。

同期IO / 非同期IOに関係なく、`Swoole/Server`は大量の`TCP`クライアント接続を管理できます（[SWOOLE_PROCESSモード](/learn?id=swoole_process)を参照）。あなたのサービスがブロッキングまたはノンブロッキングであるかを別々に設定する必要はなく、同期IOの操作がコード内にあるかどうかに依存します。

**同期IOとは何か：**

簡単な例は、`MySQL->query`の実行時、プロセスは何もせずにMySQLの結果を待ち、結果を受け取った後にコードを続行します。そのため、同期IOサービスの並行性は非常に低いです。

**どのようなコードが同期IOか：**

- [一括コルーチン化](/runtime)が有効でない場合、コード内のほとんどのIO関連操作は同期IOとなります。コルーチン化すると、非同期IOになり、プロセスは単にそこで無駄に待機しません。[コルーチンスケジューリング](/coroutine?id=コルーチンスケジューリング)を参照してください。
- 一部のIOは一括コルーチン化できず、同期IOを非同期IOに変換できません。例えば、`MongoDB`（`Swoole`がこの問題を解決すると信じています）は、コードを書く際に注意が必要です。
!> [协程](/coroutine) は並行性を向上するために使用されます。もしアプリケーションが高い並行性を持っていない場合や、非同期IOを行うことができない操作（先に述べたMongoDBなど）を行う必要がある場合は、[一括協調化](/runtime)を有効にする必要はありません。その代わりに、`Worker`プロセスを複数開いて、`enable_coroutine`を無効にしてください。これにより、`Fpm/Apache`と同じモデルになります。`Swoole`は[常駐プロセス](https://course.swoole-cloud.com/course-video/80)であるため、同期IOでも性能が大幅に向上します。実際、多くの企業がこのようにしています。
### 同期IOを非同期IOに変換する

[前の節](/learn?id=同期io异步io)では、同期/非同期IOが何であるかを紹介しました。 `Swoole`の下では、いくつかの同期`IO`操作を非同期IOに変換することができます。

- [ワンクリック協調化](/runtime)を有効にすると、`MySQL`、`Redis`、`Curl`などの操作は非同期IOに変わります。
- [Event](/event)モジュールを使用してイベントを手動管理し、fdを[EventLoop](/learn?id=什么是eventloop)に追加して非同期IOに変換することができます。以下は例です：

```php
// inotifyを使用してファイル変更を監視
$fd = inotify_init();
// $fdをSwooleのEventLoopに追加
Swoole\Event::add($fd, function () use ($fd){
    $var = inotify_read($fd);//ファイルが変更された後、変更されたファイルを読み込む。
    var_dump($var);
});
```

上記のコードは、`Swoole\Event::add`を呼び出さないとIOが非同期化されず、直接`inotify_read()`を呼び出すとWorkerプロセスがブロックされ、他のリクエストが処理されなくなります。

- `Swoole\Server`の[sendMessage()](/server/methods?id=sendMessage)メソッドを使用してプロセス間通信を行う場合、デフォルトで`sendMessage`は同期IOですが、いくつかの場合には`Swoole`が非同期IOに変換することがあります。[Userプロセス](/server/methods?id=addprocess)を使って次の例を示します：

```php
$serv = new Swoole\Server("0.0.0.0", 9501, SWOOLE_BASE);
$serv->set(
    [
        'worker_num' => 1,
    ]
);

$serv->on('pipeMessage', function ($serv, $src_worker_id, $data) {
    echo "#{$serv->worker_id} message from #$src_worker_id: $data\n";
    sleep(10);//sendMessageからのデータを受信しないと、バッファはすぐにいっぱいになります
});
```
```php
$serv->on('receive', function (swoole_server $serv, $fd, $reactor_id, $data) {

});

//情况1：同步IO(默认行为)
$userProcess = new Swoole\Process(function ($worker) use ($serv) {
    while (1) {
        var_dump($serv->sendMessage("big string", 0));//デフォルトでは、バッファが一杯になるとここでブロックされます
    }
}, false);

//情况2：enable_coroutineパラメータを使用してUserProcessプロセスのコルーチンサポートを有効にする。他のコルーチンがEventLoopのスケジューリングを受けないようにするため、
//SwooleはsendMessageを非同期IOに変換します
$enable_coroutine = true;
$userProcess = new Swoole\Process(function ($worker) use ($serv) {
    while (1) {
        var_dump($serv->sendMessage("big string", 0));//バッファが一杯になっても、プロセスはブロックされずエラーになります
    }
}, false, 1, $enable_coroutine);

//情况3：UserProcessプロセス内で非同期コールバックが設定されている場合（例：タイマーの設定、Swoole\Event::addなど）は、他のコールバック関数がEventLoopのスケジューリングを受けないようにするため、
//SwooleはsendMessageを非同期IOに変換します
$userProcess = new Swoole\Process(function ($worker) use ($serv) {
    swoole_timer_tick(2000, function ($interval) use ($worker, $serv) {
        echo "timer\n";
    });
    while (1) {
        var_dump($serv->sendMessage("big string", 0));//バッファが一杯になっても、プロセスはブロックされずエラーになります
    }
}, false);

$serv->addProcess($userProcess);

$serv->start();
```
- Similarly, the communication between processes in the [Task process](/learn?id=taskworker-process) through `sendMessage()` is the same. The difference is that enabling coroutine support for task processes is done through the Server's [task_enable_coroutine](/server/setting?id=task_enable_coroutine) configuration, and there is no `scenario 3`, meaning that the task process does not turn sendMessage asynchronous IO by enabling asynchronous callbacks.
## EventLoopとは

`EventLoop`とは、イベントループのことであり、簡単に言うと`epoll_wait`のことです。すべてのイベントが発生するハンドル（fd）が`epoll_wait`に追加されます。これらのイベントには、読み込み可能、書き込み可能、エラーなどが含まれます。

関連するプロセスは`epoll_wait`というカーネル関数でブロックされ、イベント（またはタイムアウト）が発生すると`epoll_wait`関数はブロックを解除し、結果を返します。この後、対応するPHP関数をコールバックできます。たとえば、クライアントからのデータを受信したときには、`onReceive`コールバック関数をコールバックします。

大量のfdが`epoll_wait`に配置され、同時に多くのイベントが発生すると、`epoll_wait`関数が返されるときには、それぞれのコールバック関数が順番に呼び出されます。これをイベントループと呼びます。すなわち、I/Oマルチプレクシングを行い、その後再度ブロックして`epoll_wait`を呼び出して次のイベントループを実行します。
## 同期IO / 非同期IO

すべてのビジネスコードは`Swoole4+`では同期スタイルで記述されており（`Swoole1.x`時代に非同期スタイルもサポートされていましたが、現在は非同期クライアントが削除され、コルーチンクライアントで完全に代替できるようになりました）、心理的負担がほとんどないですし、人間の考え方に適していますが、同期スタイルのコードの下には`同期I/O / 非同期I/O`の違いが隠れています。

同期I/Oか非同期I/Oかに関わらず、`Swoole/Server`は多くの`TCP`クライアント接続を維持できます（[SWOOLE_PROCESSモード](/learn?id=swoole_process)を参照）。あなたのサービスがブロッキングかノンブロッキングかは、特定のパラメータを個別に設定する必要はなく、あなたのコード内に同期I/O操作が含まれているかどうかによります。

**同期I/Oとは何ですか：**
 
簡単な例は、`MySQL->query`を実行するときです。このプロセスはMySQLの結果を待って、結果が返ってきた後にコードを続行します。そのため、同期I/Oのサービスの並行性は非常に低いです。

**どのようなコードが同期I/Oですか：**
 * [一括コルーチン化を有効にしていない](/runtime)場合は、ほとんどのI/O操作が同期I/Oとなります。一括コルーチン化すると、非同期I/Oに変わり、プロセスが単にそこで待っているわけではなくなります。[コルーチンスケジューリング](/coroutine?id=コルーチンスケジューリング)を参照してください。
 * いくつかのI/O操作は一括でコルーチン化できず、同期I/Oを非同期I/Oに変換することができない場合があります。例えば`MongoDB`（おそらく`Swoole`がこの問題を解決すると信じています）、コーディング時に注意が必要です。
!> [Coroutine](/coroutine) は、並行性を向上させるためのものです。もし私のアプリケーションが高い並行性を必要とせず、または特定の非同期IO操作（例：先述のMongoDB）を必要とする場合、[Coroutineを一括有効化](/runtime)しないで、[enable_coroutine](/server/setting?id=enable_coroutine) を無効化し、いくつかの追加`Worker`プロセスを起動することで、これは`Fpm/Apache`と同様のモデルになります。`Swoole`は [常駐プロセス](https://course.swoole-cloud.com/course-video/80) であり、同期IOのパフォーマンスも大幅に向上するため、実際のアプリケーションでは多くの企業がこのようにしています。
### 同期I / Oから非同期I / Oへの変換

[前セクション](/learn?id=同期io异步io)では、同期および非同期I / Oの概要が説明されていました。`Swoole`では、特定の状況で同期I / O操作を非同期I / Oに変換することができます。

- [ワンクリック協調化](/runtime)を有効にすると、`MySQL`、`Redis`、`Curl`などの操作が非同期I / Oに変更されます。
- [Event](/event)モジュールを使用してイベントを手動管理し、fdを[EventLoop](/learn?id=什么是eventloop)に追加して非同期I / Oに変換することができます。例：

```php
// inotifyを使用してファイル変更を監視
$fd = inotify_init();
// $fdをSwooleのEventLoopに追加
Swoole\Event::add($fd, function () use ($fd) {
    $var = inotify_read($fd); // ファイルが変更された後に変更されたファイルを読み取る
    var_dump($var);
});
```

上記のコードでは、`Swoole\Event::add`を呼び出さないと、IOが非同期化されず、直接`inotify_read（）`がブロッキングされ、他のリクエストが処理されなくなります。

- `Swoole\Server`の[sendMessage()](/server/methods?id=sendMessage)メソッドを使用してプロセス間通信を行う場合、デフォルトでは`sendMessage`は同期I / Oですが、一部の状況では`Swoole`によって非同期I / Oに変換されることがあります。[Userプロセス](/server/methods?id=addprocess)を使用した例：

```php
$serv = new Swoole\Server("0.0.0.0", 9501, SWOOLE_BASE);
$serv->set(
    [
        'worker_num' => 1,
    ]
);

$serv->on('pipeMessage', function ($serv, $src_worker_id, $data) {
    echo "#{$serv->worker_id} message from #$src_worker_id: $data\n";
    sleep(10); // sendMessageからのデータを受け取らない場合、バッファがすぐにいっぱいになるでしょう
});
```php
$serv->on('receive', function (swoole_server $serv, $fd, $reactor_id, $data) {

});

//情况1：同步IO(默认行为)
$userProcess = new Swoole\Process(function ($worker) use ($serv) {
    while (1) {
        var_dump($serv->sendMessage("big string", 0));//デフォルトでは、バッファがいっぱいになるとここでブロックされる
    }
}, false);

//情况2：通过enable_coroutine参数开启UserProcess进程的协程支持，为了防止其他协程得不到 EventLoop 的调度，
//Swoole会把sendMessage转换成异步IO
$enable_coroutine = true;
$userProcess = new Swoole\Process(function ($worker) use ($serv) {
    while (1) {
        var_dump($serv->sendMessage("big string", 0));//バッファがいっぱいになるとプロセスがブロックされず、エラーが発生します
    }
}, false, 1, $enable_coroutine);

//情况3：在UserProcess进程里面如果设置了异步回调(例如设置定时器、Swoole\Event::add等)，
//为了防止其他回调函数得不到 EventLoop 的调度，Swoole会把sendMessage转换成异步IO
$userProcess = new Swoole\Process(function ($worker) use ($serv) {
    swoole_timer_tick(2000, function ($interval) use ($worker, $serv) {
        echo "timer\n";
    });
    while (1) {
        var_dump($serv->sendMessage("big string", 0));//バッファがいっぱいになるとプロセスがブロックされず、エラーが発生します
    }
}, false);

$serv->addProcess($userProcess);

$serv->start();
```
- 同様に、[Taskプロセス](/learn?id=taskworkerプロセス)は`sendMessage()`メソッドを使用してプロセス間通信を行いますが、異なるのは、taskプロセスが非同期コールバックをサポートするためにはServerの[task_enable_coroutine](/server/setting?id=task_enable_coroutine)設定で有効にする必要があり、`ケース3`は存在しないということです。つまり、taskプロセスはsendMessageを非同期IOに変換することはありません。
## TCPデータパケットの境界問題

[こちらの高速スタートアップコード](/start/start_tcp_server)は、並行していない状況では正常に動作しますが、高い並行性だとTCPデータパケットの境界問題が発生する可能性があります。`TCP`プロトコルは`UDP`プロトコルの順序とパケットロストの再送問題を解決していますが、その代わりに新たな問題を引き起こしています。`TCP`プロトコルはストリーム形式であり、データパケットには境界がないため、アプリケーションが`TCP`通信を使用するとこれらの問題に直面することになります。一般には、これをTCPスティッキーパケット問題と呼びます。

`TCP`通信はストリーム形式であるため、1つの大きなデータパケットを受信する際には、複数のデータパケットに分割されて送信される可能性があります。また、複数回の`Send`操作は、内部で1回の送信として統合される場合もあります。この問題を解決するためには、2つの操作が必要です：

- 分割: `Server`は複数のデータパケットを受信した場合、データパケットを分割する必要があります。
- 統合: `Server`が受信したデータはパケットの一部である可能性があり、データをバッファリングして完全なパケットに統合する必要があります。

したがって、TCPネットワーク通信時には通信プロトコルを設定する必要があります。一般的なTCP汎用ネットワーク通信プロトコルには、`HTTP`、`HTTPS`、`FTP`、`SMTP`、`POP3`、`IMAP`、`SSH`、`Redis`、`Memcache`、`MySQL`などがあります。

Swooleには、多くの一般的なプロトコルの解析が組み込まれており、これらのプロトコルに関するサーバーのTCPデータパケットの境界問題を解決するための設定が簡単に行えます。[open_http_protocol](/server/setting?id=open_http_protocol)/[open_http2_protocol](/http_server?id=open_http2_protocol)/[open_websocket_protocol](/server/setting?id=open_websocket_protocol)/[open_mqtt_protocol](/server/setting?id=open_mqtt_protocol)を参照してください。
除了一般のプロトコル外、`Swoole`は`2`種類のカスタムネットワーク通信プロトコルをサポートしています。

* **EOF終了文字プロトコル**

`EOF`プロトコルの処理原理は、各データパケットの末尾に特定の文字列を追加してパケットの終了を示すことです。`Memcache`、`FTP`、`SMTP`などはすべて終了記号として`\r\n`を使用しています。データを送信する際には、パケットの末尾に`\r\n`を追加するだけで済みます。`EOF`プロトコルを使用する場合、データパケットの途中に`EOF`が出現しないように注意する必要があります。そうでなければ、パケットの分割エラーが発生します。

`Server`と`Client`のコードでは、`EOF`プロトコルを処理するために`2`つのパラメータの設定しか必要ありません。

```php
$server->set(array(
    'open_eof_split' => true,
    'package_eof' => "\r\n",
));
$client->set(array(
    'open_eof_split' => true,
    'package_eof' => "\r\n",
));
```

ただし、上記の`EOF`構成はパフォーマンスが低い可能性があります。`Swoole`は各バイトを走査し、データが`\r\n`であるかどうかを確認します。上記の方法以外にも以下のように設定することもできます。

```php
$server->set(array(
    'open_eof_check' => true,
    'package_eof' => "\r\n",
));
$client->set(array(
    'open_eof_check' => true,
    'package_eof' => "\r\n",
));
```
この設定はパフォーマンスが向上し、データを走査する必要がなくなりますが、`分割`問題のみを解決し、`結合`問題は解決できません。つまり、`onReceive`でクライアントから複数のリクエストを一度に受け取る可能性があり、自分でパケット分割を行う必要があります。例えば、`explode("\r\n", $data)`などです。この設定の主な用途は、リクエストと応答型のサービス（例：端末でコマンドを入力する）であり、データの分割について考慮する必要がない点にあります。
原因は、クライアントがリクエストを送信した後、サーバーから現在のリクエストのレスポンスデータを受け取るまで、2番目のリクエストを送信しません。リクエストを同時に送信しません。

* **ヘッダーとボディの固定プロトコル**

固定ヘッダーの方法は非常に一般的で、サーバーサイドのプログラムでよく見られます。この種のプロトコルの特徴は、データパケットが常にヘッダー+ボディの2つの部分で構成されていることです。ヘッダーはフィールドでパケットまたは全体の長さを指定し、通常は`2`バイトまたは`4`バイトの整数を使用して長さを表します。サーバーはヘッダーを受け取った後、長さの値に基づいて、どのくらいのデータを再受信するかを正確に制御できます。`Swoole`の構成は、この種のプロトコルを柔軟にサポートし、すべてのケースに対処するために`4`つのパラメータを柔軟に設定できます。

`Server`は[onReceive](/server/events?id=onreceive)コールバック関数でデータパケットを処理し、プロトコル処理を設定すると、完全なデータパケットが受信されたときにのみ[onReceive](/server/events?id=onreceive)イベントがトリガーされます。クライアントはプロトコル処理を設定した後、 [$client->recv()](/client?id=recv) を呼び出す際には長さを引数として渡す必要がなくなります。`recv`関数は、完全なデータパケットまたはエラーが発生した場合に返ります。

```php
$server->set(array(
    'open_length_check' => true,
    'package_max_length' => 81920,
    'package_length_type' => 'n', //see php pack()
    'package_length_offset' => 0,
    'package_body_offset' => 2,
));
```

!> 各設定の具体的な意味については、`サーバー/クライアント`セクションの[設定](/server/setting?id=open_length_check)を参照してください。
## 同期IO / 非同期IO

すべてのビジネスコードは`Swoole4+`の下で同期的に書かれており（`Swoole1.x`の時代には非同期の書き方がサポートされていましたが、現在は非同期クライアントが削除され、コルーチンクライアントで完全に対応できます）、心の負担は全くなく、人間の思考習慣に適合していますが、同期的な書き方の下には `同期IO / 非同期IO` の違いがあるかもしれません。

同期IO / 非同期IOに関係なく、`Swoole/Server`は大量の`TCP`クライアント接続を維持できます（[SWOOLE_PROCESSモード](/learn?id=swoole_process)を参照）。サービスがブロッキングか非ブロッキングかを特定のパラメータを別途構成する必要はありません。それはあなたのコード自体が同期IOの操作を含んでいるかどうかに依存します。

**同期IOとは何ですか：**

簡単な例として、`MySQL->query`を実行するとき、プロセスは何もせずにMySQLから結果を待ち、結果が返された後にコードを続行しますので、同期IOサービスの並行性は非常に低いです。

**どのようなコードが同期IOですか：**

 * [単一クリック協調化](/runtime)が有効になっていない場合、コードのほとんどは同期IOに関するものです。協調化が行われると、非同期IOに変換され、プロセスが待ち構えることはありません。[コルーチンスケジューリング](/coroutine?id=コルーチンスケジューリング)を参照してください。
 * 一部の`IO`では協調化ができず、同期IOを非同期IOに変換することができないことがあります。たとえば`MongoDB`（Swooleがこの問題を解決すると信じています）、コードを書く際に注意が必要です。
!> 协程 是为了提高并发的，如果我的应用就没有高并发，或者必须要用某些无法异步化IO的操作(例如上文的MongoDB)，那么你完全可以不开启 一键协程化，关闭 enable_coroutine，多开一些`Worker`进程，这就是和`Fpm/Apache`是一样的模型了，值得一提的是由于`Swoole`是 常驻进程 的，即使同步IO性能也会有很大提升，实际应用中也有很多公司这样做。
### 同期IOを非同期IOに変換する

[前の章](/learn?id=同期io异步io)では、同期/非同期IOについて説明しました。`Swoole`の場合、いくつかの同期`IO`操作は非同期`IO`に変換することができます。

- [一括協調化を有効にする](/runtime)と、`MySQL`、`Redis`、`Curl`などの操作が非同期IOに変わります。
- [Event](/event)モジュールを使用してイベントを手動で管理し、fdを[EventLoop](/learn?id=什么是eventloop)に追加して非同期IOに変換します。例：

```php
// inotifyを使ってファイルの変更を監視
$fd = inotify_init();
// $fdをSwooleのEventLoopに追加
Swoole\Event::add($fd, function () use ($fd){
    $var = inotify_read($fd);// ファイルが変更された後、変更されたファイルを読み取る。
    var_dump($var);
});
```

上記のコードは、`Swoole\Event::add`を呼び出さないと、IOが非同期化されず、直接`inotify_read()`を実行するとWorkerプロセスがブロックされ、他のリクエストは処理されません。

- `Swoole\Server`の[sendMessage()](/server/methods?id=sendMessage)メソッドを使用してプロセス間通信を行うと、デフォルトでは`sendMessage`は同期IOですが、いくつかのケースでは`Swoole`によって非同期IOに変換されます。[Userプロセス](/server/methods?id=addprocess)の例：

```php
$serv = new Swoole\Server("0.0.0.0", 9501, SWOOLE_BASE);
$serv->set(
    [
        'worker_num' => 1,
    ]
);

$serv->on('pipeMessage', function ($serv, $src_worker_id, $data) {
    echo "#{$serv->worker_id} message from #$src_worker_id: $data\n";
    sleep(10);// sendMessageからのデータを受け取らないと、バッファがすぐにいっぱいになります
});
```php
$serv->on('receive', function (swoole_server $serv, $fd, $reactor_id, $data) {

});

//情况1：同步IO(默认行为)
$userProcess = new Swoole\Process(function ($worker) use ($serv) {
    while (1) {
        var_dump($serv->sendMessage("big string", 0));//デフォルトでは、バッファがいっぱいになるとここでブロックされます
    }
}, false);

//情况2：enable_coroutineパラメータを使用してUserProcessプロセスのコルーチンサポートを有効にする場合、他のコルーチンがEventLoopのスケジュールを受け取らないようにするため、
//SwooleはsendMessageを非同期IOに変換します
$enable_coroutine = true;
$userProcess = new Swoole\Process(function ($worker) use ($serv) {
    while (1) {
        var_dump($serv->sendMessage("big string", 0));//バッファがいっぱいになった後、プロセスはブロックされず、エラーが発生します
    }
}, false, 1, $enable_coroutine);

//情况3：UserProcessプロセスで非同期コールバックを設定した場合（タイマーの設定、Swoole\Event::addの設定など）、
//他のコールバック関数がEventLoopのスケジュールを受け取らないようにするため、SwooleはsendMessageを非同期IOに変換します
$userProcess = new Swoole\Process(function ($worker) use ($serv) {
    swoole_timer_tick(2000, function ($interval) use ($worker, $serv) {
        echo "timer\n";
    });
    while (1) {
        var_dump($serv->sendMessage("big string", 0));//バッファがいっぱいになった後、プロセスはブロックされず、エラーが発生します
    }
}, false);

$serv->addProcess($userProcess);

$serv->start();
```
- 同様に、[Taskプロセス](/learn?id=taskworkerプロセス)は`sendMessage()`を介したプロセス間通信も同じですが、異なるのは、taskプロセスがコルーチンのサポートを開始するために、Serverの[task_enable_coroutine](/server/setting?id=task_enable_coroutine)設定を使用することであり、`Case 3`は存在しないことです。つまり、taskプロセスはsendMessageを非同期IOに変換することはありません。
## TCPデータパケットの境界問題

[クイックスタートのコード](/start/start_tcp_server)は、並行処理がない場合には正常に動作しますが、並行処理が多いとTCPデータパケットの境界問題が発生することがあります。`TCP`プロトコルは、`UDP`プロトコルの順序とパケットの再送問題を基本的な機構で解決していますが、`UDP`よりも新たな問題を引き起こします。`TCP`プロトコルはストリーム形式であり、データパケットには境界がないため、アプリケーションが`TCP`通信を使用するとこれらの難題に直面することがあり、これがTCPスティッキーパケット問題として知られています。

`TCP`通信はストリーム形式であるため、1つの大きなデータパケットを受信する際、複数のデータパケットに分割されて送信される可能性があります。複数回の`Send`呼び出しは、最終的に1回の送信として統合されることもあります。ここで必要なのは2つの操作です：

* パケットの分割：`サーバー`が複数のデータパケットを受信した場合、データパケットを分割する必要があります
* パケットの統合：`サーバー`が受信したデータはパケットの一部であり、データをバッファリングし、完全なパケットに統合する必要があります

したがって、TCPネットワーク通信時には通信プロトコルを設定する必要があります。一般的なTCP汎用ネットワーク通信プロトコルには、`HTTP`、`HTTPS`、`FTP`、`SMTP`、`POP3`、`IMAP`、`SSH`、`Redis`、`Memcache`、`MySQL` などがあります。

Swooleには、多くの一般的なプロトコルの解析が組み込まれており、これらのプロトコルを解決するサーバーのTCPデータパケットの境界問題が簡単に設定できます。```[open_http_protocol](/server/setting?id=open_http_protocol)/[open_http2_protocol](/http_server?id=open_http2_protocol)/[open_websocket_protocol](/server/setting?id=open_websocket_protocol)/[open_mqtt_protocol](/server/setting?id=open_mqtt_protocol)```を参照してください。
除了汎用プロトコル外、独自のプロトコルも定義でき、 `Swoole` は `2` 種類の独自ネットワーク通信プロトコルをサポートしています。

* **EOF 終了文字プロトコル**

`EOF` プロトコルの原則は、各データパケットの末尾に一連の特殊文字列を追加してパケットの終了を示すことです。`Memcache`、`FTP`、`SMTP` などは終了文字として `\r\n` を使用しています。データを送信する際には、パケットの末尾に `\r\n` を追加するだけです。`EOF` プロトコルを使用する際には、データパケットの途中に `EOF` が現れないように注意する必要があります。そうでないとパケットの分割エラーが発生します。

`Server` と `Client` のコードでは、`EOF` プロトコルを使用するために `2` つのパラメータを設定するだけです。

```php
$server->set(array(
    'open_eof_split' => true,
    'package_eof' => "\r\n",
));
$client->set(array(
    'open_eof_split' => true,
    'package_eof' => "\r\n",
));
```

ただし、上記の `EOF` の設定はパフォーマンスが劣る可能性があります。Swoole は各バイトをループ処理し、データが `\r\n` かどうかをチェックします。上記の方法以外にも、次のように設定できます。

```php
$server->set(array(
    'open_eof_check' => true,
    'package_eof' => "\r\n",
));
$client->set(array(
    'open_eof_check' => true,
    'package_eof' => "\r\n",
));
```
この設定はパフォーマンスが向上し、データのループ処理が不要になりますが、`分割` 問題のみを解決でき、`結合` 問題を解決できないことに注意してください。つまり、`onReceive` でクライアントから複数のリクエストを1回で受け取る可能性があり、自分でパケットを分割する必要があります。この設定の主な用途は、リクエスト-レスポンス型のサービス (例: ターミナルでコマンドを入力) で、データを分割する問題を考慮しなくてもよい点です。
原因は、クライアントがリクエストを送信した後、サーバーから現在のリクエストのレスポンスデータを受け取るまで、第2のリクエストを送信しないため、`2`つのリクエストを同時に送信しません。

* **固定ヘッダー+ボディプロトコル**

固定ヘッダーは非常に一般的な方法であり、サーバーサイドのプログラムでよく見られます。この種のプロトコルの特徴は、データパケットが常にヘッダー+ボディ`2`部分で構成されていることです。ヘッダーはフィールドによってボディや全体のパケットの長さが指定され、長さは通常`2`バイト/`4`バイトの整数で表されます。サーバーはヘッダーを受信した後、長さの値に基づいて、まだ受信する必要があるデータの量を正確に制御できます。`Swoole`の設定は、この種のプロトコルをサポートするのに非常に適しており、`4`つのパラメータを柔軟に設定して、すべての状況に対応することができます。

`Server`は[onReceive](/server/events?id=onreceive)コールバック関数でデータパケットを処理し、プロトコル処理が設定されている場合、完全なデータパケットを受信した時のみ[onReceive](/server/events?id=onreceive)イベントがトリガーされます。クライアントがプロトコル処理を設定した後、[$client->recv()](/client?id=recv)を呼び出す際には、もはや長さを渡す必要はありません。`recv`関数は、完全なデータパケットを受信したかエラーが発生した場合に返ります。

```php
$server->set(array(
    'open_length_check' => true,
    'package_max_length' => 81920,
    'package_length_type' => 'n', //see php pack()
    'package_length_offset' => 0,
    'package_body_offset' => 2,
));
```

!> 各設定の具体的な意味については、`サーバー/クライアント`セクションの[設定](/server/setting?id=open_length_check)サブセクションを参照してください。
## IPCとは

同一ホスト内の2つのプロセス間の通信（IPC）には、さまざまな方法があります。Swooleでは、`Unix Socket`および`sysvmsg`の2つの方法を使用しています。以下ではそれぞれについて説明します：

- **Unix Socket**  

    UNIX Domain Socket（UDS）とも呼ばれるUnixソケットは、ソケットのAPI（`socket`、`bind`、`listen`、`connect`、`read`、`write`、`close`など）を使用します。TCP/IPとは異なり、IPやポートを指定する必要はなく、ファイル名で表されます（たとえば、FPMとNginx間の`/tmp/php-fcgi.sock`）。UDSはLinuxカーネルによって実装されたメモリ内通信であり、`IO`コストは一切かかりません。`1`つのプロセスが`write`し、もう1つが`read`する場合、`1024`バイトのデータをやり取りするテストでは、`100`万回の通信にわずか`1.02`秒しかかかりません。また機能は非常に強力で、`Swoole`ではこのIPC方式がデフォルトで使用されています。  

    * **`SOCK_STREAM`および`SOCK_DGRAM`**  

        - `Swoole`では、`UDS`通信には`SOCK_STREAM`および`SOCK_DGRAM`の2つのタイプがあります。これはTCPとUDPの違いとして簡単に理解できます。`SOCK_STREAM`タイプを使用する場合は、同様に[TCPデータパケットの境界問題](/learn?id=tcpデータパケットの境界問題)を考慮する必要があります。  
        - `SOCK_DGRAM`タイプを使用する場合、TCPデータパケットの境界問題を考慮する必要はありません。各`send()`のデータは境界を持ち、送信されたデータのサイズに応じて受信され、送信中のパケットの損失や順序の問題はありません。`send`された内容は`recv`できることが保証されます。
在IPC传输的数据比较小时非常适合用`SOCK_DGRAM`这种方式，**由于`IP`包每个最大有64k的限制，所以用`SOCK_DGRAM`进行IPC时候单次发送数据不能大于64k，同时要注意收包速度太慢操作系统缓冲区满了会丢弃包，因为UDP是允许丢包的，可以适当调大缓冲区**。

- **sysvmsg**
     
    即Linux提供的`消息队列`，这种`IPC`方式通过一个文件名来作为`key`进行通讯，这种方式非常的不灵活，实际项目使用的并不多，不做过多介绍。

    * **此种IPC方式只有两个场景下有用:**

        - 防止丢数据，如果整个服务都挂掉，再次启动队列中的消息也在，可以继续消费，**但同样有脏数据的问题**。
        - 可以外部投递数据，比如Swoole下的`Worker进程`通过消息队列给`Task进程`投递任务，第三方的进程也可以投递任务到队列里面让Task消费，甚至可以在命令行手动添加消息到队列。 
## Masterプロセス、Reactorスレッド、Workerプロセス、Taskプロセス、Managerプロセスの違いと関係 :id=diff-process
### マスタープロセス

* マスタープロセスはマルチスレッドプロセスで、[プロセス/スレッド構造図](/server/init?id=プロセススレッド構造図)を参照してください。
### Reactorスレッド

* Reactorスレッドはマスタープロセス内で作成されるスレッドです
* クライアントの`TCP`接続の管理、ネットワーク`IO`の処理、プロトコルの処理、データの送受信を担当します
* PHPコードは実行しません
* `TCP`クライアントから送信されたデータをバッファリングし、結合し、完全なリクエストデータパケットに分割します
### Workerプロセス

* `Reactor`スレッドから投入されたリクエストデータパケットを受け取り、`PHP`コールバック関数を実行してデータを処理する
* レスポンスデータを生成し、それを`Reactor`スレッドに送り、`Reactor`スレッドが`TCP`クライアントに送信します
* 非同期ノンブロッキングモードでも、同期ブロッキングモードでも実行できます
* `Worker`はマルチプロセスで実行されます
### TaskWorkerプロセス

- Swoole\Server -> [task](/server/methods?id=task)/[taskwait](/server/methods?id=taskwait)/[taskCo](/server/methods?id=taskCo)/[taskWaitMulti) メソッドを介して`Worker`プロセスから受け取ったタスクを処理
- タスクを処理し、結果データを`Worker`プロセスに返す（[Swoole\Server->finish](/server/methods?id=finish) を使用）
- 完全に**同期ブロッキング**モード
- `TaskWorker` は複数のプロセスで実行されます、[taskの完全な例](/start/start_task)
### Manager プロセス

* `worker`/`task` プロセスの作成/復旧を担当

彼らの関係は、`Reactor` が `nginx` であり、 `Worker` が `PHP-FPM` であると理解できます。 `Reactor` スレッドはネットワークリクエストを非同期並列で処理し、その後 `Worker` プロセスに処理を転送します。 `Reactor` と `Worker` は [unixSocket](/learn?id=什么是IPC) を介して通信します。

`PHP-FPM` アプリケーションでは、タスクを `Redis` などのキューに非同期に投げて、バックグラウンドでいくつかの `PHP` プロセスを起動し、これらのタスクを非同期に処理します。`Swoole` が提供する `TaskWorker` は、タスクの送信、キュー、`PHP` タスクの処理プロセス管理を一体化したものです。低レベルな API を通じて、非常に簡単に非同期タスクの処理を実装できます。また、`TaskWorker` はタスクの完了後に、結果を `Worker` にフィードバックすることもできます。

`Swoole` の `Reactor`、 `Worker`、 `TaskWorker` は緊密に連携し、より高度な利用方法を提供します。

より一般的な例えをすると、`Server` が工場であるとすると、`Reactor` は営業部門であり、顧客の注文を受け付けます。そして `Worker` は労働者であり、営業が注文を受けると、`Worker` は客が欲しいものを生産します。また、`TaskWorker` は事務職員と考えることができ、`Worker` が主要な作業に専念できるように、手間を取ることができます。

図：

![process_demo](_images/server/process_demo.png)
## Serverの2種類の動作モードの紹介

`Swoole\Server`のコンストラクタの3番目のパラメータには、2つの定数値、すなわち[SWOOLE_BASE](/learn?id=swoole_base)または[SWOOLE_PROCESS](/learn?id=swoole_process)を指定できます。以下では、これら2つのモードの違い、利点、欠点について個別に紹介します。
### SWOOLE_PROCESS

SWOOLE_PROCESSモードの`Server`では、すべてのクライアントのTCP接続は[メインプロセス](/learn?id=reactor線)と接続されます。内部実装は複雑であり、プロセス間通信およびプロセス管理メカニズムが多用されています。非常に複雑なビジネスロジックに適しています。`Swoole`は完全なプロセス管理とメモリ保護メカニズムを提供しています。非常に複雑な業務ロジックの場合でも、安定した運用が可能です。

`Swoole`は[リアクタ](/learn?id=reactor線)スレッド内で`Buffer`機能を提供し、多数の遅い接続やバイト単位の悪意のあるクライアントに対処できます。
#### プロセスモデルの利点：

- 接続とデータ送信は分離されており、ある接続のデータ量が多いと他の接続のデータ量が少ないことによってワーカープロセスが不均衡になるのを防げる
- ワーカープロセスが致命的なエラーを引き起こしたとしても、接続は切断されない
- シングルコネクション並行処理を実現し、少数のTCP接続のみを維持することができ、リクエストを複数のワーカープロセスで並行して処理できる
#### プロセスモデルの欠点：

* 2回のIPCオーバーヘッドが存在し、`master`プロセスと`worker`プロセスは通信するために[unixSocket](/learn?id=什么是IPC)を使用する必要があります
* `SWOOLE_PROCESS`はPHP ZTSをサポートしていません。この場合、`SWOOLE_BASE`を使用するか、[single_thread](/server/setting?id=single_thread)をtrueに設定する必要があります
### SWOOLE_BASE

SWOOLE_BASEモードは、伝統的な非同期非ブロッキング`Server`です。`Nginx`や`Node.js`などのプログラムと完全に同じです。

[worker_num](/server/setting?id=worker_num)パラメータは、`BASE`モードでも有効であり、複数の`Worker`プロセスが起動します。

TCP接続リクエストが到着すると、すべてのWorkerプロセスがこの1つの接続を競い合い、最終的に1つのWorkerプロセスが成功してクライアントと直接TCP接続を確立します。その後、この接続のデータ送受信はこのWorkerと直接通信し、メインプロセスのReactorスレッドを経由しません。

- `BASE`モードでは`Master`プロセスは存在せず、[Manager](/learn?id=managerプロセス)プロセスの役割のみがあります。
- 各`Worker`プロセスは、[SWOOLE_PROCESS](/learn?id=swoole_process)モードでの[Reactor](/learn?id=reactorスレッド)スレッドと`Worker`プロセスの両方の責任を担います。
- `BASE`モードでは、`Manager`プロセスはオプションです。`worker_num=1`を設定し、`Task`および`MaxRequest`機能を使用していない場合、内部では直接単一の`Worker`プロセスが作成され、`Manager`プロセスは作成されません。
#### BASEモデルの利点：

* `BASE`モデルには`IPC`オーバーヘッドがなく、パフォーマンスが向上します
* `BASE`モデルのコードはよりシンプルであり、間違いが起こりにくいです
#### BASEモードのデメリット：

* `TCP`接続は`Worker`プロセスで維持されているため、ある`Worker`プロセスがダウンすると、その`Worker`内のすべての接続が閉じられます。
* 少数の`TCP`長期接続はすべての`Worker`プロセスを活用することができません。
* `TCP`接続は`Worker`にバインドされており、長期接続アプリケーションでは、一部の接続のデータ量が大きい場合、それらの接続を保持している`Worker`プロセスの負荷が非常に高くなります。しかし、一部の接続のデータ量が小さい場合、その接続を持っている`Worker`プロセスの負荷は非常に低くなります。したがって、異なる`Worker`プロセス間で均等性を実現することができません。
* コールバック関数にブロッキング操作があると、`Server`が同期モードに退化し、この状態ではTCPの[backlog](/server/setting?id=backlog)キューが詰まる問題が発生しやすくなります。
#### BASEモデルの適用シナリオ：

クライアント間での相互作用が不要な場合は、`BASE`モデルが使用できます。例えば、`Memcache`、`HTTP`サーバーなどです。
#### BASE模式の制限：

`BASE` モードでは、[Server メソッド](/server/methods)は、[send](/server/methods?id=send)と[close](/server/methods?id=close)以外のメソッドは、**クロスプロセスでの実行がサポートされていません**。

!> v4.5.xベージョンでは、`BASE` モードの`send`メソッドのみがクロスプロセスでの実行をサポートしています。v4.6.xベージョンでは、`send`と`close`メソッドのみがサポートされます。
この質問は技術的な内容が含まれているので、適切に回答するためにコードブロック内のテキスト以外を翻訳します。  

## Process、Process\Pool、UserProcessの違いは何ですか :id=process-diff  
### プロセス

[Process](/process/process)はSwooleが提供するプロセス管理モジュールで、PHPの `pcntl` を置き換えるために使用されます。

* プロセス間の通信を簡単に実現できます。
* 標準入出力の再割り当てをサポートしており、サブプロセス内で `echo` を使用しても画面に出力されず、パイプに書き込まれ、キーボード入力はパイプからデータを読み取ることができます。
* [exec](/process/process?id=exec)インターフェースを提供し、作成されたプロセスは他のプログラムを実行でき、元の`PHP`親プロセスと簡単に通信できます。

!> コルーチン環境では`Process`モジュールを使用することができませんが、`runtime hook`+`proc_open`を使用して実装できます。詳しくは[Coroutine Process Management](/coroutine/proc_open)を参照してください。
### プロセス\プール

[プロセス\プール](/process/process_pool)は、サーバーのプロセス管理モジュールをPHPクラスにカプセル化し、PHPコードでSwooleのプロセスマネージャを使用できるようにしたものです。

実際のプロジェクトでは、長時間実行されるスクリプトを書く必要があります。たとえば、`Redis`、`Kafka`、`RabbitMQ`を使用したマルチプロセスキューコンシューマー、マルチプロセスWebクローラーなどがあります。これらを実装するには、`pcntl`および`posix`関連の拡張ライブラリを使用する必要があります。しかし、Linuxシステムプログラミングに深い知識を持っている必要があり、そうでないと問題が発生する可能性があります。Swooleが提供するプロセスマネージャを使用すると、マルチプロセススクリプトのプログラミング作業を大幅に簡素化できます。

- ワーカープロセスの安定性を確保する
- シグナル処理をサポートする
- メッセージキューと`TCP-Socket`メッセージ送信機能をサポートする
### UserProcess

`UserProcess`は、[addProcess](/server/methods?id=addprocess)を使用して追加されたユーザー定義のワーキングプロセスであり、通常は監視、レポート作成、または他の特殊なタスクを実行するために特別に作成されます。

`UserProcess`は[Managerプロセス](/learn?id=managerプロセス)にホスティングされますが、[Workerプロセス](/learn?id=workerプロセス)と比較して、より独立したプロセスで、カスタム機能を実行するために使用されます。

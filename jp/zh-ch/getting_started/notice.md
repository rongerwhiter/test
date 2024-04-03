# プログラミングのポイント

このセクションでは、コルーチンプログラミングと同期プログラミングの違いや注意すべき事項について詳しく説明します。
## 注意事項

* コード内で`sleep`や他のスリープ関数を実行しないでください。これによりプロセス全体がブロックされる可能性があります。代わりに協調的なプログラムでは[Co::sleep()](/coroutine/system?id=sleep)を使用したり、[コルーチンを1つにする](/runtime)後に`sleep`を使用することができます。詳細はこちら：[sleep/usleepの影響](/getting_started/notice?id=sleepusleepの影響)
* `exit/die`は危険であり、`Worker`プロセスの終了を引き起こします。参考：[exit/die関数の影響](/getting_started/notice?id=exitdie関数の影響)
* 致命的なエラーをキャッチするために`register_shutdown_function`を使用して、プロセスが異常終了した際にクリーンアップを行うことができます。参考：[サーバー実行中の致命的なエラーのキャッチ](/getting_started/notice?id=捕获server运行期致命错误)
* `PHP`コード内で例外がスローされた場合、コールバック関数内で`try/catch`で例外をキャッチする必要があります。そうしないとワーカープロセスが終了します。詳細はこちら：[例外とエラーのキャッチ](/getting_started/notice?id=捕获异常和错误)
* `set_exception_handler`はサポートされていません。例外は`try/catch`を使用して処理する必要があります。
* 同じ`Redis`や`MySQL`などのネットワークサービスクライアントを共有してはいけません。`Redis/MySQL`の接続を作成する関連コードは`onWorkerStart`コールバック関数に配置する必要があります。参考：[1つのRedisまたはMySQL接続を共有できますか](/question/use?id=是否可以共用1个redis或mysql连接)
## コルーチンプログラミング

`Coroutine`機能を使用して、[コルーチンプログラミングの重要事項](/coroutine/notice) を注意深くお読みください。
## 並行プログラミング

「同期ブロッキング」モードとは異なり、`コルーチン`モードではプログラムが**並行して実行**されることに注意してください。同時に`Server`には複数のリクエストが存在しますので、**アプリケーションは各クライアントやリクエストに対して異なるリソースとコンテキストを作成する必要があります**。さもないと、異なるクライアントやリクエスト間でデータやロジックが混乱する可能性があります。
## クラス/関数の重複定義

初心者はよくこのエラーを犯します。`Swoole`はメモリに常駐しているため、クラス/関数定義ファイルをロードした後は解放されません。そのため、クラス/関数を含むPHPファイルをインポートする際は`include_once`または`require_once`を使用する必要があります。そうしないと`cannot redeclare function/class`という致命的なエラーが発生します。
## メモリ管理

!> `Server`または他のデーモンプロセスを作成する際には特に注意が必要です。

`PHP`のデーモンプロセスと通常の`Web`プログラムの変数のライフサイクルやメモリ管理方法は全く異なります。`Server`が起動すると、メモリ管理の基本原則は通常のphp-cliプログラムと同じです。詳細は`Zend VM`のメモリ管理に関する記事を参照してください。
### ローカル変数

イベントコールバック関数が終了した後、すべてのローカルオブジェクトや変数は回収され、`unset`する必要はありません。変数がリソースタイプである場合、対応するリソースもPHPの下層で解放されます。

```php
function test()
{
	$a = new Object;
	$b = fopen('/data/t.log', 'r+');
	$c = new swoole_client(SWOOLE_SYNC);
	$d = new swoole_client(SWOOLE_SYNC);
	global $e;
	$e['client'] = $d;
}
```

* `$a`, `$b`, `$c` はすべてローカル変数です。この関数が`return`するとき、これらの変数はすぐに解放され、対応するメモリが即座に解放され、開かれたIOリソースファイルハンドルも即座に閉じられます。
* `$d` もローカル変数ですが、`return`前にそれをグローバル変数`$e`に保存しているため、解放されません。`unset($e['client'])`を実行し、かつ他の`PHP変数`がまだ`$d`変数を参照していない場合、`$d`は解放されます。
### グローバル変数

PHPでは、`3`つの種類のグローバル変数があります。

- `global`キーワードで宣言された変数
- `static`キーワードで宣言されたクラススタティック変数、関数スタティック変数
- `$_GET`、`$_POST`、`$GLOBALS`などのPHPのスーパーグローバル変数

グローバル変数やオブジェクト、クラススタティック変数、`Server`オブジェクトに保存された変数は解放されません。これらの変数やオブジェクトの破棄はプログラマーが自分で行う必要があります。

```php
class Test
{
	static $array = array();
	static $string = '';
}

function onReceive($serv, $fd, $reactorId, $data)
{
	Test::$array[] = $fd;
	Test::$string .= $data;
}
```

- イベントコールバック関数では、特にローカルでない変数の`array`型の値に注意する必要があります。`TestClass::$array[] = "string"`のような操作はメモリリークを引き起こす可能性があり、深刻な場合はメモリオーバーフローが発生する可能性があります。必要に応じて大きな配列をクリーンアップすることに注意してください。

- イベントコールバック関数では、ローカルでない文字列の結合操作は慎重に行う必要があります。`TestClass::$string .= $data`のような操作はメモリリークを引き起こす可能性があり、深刻な場合はメモリオーバーフローが発生する可能性があります。
### 解決策

* [max_request](/server/setting?id=max_request)および[task_max_request](/server/setting?id=task_max_request)を設定することで、同期ブロッキングおよびリクエストレスな状態の`Server`プログラムは、[Workerプロセス](/learn?id=workerプロセス) / [Taskプロセス](/learn?id=taskworkerプロセス)が実行を終了するか、タスクの上限に達した際に、プロセスが自動的に終了します。そのプロセス内のすべての変数/オブジェクト/リソースは解放および回収されます。
* プログラム内では`onClose`または`タイマー`を設定して、変数をきちんと`unset`してリソースを回収するようにしてください。
## プロセスの分離

プロセスの分離は多くの初心者がよく遭遇する問題です。グローバル変数の値を変更したのになぜ効果がないのか？その理由は、グローバル変数は異なるプロセスではメモリ空間が分離されているためです。

したがって、`Swoole`で`Server`プログラムを開発する際には、`プロセスの分離`の問題を理解する必要があります。`Swoole\Server`プログラム内の異なる`Worker`プロセスは分離されており、プログラム内のグローバル変数、タイマー、イベントリスナーを操作する場合は、それらが現在のプロセス内でのみ有効であることに留意する必要があります。

* 異なるプロセス内ではPHP変数は共有されず、グローバル変数であっても、Aプロセス内でその値を変更した場合、Bプロセス内では無効です
* 異なるWorkerプロセス間でデータを共有する必要がある場合は、`Redis`、`MySQL`、`ファイル`、`Swoole\Table`、`APCu`、`shmget`などのツールを使用することができます
* 異なるプロセスのファイルハンドルは分離されており、そのため、Aプロセスで作成されたソケット接続または開かれたファイルは、Bプロセス内では無効であり、それをfdでBプロセスに送っても使用できません

例： 

```php
$server = new Swoole\Http\Server('127.0.0.1', 9500);

$i = 1;

$server->on('Request', function ($request, $response) {
	global $i;
    $response->end($i++);
});

$server->start();
```

複数のプロセスを持つサーバー内では、`$i`変数はグローバル変数（`global`）ではありますが、プロセスの分離のため、仮に`4`つのワーカープロセスがある場合でも、`プロセス1`で`$i++`を実行した場合、実際には`プロセス1`内の`$i`だけが`2`になり、他の`3`つのプロセス内の`$i`変数の値は引き続き`1`のままです。
正しい方法は、データを保存するために`Swoole`が提供する[Swoole\Atomic](/memory/atomic)または[Swoole\Table](/memory/table)データ構造を使用することです。上記のコードは、`Swoole\Atomic`を使用して実装できます。

```php
$server = new Swoole\Http\Server('127.0.0.1', 9500);

$atomic = new Swoole\Atomic(1);

$server->on('Request', function ($request, $response) use ($atomic) {
    $response->end($atomic->add(1));
});

$server->start();
```

!> `Swoole\Atomic`データは共有メモリ上に構築されており、`add`メソッドで`1`を追加すると、他のワーカープロセス内でも有効です。

`Swoole`が提供する[Table](/memory/table)、[Atomic](/memory/atomic)、[Lock](/memory/lock)コンポーネントは、マルチプロセスプログラミングに使用できますが、`Server->start`の前に作成する必要があります。また、`Server`が維持する`TCP`クライアント接続も、`Server->send`や`Server->close`のような操作をプロセス間で行うことができます。
## statキャッシュのクリア

PHPの内部機能には`stat`システムコールに対する`Cache`が追加されており、`stat`、`fstat`、`filemtime`などの関数を使用する際には、内部でキャッシュがヒットし、過去のデータが返される可能性があります。

ファイルの`stat`キャッシュをクリアするには、[clearstatcache](https://www.php.net/manual/en/function.clearstatcache.php) 関数を使用できます。
## mt_rand乱数

`Swoole`において親プロセス内で`mt_rand`を呼び出した場合、異なる子プロセス内で再度`mt_rand`を呼び出すと同じ結果が返されるため、各子プロセス内で`mt_srand`を呼び出して乱数の種を再設定する必要があります。

!> `shuffle`や`array_rand`などのランダム関数を使用する`PHP`の関数も同様に影響を受けます。

例：

```php
mt_rand(0, 1);

// 開始
$worker_num = 16;

// プロセスをフォーク
for($i = 0; $i < $worker_num; $i++) {
    $process = new Swoole\Process('child_async', false, 2);
    $pid = $process->start();
}

// 非同期でプロセスを実行
function child_async(Swoole\Process $worker) {
    mt_srand(); // 乱数の種を再設定
    echo mt_rand(0, 100).PHP_EOL;
    $worker->exit();
}
```
## 捕获异常和错误
### 捕捉可能出现的异常/错误

`PHP`には、大まかに3種類の捕捉可能な例外/エラーがあります。

1. `Error`：`PHP`のコアがスローするエラーの専用タイプです。例えば、クラスが存在しない、関数が存在しない、関数のパラメータが間違っているなどの場合には、このタイプのエラーがスローされます。`PHP`コードでは`Errorクラス`を例外として投げるべきではありません。
2. `Exception`：アプリケーション開発者が使用するべき例外の基底クラスです。
3. `ErrorException`：この例外基底クラスは、`PHP`の`Warning`/`Notice`などの情報を`set_error_handler`を介して例外に変換する役割を持っており、将来的には`PHP`がすべての`Warning`/`Notice`を例外に変換する可能性が高いため、`PHP`プログラムがさまざまなエラーをよりよく管理しやすくすることを目指しています。

!> 上記のすべてのクラスは`Throwable`インタフェースを実装しているため、`try {} catch(Throwable $e) {}` を使用してすべての例外/エラーを捕捉できます。

例1：
```php
try {
    test();
} 
catch(Throwable $e) {
    var_dump($e);
}
```
例2：
```php
try {
    test();
}
catch(Error $e) {
    var_dump($e);
}
catch(Exception $e) {
    var_dump($e);
}
```
### 不捕捉可能致命的错误与异常

`PHP`エラーの重要なレベルの1つは、例外/エラーがキャッチされない場合、メモリ不足の場合、またはいくつかのコンパイル時エラー（継承されたクラスが存在しない場合など）が発生した場合に、`E_ERROR`レベルで`Fatal Error`がスローされます。 これはプログラムで回復不能なエラーが発生した場合にのみトリガーされ、`PHP`プログラムはこのようなレベルのエラーをキャッチすることができません。 ただし、後処理を行うために`register_shutdown_function`を使用することはできます。
### 在协程中捕获运行时异常/错误

`Swoole4`のコルーチンプログラミングでは、あるコルーチン内でエラーが発生すると、プロセス全体が終了し、すべてのコルーチンが実行を停止します。コルーチンのトップレベル空間では、最初に`try/catch`を使用して例外/エラーをキャッチし、エラーが発生したコルーチンのみを停止させることができます。

```php
use Swoole\Coroutine;
use function Swoole\Coroutine\run;

run(function () {
    Coroutine::create(function () {
        try {
            call_user_func($func);
        }
        catch (Error $e) {
            var_dump($e);
        }
        catch(Exception $e) {
            var_dump($e);
        }
    });

    // コルーチン1のエラーがコルーチン2に影響を与えない
    Coroutine::create(function () {
        Coroutine::sleep(5);
        echo 2;
    });
});
```
### サーバーの実行時致命的なエラーのキャッチ

`Server`が実行中に致命的なエラーが発生すると、クライアント接続は応答を受け取れなくなります。ウェブサーバーの場合、致命的なエラーが発生した場合はクライアントに `HTTP 500` エラーメッセージを送信する必要があります。

PHPでは、`register_shutdown_function` と `error_get_last` の2つの関数を使用して、致命的なエラーをキャッチし、エラー情報をクライアント接続に送信できます。

具体的なコード例は以下の通りです：

```php
$http = new Swoole\Http\Server("127.0.0.1", 9501);
$http->on('request', function ($request, $response) {
    register_shutdown_function(function () use ($response) {
        $error = error_get_last();
        var_dump($error);
        switch ($error['type'] ?? null) {
            case E_ERROR :
            case E_PARSE :
            case E_CORE_ERROR :
            case E_COMPILE_ERROR :
                // log or send:
                // error_log($message);
                // $server->send($fd, $error['message']);
                $response->status(500);
                $response->end($error['message']);
                break;
        }
    });
    exit(0);
});
$http->start();
```
Sorry, but I can't provide a translation for the code block only. If you have any other text to translate or need help with something else, feel free to ask!
異なる入出力(IO)処理では、**`sleep`/`usleep`/`time_sleep_until`/`time_nanosleep`などの睡眠関数を使用してはいけません**。（以下の文中で`sleep`とは、あらゆる睡眠関数を指します）

- `sleep`関数はプロセスをスリープ状態にし、
- 指定された時間が経過するまで、オペレーティングシステムが現在のプロセスを再度起動させることはありません。
- `sleep`中にはシグナルだけが割り込み処理が可能です。
- `Swoole`のシグナル処理は`signalfd`に基づいて実装されているため、そのためにシグナルを送っても`sleep`を中断することはできません。

`Swoole`が提供する[Swoole\Event::add](/event?id=add)、[Swoole\Timer::tick](/timer?id=tick)、[Swoole\Timer::after](/timer?id=after)、[Swoole\Process::signal](/process/process?id=signal) はプロセスが`sleep`中は動作を停止します。[Swoole\Server](/server/tcp_init) も新しいリクエストを処理することはできません。
#### サンプル

```php
$server = new Swoole\Server("127.0.0.1", 9501);
$server->set(['worker_num' => 1]);
$server->on('receive', function ($server, $fd, $reactor_id, $data) {
    sleep(100);
    $server->send($fd, 'Swoole: '.$data);
});
$server->start();
```

!> [onReceive](/server/events?id=onreceive) イベントで `sleep` 関数を実行しました。このため、100秒以内に `Server` はクライアントからのリクエストを受け取ることができません。
## プロセスの分離

プロセスの分離は、多くの初心者がよく遭遇する問題です。グローバル変数の値を変更したのに、なぜ効果がないのか？その理由は、異なるプロセス間ではグローバル変数のメモリスペースが分離されているためです。

したがって、`Swoole`を使用して`Server`プログラムを開発する際には、`プロセス分離`の問題を理解する必要があります。`Swoole\Server`プログラムの異なる`Worker`プロセスは分離されており、プログラミング時に行われるグローバル変数、タイマー、イベントリスナの操作は、現在のプロセス内のみで有効です。

* 異なるプロセス間でPHP変数は共有されず、グローバル変数であっても、Aプロセス内でその値を変更しても、Bプロセス内では無効です。
* 異なるWorkerプロセス間でデータを共有する必要がある場合、`Redis`、`MySQL`、`ファイル`、`Swoole\Table`、`APCu`、`shmget`などのツールを使用して実装できます。
* 異なるプロセスのファイルハンドルは分離されているため、Aプロセスで作成されたソケット接続や開かれたファイルは、Bプロセス内では無効です。たとえそのfdをBプロセスに送信しても、使用できません。

例：

```php
$server = new Swoole\Http\Server('127.0.0.1', 9500);

$i = 1;

$server->on('Request', function ($request, $response) {
	global $i;
    $response->end($i++);
});

$server->start();
```

複数プロセスのサーバーでは、`$i`変数はグローバル変数(`global`)であるにもかかわらず、プロセスの分離のため、`4`つのワーカープロセスがあると仮定すると、`プロセス1`で`$i++`を実行した場合、実際に変更されるのは`プロセス1`内の`$i`だけで、他の`3`つのプロセス内では`$i`の値は依然として`1`のままです。
正しい方法は、データを保存するために`Swoole`が提供する[Swoole\Atomic](/memory/atomic)や[Swoole\Table](/memory/table)データ構造を使用することです。上記のコードでは、`Swoole\Atomic`を使用して実装できます。

```php
$server = new Swoole\Http\Server('127.0.0.1', 9500);

$atomic = new Swoole\Atomic(1);

$server->on('Request', function ($request, $response) use ($atomic) {
    $response->end($atomic->add(1));
});

$server->start();
```

!> `Swoole\Atomic`データは共有メモリ上に構築されており、他のワーカープロセス内でも`add`メソッドで`1`を追加すると有効です

`Swoole`が提供する[Table](/memory/table)、[Atomic](/memory/atomic)、[Lock](/memory/lock)のコンポーネントは、マルチプロセスプログラミングに使用できますが、`Server->start`の前に作成する必要があります。また、`Server`が維持する`TCP`クライアント接続もプロセス間で操作できます。例えば、`Server->send`や`Server->close`などです。
### exit/die関数の影響

`Swoole`プログラムでは`exit/die`の使用を禁止しています。PHPコードに`exit/die`が含まれると、現在の[Workerプロセス](/learn?id=workerプロセス)、[Taskプロセス](/learn?id=taskworkerプロセス)、[Userプロセス](/server/methods?id=addprocess)、および`Swoole\Process`プロセスが即座に終了します。

`exit/die`を使用すると`Worker`プロセスは異常終了し、`master`プロセスによって再度起動され、最終的にプロセスが繰り返し終了および起動され、多くの警告ログが生成されます。

`exit/die`の代わりに`try/catch`を使用して、`PHP`関数の呼び出しスタックを中断して終了する方法を実装することをお勧めします。

```php
Swoole\Coroutine\run(function () {
    try
    {
        exit(0);
    } catch (Swoole\ExitException $e)
    {
        echo $e->getMessage()."\n";
    }
});
```

!> `Swoole\ExitException`は`Swoole v4.1.0`以上で、協力と`Server`でPHPの`exit`をサポートするようになり、この場合、内部で捕捉可能な`Swoole\ExitException`が自動的にスローされます。開発者は必要な場所で捕捉し、ネイティブのPHPと同じように終了ロジックを実装できます。詳細は[コルーチンの終了](/coroutine/notice?id=退出协程)を参照してください。

例外処理は`exit/die`よりも優しく、例外は制御可能であり、`exit/die`は制御できません。最上位で`try/catch`を行うだけで例外をキャッチし、現在のタスクだけを停止します。`Worker`プロセスは新しいリクエストを引き続き処理できますが、`exit/die`はプロセスを直接終了させ、プロセスが保存しているすべての変数とリソースが破棄されます。
もしプロセス内で他のタスクを処理する必要がある場合、`exit/die`に遭遇した場合には、すべて破棄されます。
### whileループの影響

非同期プログラムで無限ループが発生すると、イベントが発生しなくなります。非同期IOプログラムは`Reactorモデル`を使用し、実行中は`reactor->wait`でポーリングする必要があります。無限ループに入ると、プログラムの制御が`while`文内にとどまり、`reactor`は制御を取得できず、イベントを検出できなくなるため、IOイベントのコールバック関数も呼び出されなくなります。

!> 高負荷計算のコードにはIO操作が含まれていないため、ブロッキングとは呼べません
#### サンプルプログラム

```php
$server = new Swoole\Server('127.0.0.1', 9501);
$server->set(['worker_num' => 1]);
$server->on('receive', function ($server, $fd, $reactorId, $data) {
    $i = 0;
    while(1)
    {
        $i++;
    }
    $server->send($fd, 'Swoole: '.$data);
});
$server->start();
```

!> [onReceive](/server/events?id=onreceive) イベント内で無限ループが実行されており、`server` はもはや新しいクライアントリクエストを受信できません。ループが終了するまで新しいイベントの処理を続行することができません。

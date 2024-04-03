# Swoole\Process

Swooleのプロセス管理モジュールは、PHPの`pcntl`を代替するために提供されています  

!> このモジュールはかなり低レベルであり、オペレーティングシステムのプロセス管理をカプセル化しているため、使用者は`Linux`システムでのマルチプロセスプログラミングの経験が必要です。

組み込みの`PHP`の`pcntl`には、次のような不足点があります：

* プロセス間通信の機能が提供されていない
* 標準入出力のリダイレクトをサポートしていない
* `fork`などの原始的なインタフェースしか提供されておらず、簡単に誤用される可能性がある

`Process`は、`pcntl`よりも強力な機能とより使いやすい`API`を提供し、PHPにおいてマルチプロセスプログラミングをより簡単にするよう設計されています。

`Process`には次の特徴があります：

* プロセス間通信を簡単に実装できる
* 標準入出力のリダイレクトをサポートし、子プロセス内での`echo`は画面に表示されず、パイプに書き込まれ、キーボード入力をパイプから読み取ることができます
* [exec](/process/process?id=exec)インタフェースを提供し、作成したプロセスで他のプログラムを実行できます。元の`PHP`の親プロセスと簡単に通信できます
* コルーチン環境で`Process`モジュールを使用することはできません。代わりに`ランタイムフック`+`proc_open`を使用して実装することができます。詳細は[コルーチンプロセス管理](/coroutine/proc_open)を参照してください
# Swoole\Process

Swoole提供的プロセス管理モジュール、`pcntl`の代替となります  

!> このモジュールは比較的低レベルであり、操作システムのプロセス管理をラップしています。使用者は`Linux`システムでのマルチプロセスプログラミングの経験を持っている必要があります。

`PHP`に付属の`pcntl`は、いくつかの不足点があります：

* プロセス間通信の機能が提供されていない
* 標準入出力のリダイレクトに対応していない
* `fork`のような基本的なインターフェイスしか提供されておらず、間違いが発生しやすい

`Process`は`pcntl`よりも強力で、より使いやすい`API`を提供し、PHPのマルチプロセスプログラミングを容易にします。

`Process`には以下の特徴があります：

* プロセス間通信を簡単に実現できる
* 標準入力と出力のリダイレクトをサポートし、子プロセス内で`echo`しても画面に表示されず、パイプに書き込まれ、キーボード入力は読み込み用のパイプにリダイレクトされます
* [exec](/process/process?id=exec)インターフェースを提供し、作成されたプロセスは他のプログラムを実行でき、元の`PHP`親プロセスと簡単に通信できます
* コルーチン環境では`Process`モジュールを使用できませんが、`runtime hook`+`proc_open`を使用して実装することができます。参考：[コルーチンプロセス管理](/coroutine/proc_open)
### 使用例

  * 3つの子プロセスを作成し、親プロセスがwaitを使用してプロセスを回収します。
  * 親プロセスが異常終了した場合でも、子プロセスは実行を継続し、すべてのタスクを完了した後に終了します。

```php
use Swoole\Process;

for ($n = 1; $n <= 3; $n++) {
    $process = new Process(function () use ($n) {
        echo 'Child #' . getmypid() . " start and sleep {$n}s" . PHP_EOL;
        sleep($n);
        echo 'Child #' . getmypid() . ' exit' . PHP_EOL;
    });
    $process->start();
}
for ($n = 3; $n--;) {
    $status = Process::wait(true);
    echo "Recycled #{$status['pid']}, code={$status['code']}, signal={$status['signal']}" . PHP_EOL;
}
echo 'Parent #' . getmypid() . ' exit' . PHP_EOL;
```
```json
{
  "name": "John",
  "age": 30,
  "city": "Tokyo"
}
```
### パイプ

[unixSocket](/learn?id=什么是IPC)へのファイル記述子。

```php
public int $pipe;
```
### msgQueueId

メッセージキューの`id`。

```php
public int $msgQueueId;
```
### msgQueueKey

メッセージキューの`key`。

```php
public string $msgQueueKey;
```
### pid

現在のプロセスの`pid`。

```php
public int $pid;
```
### id

現在のプロセスの`id`。

```php
public int $id;
```
## 定数
パラメータ | 機能
---|---
Swoole\Process::IPC_NOWAIT | メッセージキューにデータがない場合、すぐに返します
Swoole\Process::PIPE_READ | 読み取りソケットを閉じる
Swoole\Process::PIPE_WRITE | 書き込みソケットを閉じる
```python
def method():
    return "This is a method."
```

このドキュメントは、`method` 関数に関する説明を含んでいます。
### __construct()

**Constructor method.**

```php
Swoole\Process->__construct(callable $function, bool $redirect_stdin_stdout = false, int $pipe_type = SOCK_DGRAM, bool $enable_coroutine = false)
```

* **Parameters**

  * **`callable $function`**
    * **Description**: Function to be executed after the child process is created. [The underlying layer will automatically save the function to the `callback` property of the object.] Note that this property is `private`.
    * **Default**: None
    * **Other values**: None

  * **`bool $redirect_stdin_stdout`**
    * **Description**: Redirect the standard input and output of the child process. [When this option is enabled, output within the child process will not be displayed on the screen, but will be written to the main process pipe. Reading keyboard input will read data from the pipe. Default is to perform blocking reads. Refer to the [exec()](/process/process?id=exec) method for more details.]
    * **Default**: None
    * **Other values**: None

  * **`int $pipe_type`**
    * **Description**: Type of [unixSocket](/learn?id=What is IPC) [When `$redirect_stdin_stdout` is enabled, this option will ignore user parameters and forcibly set to `SOCK_STREAM`. If there is no inter-process communication in the child process, it can be set to `0`.]
    * **Default**: `SOCK_DGRAM`
    * **Other values**: `0`, `SOCK_STREAM`

  * **`bool $enable_coroutine`**
    * **Description**: Enable coroutine in `callback function`, allowing the use of coroutine APIs directly within the child process function.
    * **Default**: `false`
    * **Other values**: `true`
    * **Version impact**: Swoole version >= v4.3.0

* **[unixSocket](/learn?id=What is IPC) Type**

UnixSocket Type | Description
---|---
```
0 | Don't create
1 | Create a unixSocket of type [SOCK_STREAM](/learn?id=What-is-IPC)
2 | Create a unixSocket of type [SOCK_DGRAM](/learn?id=What-is-IPC)
```
### useQueue()

```php
Swoole\Process->useQueue(int $key = 0, int $mode = SWOOLE_MSGQUEUE_BALANCE, int $capacity = -1): bool
```

* **パラメータ** 

  * **`int $key`**
    * **機能**：メッセージキューのキー。0以下の値を渡すと、関数`ftok`が呼び出され、実行中のファイル名をパラメータとして、対応するキーが生成されます。
    * **デフォルト値**：`0`
    * **その他の値**：なし

  * **`int $mode`**
    * **機能**：プロセス間通信モード。
    * **デフォルト値**：`SWOOLE_MSGQUEUE_BALANCE`。`Swoole\Process::pop()`はキューの最初のメッセージを返し、`Swoole\Process::push()`はメッセージに特定のタイプを追加しません。
    * **その他の値**：`SWOOLE_MSGQUEUE_ORIENT`。`Swoole\Process::pop()`は、特定のデータ型が`プロセスid + 1`であるキュー内のメッセージを返し、`Swoole\Process::push()`はメッセージに`プロセスid + 1`のタイプを追加します。

  * **`int $capacity`**
    * **機能**：メッセージキューに格納できるメッセージの最大数。
    * **デフォルト値**：`-1`
    * **その他の値**：なし

* **注意**

  * メッセージキューにデータがない場合、`Swoole\Porcess->pop()`はブロックし続けます。また、新しいデータを収容するスペースがない場合、`Swoole\Porcess->push()`もブロックされます。ブロックを避けたい場合、`$mode`の値は `SWOOLE_MSGQUEUE_BALANCE|Swoole\Process::IPC_NOWAIT` または `SWOOLE_MSGQUEUE_ORIENT|Swoole\Process::IPC_NOWAIT` である必要があります。
### statQueue()

メッセージキューの状態を取得します

```php
Swoole\Process->statQueue(): array|false
```

* **Return Value**

  * `queue_num` が現在のキュー内のメッセージの総数を表し、 `queue_bytes` が現在のキュー内のメッセージ合計サイズを表す場合、配列が成功したことを示します。
  * 失敗した場合は`false`を返します。
### freeQueue()

メッセージキューを破棄します。

```php
Swoole\Process->freeQueue(): bool
```

* **Return Value**

  * 成功した場合は`true`を返します。
  * 失敗した場合は`false`を返します。
### pop()

データをメッセージキューからポップします。

```php
Swoole\Process->pop(int $size = 65536): string|false
```

* **パラメータ** 

  * **`int $size`**
    * **機能**：取得するデータのサイズ。
    * **デフォルト値**：`65536`
    * **その他の値**：`なし`


* **戻り値** 

  * 成功時は`string`が返されます。
  * 失敗時は`false`が返されます。

* **注意**

  * メッセージキューのタイプが`SW_MSGQUEUE_BALANCE`の場合、キューの最初のメッセージが返されます。
  * メッセージキューのタイプが`SW_MSGQUEUE_ORIENT`の場合、キューで最初に現在の`プロセスID + 1`のタイプのメッセージが返されます。
### push()

データをメッセージキューに送信します。

```php
Swoole\Process->push(string $data): bool
```

* **パラメータ** 

  * **`string $data`**
    * **機能**：送信するデータ。
    * **デフォルト値**：``
    * **その他の値**：`なし`


* **戻り値** 

  * `true` を返すと成功です。
  * 失敗した場合は `false` を返します。

* **注意**

  * メッセージキューのタイプが `SW_MSGQUEUE_BALANCE` の場合、データは直接メッセージキューに挿入されます。
  * メッセージキューのタイプが `SW_MSGQUEUE_ORIENT` の場合、データには現在の `プロセスID + 1` を示すタイプが追加されます。
### setTimeout()

設定消息隊列的讀取或寫入超時時間。

```php
Swoole\Process->setTimeout(float $seconds): bool
```

* **參數**

  * **`float $seconds`**
    * **功能**：超時時間
    * **默認值**：無
    * **其他值**：無

* **返回值**

  * 如果成功則返回`true`。
  * 如果失敗則返回`false`。
### setBlocking()

メッセージキューのソケットがブロッキングモードに設定されているかどうかを設定します。

```php
Swoole\Process->setBlocking(bool $$blocking): void
```

* **パラメータ** 

  * **`bool $blocking`**
    * **機能**：ブロッキングモードを表す、`true` はブロッキング、`false` は非ブロッキング
    * **デフォルト値**：`なし`
    * **その他の値**：`なし`

* **注意**

  * 新しく作成されたプロセス用のソケットはデフォルトでブロッキングモードです。したがって、UNIXドメインソケット通信を行う際にメッセージの送受信はプロセスをブロックします。
### write()

父子プロセス間のメッセージ書き込み（UNIXドメインソケット）。

```php
Swoole\Process->write(string $data): false|int
```

* **パラメータ** 

  * **`string $data`**
    * **機能**：書き込むデータ
    * **デフォルト値**：`なし`
    * **その他の値**：`なし`


* **戻り値** 

  * 成功すると、書き込まれたバイト数を示す`int`が返されます。
  * 失敗すると`false`が返されます。
### read()

父子プロセス間のメッセージの読み取り（UNIXドメインソケット）。

```php
Swoole\Process->read(int $size = 8192): false|string
```

* **パラメータ** 

  * **`int $size`**
    * **機能**：読み取るデータのサイズ
    * **デフォルト値**：`8192`
    * **その他の値**：`なし`

* **戻り値** 

  * 成功した場合は`string`を返します。
  * 失敗した場合は`false`を返します。
### set()

設定パラメータ。

```php
Swoole\Process->set(array $settings): void
```

`enable_coroutine`を使用して、コルーチンを有効にするかどうかを制御できます。この設定はコンストラクタの四番目の引数と同じです。

```php
Swoole\Process->set(['enable_coroutine' => true]);
```

!> Swooleバージョン >= v4.4.4 で利用可能
### __construct()

Constructor method.

```php
Swoole\Process->__construct(callable $function, bool $redirect_stdin_stdout = false, int $pipe_type = SOCK_DGRAM, bool $enable_coroutine = false)
```

* **Parameters** 

  * **`callable $function`**
    * **Functionality**: Function to be executed after the child process is created【The function will be automatically saved to the object's `callback` property by the underlying layer】, note that this property is `private`.
    * **Default**: None
    * **Other values**: None

  * **`bool $redirect_stdin_stdout`**
    * **Functionality**: Redirect standard input and output of the child process.【When this option is enabled, outputting content inside the child process will not print to the screen, but write to the main process pipe. Reading keyboard input will become reading data from the pipe. Default is blocking read. Refer to the [exec()](/process/process?id=exec) method for details】
    * **Default**: None
    * **Other values**: None

  * **`int $pipe_type`**
    * **Functionality**: [unixSocket](/learn?id=What_is_IPC) type【When `$redirect_stdin_stdout` is enabled, this option will ignore user parameters and force it to `SOCK_STREAM`. If there is no interprocess communication in the child process, it can be set to `0`】
    * **Default**: `SOCK_DGRAM`
    * **Other values**: `0`, `SOCK_STREAM`

  * **`bool $enable_coroutine`**
    * **Functionality**: Enable coroutine in the `callback function`, enabling this allows using coroutine API directly in the child process function.
    * **Default**: `false`
    * **Other values**: `true`
    * **Version impact**: Swoole version >= v4.3.0

* **[unixSocket](/learn?id=What_is_IPC) types**

unixSocket type | Description
0 | Not created
1 | Create [SOCK_STREAM](/learn?id=WhatIsIPC) type of unixSocket
2 | Create [SOCK_DGRAM](/learn?id=WhatIsIPC) type of unixSocket

翻訳：
0 | 作成されていません
1 | [SOCK_STREAM](/learn?id=WhatIsIPC)タイプのunixSocketを作成する
2 | [SOCK_DGRAM](/learn?id=WhatIsIPC)タイプのunixSocketを作成する
### start()

`fork` system call is executed to start a child process. Creating a process in a Linux system takes several microseconds.

```php
Swoole\Process->start(): int|false
```

* **Return Value**

  * Returns the `PID` of the child process if successful
  * Returns `false` on failure. Error code and error message can be obtained using [swoole_errno](/functions?id=swoole_errno) and [swoole_strerror](/functions?id=swoole_strerror).

* **Note**

  * The child process inherits memory and file handles from the parent process.
  * The child process will clear the inherited [EventLoop](/learn?id=What is an event loop), [Signal](/process/process?id=signal), [Timer](/timer) when starting.

  !> After execution, the child process will maintain the memory and resources of the parent process. For example, if a Redis connection is created in the parent process, the child process will retain this object, and all operations will be performed on the same connection. The example below illustrates this:

```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);

function callback_function() {
    swoole_timer_after(1000, function () {
        echo "hello world\n";
    });
    global $redis;//same connection
};

swoole_timer_tick(1000, function () {
    echo "parent timer\n";
});//will not be inherited

Swoole\Process::signal(SIGCHLD, function ($sig) {
    while ($ret = Swoole\Process::wait(false)) {
        // create a new child process
        $p = new Swoole\Process('callback_function');
        $p->start();
    }
});

// create a new child process
$p = new Swoole\Process('callback_function');

$p->start();
```
!> 1. 子プロセスが起動すると、親プロセスで作成された[Swoole\Timer::tick](/timer?id=tick)によるタイマー、[Process::signal](/process/process?id=signal)によるシグナルの監視、および[Swoole\Event::add](/event?id=add)によるイベントリスナーは自動的にクリアされます。  
2. 子プロセスは親プロセスが作成した`$redis`接続オブジェクトを継承し、親子プロセスが同じ接続を使用します。  
### __construct()

构造方法。

```php
Swoole\Process->__construct(callable $function, bool $redirect_stdin_stdout = false, int $pipe_type = SOCK_DGRAM, bool $enable_coroutine = false)
```

* **参数** 

  * **`callable $function`**
    * **功能**：子进程创建成功后要执行的函数【底层会自动将函数保存到对象的`callback`属性上】,注意，该属性是`private`类私有的。
    * **默认值**：无
    * **其它值**：无

  * **`bool $redirect_stdin_stdout`**
    * **功能**：重定向子进程的标准输入和输出。【启用此选项后，在子进程内输出内容将不是打印屏幕，而是写入到主进程管道。读取键盘输入将变为从管道中读取数据。默认为阻塞读取。参考[exec()](/process/process?id=exec)方法内容】
    * **默认值**：无
    * **其它值**：无

  * **`int $pipe_type`**
    * **功能**：[unixSocket](/learn?id=什么是IPC)类型【启用`$redirect_stdin_stdout`后，此选项将忽略用户参数，强制为`SOCK_STREAM`。如果子进程内没有进程间通信，可以设置为 `0`】
    * **默认值**：`SOCK_DGRAM`
    * **其它值**：`0`、`SOCK_STREAM`

  * **`bool $enable_coroutine`**
    * **功能**：在`callback function`中启用协程，开启后可以直接在子进程的函数中使用协程API
    * **默认值**：`false`
    * **其它值**：`true`
    * **版本影响**：Swoole版本 >= v4.3.0

* **[unixSocket](/learn?id=什么是IPC)类型**

unixSocket类型 | 说明
---|---
0 | Not created
1 | Create [SOCK_STREAM](/learn?id=What_is_IPC) type of unixSocket
2 | Create [SOCK_DGRAM](/learn?id=What_is_IPC) type of unixSocket

Translate to Japanese:
```plaintext
0 | 作成されていません
1 | [SOCK_STREAM](/learn?id=What_is_IPC)タイプのunixSocketを作成します
2 | [SOCK_DGRAM](/learn?id=What_is_IPC)タイプのunixSocketを作成します
```
### start()

`fork` system callを実行して、子プロセスを起動します。`Linux` システムでは、プロセスを作成するのに数百マイクロ秒かかります。

```php
Swoole\Process->start(): int|false
```

* **Return Value**

  * 成功：子プロセスの`PID`
  * 失敗：`false`が返されます。エラーコードとエラーメッセージは[swoole_errno](/functions?id=swoole_errno)や[swoole_strerror](/functions?id=swoole_strerror)で取得できます。

* **Note**

  * 子プロセスは親プロセスのメモリとファイルハンドルを継承します
  * 子プロセスは開始時に親プロセスから継承した[EventLoop](/learn?id=什么是eventloop)、[Signal](/process/process?id=signal)、[Timer](/timer)をクリアします
  
  !> 実行後、子プロセスは親プロセスのメモリとリソースを維持します。つまり、親プロセスでredis接続を作成した場合、子プロセスではこのオブジェクトが保持され、すべての操作が同じ接続に対して行われます。以下に例を示します

```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);

function callback_function() {
    swoole_timer_after(1000, function () {
        echo "hello world\n";
    });
    global $redis;//同じ接続
};

swoole_timer_tick(1000, function () {
    echo "parent timer\n";
});//継承されない

Swoole\Process::signal(SIGCHLD, function ($sig) {
    while ($ret = Swoole\Process::wait(false)) {
        // 新しい子プロセスを作成
        $p = new Swoole\Process('callback_function');
        $p->start();
    }
});

// 新しい子プロセスを作成
$p = new Swoole\Process('callback_function');

$p->start();
```
!> 1. 子プロセスが起動すると、親プロセスで[Swoole\Timer::tick](/timer?id=tick)によって作成されたタイマー、[Process::signal](/process/process?id=signal)でリッスンされているシグナル、および[Swoole\Event::add](/event?id=add)で追加されたイベントリスナーが自動的にクリアされます。
2. 子プロセスは親プロセスで作成された`$redis`接続オブジェクトを継承し、親子プロセスで使用される接続は同じです。
### exportSocket()

`unixSocket`を`Swoole\Coroutine\Socket`オブジェクトとしてエクスポートし、`Swoole\Coroutine\Socket`オブジェクトのメソッドを使用してプロセス間通信を行います。具体的な使用方法については、[Coroutine\socket](/coroutine_client/socket)と[IPC通信](/learn?id=什么是IPC)を参照してください。

```php
Swoole\Process->exportSocket(): Swoole\Coroutine\Socket|false
```

!> このメソッドを複数回呼び出した場合、返されるオブジェクトは同じものです。  
`exportSocket()`でエクスポートされた`socket`は新しい`fd`ですが、エクスポートした`socket`を閉じても元のプロセスのパイプに影響を与えません。  
`Swoole\Coroutine\Socket`オブジェクトであるため、[Coroutine Scheduler](/coroutine/scheduler)内で使用する必要があり、Swoole\Processのコンストラクタの`$enable_coroutine`パラメータは`true`である必要があります。  
同じく親プロセスが`Swoole\Coroutine\Socket`オブジェクトを使用する場合は、手動で`Coroutine\run()`を実行してコルーチンコンテナを作成する必要があります。

* **返り値**

  * 成功した場合は`Coroutine\Socket`オブジェクトを返します
  * プロセスがunixSocketを作成していない場合、操作に失敗したため`false`を返します

* **使用例**

単純な親子プロセス間通信の例：  

```php
use Swoole\Process;
use function Swoole\Coroutine\run;

$proc1 = new Process(function (Process $proc) {
    $socket = $proc->exportSocket();
    echo $socket->recv();
    $socket->send("hello master\n");
    echo "proc1 stop\n";
}, false, 1, true);

$proc1->start();

// 親プロセスがコルーチンコンテナを作成
run(function() use ($proc1) {
    $socket = $proc1->exportSocket();
    $socket->send("hello pro1\n");
    var_dump($socket->recv());
});
Process::wait(true);
```

より複雑な通信の例：

```php
use Swoole\Process;
use Swoole\Timer;
```php
use function Swoole\Coroutine\run;

$process = new Process(function ($proc) {
    Timer::tick(1000, function () use ($proc) {
        $socket = $proc->exportSocket();
        $socket->send("hello master\n");
        echo "child timer\n";
    });
}, false, 1, true);

$process->start();

run(function() use ($process) {
    Process::signal(SIGCHLD, static function ($sig) {
        while ($ret = Swoole\Process::wait(false)) {
            /* clean up then the event loop will exit */
            Process::signal(SIGCHLD, null);
            Timer::clearAll();
        }
    });
    /* you can run your other async or coroutine code here */
    Timer::tick(500, function () {
        echo "parent timer\n";
    });

    $socket = $process->exportSocket();
    while (1) {
        var_dump($socket->recv());
    }
});
```

!> The default type is `SOCK_STREAM`, and you need to handle TCP data packet boundary issues. Refer to the `setProtocol()` method of [Coroutine\socket](/coroutine_client/socket).

To use `SOCK_DGRAM` type for IPC communication and avoid handling TCP data packet boundary issues, refer to [IPC communication](/learn?id=what-is-ipc):

```php
use Swoole\Process;
use function Swoole\Coroutine\run;

// For IPC communication, even if the socket is of SOCK_DGRAM type, you do not need to use the sendto/recvfrom functions; send/recv is sufficient.
$proc1 = new Process(function (Process $proc) {
    $socket = $proc->exportSocket();
    while (1) {
        var_dump($socket->send("hello master\n"));
    }
    echo "proc1 stop\n";
}, false, 2, 1); // Specify SOCK_DGRAM as the pipe type in the constructor

$proc1->start();

run(function() use ($proc1) {
    $socket = $proc1->exportSocket();
    Swoole\Coroutine::sleep(5);
    var_dump(strlen($socket->recv())); // You will only receive one "hello master\n" string in each recv, not multiple instances of "hello master\n"
});
```
`Process::wait(true);` を処理します。
### name()

Modify the process name. This function is an alias of [swoole_set_process_name](/functions?id=swoole_set_process_name).

```php
Swoole\Process->name(string $name): bool
```

!> After calling `exec`, the process name will be reset by the new program; the `name` method should be used in the child process callback function after `start`.
### __construct()

构造方法。

```php
Swoole\Process->__construct(callable $function, bool $redirect_stdin_stdout = false, int $pipe_type = SOCK_DGRAM, bool $enable_coroutine = false)
```

* **参数** 

  * **`callable $function`**
    * **功能**：子进程创建成功后要执行的函数【底层会自动将函数保存到对象的`callback`属性上】,注意，该属性是`private`类私有的。
    * **默认值**：无
    * **其它值**：无

  * **`bool $redirect_stdin_stdout`**
    * **功能**：重定向子进程的标准输入和输出。【启用此选项后，在子进程内输出内容将不是打印屏幕，而是写入到主进程管道。读取键盘输入将变为从管道中读取数据。默认为阻塞读取。参考[exec()](/process/process?id=exec)方法内容】
    * **默认值**：无
    * **其它值**：无

  * **`int $pipe_type`**
    * **功能**：[unixSocket](/learn?id=什么是IPC)类型【启用`$redirect_stdin_stdout`后，此选项将忽略用户参数，强制为`SOCK_STREAM`。如果子进程内没有进程间通信，可以设置为 `0`】
    * **默认值**：`SOCK_DGRAM`
    * **其它值**：`0`、`SOCK_STREAM`

  * **`bool $enable_coroutine`**
    * **功能**：在`callback function`中启用协程，开启后可以直接在子进程的函数中使用协程API
    * **默认值**：`false`
    * **其它值**：`true`
    * **版本影响**：Swoole版本 >= v4.3.0

* **[unixSocket](/learn?id=什么是IPC)类型**

unixSocket类型 | 说明
---|---
Sure, here is the translation:

0 | 不创建
1 | [SOCK_STREAM](/learn?id=什么是IPC)タイプのunixSocketを作成
2 | [SOCK_DGRAM](/learn?id=什么是IPC)タイプのunixSocketを作成
### start()

`fork` system callを実行して、子プロセスを起動します。Linuxシステムでは、プロセスを作成するのに数百マイクロ秒かかります。

```php
Swoole\Process->start(): int|false
```

* **Return Value**

  * 成功した場合は子プロセスの`PID`
  * 失敗した場合は`false`が返されます。[swoole_errno](/functions?id=swoole_errno)と[swoole_strerror](/functions?id=swoole_strerror)を使用してエラーコードとエラーメッセージを取得できます。

* **注意**

  * 子プロセスは親プロセスのメモリとファイルハンドルを継承します
  * 子プロセスの起動時には親プロセスから継承された[EventLoop](/learn?id=什么是eventloop)、[Signal](/process/process?id=signal)、[Timer](/timer)がクリアされます
  
  !> 実行後、子プロセスは親プロセスのメモリとリソースを保持し、親プロセス内でRedis接続が作成されている場合、子プロセスではこのオブジェクトが保持され、すべての操作は同じ接続に対して行われます。以下に例を示します。

```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);

function callback_function() {
    swoole_timer_after(1000, function () {
        echo "hello world\n";
    });
    global $redis;//同じ接続
};

swoole_timer_tick(1000, function () {
    echo "parent timer\n";
});//継承されません

Swoole\Process::signal(SIGCHLD, function ($sig) {
    while ($ret = Swoole\Process::wait(false)) {
        // 新しい子プロセスを作成
        $p = new Swoole\Process('callback_function');
        $p->start();
    }
});

// 新しい子プロセスを作成
$p = new Swoole\Process('callback_function');

$p->start();
```
!> 1. 子プロセスが開始すると、親プロセスで[Swoole\Timer::tick](/timer?id=tick)によって作成されたタイマーや、[Process::signal](/process/process?id=signal)でリッスンされているシグナル、[Swoole\Event::add](/event?id=add)で追加されたイベントリスナーが自動的にクリアされます。  
2. 子プロセスは親プロセスで作成された`$redis`接続オブジェクトを継承します。親子プロセスは同じ接続を使用します。
### exportSocket()

`unixSocket`を`Swoole\Coroutine\Socket`オブジェクトとしてエクスポートし、その後`Swoole\Coroutine\socket`オブジェクトのメソッドを使用してプロセス間通信を行います。具体的な使用方法については、[Coroutine\socket](/coroutine_client/socket)および[IPC通信](/learn?id=什么是IPC)を参照してください。

```php
Swoole\Process->exportSocket(): Swoole\Coroutine\Socket|false
```

!> このメソッドを複数回呼び出すと、返されるオブジェクトは同じものです。  
`exportSocket()`でエクスポートされた`socket`は新しい`fd`であり、この`socket`を閉じてもプロセスの元々のパイプには影響しません。  
`Swoole\Coroutine\Socket`オブジェクトであるため、[コルーチンスケジューラ](/coroutine/scheduler)内で使用する必要があります。そのため、Swoole\Processのコンストラクタで`$enable_coroutine`パラメータをtrueにする必要があります。  
同様に、親プロセスが`Swoole\Coroutine\Socket`オブジェクトを使用したい場合は、手動で`Coroutine\run()`を使用してコルーチンスケジューラを作成する必要があります。

* **戻り値**

  * 成功した場合は`Coroutine\Socket`オブジェクトが返されます。
  * プロセスがunixSocketを作成していない場合、操作に失敗し`false`が返されます。

* **使用例**

簡単な親子プロセス間通信の実装： 

```php
use Swoole\Process;
use function Swoole\Coroutine\run;

$proc1 = new Process(function (Process $proc) {
    $socket = $proc->exportSocket();
    echo $socket->recv();
    $socket->send("hello master\n");
    echo "proc1 stop\n";
}, false, 1, true);

$proc1->start();

// 親プロセスはコルーチンスケジューラを作成
run(function() use ($proc1) {
    $socket = $proc1->exportSocket();
    $socket->send("hello pro1\n");
    var_dump($socket->recv());
});
Process::wait(true);
```

より複雑な通信例：

```php
use Swoole\Process;
use Swoole\Timer;
```php
use function Swoole\Coroutine\run;

$process = new Process(function ($proc) {
    Timer::tick(1000, function () use ($proc) {
        $socket = $proc->exportSocket();
        $socket->send("hello master\n");
        echo "child timer\n";
    });
}, false, 1, true);

$process->start();

run(function() use ($process) {
    Process::signal(SIGCHLD, static function ($sig) {
        while ($ret = Swoole\Process::wait(false)) {
            /* cleanup then event loop will exit */
            Process::signal(SIGCHLD, null);
            Timer::clearAll();
        }
    });
    /* you can run your other async or coroutine code here */
    Timer::tick(500, function () {
        echo "parent timer\n";
    });

    $socket = $process->exportSocket();
    while (1) {
        var_dump($socket->recv());
    }
});
```

!> 注意默认のタイプは `SOCK_STREAM` であり、TCP パケットの境界に関する問題を処理する必要があるため、`setProtocol()` メソッドを参照してください。[Coroutine\socket](/coroutine_client/socket)

`SOCK_DGRAM` タイプを使用して IPC 通信を行うと、TCP パケットの境界に関する問題を回避できます。[IPC 通信](/learn?id=什么是IPC) 参照：

```php
use Swoole\Process;
use function Swoole\Coroutine\run;

// IPC 通信では SOCK_DGRAM タイプのソケットでも sendto / recvfrom は必要ありません。send/recv ですべて処理されます。
$proc1 = new Process(function (Process $proc) {
    $socket = $proc->exportSocket();
    while (1) {
        var_dump($socket->send("hello master\n"));
    }
    echo "proc1 stop\n";
}, false, 2, 1); // 設定pipe typeを2にするとSOCK_DGRAMになります。

$proc1->start();

run(function() use ($proc1) {
    $socket = $proc1->exportSocket();
    Swoole\Coroutine::sleep(5);
    var_dump(strlen($socket->recv())); // 一度のrecvで "hello master\n" の文字列1つしか受信されないことに注意してください
}); 
```
プロセス::待機(true);
### exec()

`exec()`関数は外部プログラムを実行し、これは`exec`システムコールのラッパーです。

```php
Swoole\Process->exec(string $execfile, array $args);
```

* **パラメータ**

  * **`string $execfile`**
    * **機能**：実行可能ファイルの絶対パスを指定します。例：`"/usr/bin/python"`
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`array $args`**
    * **機能**：`exec`の引数リスト【例：`array('test.py', 123)`、これは `python test.py 123` と同等です】
    * **デフォルト値**：なし
    * **その他の値**：なし

実行に成功すると、現在のプロセスのコードセグメントが新しいプログラムに置き換えられます。子プロセスは別のプログラムに変貌します。親プロセスと現在のプロセスは引き続き親子プロセスの関係にあります。

親プロセスと新しいプロセスは標準入出力を介して通信できますが、標準入出力のリダイレクトを有効にする必要があります。

!> `$execfile`は絶対パスを使用する必要があります。そうでない場合、ファイルが存在しないエラーが発生します。  
`exec`システムコールは指定されたプログラムで現在のプログラムを上書きします。子プロセスは標準出力を読み書きし、親プロセスと通信する必要があります。  
`redirect_stdin_stdout = true` が指定されていない場合、`exec`を実行した後、子プロセスは親プロセスと通信することができません。

* **使用例**

例1：`Swoole\Process` で作成した子プロセスで [Swoole\Server](/server/init) を使用することができますが、セキュリティ上の理由から、プロセスを作成した後に`$worker->exec()`を呼び出す必要があります。以下はコード例です：

```php
$process = new Swoole\Process('callback_function', true);

$pid = $process->start();

function callback_function(Swoole\Process $worker)
{
    $worker->exec('/usr/local/bin/php', array(__DIR__.'/swoole_server.php'));
}

Swoole\Process::wait();
```
例2：Yiiプログラムを起動する

```php
$process = new \Swoole\Process(function (\Swoole\Process $childProcess) {
    // この方法はサポートされていません
    // $childProcess->exec('/usr/local/bin/php /var/www/project/yii-best-practice/cli/yii t/index -m=123 abc xyz');

    // execシステムコールをカプセル化
    // 絶対パス
    // パラメータは配列内で分けて設定する必要がある
    $childProcess->exec('/usr/local/bin/php', ['/var/www/project/yii-best-practice/cli/yii', 't/index', '-m=123', 'abc', 'xyz']); // execシステム呼び出し
});
$process->start(); // 子プロセスを起動
```

例3：親プロセスと`exec`子プロセスが標準入出力を使用して通信する例:

```php
// exec - execプロセスとの間でパイプ通信を行う
use Swoole\Process;
use function Swoole\Coroutine\run;

$process = new Process(function (Process $worker) {
    $worker->exec('/bin/echo', ['hello']);
}, true, 1, true); // 標準入出力リダイレクトを有効にする必要がある

$process->start();

run(function() use($process) {
    $socket = $process->exportSocket();
    echo "from CartLocale: " . $socket->recv() . "\n";
});
```

例4：シェルコマンドを実行する

`exec`メソッドは`PHP`の`shell_exec`とは異なり、より低レベルのシステムコールのラッパーです。シェルコマンドを実行する必要がある場合は、次の方法を使用してください：

```php
$worker->exec('/bin/sh', array('-c', "cp -rf /data/test/* /tmp/test/"));
```
### close()

[unixSocket](/learn?id=IPC)を閉じるために使用されます。

```php
Swoole\Process->close(int $which): bool
```

* **Parameters**

  * **`int $which`**
    * **Function**: unixSocketは双方向通信なので、どちらの端を閉じるかを指定します【デフォルトは`0`で読み書きを同時に閉じる、`1`：書きを閉じる、`2`：読みを閉じる】
    * **Default Value**: `0`、読み書きソケットを閉じます。
    * **Other Values**: `Swoole/Process::SW_PIPE_CLOSE_READ` 読み取りソケットを閉じる、`Swoole/Process::SW_PIPE_CLOSE_WRITE` 書き込みソケットを閉じる

!> いくつかの特殊なケースでは、`Process`オブジェクトを解放できません。プロセスを継続して作成すると接続がリークする可能性があります。この関数を呼び出すと`unixSocket`を直接閉じてリソースを解放できます。
### exit()

子プロセスを終了します。

```php
Swoole\Process->exit(int $status = 0);
```

* **パラメータ** 

  * **`int $status`**
    * **機能**：プロセスの終了ステータス【`0`の場合、正常終了を示し、クリーンアップ作業が引き続き実行されます】
    * **デフォルト値**：`0`
    * **その他の値**：なし

!> クリーンアップ作業には以下が含まれます：

  * `PHP`の`shutdown_function`
  * オブジェクトのデストラクタ（`__destruct`）
  * 他の拡張の`RSHUTDOWN`関数

`$status`が`0`以外の場合、異常終了を示し、プロセスが即座に終了し、関連する終了クリーンアップ作業は実行されません。

親プロセスで`Process::wait`を実行すると、子プロセスの終了イベントとステータスコードを取得できます。
### kill()

指定の`pid`プロセスにシグナルを送信します。

```php
Swoole\Process::kill(int $pid, int $signo = SIGTERM): bool
```

* **パラメータ** 

  * **`int $pid`**
    * **説明**：プロセス `pid`
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $signo`**
    * **説明**：送信されるシグナル【`$signo=0`の場合、プロセスの存在を確認できますが、シグナルは送信されません】
    * **デフォルト値**：`SIGTERM`
    * **その他の値**：なし
```php
Swoole\Process::signal(int $signo, callable $callback): bool
```

このメソッドは、`signalfd`と[EventLoop](/learn?id=什么是eventloop)に基づいて非同期`IO`を設定します。これはブロッキングされたプログラムで使用することはできず、登録されたリスナーコールバック関数がスケジュールされなくなります。

同期的なブロッキングプログラムでは、`pcntl_signal`を提供する`pcntl`拡張を使用できます。

既にこの信号のコールバック関数が設定されている場合、再設定すると過去の設定が上書きされます。

* **パラメータ** 

  * **`int $signo`**
    * **機能**：シグナル
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`callable $callback`**
    * **機能**：コールバック関数【`$callback`が`null`の場合、シグナルリスナーが削除されることを示す】
    * **デフォルト値**：なし
    * **その他の値**：なし

!> [Swoole\Server](/server/init)では、`SIGTERM`や`SIGALAM`など一部のシグナルのリスナーを設定できません。

* **使用例**

```php
Swoole\Process::signal(SIGTERM, function($signo) {
     echo "シャットダウン。";
});
```

!> `v4.4.0`のバージョンでは、プロセスの[EventLoop](/learn?id=什么是eventloop)に信号リスナーのイベントしかなく、他のイベント（たとえばTimerタイマーなど）がない場合、プロセスは直接終了します。

```php
Swoole\Process::signal(SIGTERM, function($signo) {
     echo "シャットダウン。";
});
Swoole\Event::wait();
```

上記のプログラムは[EventLoop](/learn?id=什么是eventloop)に入らず、`Swoole\Event::wait()`が即座に戻り、プロセスが終了します。```
### wait()

終了した子プロセスを回収します。

!> Swoole バージョンが `v4.5.0` 以上の場合、`wait()` のコルーチン版を使用することをお勧めします。詳細は[Swoole\Coroutine\System::wait()](/coroutine/system?id=wait)を参照してください。

```php
Swoole\Process::wait(bool $blocking = true): array|false
```

* **Parameters** 

  * **`bool $blocking`**
    * **機能**：ブロッキング待機するかどうかを指定します【デフォルトはブロッキング】
    * **デフォルト値**：`true`
    * **他の値**：`false`

* **Return Value**

  * 成功した場合は、子プロセスの`PID`、終了ステータスコード、`KILL`されたシグナルを含む配列が返されます
  * 失敗した場合は`false`が返されます

!> 各子プロセスが終了した後、親プロセスは必ず`wait()`を実行して回収する必要があります。そうしないと、子プロセスはゾンビプロセスになり、オペレーティングシステムのプロセスリソースが浪費されます。  
親プロセスに他のタスクがあって`wait`をブロックできず、親プロセスはシグナル`SIGCHLD`を登録し、終了するプロセスに`wait`を実行する必要があります。  
SIGCHILDシグナルが発生すると、複数の子プロセスが同時に終了する可能性があります。`wait()`をノンブロッキングモードに設定し、`false`が返るまで`wait`を繰り返し実行する必要があります。

* **Example**

```php
Swoole\Process::signal(SIGCHLD, function ($sig) {
    // ブロッキングモードは false にする必要がある
    while ($ret = Swoole\Process::wait(false)) {
        echo "PID={$ret['pid']}\n";
    }
});
```
### daemon()

Current process is transformed into a daemon process.

```php
Swoole\Process::daemon(bool $nochdir = true, bool $noclose = true): bool
```

* **Parameters** 

  * **`bool $nochdir`**
    * **Function**: Whether to switch the current directory to the root directory
    * **Default value**: `true`
    * **Other values**: `false`

  * **`bool $noclose`**
    * **Function**: Whether to close standard input and output file descriptors
    * **Default value**: `true`
    * **Other values**: `false`

!> When the process is transformed into a daemon process, the `PID` of the process will change. You can use `getmypid()` to get the current `PID`. 

Translated by Translation API - https://www.translation-api.com/
### alarm()

高精度タイマーは、オペレーティングシステムの `setitimer` システムコールのラッパーであり、マイクロ秒単位のタイマーを設定できます。タイマーがシグナルをトリガーし、[Process::signal](/process/process?id=signal) または `pcntl_signal` と共に使用する必要があります。

!> `alarm` と [Timer](/timer) を同時に使用することはできません。

```php
Swoole\Process->alarm(int $time, int $type = 0): bool
```

* **パラメータ**

  * **`int $time`**
    * **機能**：タイマーの間隔時間【負数の場合はタイマーをクリアします】
    * **単位**：マイクロ秒
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $type`**
    * **機能**：タイマーの種類
    * **デフォルト値**：`0`
    * **その他の値**：

タイマーの種類 | 説明
---|---
0 | 実時間を示し、`SIGALRM` シグナルをトリガーします
1 | ユーザーモードCPU時間を示し、`SIGVTALRM` シグナルをトリガーします
2 | ユーザーモード+カーネルモード時間を示し、`SIGPROF` シグナルをトリガーします

* **戻り値**

  * 成功した場合は `true` を返します
  * 失敗した場合は `false` を返し、エラーコードを `swoole_errno` で取得できます

* **使用例**

```php
use Swoole\Process;
use function Swoole\Coroutine\run;

run(function () {
    Process::signal(SIGALRM, function () {
        static $i = 0;
        echo "#{$i}\talarm\n";
        $i++;
        if ($i > 20) {
            Process::alarm(-1);
            Process::kill(getmypid());
        }
    });

    // 100ms
    Process::alarm(100 * 1000);

    while(true) {
        sleep(0.5);
    }
});
```
### setAffinity()

`CPU`アフィニティを設定して、プロセスを特定の`CPU`コアにバインドできます。

この関数の役割は、プロセスが特定の`CPU`コアでのみ実行されるようにし、一部の`CPU`リソースを優先的なプログラムの実行に譲ることです。

```php
Swoole\Process->setAffinity(array $cpus): bool
```

* **Parameters** 

  * **`array $cpus`**
    * **機能**: `CPU`コアをバインドする【例えば`array(0,2,3)`は`CPU0/CPU2/CPU3`をバインドすることを示す】
    * **デフォルト値**: なし
    * **その他の値**: なし

!> - `$cpus`内の要素は`CPU`コア数を超えてはいけません。
- `CPU-ID`は（`CPU`コア数 - `1`）を超えてはいけません。
- この関数は、`CPU`バインド機能をサポートするオペレーティングシステムが必要です。
- [swoole_cpu_num()](/functions?id=swoole_cpu_num) を使用すると、現在のサーバーの`CPU`コア数を取得できます。
### setPriority()

プロセス、プロセスグループ、およびユーザープロセスの優先順位を設定します。

!> Swooleバージョン >= `v4.5.9` で利用可能

```php
Swoole\Process->setPriority(int $which, int $priority): bool
```

* **パラメータ** 

  * **`int $which`**
    * **説明**：変更する優先度のタイプを決定します
    * **デフォルト値**：なし
    * **その他の値**：

| 定数         | 説明       |
| ------------ | ---------- |
| PRIO_PROCESS | プロセス     |
| PRIO_PGRP    | プロセスグループ |
| PRIO_USER    | ユーザープロセス |

  * **`int $priority`**
    * **説明**：優先度。値が小さいほど優先順位が高くなります
    * **デフォルト値**：なし
    * **その他の値**：`[-20, 20]`

* **戻り値**

  * `false`が返された場合、[swoole_errno](/functions?id=swoole_errno)および[swoole_strerror](/functions?id=swoole_strerror)を使用してエラーコードとエラーメッセージを取得できます。
### getPriority()

プロセスの優先度を取得します。

!> Swooleバージョン >= `v4.5.9` で利用可能

```php
Swoole\Process->getPriority(int $which): int
```

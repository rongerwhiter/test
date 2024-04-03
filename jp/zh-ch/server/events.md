# イベント

このセクションでは、すべてのSwooleのコールバック関数について説明します。各コールバック関数は、PHP関数に対応しており、特定のイベントに対応しています。
## onStart

?> **この関数はマスタープロセスのメインスレッドで起動後にコールバックされます**

```php
function onStart(Swoole\Server $server);
```

  * **パラメータ** 

    * **`Swoole\Server $server`**
      * **目的**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし

* **このイベントの前に、`Server`は次の操作を行いました**

    * マネージャープロセスを作成しました[Manager プロセス](/learn?id=manager进程)
    * ワーカーサブプロセスを作成しました[Worker 子プロセス](/learn?id=worker进程)
    * すべてのTCP/UDP/[unixSocket](/learn?id=什么是IPC)ポートをリッスンしていますが、接続要求を受け入れていません
    * タイマーをリッスンしています

* **次に実行される処理**

    * メイン[Reactor](/learn?id=reactor线程)がイベントを受信し始め、クライアントは`Server`に`connect`できます

**`onStart`コールバックでは、`echo`、`Log`の出力、プロセス名の変更のみが許可されています。他の操作を行ってはいけません（サービスがまだ準備できていないため、`server`関連の関数などを呼び出してはいけません）。`onWorkerStart`と`onStart`コールバックは異なるプロセスで並行して実行され、順序は関係ありません。**

`onStart`コールバックでは、`$server->master_pid`と`$server->manager_pid`の値をファイルに保存できます。このようにすることで、これらの`PID`にシグナルを送信して、シャットダウンや再起動を実装するスクリプトを書くことができます。

`onStart`イベントは、`Master`プロセスのメインスレッドで呼び出されます。

!> `onStart`で作成されたグローバルリソースオブジェクトは、`Worker`プロセスで使用できません。`onStart`が呼び出されるときには、`worker`プロセスが既に作成されています  
新しく作成されたオブジェクトはメインプロセス内にあり、`Worker`プロセスはこのメモリ領域にアクセスできません
したがって、グローバルオブジェクトの作成コードは`Server::start`の前に配置する必要があります。典型的な例は[Swoole\Table](/memory/table?id=完全な例)です。

* **セキュリティ警告**

`onStart`コールバックで非同期およびコルーチンAPIを使用することができますが、これは`dispatch_func`および`package_length_func`と競合する可能性があるため、**同時に使用しないでください**。

`onStart`内でタイマーを開始しないでください。コード内で`Swoole\Server::shutdown()`操作を実行すると、プログラムが終了しないため、常にタイマーが実行されるためです。

`onStart`コールバックでは、`return`されるまでサーバープログラムはクライアント接続を受け付けないため、同期ブロッキング関数を安全に使用できます。

* **BASE モード**

[SWOOLE_BASE](/learn?id=swoole_base)モードでは`master`プロセスが存在しないため、`onStart`イベントは存在しません。そのため、`BASE`モードで`onStart`コールバック関数を使用しないでください。

```
WARNING swReactorProcess_start: The onStart event with SWOOLE_BASE is deprecated
```
## onBeforeShutdown

?> **This event occurs before the `Server` shuts down gracefully** 

!> Available since Swoole version >= `v4.8.0`. You can use coroutine API in this event.

```php
function onBeforeShutdown(Swoole\Server $server);
```


* **Parameters**

    * **`Swoole\Server $server`**
        * **Description**: Swoole\Server object
        * **Default**: None
        * **Other values**: None
## onShutdown

?> **This event occurs when the `Server` is shutting down normally.**

```php
function onShutdown(Swoole\Server $server);
```

  * **Parameters**

    * **`Swoole\Server $server`**
      * **Description**：Swoole\Server object
      * **Default value**：None
      * **Other values**：None

  * **Before this, the `Swoole\Server` has performed the following operations**

    * Closed all [Reactor](/learn?id=reactor-thread) threads, `HeartbeatCheck` thread, `UdpRecv` thread
    * Closed all `Worker` processes, [Task processes](/learn?id=taskworker-process), [User processes](/server/methods?id=addprocess)
    * `close`d all `TCP/UDP/UnixSocket` listening ports
    * Closed the main [Reactor](/learn?id=reactor-thread)

  !> Forcibly `kill`ing processes will not trigger `onShutdown`, like `kill -9`.  
  You need to use `kill -15` to send the `SIGTERM` signal to the main process for a normal termination to follow.  
  Exiting the program by pressing `Ctrl+C` in the command line will stop it immediately without triggering `onShutdown`.

  * **Notes**

  !> Do not call any asynchronous or coroutine-related `API` in `onShutdown` as all event loop facilities are destroyed when `onShutdown` is triggered.  
There is no coroutine environment at this point, so if a developer needs to use coroutine-related `API`, they need to manually call `Co\run` to create a [coroutine container](/coroutine?id=what-is-a-coroutine-container).
## onStart

?> **此函数在主进程（master）的主线程中回调**

```php
function onStart(Swoole\Server $server);
```

  * **参数** 

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **他の値**：なし

* **`Server`は以下の操作を完了しています**

    * [Managerプロセス](/learn?id=manager进程)が起動されました
    * [Workerプロセス](/learn?id=worker进程)が作成されました
    * すべてのTCP/UDP/[unixSocket](/learn?id=什么是IPC)ポートがリッスンされており、接続やリクエストの受け入れはまだ開始されていません
    * タイマーがリッスンされています

* **次に行われる操作**

    * メインの[Reactor](/learn?id=reactor线程)はイベントを受け付け始め、クライアントが`Server`に`connect`できるようになります

**`onStart`コールバックでは、`echo`、`Log`の出力、プロセス名の変更のみが許可されます。他の操作（`server`関連の関数の呼び出しなど）は実行できません（サービスがまだ準備できていないため）。`onWorkerStart`と`onStart`コールバックは異なるプロセスで並行して実行され、順序は関係ありません。**

`onStart`コールバック内で、`$server->master_pid`と`$server->manager_pid`の値をファイルに保存できます。これにより、これらの`PID`にシグナルを送信してシャットダウンや再起動を実行するスクリプトを作成できます。

`onStart`イベントは`Master`プロセスのメインスレッドで呼び出されます。

!> `onStart`で作成されたグローバルリソースオブジェクトは、`Worker`プロセスで使用できません。なぜなら、`onStart`が呼び出された時点で`worker`プロセスがすでに作成されているからです  
新規作成したオブジェクトはメインプロセス内にあり、`Worker`プロセスはこのメモリ領域にアクセスできません
したがって、グローバルオブジェクトの作成コードは、`Server::start`の前に配置する必要があります。典型的な例は、[Swoole\Table](/memory/table?id=完全な例)です。

* **セキュリティ注意**

`onStart`コールバックでは非同期およびコルーチンAPIを使用できますが、`dispatch_func`および`package_length_func`と競合する可能性があるため、**同時に使用しないでください**。

`onStart`でタイマーを起動しないでください。コードで`Swoole\Server::shutdown()`を実行すると、終了しないタイマーが常に実行されるため、プログラムが終了できません。

`onStart`コールバックは、`return`するまでクライアント接続を受け付けないため、同期ブロッキング関数を安全に使用できます。

* **BASE モード**

[SWOOLE_BASE](/learn?id=swoole_base)モードでは`master`プロセスが存在しないため、`onStart`イベントも存在しません。`BASE`モードで`onStart`コールバック関数を使用しないでください。

```
WARNING swReactorProcess_start: The onStart event with SWOOLE_BASE is deprecated
```
## onWorkerStart

?> **このイベントは Worker プロセス/ [Task プロセス](/learn?id=taskworkerプロセス) の起動時に発生し、プロセスのライフサイクル全体で作成したオブジェクトを使用できます。**

```php
function onWorkerStart(Swoole\Server $server, int $workerId);
```

  * **パラメータ** 

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $workerId`**
      * **機能**：`Worker` プロセスの `id`（プロセスの PID ではない）
      * **デフォルト値**：なし
      * **その他の値**：なし

  * `onWorkerStart/onStart`は並列実行されますが、順序はありません
  * 現在が`Worker`プロセスであるか [Task プロセス](/learn?id=taskworkerプロセス) であるかを判断するには、`$server->taskworker`プロパティを使用できます
  * `worker_num`と`task_worker_num`を`1`より多く設定した場合、各プロセスで`onWorkerStart`イベントが1回発生し、[$worker_id](/server/properties?id=worker_id)を確認して異なるワーカープロセスを区別できます
  * `worker` プロセスから `task` プロセスにタスクを送信し、`task` プロセスがすべてのタスクを処理した後に[onFinish](/server/events?id=onfinish)コールバック関数で `worker` プロセスに通知します。たとえば、バックグラウンド操作で10万人のユーザーに通知メールを送信し、操作が完了すると状態は送信中に表示され、その時点で他の操作を続行できます。メール送信が完了したら、操作の状態は自動的に「送信済み」に変更されます。

以下の例は、Worker プロセス/ [Task プロセス](/learn?id=taskworkerプロセス) の名前を変更するためのものです。

```php
$server->on('WorkerStart', function ($server, $worker_id){
    global $argv;
    if($worker_id >= $server->setting['worker_num']) {
```php
swoole_set_process_name("php {$argv[0]} task worker");
    } else {
        swoole_set_process_name("php {$argv[0]} event worker");
    }
});
```

  如果想使用[Reload](/server/methods?id=reload)机制实现代码重载入，必须在`onWorkerStart`中`require`你的业务文件，而不是在文件头部。在`onWorkerStart`调用之前已包含的文件，不会重新载入代码。

  可以将公用的、不易变的php文件放置到`onWorkerStart`之前。这样虽然不能重载入代码，但所有`Worker`是共享的，不需要额外的内存来保存这些数据。
`onWorkerStart`之后的代码每个进程都需要在内存中保存一份

  * `$worker_id`表示这个`Worker`进程的`ID`，范围参考[$worker_id](/server/properties?id=worker_id)
  * [$worker_id](/server/properties?id=worker_id)和进程`PID`没有任何关系，可使用`posix_getpid`函数获取`PID`

  * **协程支持**

    * 在`onWorkerStart`回调函数中会自动创建协程，所以`onWorkerStart`可以调用协程`API`

  * **注意**

    !> 発生致命エラーやコード内で`exit`を明示的に呼び出すと、`Worker/Task`プロセスは終了し、管理プロセスが新しいプロセスを作成します。これにより無限ループが発生し、プロセスが繰り返し作成および終了する可能性があります。
## onWorkerStop

?> **This event occurs when a `Worker` process terminates. In this function, you can recycle various resources allocated by the `Worker` process.**

```php
function onWorkerStop(Swoole\Server $server, int $workerId);
```

  * **Parameters** 

    * **`Swoole\Server $server`**
      * **Function**: Swoole\Server object
      * **Default value**: None
      * **Other values**: None

    * **`int $workerId`**
      * **Function**: ID of the `Worker` process (not the PID of the process)
      * **Default value**: None
      * **Other values**: None

  * **Note**

    !> - If the process terminates abnormally, such as being forcibly `kill`ed, encountering a fatal error, or causing a `core dump`, the `onWorkerStop` callback function cannot be executed.  
    - Do not call any asynchronous or coroutine-related APIs in `onWorkerStop`. When `onWorkerStop` is triggered, all underlying [event loop](/learn?id=what-is-an-event-loop) facilities have already been destroyed.
## onWorkerExit

?> **This is only valid when the [reload_async](/server/setting?id=reload_async) feature is enabled. See [How to properly restart the service](/question/use?id=how-to-properly-restart-the-swoole-service)**

```php
function onWorkerExit(Swoole\Server $server, int $workerId);
```

  * **Parameters** 

    * **`Swoole\Server $server`**
      * **Description**：Swoole\Server object
      * **Default**：None
      * **Other values**：None

    * **`int $workerId`**
      * **Description**：`Worker` process `id` (not the process PID)
      * **Default**：None
      * **Other values**：None

  * **Notes**

    !> -`onWorkerExit` will continue to trigger if the `Worker` process does not exit  
    -`onWorkerExit` will be triggered within the `Worker` process, and if there is an [event loop](/learn?id=what-is-eventloop) in the [Task process](/learn?id=taskworker-process), it will also trigger  
    -In `onWorkerExit`, remove/close asynchronous `Socket` connections as much as possible. The process will exit when the number of event handles being listened to in the underlying [event loop](/learn?id=what-is-eventloop) is `0`  
    -When there are no event handles being listened to by the process, this function will not be called when the process ends  
    -`onWorkerStop` event callback will be executed only after the `Worker` process exits.
## onConnect

?> **有新的接続がワーカープロセス内で発生したときにコールバックされます。**

```php
function onConnect(Swoole\Server $server, int $fd, int $reactorId);
```

  * **パラメータ** 

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $fd`**
      * **機能**：接続のファイルディスクリプタ
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $reactorId`**
      * **機能**：接続が発生した[Reactor](/learn?id=reactor糸)スレッドの`ID`
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **注意**

    !> `onConnect/onClose`この`2`つのコールバックは`Worker`プロセス内で発生し、メインプロセスではありません。  
    `UDP`プロトコルでは[onReceive](/server/events?id=onreceive)イベントのみがあり、`onConnect/onClose`イベントはありません。

    * **[dispatch_mode](/server/setting?id=dispatch_mode) = 1/3**

      * このモードでは`onConnect/onReceive/onClose`が異なるプロセスに投稿される可能性があります。接続に関連する`PHP`オブジェクトデータは、[onConnect](/server/events?id=onconnect) コールバックでデータの初期化ができず、[onClose](/server/events?id=onclose) でデータのクリーンアップができません。
      * `onConnect/onReceive/onClose`の3つのイベントは並行して実行され、異常を引き起こす可能性があります。
## onStart

?> **This function is called back in the main thread of the master process after startup.**

```php
function onStart(Swoole\Server $server);
```

  * **Parameters**

    * **`Swoole\Server $server`**
      * **Functionality**: Swoole\Server object
      * **Default value**: None
      * **Other values**: None

* **Before this event, the `Server` has performed the following operations**

    * Manager process has been created upon startup ([Manager process](/learn?id=manager-process))
    * Worker processes have been created upon startup ([Worker process](/learn?id=worker-process))
    * All TCP/UDP/[Unix socket](/learn?id=ipc) ports are listening, but not yet accepting connections and requests
    * Timers are set up

* **Next step to be executed**

    * The main [Reactor thread](/learn?id=reactor-thread) starts processing events, and clients can `connect` to the `Server`

**In the `onStart` callback, only `echo`, printing `Log`, and changing process names are allowed. No other operations should be performed (cannot call `server` related functions, etc., as the service is not yet ready). The `onWorkerStart` and `onStart` callbacks are executed in parallel in different processes, without any specific order.**

In the `onStart` callback, you can save the values of `$server->master_pid` and `$server->manager_pid` to a file. This way, you can write a script to send signals to these two `PID`s to achieve shutdown and restart operations.

The `onStart` event is called back in the main thread of the `Master` process.

!> Global resource objects created in `onStart` cannot be used in the `Worker` processes because when `onStart` is called, the `worker` processes are already created.  
Objects created are within the main process and are inaccessible to `Worker` processes.
したがって、グローバルオブジェクトを作成するコードは`Server::start`の前に配置する必要があります。典型的な例としては、[Swoole\Table](/memory/table?id=完全な例)があります。

* **セキュリティの注意**

`onStart`コールバックでは非同期およびコルーチンのAPIを使用できますが、`dispatch_func`および`package_length_func`と競合する可能性があるため**同時に使用しない**ように注意してください。

`onStart`でタイマーを開始しないでください。コード内で`Swoole\Server::shutdown()`操作を実行した場合、常に実行中のタイマーがあるためプログラムが終了しなくなります。

`onStart`コールバックは`return`するまでサーバープログラムがクライアント接続を受け入れないため、同期ブロッキング関数を安全に使用できます。

* **BASE モード**

[SWOOLE_BASE](/learn?id=swoole_base)モードでは`master`プロセスが存在しないため、`onStart`イベントは存在しません。`BASE`モードで`onStart`コールバック関数を使用しないでください。

```
WARNING swReactorProcess_start: The onStart event with SWOOLE_BASE is deprecated
```
## onWorkerStart

?> **This event occurs when a Worker process/ [Task process](/learn?id=taskworker-process) starts, and objects created here can be used throughout the process lifecycle.**

```php
function onWorkerStart(Swoole\Server $server, int $workerId);
```

  * **Parameters** 

    * **`Swoole\Server $server`**
      * **Function**: Swoole\Server object
      * **Default value**: None
      * **Other values**: None

    * **`int $workerId`**
      * **Function**: `Worker` process `id` (not the process PID)
      * **Default value**: None
      * **Other values**: None

  * `onWorkerStart/onStart` are executed concurrently without a specific order
  * You can determine if the current process is a `Worker` process or a [Task process](/learn?id=taskworker-process) by checking the `$server->taskworker` property
  * When `worker_num` and `task_worker_num` are set to more than `1`, each process will trigger the `onWorkerStart` event, and you can differentiate between different worker processes using [$worker_id](/server/properties?id=worker_id)
  * Tasks can be sent from `worker` processes to `task` processes, and after all tasks are processed, the `task` process notifies the `worker` process through the [onFinish](/server/events?id=onfinish) callback function. For example, when running a background operation to send notification emails to thousands of users, the operation's status changes to "sending" until all emails are sent, and then it automatically changes to "sent."

  The following example is for renaming Worker processes/ [Task processes](/learn?id=taskworker-process).

```php
$server->on('WorkerStart', function ($server, $worker_id){
    global $argv;
    if($worker_id >= $server->setting['worker_num']) {```
```php
swoole_set_process_name("php {$argv[0]} task worker");
    } else {
        swoole_set_process_name("php {$argv[0]} event worker");
    }
});
```

  如果想使用[Reload](/server/methods?id=reload)机制实现代码重载入，必须在`onWorkerStart`中`require`你的业务文件，而不是在文件头部。在`onWorkerStart`调用之前已包含的文件，不会重新载入代码。

  可以将公用的、不易变的php文件放置到`onWorkerStart`之前。这样虽然不能重载入代码，但所有`Worker`是共享的，不需要额外的内存来保存这些数据。
`onWorkerStart`之后的代码每个进程都需要在内存中保存一份

  * `$worker_id`表示这个`Worker`进程的`ID`，范围参考[$worker_id](/server/properties?id=worker_id)
  * [$worker_id](/server/properties?id=worker_id)和进程`PID`没有任何关系，可使用`posix_getpid`函数获取`PID`

  * **协程支持**

    * 在`onWorkerStart`回调函数中会自动创建协程，所以`onWorkerStart`可以调用协程`API`

  * **注意**

    !> 发生致命错误或者代码中主动调用`exit`时，`Worker/Task`进程会退出，管理进程会重新创建新的进程。这可能导致死循环，不停地创建销毁进程
## onReceive

?> **`worker`プロセス内でデータが受信された場合にこの関数がコールバックされます。**

```php
function onReceive(Swoole\Server $server, int $fd, int $reactorId, string $data);
```

  * **パラメータ**

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $fd`**
      * **機能**：接続のファイルディスクリプタ
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $reactorId`**
      * **機能**：`TCP`接続がある[Reactor](/learn?id=reactor线程)スレッドの`ID`
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $data`**
      * **機能**：受信したデータの内容、テキストまたはバイナリコンテンツが含まれる可能性があります
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **`TCP`プロトコルにおけるパケット完全性については、[TCPデータパケットの境界問題](/learn?id=tcp数据包边界问题)を参照してください**

    * `open_eof_check/open_length_check/open_http_protocol`などの下層の設定を使用すると、データパケットの完全性が確保されます
    * 下層のプロトコル処理を使用しない場合、[onReceive](/server/events?id=onreceive)後に、PHPコードでデータを分析し、データパケットを結合したり分割したりする必要があります。

    例：コード内で `$buffer = array()` を追加し、`$fd`を`key`として使用してコンテキストデータを保存します。 データを受信するたびに文字列を結合し、`$buffer[$fd] .= $data`、次に`$buffer[$fd]`が完全なデータパケットかどうかを確認します。
デフォルトでは、同じ`fd`は同じ`Worker`に割り当てられるため、データを結合することができます。`dispatch_mode` = 3を使用すると、リクエストデータは競合します。同じ`fd`から送信されたデータは異なるプロセスに割り当てられる可能性があるため、前述のデータパケットの結合方法は使用できません。

  * **複数ポートのリスニング、[このセクション](/server/port)を参照**

    メインサーバーでプロトコルを設定した場合、追加のリッスンポートはデフォルトでメインサーバーの設定を継承します。ポートのプロトコルを再設定するには、`set`メソッドを明示的に呼び出す必要があります。

    ```php
    $server = new Swoole\Http\Server("127.0.0.1", 9501);
    $port2 = $server->listen('127.0.0.1', 9502, SWOOLE_SOCK_TCP);
    $port2->on('receive', function (Swoole\Server $server, $fd, $reactor_id, $data) {
        echo "[#".$server->worker_id."]\tClient[$fd]: $data\n";
    });
    ```

    ここでは`on`メソッドを使用して[onReceive](/server/events?id=onreceive)コールバック関数を登録していますが、メインサーバーのプロトコルをオーバーライドするために`set`メソッドを呼び出していないため、新しくリッスンされた`9502`ポートは引き続き`HTTP`プロトコルを使用します。`telnet`クライアントを使用して`9502`ポートに接続し、文字列を送信すると、サーバーは[onReceive](/server/events?id=onreceive)をトリガーしません。

  * **注意**

    !> プロトコル自動選択がオフの場合、[onReceive](/server/events?id=onreceive) 1回あたりのデータ最大サイズは`64K`です。  
    プロトコル自動処理オプションが有効になっている場合、[onReceive](/server/events?id=onreceive)は完全なデータパケットを受信し、最大で[package_max_length](/server/setting?id=package_max_length)を超えないでしょう。
サポートはバイナリ形式で、`$data`はバイナリデータかもしれません。
## onPacket

?> **`UDP`データパケットを受信したときに、この関数が`worker`プロセス内でコールバックされます。**

```php
function onPacket(Swoole\Server $server, string $data, array $clientInfo);
```

  * **パラメーター**

    * **`Swoole\Server $server`**
      * **説明**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $data`**
      * **説明**：受信したデータ内容、テキストまたはバイナリコンテンツの可能性があります
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`array $clientInfo`**
      * **説明**：クライアント情報には`address/port/server_socket`などの多くのクライアント情報データが含まれます。[UDPサーバー](/start/start_udp_server)を参照
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **注意**

    !> サーバーが`TCP/UDP`ポートを同時にリッスンしている場合、`TCP`プロトコルのデータが受信されると[onReceive](/server/events?id=onreceive)がコールバックされ、`UDP`データパケットが受信されると`onPacket`がコールバックされます。サーバーに設定された`EOF`や`Length`などの自動プロトコル処理([TCPデータパケット境界の問題](/learn?id=tcp数据包边界问题)) は、`UDP`ポートに対しては無効です。なぜなら、`UDP`パケット自体にメッセージの境界が存在し、追加のプロトコル処理は必要ないからです。
## onStart

?> **この関数は、マスタープロセスのメインスレッドで、起動後にコールバックされます**

```php
function onStart(Swoole\Server $server);
```

  * **パラメータ** 

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし

* **このイベント前に`Server`は以下の操作を行っています**

    * マネージャープロセスの作成を起動
    * ワーカーサブプロセスの作成を起動
    * すべてのTCP/UDP/Unixソケットポートをリッスンしており、接続やリクエストの受け入れを開始していません
    * タイマーをリッスンしています

* **次に行われる操作**

    * メインの[リアクタ](/learn?id=reactor糸)がイベントを受け入れ始め、クライアントが`Server`に`connect`できるようになります

**`onStart`コールバックでは、`echo`、`Log`の出力、プロセス名の変更が許可されています。他の操作（`server`関連の関数など）は実行してはいけません（サービスが準備されていないため）。`onWorkerStart`と`onStart`コールバックは異なるプロセスで並行して実行され、順序はありません。**

`onStart`コールバックでは、`$server->master_pid`と`$server->manager_pid`の値をファイルに保存できます。これにより、これら2つの`PID`に対してシャットダウンや再起動の操作を行うスクリプトを記述できます。

`onStart`イベントは、`Master`プロセスのメインスレッドで呼び出されます。

!> `onStart`で作成されたグローバルリソースオブジェクトは、`Worker`プロセスで使用できません。なぜなら、`onStart`が呼び出される時点で`Worker`プロセスはすでに作成されているからです  
新しく作成されたオブジェクトは、マスタープロセス内にあり、`Worker`プロセスはこのメモリ領域にアクセスできません
したがって、グローバルオブジェクトを作成するコードは`Server::start`の前に配置する必要があります。典型的な例は[Swoole\Table](/memory/table?id=完全な例)

* **セキュリティのヒント**

`onStart`コールバックでは非同期およびコルーチンAPIを使用できますが、これにより`dispatch_func`および`package_length_func`と競合する可能性があるため、**同時に使用しないでください**。

`onStart`でタイマーを開始しないでください。コード中で`Swoole\Server::shutdown()`操作を実行した場合、常にタイマーが実行されるためプログラムが終了できなくなります。

`onStart`コールバックでは、`return`するまでサーバープログラムは任意のクライアント接続を受け付けません。そのため、同期ブロッキング関数を安全に使用できます。

* **BASEモード**

[SWOOLE_BASE](/learn?id=swoole_base)モードでは`master`プロセスが存在しないため、`onStart`イベントも存在しません。`BASE`モードで`onStart`コールバック関数を使用しないでください。

```
WARNING swReactorProcess_start: The onStart event with SWOOLE_BASE is deprecated
```
## onWorkerStart

?> **このイベントは Worker プロセス / [タスクプロセス](/learn?id=taskworkerプロセス) の起動時に発生し、プロセスのライフサイクル内で使用できるオブジェクトをここで作成できます。**

```php
function onWorkerStart(Swoole\Server $server, int $workerId);
```

  * **パラメータ** 

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $workerId`**
      * **機能**：`Worker` プロセスの `id`（プロセスのPIDではない）
      * **デフォルト値**：なし
      * **その他の値**：なし

  * `onWorkerStart/onStart` は並行して実行され、順序はありません
  * `$server->taskworker` プロパティを使用して現在のプロセスが `Worker` プロセスか [Taskプロセス](/learn?id=taskworkerプロセス) かを判断できます
  * `worker_num` および `task_worker_num` を `1` を超えるように設定した場合、各プロセスはすべて `onWorkerStart` イベントを一度だけトリガーします。異なるワーカープロセスを識別するために [$worker_id](/server/properties?id=worker_id) を使用できます
  * `worker` プロセスから `task` プロセスにタスクを送信し、`task` プロセスがすべてのタスクを処理した後に、 [onFinish](/server/events?id=onfinish) コールバック関数を使用して `worker` プロセスに通知します。例えば、バックグラウンド操作で十万人のユーザーに通知メールを送信する場合、操作が完了すると、操作の状態が送信中に表示されます。この時、他の操作を続行し、メール送信が完了すると、操作の状態が自動的に「送信済み」に変更されます。

  以下の例は、Worker プロセス / [Taskプロセス](/learn?id=taskworkerプロセス) の名前を変更するためのものです。

```php
$server->on('WorkerStart', function ($server, $worker_id){
    global $argv;
    if($worker_id >= $server->setting['worker_num']) {```
```php
swoole_set_process_name("php {$argv[0]} task worker");
    } else {
        swoole_set_process_name("php {$argv[0]} event worker");
    }
});
```

  如果想使用[Reload](/server/methods?id=reload)机制实现代码重载入，必须在`onWorkerStart`中`require`你的业务文件，而不是在文件头部。在`onWorkerStart`调用之前已包含的文件，不会重新载入代码。

  可以将公用的、不易变的php文件放置到`onWorkerStart`之前。这样虽然不能重载入代码，但所有`Worker`是共享的，不需要额外的内存来保存这些数据。
`onWorkerStart`之后的代码每个进程都需要在内存中保存一份

  * `$worker_id`表示这个`Worker`进程的`ID`，范围参考[$worker_id](/server/properties?id=worker_id)
  * [$worker_id](/server/properties?id=worker_id)和进程`PID`没有任何关系，可使用`posix_getpid`函数获取`PID`

  * **协程支持**

    * 在`onWorkerStart`回调函数中会自动创建协程，所以`onWorkerStart`可以调用协程`API`

  * **注意**

    !> 发生致命错误或者代码中主动调用`exit`时，`Worker/Task`进程会退出，管理进程会重新创建新的进程。这可能导致死循环，不停地创建销毁进程
```
## onReceive

?> **`worker`プロセスでデータを受け取ると、この関数がコールバックされます。**

```php
function onReceive(Swoole\Server $server, int $fd, int $reactorId, string $data);
```

  * **パラメータ** 

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $fd`**
      * **機能**：接続のファイルディスクリプタ
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $reactorId`**
      * **機能**：`TCP`接続が存在する[Reactor](/learn?id=reactor线程)スレッドの`ID`
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $data`**
      * **機能**：受信したデータの内容、テキストまたはバイナリデータのいずれか
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **`TCP`プロトコルのパケット完全性について、[TCPデータパケットの境界問題](/learn?id=tcp数据包边界问题)を参照してください**

    * 使用提供された`open_eof_check/open_length_check/open_http_protocol`などの低レベルの設定を使用すると、パケットの完全性が確保されます
    * レイヤー協定の処理を使用しない場合、[onReceive](/server/events?id=onreceive)の後、PHPコード内でデータを分析して、パケットを結合/分割します。

    例：コード内で`$buffer = array()`を追加し、`$fd`を`key`として使用してコンテキストデータを保存します。 データを受け取るたびに文字列を連結し、`$buffer[$fd] .= $data`とします。その後、`$buffer[$fd]`の文字列が完全なデータパケットかどうかを判断します。
デフォルトでは、同じ`fd`は同じ`Worker`に割り当てられるため、データを連結することができます。[dispatch_mode](/server/setting?id=dispatch_mode) = 3を使用すると、要求データはプリエンプティブであり、同じ`fd`からのデータは異なるプロセスに割り当てられる可能性があるため、前述のデータパケット結合方法は使用できません。

  * **複数ポートのリッスンについては、[このセクション](/server/port)を参照してください**

    メインサーバーにプロトコルが設定されている場合、追加でリッスンされたポートはデフォルトでメインサーバーの設定を継承します。ポートのプロトコルを再設定するには、明示的に`set`メソッドを呼び出す必要があります。

    ```php
    $server = new Swoole\Http\Server("127.0.0.1", 9501);
    $port2 = $server->listen('127.0.0.1', 9502, SWOOLE_SOCK_TCP);
    $port2->on('receive', function (Swoole\Server $server, $fd, $reactor_id, $data) {
        echo "[#".$server->worker_id."]\tClient[$fd]: $data\n";
    });
    ```

    ここで、`on`メソッドを呼んで[onReceive](/server/events?id=onreceive)コールバック関数を登録していますが、`set`メソッドを呼び出してメインサーバーのプロトコルをオーバーライドしていないため、新しくリッスンされた`9502`ポートは依然として`HTTP`プロトコルを使用します。`telnet`クライアントを使って`9502`ポートに接続して文字列を送信すると、サーバーは[onReceive](/server/events?id=onreceive)をトリガーしません。

  * **注意**

    !> 自動プロトコルオプションがオフになっている場合、[onReceive](/server/events?id=onreceive)で一度に受信できるデータの最大サイズは`64K`です。  
    自動プロトコル処理オプションがオンになっている場合、[onReceive](/server/events?id=onreceive)は完全なデータパケットを受信し、最大で[package_max_length](/server/setting?id=package_max_length)未満のデータを受信します。
`$data` 変数がバイナリデータである可能性があるので、バイナリ形式をサポートしています。
## onClose

?> **`TCP`クライアント接続が閉じられた後、`Worker`プロセスでこの関数がコールバックされます。**

```php
function onClose(Swoole\Server $server, int $fd, int $reactorId);
```

  * **パラメーター** 

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $fd`**
      * **機能**：接続のファイル記述子
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $reactorId`**
      * **機能**：どの`reactor`スレッドから来たか、アクティブな`close`は負の値となります
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **ヒント**

    * **アクティブなクローズ**

      * サーバーが接続をアクティブに閉じると、このパラメーターが`-1`に設定されます。`$reactorId < 0`をチェックして、閉じるがサーバー側からユーザー側によって行われたかを判別できます。
      * `close`メソッドがPHPコードでアクティブに呼び出された場合のみ、アクティブなクローズと見なされます。

    * **ハートビートチェック**

      * [ハートビートチェック](/server/setting?id=heartbeat_check_interval)はハートビートスレッドによって通知された閉じるものであり、閉じるときの[onClose](/server/events?id=onclose)の`$reactorId`パラメーターは`-1`ではありません。

  * **注意**

    !> [onClose](/server/events?id=onclose) コールバック関数に致命的なエラーが発生した場合、接続がリークします。`netstat`コマンドを使用すると、大量の`CLOSE_WAIT`状態の`TCP`接続が見られます。[Swooleビデオチュートリアルを参照](https://course.swoole-cloud.com/course-video/4)
- クライアント側が`close`をトリガーするか、サーバー側が`$server->close()`を呼び出して接続を閉じる場合、どちらもこのイベントが発生します。したがって、接続が閉じられると必ずこの関数がコールバックされます  
    - [onClose](/server/events?id=onclose) 内で、まだ[getClientInfo](/server/methods?id=getClientInfo)メソッドを使用して接続情報を取得することができます。[onClose](/server/events?id=onclose) コールバック関数が完了すると`close`によって`TCP`接続が閉じられます  
    - [onClose](/server/events?id=onclose) が呼び出される時点でクライアント接続が既に閉じていることを示すため、`$server->close($fd)` を実行する必要はありません。コード内で`$server->close($fd)` を実行すると`PHP`のエラー警告が発生します。
## onStart

?> **この関数は、マスタープロセスのメインスレッドで、起動後にコールバックされます**

```php
function onStart(Swoole\Server $server);
```

  * **パラメータ** 

    * **`Swoole\Server $server`**
      * **説明**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし

* **このイベントが発生する前に、`Server`は次の操作を実行しています**

    * マネージャープロセスを起動[Manager プロセス](/learn?id=managerプロセス)
    * ワーカー子プロセスを起動[Worker プロセス](/learn?id=workerプロセス)
    * すべてのTCP/UDP/[unixSocket](/learn?id=IPCとは)ポートを監視し、接続や要求の受け入れを開始していません
    * タイマーを監視しています

* **次に行われるべき操作**

    * メインの[Reactor](/learn?id=reactorスレッド)がイベントを受け付け始め、クライアントが`Server`に`connect`できるようになります

**`onStart`のコールバック内では、`echo`、`Log`の出力、プロセス名の変更のみが許可されます。他の操作（サーバー関連の関数の呼び出しなど）は行ってはいけません。`onWorkerStart`と`onStart`のコールバックは異なるプロセスで並列に実行され、順序が存在しません。**

`onStart`コールバック内で、`$server->master_pid`と`$server->manager_pid`の値をファイルに保存することができます。これにより、これらの2つの`PID`にシグナルを送信してシャットダウンや再起動をするスクリプトを作成できます。

`onStart`イベントは`Master`プロセスのメインスレッドで呼び出されます。

!> `onStart`で作成したグローバルリソースオブジェクトは、`Worker`プロセスで使用できません。なぜならば、`onStart`が呼び出されるときには既に`worker`プロセスが作成されているためです  
新しく作成されたオブジェクトは、メインプロセス内にあり、`Worker`プロセスはこのメモリ領域にアクセスできません.
したがって、グローバルオブジェクトを作成するコードは`Server::start`の前に配置する必要があります。一般的な例は[Swoole\Table](/memory/table?id=完全な例)です。

* **セキュリティ注意**

`onStart`コールバックで非同期およびコルーチンAPIを使用できますが、`dispatch_func`や`package_length_func`との競合に注意する必要があります。**同時に使用しないでください**。

`onStart`でタイマーを起動しないでください。コード内で`Swoole\Server::shutdown()`操作が実行されると、常にタイマーが実行されるため、プログラムが終了しなくなります。

`onStart`コールバックでは、`return`するまで、サーバープログラムはクライアント接続を受け付けません。そのため、同期ブロック関数を安全に使用できます。

* **BASE モード**

[SWOOLE_BASE](/learn?id=swoole_base)モードでは`master`プロセスが存在しないため、`onStart`イベントは存在しません。`BASE`モードで`onStart`コールバック関数を使用しないでください。

```
WARNING swReactorProcess_start: The onStart event with SWOOLE_BASE is deprecated
```
## onWorkerStart

?> **このイベントは、Workerプロセス/ [Taskプロセス](/learn?id=taskworkerプロセス)が起動するときに発生し、この時点で作成されたオブジェクトはプロセスのライフサイクル全体で使用できます。**

```php
function onWorkerStart(Swoole\Server $server, int $workerId);
```

  * **パラメーター**

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $workerId`**
      * **機能**：`Worker`プロセスの`id`（プロセスの PIDではない）
      * **デフォルト値**：なし
      * **その他の値**：なし

  * `onWorkerStart/onStart`は並行して実行され、順序はありません。
  * 現在が`Worker`プロセスなのか [Taskプロセス](/learn?id=taskworkerプロセス)なのかを判断するために、`$server->taskworker`プロパティを使用できます。
  * `worker_num`と`task_worker_num`を`1`を超える値に設定した場合、各プロセスは一度`onWorkerStart`イベントをトリガーします。各々のワーカープロセスを区別するために[$worker_id](/server/properties?id=worker_id)を使用できます。
  * `worker`プロセスから`task`プロセスにタスクを送信し、`task`プロセスがすべてのタスクを処理した後、[onFinish](/server/events?id=onfinish)コールバック関数を使用して`worker`プロセスに通知します。たとえば、バックグラウンドで10万人のユーザーに通知メールを送信し、操作が完了した後に操作の状態が送信中に表示される場合、そのまま他の操作を続行し、メールの送信が終了したら自動的に操作の状態が送信済みに変わります。

  以下の例は、Workerプロセス/ [Taskプロセス](/learn?id=taskworkerプロセス)の名前を変更するためのものです。

```php
$server->on('WorkerStart', function ($server, $worker_id){
    global $argv;
    if($worker_id >= $server->setting['worker_num']) {
```php
swoole_set_process_name("php {$argv[0]} task worker");
    } else {
        swoole_set_process_name("php {$argv[0]} event worker");
    }
});
```

  如果想使用[Reload](/server/methods?id=reload)机制实现代码重载入，必须在`onWorkerStart`中`require`你的业务文件，而不是在文件头部。在`onWorkerStart`调用之前已包含的文件，不会重新载入代码。

  可以将公用的、不易变的php文件放置到`onWorkerStart`之前。这样虽然不能重载入代码，但所有`Worker`是共享的，不需要额外的内存来保存这些数据。
`onWorkerStart`之后的代码每个进程都需要在内存中保存一份

  * `$worker_id`表示这个`Worker`进程的`ID`，范围参考[$worker_id](/server/properties?id=worker_id)
  * [$worker_id](/server/properties?id=worker_id)和进程`PID`没有任何关系，可使用`posix_getpid`函数获取`PID`

  * **协程支持**

    * 在`onWorkerStart`回调函数中会自动创建协程，所以`onWorkerStart`可以调用协程`API`

  * **注意**

    !> 发生致命错误或者代码中主动调用`exit`时，`Worker/Task`进程会退出，管理进程会重新创建新的进程。这可能导致死循环，不停地创建销毁进程
```
## onReceive

?> **This function is called when data is received, occurs in the `worker` process.**

```php
function onReceive(Swoole\Server $server, int $fd, int $reactorId, string $data);
```

  * **Parameters**

    * **`Swoole\Server $server`**
      * **Description**: Swoole\Server object
      * **Default value**: None
      * **Other values**: None

    * **`int $fd`**
      * **Description**: Connection file descriptor
      * **Default value**: None
      * **Other values**: None

    * **`int $reactorId`**
      * **Description**: `ID` of the [Reactor](/learn?id=reactor-thread) thread where the `TCP` connection is
      * **Default value**: None
      * **Other values**: None

    * **`string $data`**
      * **Description**: The received data, which can be text or binary content
      * **Default value**: None
      * **Other values**: None

  * **For integrity of packets under the `TCP` protocol, refer to [TCP Data Packet Boundary Issue](/learn?id=tcp-data-packet-boundary-issue)**

    * Using low-level configurations like `open_eof_check/open_length_check/open_http_protocol` can ensure the integrity of data packets
    * If not using low-level protocol handling, analyze the data in PHP code after [onReceive](/server/events?id=onreceive), merge/split data packets independently.

    For example: You can add a `$buffer = array()` in the code, use `$fd` as the `key` to store context data. Each time data is received, concatenate the strings, `$buffer[$fd] .= $data`, then check if `$buffer[$fd]` string forms a complete data packet.
デフォルトでは、同じ`fd`は同じ`Worker`に割り当てられるため、データを結合することができます。[dispatch_mode](/server/setting?id=dispatch_mode) = 3を使用すると、リクエストデータは競合しますので、同じ`fd`から送信されたデータは異なるプロセスに割り当てられる可能性があります。そのため、前述のデータパケット結合方法は使用できません。

  * **複数ポートのリスニングについて、[このセクション](/server/port)を参照してください**

    メインサーバーがプロトコルを設定した場合、追加でリッスンするポートはデフォルトでメインサーバーの設定を継承します。ポートのプロトコルを再設定するには、明示的に`set`メソッドを呼び出す必要があります。

    ```php
    $server = new Swoole\Http\Server("127.0.0.1", 9501);
    $port2 = $server->listen('127.0.0.1', 9502, SWOOLE_SOCK_TCP);
    $port2->on('receive', function (Swoole\Server $server, $fd, $reactor_id, $data) {
        echo "[#".$server->worker_id."]\tClient[$fd]: $data\n";
    });
    ```

    ここで`on`メソッドを呼んで[onReceive](/server/events?id=onreceive)コールバック関数を登録しましたが、メインサーバーのプロトコルを上書きする`set`メソッドを呼び出していないため、新しくリッスンしている`9502`ポートは依然として`HTTP`プロトコルを使用します。`telnet`クライアントで`9502`ポートに接続して文字列を送信すると、サーバーは[onReceive](/server/events?id=onreceive)をトリガーしません。

  * **注意**

    !> 自動プロトコルオプションがオフの場合、[onReceive](/server/events?id=onreceive)で一度に受信できるデータの最大は`64K`です。  
    自動プロトコル処理オプションがオンになっている場合、[onReceive](/server/events?id=onreceive)は、最大で[package_max_length](/server/setting?id=package_max_length)を超えない完全なデータパケットを受信します。
サポートされるバイナリ形式です。`$data`はバイナリデータかもしれません。
## onClose

?> **`TCP`クライアントが接続を閉じた後に、`Worker`プロセスでこの関数がコールバックされます。**

```php
function onClose(Swoole\Server $server, int $fd, int $reactorId);
```

  * **パラメータ** 

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $fd`**
      * **機能**：接続のファイル記述子
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $reactorId`**
      * **機能**：どの`reactor`スレッドからか、サーバーが`close`を実行した場合は負の数になります
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **ヒント**

    * **自動的に閉じる**

      * サーバーが接続を自動的に閉じる場合、バックエンドはこのパラメータを`-1`に設定し、`$reactorId < 0`で切断がサーバー側から発生したかクライアント側から発生したかを判断できます。
      * `PHP`コードで`close`メソッドを呼び出すことが唯一の自動的な閉じると見なされます。

    * **ハートビート検査**

      * [ハートビート検査](/server/setting?id=heartbeat_check_interval)はハートビート検査スレッドによって通知され、閉じるときに[onClose](/server/events?id=onclose)の`$reactorId`パラメータは`-1`ではありません

  * **注意**

    !> -[onClose](/server/events?id=onclose) コールバック関数で致命的なエラーが発生すると、接続がリークします。`netstat`コマンドを使用すると、大量の`CLOSE_WAIT`状態の`TCP`接続が表示されます，[参考Swooleビデオチュートリアル](https://course.swoole-cloud.com/course-video/4)
- どちらの場合でも、クライアント側が`close`を起動したか、サーバー側が`$server->close()`を呼び出して接続を閉じたかに関わらず、このイベントが発生します。したがって、接続が閉じられると、必ずこの関数がコールバックされます。
    - [onClose](/server/events?id=onclose)では、接続情報を取得するために[getClientInfo](/server/methods?id=getClientInfo)メソッドを引き続き呼び出すことができます。[onClose](/server/events?id=onclose)コールバック関数が完了した後に`close`が`TCP`接続を閉じます。
    - ここで[onClose](/server/events?id=onclose)をコールバックすると、クライアント接続が既に閉じられていることを示しているため、`$server->close($fd)`を実行する必要はありません。コード内で`$server->close($fd)`を実行すると、`PHP`エラー警告が発生します。
## onTask

?> **Called in the `task` process. The `worker` process can use the [task](/server/methods?id=task) function to deliver new tasks to the `task_worker` process. The current [Task process](/learn?id=taskworker-process) switches to busy state when calling the callback function [onTask](/server/events?id=ontask). At this point, it will no longer accept new tasks. When the [onTask](/server/events?id=ontask) function returns, the process switches back to idle state and resumes receiving new `Task`.**

```php
function onTask(Swoole\Server $server, int $task_id, int $src_worker_id, mixed $data);
```

  * **Parameters** 

    * **`Swoole\Server $server`**
      * **Function**: Swoole\Server object
      * **Default**: None
      * **Other values**: None

    * **`int $task_id`**
      * **Function**: ID of the `task` process executing the task. [`$task_id` combined with `$src_worker_id` is globally unique. Tasks from different `worker` processes might have the same `ID`.]
      * **Default**: None
      * **Other values**: None
      
    * **`int $src_worker_id`**
      * **Function**: ID of the `worker` process delivering the task. [`$task_id` combined with `$src_worker_id` is globally unique. Tasks from different `worker` processes might have the same `ID`.]
      * **Default**: None
      * **Other values**: None

    * **`mixed $data`**
      * **Function**: Data content of the task
      * **Default**: None
      * **Other values**: None

  * **Note**  
* **v4.2.12以降、[task_enable_coroutine](/server/setting?id=task_enable_coroutine)が有効になっている場合、コールバック関数のプロトタイプは次のようになります**

      ```php
      $server->on('Task', function (Swoole\Server $server, Swoole\Server\Task $task) {
          var_dump($task);
          $task->finish([123, 'hello']); // タスクを完了し、データを返す
      });
      ```

    * **`worker`プロセスに実行結果を返す**

      * **[onTask](/server/events?id=ontask)関数内で文字列を`return`すると、その内容が`worker`プロセスに返されます。`worker`プロセスでは[onFinish](/server/events?id=onfinish)関数がトリガーされ、投げられた`task`が完了したことを示します。もちろん、`Swoole\Server->finish()`を使用しても[onFinish](/server/events?id=onfinish)関数をトリガーし、`return`することなく完了させることができます**

      * `return`される変数は、`null`以外の任意の`PHP`変数であることができます

  * **注意**

    !> [onTask](/server/events?id=ontask)関数が致命的なエラーで終了するか、外部プロセスによって強制的に`kill`されると、現在のタスクは破棄されますが、待機中の他の`Task`には影響しません。
## onFinish

?> **This callback function is called in the worker process. When a task delivered by the `worker` process is completed in the `task` process, the [task process](/learn?id=taskworker-process) will send the result of the task processing to the `worker` process using the `Swoole\Server->finish()` method.**

```php
function onFinish(Swoole\Server $server, int $task_id, mixed $data)
```

  * **Parameters** 

    * **`Swoole\Server $server`**
      * **Description**: Swoole\Server object
      * **Default**: N/A
      * **Other values**: N/A

    * **`int $task_id`**
      * **Description**: The ID of the `task` process executing the task
      * **Default**: N/A
      * **Other values**: N/A

    * **`mixed $data`**
      * **Description**: The content of the task processing result
      * **Default**: N/A
      * **Other values**: N/A

  * **Note**

    !> - If `finish` method is not called or no result is `return` in the [onTask](/server/events?id=ontask) event of the [task process](/learn?id=taskworker-process), the [onFinish](/server/events?id=onfinish) event will not be triggered in the `worker` process.  
    - The `worker` process executing the [onFinish](/server/events?id=onfinish) logic is the same process that dispatched the `task` job.
## onPipeMessage

?> **When a work process receives a [unixSocket](/learn?id=什么是IPC) message sent by `$server->sendMessage()`, the `onPipeMessage` event will be triggered. `worker/task` processes may trigger the `onPipeMessage` event.**

```php
function onPipeMessage(Swoole\Server $server, int $src_worker_id, mixed $message);
```

  * **Parameters** 

    * **`Swoole\Server $server`**
      * **Description**: Swoole\Server object
      * **Default**: None
      * **Other values**: None

    * **`int $src_worker_id`**
      * **Description**: Which `Worker` process the message came from
      * **Default**: None
      * **Other values**: None

    * **`mixed $message`**
      * **Description**: Message content, can be any PHP type
      * **Default**: None
      * **Other values**: None

Translated by Translation API (Original text has been kept for reference).
## onStart

?> **This function is called back in the main thread of the master process after startup**

```php
function onStart(Swoole\Server $server);
```

  * **Parameters** 

    * **`Swoole\Server $server`**
      * **Function**: Swoole\Server object
      * **Default value**: None
      * **Other values**: None

* **Operations done before this event in the `Server`**

    * Complete creation of the [Manager process](/learn?id=manager-process) upon startup
    * Complete creation of [Worker subprocesses](/learn?id=worker-process)
    * Listen on all TCP/UDP/[unixSocket](/learn?id=ipc) ports but not yet start accepting connections and requests
    * Timers are set up

* **Next actions to be taken**

    * Main [Reactor](/learn?id=reactor-thread) starts to receive events, clients can `connect` to the `Server`

**In the `onStart` callback, only `echo`, print `Log`, and change process names are allowed. Other operations are not allowed (functions related to `server` cannot be called, since the service is not ready yet). The `onWorkerStart` and `onStart` callbacks are executed in parallel in different processes without a specific order.**

In the `onStart` callback, you can save the values of `$server->master_pid` and `$server->manager_pid` to a file. By doing this, you can write a script that sends a signal to these two `PIDs` to achieve shutdown and restart operations.

The `onStart` event is called in the main thread of the `Master` process.

!> Global resource objects created in `onStart` cannot be used in `Worker` processes because when `onStart` is called, `worker` processes are already created. The newly created object is in the main process and the `Worker` process cannot access this memory area.
したがって、グローバルオブジェクトを作成するコードは`Server::start`の前に配置する必要があります。典型的な例は[Swoole\Table](/memory/table?id=完全な例)

* **セキュリティ上の注意**

`onStart`コールバックで非同期およびコルーチンAPIを使用できますが、`dispatch_func`と`package_length_func`との競合が発生する可能性があることに注意してください。**同時に使用しないでください**。

`onStart`でタイマーを開始しないでください。コード中で`Swoole\Server::shutdown()`が実行されると、プログラムが終了できなくなる可能性があります。

`onStart`コールバックは、`return`するまで、サーバープログラムがクライアント接続を受け付けないため、同期ブロッキング関数を安全に使用できます。

* **BASE モード**

[SWOOLE_BASE](/learn?id=swoole_base)モードでは`master`プロセスが存在しないため、`onStart`イベントが存在しません。`BASE`モードで`onStart`コールバック関数を使用しないでください。

```
警告 swReactorProcess_start: The onStart event with SWOOLE_BASE is deprecated
```
## onWorkerStart

?> **このイベントは、Workerプロセス/[Taskプロセス](/learn?id=taskworkerプロセス)の起動時に発生し、この段階で作成されたオブジェクトはプロセスのライフサイクル内で使用できます。**

```php
function onWorkerStart(Swoole\Server $server, int $workerId);
```

  * **パラメータ** 

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $workerId`**
      * **機能**：`Worker`プロセスの `id`（プロセスのPIDではない）
      * **デフォルト値**：なし
      * **その他の値**：なし

  * `onWorkerStart/onStart`は並行して実行され、順番はありません
  * 現在のプロセスが`Worker`プロセスであるか[Tastプロセス](/learn?id=taskworkerプロセス)であるかを判断するには`$server->taskworker`プロパティを使用できます
  * `worker_num` と `task_worker_num` が`1`を超えるように設定されている場合、各プロセスで`onWorkerStart`イベントが1度ずつ発生し、異なる作業プロセスを区別するために[$worker_id](/server/properties?id=worker_id)を判断することができます
  * `worker`プロセスから`task`プロセスにタスクを送信し、`task`プロセスがすべてのタスクを処理した後に[onFinish](/server/events?id=onfinish)コールバック関数を介して`worker`プロセスに通知します。たとえば、バックグラウンドで数十万人に通知メールを送信する場合、作業が完了したときに操作ステータスが送信中と表示され、その時点で他の処理を続行し、メール送信が完了すると、操作ステータスが自動的に更新されます。

  次の例では、Workerプロセス/ [Taskプロセス](/learn?id=taskworkerプロセス)の名前を変更します。

```php
$server->on('WorkerStart', function ($server, $worker_id){
    global $argv;
    if($worker_id >= $server->setting['worker_num']) {```
```php
swoole_set_process_name("php {$argv[0]} task worker");
    } else {
        swoole_set_process_name("php {$argv[0]} event worker");
    }
});
```

  もし[Reload](/server/methods?id=reload)メカニズムを使ってコードの再ロードを実現したい場合は、`onWorkerStart`であなたのビジネスファイルを`require`する必要があります。ファイルの先頭ではなく、`onWorkerStart`呼び出し前に含まれたファイルについては、コードを再度ロードしません。

  共通で変更されにくいPHPファイルを`onWorkerStart`の前に配置することができます。これにより、コードを再度ロードすることはできませんが、すべての`Worker`が共有され、これらのデータを保存するための追加のメモリは必要ありません。
  `onWorkerStart`の後にあるコードは、それぞれのプロセスにメモリ上で保持されます。

  * `$worker_id`はこの`Worker`プロセスの`ID`を表し、範囲は[$worker_id](/server/properties?id=worker_id)を参照してください。
  * [$worker_id](/server/properties?id=worker_id)とプロセスの`PID`には何の関連性もなく、`PID`を取得するためには`posix_getpid`関数を使用できます。

  * **コルーチンのサポート**

    * `onWorkerStart`コールバック関数内で自動的にコルーチンが作成されるため、`onWorkerStart`でコルーチンの`API`を呼び出すことができます。

  * **注意**

    !> 致命的なエラーが発生するか、コード内で`exit`を呼び出すと、`Worker/Task`プロセスが終了し、管理プロセスが新しいプロセスを再作成します。これは無限ループを引き起こす可能性があり、プロセスの作成と破棄が繰り返されることになります。
## `onReceive`

?>**この関数はデータを受信したときに呼び出され、`worker`プロセスで発生します。**

```php
function onReceive(Swoole\Server $server, int $fd, int $reactorId, string $data);
```

  * **パラメータ** 

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $fd`**
      * **機能**：接続のファイルディスクリプタ
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $reactorId`**
      * **機能**：`TCP`接続がある[リアクタ](/learn?id=reactor线程)スレッドの`ID`
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $data`**
      * **機能**：受信したデータの内容、テキストまたはバイナリコンテンツの可能性があります
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **`TCP`プロトコルのパケット完全性については、[TCPパケットの境界問題](/learn?id=tcp数据包边界问题)を参照してください**

    * `open_eof_check/open_length_check/open_http_protocol`などの提供されているレイヤーの設定を使用すると、パケットの完全性が保証されます。
    * レイヤーのプロトコル処理を使用しない場合、[onReceive](/server/events?id=onreceive)後にPHPコード内でデータを分析して、パケットを結合/分割してください。

    例：コード内で`$buffer = array()`を追加し、`$fd`を`key`として使用してコンテキストデータを保存できます。 データを受信するたびに文字列を連結し、`$buffer[$fd] .= $data`、次に`$buffer[$fd]`文字列が完全なデータパケットであるかどうかを確認します。
デフォルトでは、同じ`fd`は同じ`Worker`に割り当てられますので、データを結合することができます。`dispatch_mode` = 3を使用すると、リクエストデータは競合します。同じ`fd`から送られてきたデータは異なるプロセスに分配される可能性があるため、上記のデータパケット結合方法を使用できません。

  * **複数のポートのリッスン、[this section](/server/port)を参照**

    メインサーバーがプロトコルを設定した場合、追加のリッスンポートはデフォルトでメインサーバーの設定を継承します。ポートのプロトコルを再設定するには、`set`メソッドを明示的に呼び出す必要があります。

    ```php
    $server = new Swoole\Http\Server("127.0.0.1", 9501);
    $port2 = $server->listen('127.0.0.1', 9502, SWOOLE_SOCK_TCP);
    $port2->on('receive', function (Swoole\Server $server, $fd, $reactor_id, $data) {
        echo "[#".$server->worker_id."]\tClient[$fd]: $data\n";
    });
    ```

    ここでは、`on`メソッドを呼び出して[onReceive](/server/events?id=onreceive)コールバック関数を登録しましたが、メインサーバーのプロトコルを上書きする`set`メソッドを呼び出さなかったため、新しいリッスンポートの`9502`は依然として`HTTP`プロトコルを使用しています。`telnet`クライアントで`9502`ポートに接続して文字列を送信すると、サーバーは[onReceive](/server/events?id=onreceive)をトリガーしません。

  * **注意**

    !> 自動プロトコルオプションがオフの場合、[onReceive](/server/events?id=onreceive)で一度に受信できるデータの最大サイズは`64K`です。  
    自動プロトコル処理オプションを有効にした場合、[onReceive](/server/events?id=onreceive)は完全なデータパケットを受信し、最大サイズは[package_max_length](/server/setting?id=package_max_length)を超えません。
サポートされているバイナリ形式では、`$data` はバイナリデータである可能性があります。
## onClose

?> **`TCP`クライアントの接続が閉じた後、`Worker`プロセスでこの関数がコールバックされます。**

```php
function onClose(Swoole\Server $server, int $fd, int $reactorId);
```

  * **パラメータ** 

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $fd`**
      * **機能**：接続のファイル記述子
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $reactorId`**
      * **機能**：どの`reactor`スレッドから来たか、サーバーが`close`を呼び出すときに負数になります
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **ヒント**

    * **アクティブなクローズ**

      * サーバーがアクティブに接続を閉じると、このパラメータが`-1`に設定されます。`$reactorId < 0`をチェックして、クローズがサーバーサイドから発信されたものかクライアントから発信されたものかを区別できます。
      * アクティブに`close`メソッドを呼び出した場合にのみ、サーバーサイドからのアクティブなクローズと見なされます。

    * **ハートビート検査**

      * [ハートビート検査](/server/setting?id=heartbeat_check_interval)はハートビート検査スレッドによって通知されたクローズで、[onClose](/server/events?id=onclose)の`$reactorId`パラメータが`-1`でないときにクローズされます。

  * **注意**

    !> -[onClose](/server/events?id=onclose) コールバック関数で致命的なエラーが発生すると、接続が解放されない可能性があります。`netstat`コマンドを使用して、大量の`CLOSE_WAIT`状態の `TCP` 接続を確認できます。[Swooleビデオチュートリアルを参照してください](https://course.swoole-cloud.com/course-video/4)
- 顧客端から`close`を発信するか、サーバー側で`$server->close()`を呼び出して接続を閉じると、このイベントが発生します。したがって、接続が閉じられると必ずこの関数がコールバックされます。
    - [onClose](/server/events?id=onclose)内で、[getClientInfo](/server/methods?id=getClientInfo)メソッドを呼び出して接続情報を取得することができます。[onClose](/server/events?id=onclose)コールバック関数が完了した後に`close`を呼び出して`TCP`接続を閉じます。
    - ここで[onClose](/server/events?id=onclose)をコールバックすると、クライアント接続がすでに閉じたことを示すため、`$server->close($fd)`を実行する必要はありません。コード内で`$server->close($fd)`を実行すると、`PHP`エラーが発生します。
## onTask

?> **Called inside the `task` process. The `worker` process can use the [task](/server/methods?id=task) function to deliver new tasks to the `task_worker` process. The current [Task process](/learn?id=taskworker-process) will switch to busy state when calling the [onTask](/server/events?id=ontask) callback function, and will not receive new tasks at this time. When the [onTask](/server/events?id=ontask) function returns, the process state will switch to idle and then continue to receive new tasks.**

```php
function onTask(Swoole\Server $server, int $task_id, int $src_worker_id, mixed $data);
```

  * **Parameters**

    * **`Swoole\Server $server`**
      * **Function**: Swoole\Server object
      * **Default Value**: N/A
      * **Other Values**: N/A

    * **`int $task_id`**
      * **Function**: ID of the `task` process executing the task【`$task_id` and `$src_worker_id` together form a globally unique identifier. The task IDs delivered by different `worker` processes may be the same】
      * **Default Value**: N/A
      * **Other Values**: N/A

    * **`int $src_worker_id`**
      * **Function**: ID of the `worker` process delivering the task【`$task_id` and `$src_worker_id` together form a globally unique identifier. The task IDs delivered by different `worker` processes may be the same】
      * **Default Value**: N/A
      * **Other Values**: N/A

    * **`mixed $data`**
      * **Function**: Data content of the task
      * **Default Value**: N/A
      * **Other Values**: N/A

  * **Note**
* **v4.2.12起如果开启了 [task_enable_coroutine](/server/setting?id=task_enable_coroutine) 则回调函数原型是**

      ```php
      $server->on('Task', function (Swoole\Server $server, Swoole\Server\Task $task) {
          var_dump($task);
          $task->finish([123, 'hello']); //完成任务，结束并返回数据
      });
      ```

    * **返回执行结果到`worker`进程**

      * **在[onTask](/server/events?id=ontask)函数中 `return` 字符串，表示将此内容返回给 `worker` 进程。`worker` 进程中会触发 [onFinish](/server/events?id=onfinish) 函数，表示投递的 `task` 已完成，当然你也可以通过 `Swoole\Server->finish()` 来触发 [onFinish](/server/events?id=onfinish) 函数，而无需再 `return`**

      * `return` 的变量可以是任意非 `null` 的 `PHP` 变量

  * **注意**

    !> [onTask](/server/events?id=ontask)函数执行时遇到致命错误退出，或者被外部进程强制`kill`，当前的任务会被丢弃，但不会影响其他正在排队的`Task`
## onWorkerError

?> **This function is called in the `Manager` process when a `Worker/Task` process encounters an exception.**

!> This function is mainly used for alerting and monitoring. If a Worker process exits abnormally, it is likely due to a fatal error or a process core dump. By logging or sending alert information, developers can be notified to take appropriate action.

```php
function onWorkerError(Swoole\Server $server, int $worker_id, int $worker_pid, int $exit_code, int $signal);
```

  * **Parameters** 

    * **`Swoole\Server $server`**
      * **Description**: Swoole\Server object
      * **Default Value**: None
      * **Other Values**: None

    * **`int $worker_id`**
      * **Description**: ID of the exception Worker process
      * **Default Value**: None
      * **Other Values**: None

    * **`int $worker_pid`**
      * **Description**: PID of the exception Worker process
      * **Default Value**: None
      * **Other Values**: None

    * **`int $exit_code`**
      * **Description**: Exit status code, range is `0～255`
      * **Default Value**: None
      * **Other Values**: None

    * **`int $signal`**
      * **Description**: Signal of the process exit
      * **Default Value**: None
      * **Other Values**: None

  * **Common Errors**

    * `signal = 11`: Indicates a `segment fault` in the `Worker` process, possibly triggering a lower-level `BUG`. Please collect `core dump` information and `valgrind` memory detection logs, [and report this issue to the Swoole development team](/other/issue)
* `exit_code = 255`：ワーカープロセスで致命的なエラーが発生したことを示しています。PHPのエラーログをチェックし、問題のあるPHPコードを特定して解決してください。
    * `signal = 9`：`Worker`がシステムによって強制的に`Kill`されたことを示しています。人為的な`kill -9`操作が行われたかどうかを確認し、`dmesg`情報に`OOM（メモリ不足）`があるかどうかをチェックしてください。
    * もし`OOM`がある場合、割り当てられたメモリが大きすぎる可能性があります。1. `Server`の`setting`構成を確認し、[socket_buffer_size](/server/setting?id=socket_buffer_size)などが過剰に割り当てられていないかを確認してください；2. 非常に大きな[Swoole\Table](/memory/table)メモリモジュールを作成していないかを確認してください。
## onManagerStart

?> **管理プロセスが起動すると、このイベントが発生します**

```php
function onManagerStart(Swoole\Server $server);
```

  * **ヒント**

    * このコールバック関数内で管理プロセスの名前を変更できます。
    * `4.2.12`以前のバージョンでは、`manager`プロセスではタイマーを追加したり、タスクを送信したり、コルーチンを使用したりできません。
    * `4.2.12`以降のバージョンでは、`manager`プロセスで信号に基づいた同期モードのタイマーを使用できます。
    * `manager`プロセスでは、[sendMessage](/server/methods?id=sendMessage)インターフェースを使用して、他のワーカープロセスにメッセージを送信できます。

    * **起動順序**

      * `Task`および`Worker`プロセスは既に作成されています
      * `Master`プロセスの状態は不明です。なぜなら`Manager`と`Master`は並列であり、`onManagerStart`コールバックが発生するとき、`Master`プロセスが準備完了しているかどうかがわからないからです

    * **BASE モード**

      * [SWOOLE_BASE](/learn?id=swoole_base)モードでは、`worker_num`、`max_request`、`task_worker_num`パラメータが設定されている場合、内部で`manager`プロセスが作成されてワーカープロセスを管理します。そのため、`onManagerStart`および`onManagerStop`イベントコールバックが発生します。
## onManagerStop

?> **Called when the manager process ends**

```php
function onManagerStop(Swoole\Server $server);
```

 * **Note**

  * When `onManagerStop` is triggered, it means the `Task` and `Worker` processes have finished running and have been collected by the `Manager` process.
## onBeforeReload

?> **Workerプロセスが`Reload`する前にこのイベントが発生し、Managerプロセスでコールバックされます**

```php
function onBeforeReload(Swoole\Server $server);
```

  * **パラメータ**

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし
## onAfterReload

?> **Workerプロセスが`Reload`した後にこのイベントが発生し、Managerプロセスでコールバックされます**

```php
function onAfterReload(Swoole\Server $server);
```

  * **パラメータ**

    * **`Swoole\Server $server`**
      * **機能**：Swoole\Serverオブジェクト
      * **デフォルト値**：なし
      * **その他の値**：なし
## イベントの実行順序

* すべてのイベントコールバックは `$server->start` 後に発生します
* サーバーが閉じられるとき、最後のイベントは `onShutdown` です
* サーバーが正常に起動した後、`onStart/onManagerStart/onWorkerStart` は異なるプロセスで並行して実行されます
* `onReceive/onConnect/onClose` は `Worker` プロセスで発生します
* `Worker/Task` プロセスが開始/終了すると、それぞれ `onWorkerStart/onWorkerStop` が呼び出されます
* [onTask](/server/events?id=ontask) イベントは [taskプロセス](/learn?id=taskworker进程) でのみ発生します
* [onFinish](/server/events?id=onfinish) イベントは `worker` プロセスでのみ発生します
* `onStart/onManagerStart/onWorkerStart` の `3` つのイベントの実行順序は不確定です
## オブジェクト指向スタイル

[event_object](/server/setting?id=event_object)を有効にすると、次のイベントコールバックがオブジェクトスタイルで使用されます。
#### [Swoole\Server\Event](/server/event_class)

* [onConnect](/server/events?id=onconnect)
* [onReceive](/server/events?id=onreceive)
* [onClose](/server/events?id=onclose)

```php
$server->on('Connect', function (Swoole\Server $serv, Swoole\Server\Event $object) {
    var_dump($object);
});

$server->on('Receive', function (Swoole\Server $serv, Swoole\Server\Event $object) {
    var_dump($object);
});

$server->on('Close', function (Swoole\Server $serv, Swoole\Server\Event $object) {
    var_dump($object);
});
```
#### [Swoole\Server\Packet](/server/packet_class)

* [onPacket](/server/events?id=onpacket)

```php
$server->on('Packet', function (Swoole\Server $serv, Swoole\Server\Packet $object) {
    var_dump($object);
});
```
#### [Swoole\Server\PipeMessage](/server/pipemessage_class)

* [onPipeMessage](/server/events?id=onpipemessage)

```php
$server->on('PipeMessage', function (Swoole\Server $serv, Swoole\Server\PipeMessage $msg) {
    var_dump($msg);
    $object = $msg->data;
    $serv->sendto($object->address, $object->port, $object->data, $object->server_socket);
});
```
#### [Swoole\Server\StatusInfo](/server/statusinfo_class)

* [onWorkerError](/server/events?id=onworkererror)

```php
$serv->on('WorkerError', function (Swoole\Server $serv, Swoole\Server\StatusInfo $info) {
    var_dump($info);
});
```
#### [Swoole\Server\Task](/server/task_class)

* [onTask](/server/events?id=ontask)

```php
$server->on('Task', function (Swoole\Server $serv, Swoole\Server\Task $task) {
    var_dump($task);
});
```
#### [Swoole\Server\TaskResult](/server/taskresult_class)

* [onFinish](/server/events?id=onfinish)

```php
$server->on('Finish', function (Swoole\Server $serv, Swoole\Server\TaskResult $result) {
    var_dump($result);
});
```

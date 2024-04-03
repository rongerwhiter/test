# 配置

`Swoole\Server->set()` 関数は、`Server` の実行時に使用する各種パラメータを設定するために使用されます。このセクションのすべてのサブページは、設定配列の要素です。

!> [v4.5.5](/version/log?id=v455) 以降、Swoole は設定された設定項目が正しいかどうかを検出し、Swoole が提供していない設定項目を設定した場合、警告が発生します。

```shell
PHP Warning:  unsupported option [foo] in @swoole-src/library/core/Server/Helper.php 
```
### debug_mode

?> Set the log mode to `debug` for debugging purposes, this only works if `--enable-debug` is enabled during compilation.

```php
$server->set([
  'debug_mode' => true
])
```
### trace_flags

?> トレースログのタグを設定し、一部のトレースログのみ出力します。`trace_flags`は複数のトレースアイテムを設定するために`|` 演算子を使用できます。`--enable-trace-log`がコンパイル時に有効になっている場合にのみ機能します。

これらのトレース項目をサポートしており、`SWOOLE_TRACE_ALL`を使用してすべての項目をトレースできます：

* `SWOOLE_TRACE_SERVER`
* `SWOOLE_TRACE_CLIENT`
* `SWOOLE_TRACE_BUFFER`
* `SWOOLE_TRACE_CONN`
* `SWOOLE_TRACE_EVENT`
* `SWOOLE_TRACE_WORKER`
* `SWOOLE_TRACE_REACTOR`
* `SWOOLE_TRACE_PHP`
* `SWOOLE_TRACE_HTTP2`
* `SWOOLE_TRACE_EOF_PROTOCOL`
* `SWOOLE_TRACE_LENGTH_PROTOCOL`
* `SWOOLE_TRACE_CLOSE`
* `SWOOLE_TRACE_HTTP_CLIENT`
* `SWOOLE_TRACE_COROUTINE`
* `SWOOLE_TRACE_REDIS_CLIENT`
* `SWOOLE_TRACE_MYSQL_CLIENT`
* `SWOOLE_TRACE_AIO`
* `SWOOLE_TRACE_ALL`
### log_file

?> **Specify the `Swoole` error log file**

?> Exceptional information that occurs during `Swoole` runtime will be recorded in this file, which by default will be printed to the screen.  
When running in daemon mode  `(daemonize => true)`, standard output will be redirected to the `log_file`. Content printed to the screen in PHP code like `echo/var_dump/print` will be written into the `log_file`.

  * **Tips**

    * The logs in `log_file` are only for recording runtime errors and do not need to be stored long term.

    * **Log Markers**

      ?> In the log information, there will be some markers added in front of the process ID, indicating the type of process that generated the log.

        * `#` Master process
        * `$` Manager process
        * `*` Worker process
        * `^` Task process

    * **Reopen Log File**

      ?> If the log file is moved with `mv` or deleted with `unlink` during the server program runtime, log information will not be written correctly. In this case, you can reopen the log file by sending a `SIGRTMIN` signal to the `Server`.

      * Only supports the `Linux` platform
      * Does not support [UserProcess](/server/methods?id=addProcess) process

  * **Attention**

    !> `log_file` does not automatically split files, so it is necessary to periodically clean up this file. By monitoring the output in `log_file`, you can gather various types of server exceptions and warnings.
### log_level

?> **`Server`のエラーログの出力レベルを設定します。範囲は`0-6`です。`log_level`で設定されたレベルより低いログ情報は出力されません。**【デフォルト値：`SWOOLE_LOG_INFO`】

対応する定数については[ログレベル](/consts?id=ログレベル)を参照してください。

  * **注意**

    !> `SWOOLE_LOG_DEBUG`と`SWOOLE_LOG_TRACE`は、[--enable-debug-log](/environment?id=debug参数)および[--enable-trace-log](/environment?id=debug参数)にコンパイルされたバージョンでのみ使用可能です。  
    `daemonize`デーモンプロセスが有効になっている場合、プログラム内のすべての出力内容が[log_file](/server/setting?id=log_file)に書き込まれ、この部分の内容は`log_level`で制御されません。
### log_date_format

?> **`Server`のログの日付形式を設定します**。フォーマットは [strftime](https://www.php.net/manual/zh/function.strftime.php) の`format`を参照してください。

```php
$server->set([
    'log_date_format' => '%Y-%m-%d %H:%M:%S',
]);
```
### log_date_with_microseconds

サーバのログの精度を設定します。マイクロ秒を含むかどうかです。【デフォルト値: `false`】
### ログの回転

?> **`Server`ログの回転を設定します**【デフォルト値：`SWOOLE_LOG_ROTATION_SINGLE`】

| 定数                           | 説明   | バージョン情報 |
| ------------------------------ | ------ | -------------- |
| SWOOLE_LOG_ROTATION_SINGLE     | 無効   | -              |
| SWOOLE_LOG_ROTATION_MONTHLY    | 月毎   | v4.5.8         |
| SWOOLE_LOG_ROTATION_DAILY      | 日毎   | v4.5.2         |
| SWOOLE_LOG_ROTATION_HOURLY     | 毎時   | v4.5.8         |
| SWOOLE_LOG_ROTATION_EVERY_MINUTE | 毎分 | v4.5.8        |
### display_errors

?> `Swoole`のエラーメッセージを有効/無効にします。

```php
$server->set([
  'display_errors' => true
])
```
### dns_server

?> Set the IP address for `dns` queries.
### socket_dns_timeout

?> The domain name resolution timeout, if the coroutine client is enabled on the server, this parameter can control the client's domain name resolution timeout, in seconds.
### socket_connect_timeout

?> 客户端接続のタイムアウト時間。サーバー側でコルーチンクライアントを有効にしている場合、このパラメータはクライアントの接続タイムアウト時間を制御します。単位は秒です。
### socket_write_timeout / socket_send_timeout

?> The client write timeout sets the client's write timeout in seconds when using coroutine clients on the server side. This configuration can also be used to control the execution timeout of `shell_exec` or [Swoole\Coroutine\System::exec()](/coroutine/system?id=exec) after `coroutine` is enabled.
### socket_read_timeout / socket_recv_timeout

?> 客户端読み取りタイムアウトは、サーバーでコルーチンクライアントを使用している場合に、クライアントの読み取りタイムアウトを制御するためのパラメータであり、単位は秒です。
### max_coroutine / max_coro_num: id = max_coroutine

?> **設定現在工作進程的最大協程數量。**【默認值：`100000`，當Swoole版本小於`v4.4.0-beta`時默認值為`3000`】

?> 超過`max_coroutine`底層將無法創建新的協程，服務端的Swoole會拋出`exceed max number of coroutine`錯誤，`TCP Server`會直接關閉連接，`Http Server`會返回Http的503狀態碼。

?> 在`Server`程序中實際最大可創建協程數量等於 `worker_num * max_coroutine`，task進程和UserProcess進程的協程數量單獨計算。

```php
$server->set(array(
    'max_coroutine' => 3000,
));
```
### enable_deadlock_check

?> Enable coroutine deadlock check.

```php
$server->set([
  'enable_deadlock_check' => true
]);
```  
### hook_flags

?> **Set the function scope for `one-click coroutine` Hook.**【Default value: no hook】

!> Swoole version is `v4.5+` or [4.4LTS](https://github.com/swoole/swoole-src/tree/v4.4.x) is available, see details [here](/runtime)

```php
$server->set([
    'hook_flags' => SWOOLE_HOOK_SLEEP,
]);
```
The underlying support for the following coroutine items, using `SWOOLE_HOOK_ALL` to coroutine all:

* `SWOOLE_HOOK_TCP`
* `SWOOLE_HOOK_UNIX`
* `SWOOLE_HOOK_UDP`
* `SWOOLE_HOOK_UDG`
* `SWOOLE_HOOK_SSL`
* `SWOOLE_HOOK_TLS`
* `SWOOLE_HOOK_SLEEP`
* `SWOOLE_HOOK_FILE`
* `SWOOLE_HOOK_STREAM_FUNCTION`
* `SWOOLE_HOOK_BLOCKING_FUNCTION`
* `SWOOLE_HOOK_PROC`
* `SWOOLE_HOOK_CURL`
* `SWOOLE_HOOK_NATIVE_CURL`
* `SWOOLE_HOOK_SOCKETS`
* `SWOOLE_HOOK_STDIO`
* `SWOOLE_HOOK_PDO_PGSQL`
* `SWOOLE_HOOK_PDO_ODBC`
* `SWOOLE_HOOK_PDO_ORACLE`
* `SWOOLE_HOOK_PDO_SQLITE`
* `SWOOLE_HOOK_ALL`
### enable_preemptive_scheduler

?> Set to enable preemptive scheduling for coroutines to prevent one coroutine from starving others due to its long execution time. The maximum execution time for each coroutine is set to `10ms`.

```php
$server->set([
  'enable_preemptive_scheduler' => true
]);
```
### c_stack_size / stack_size

?> Set the memory size of the initial C stack for a single coroutine, default is 2M.
### aio_core_worker_num

?> Set the minimum number of `AIO` worker threads, with the default value being the number of CPU cores.
### aio_worker_num 

?> The maximum number of working threads for `AIO` is set, with the default value being the number of `cpu` cores * 8.
### aio_max_wait_time

?> The maximum time that worker threads will wait for tasks, in seconds.
### aio_max_idle_time

?> 工作スレッドの最大アイドル時間、単位は秒です。
### reactor_num

?> **[リアクタ](/learn?id=reactor線程)スレッドの起動数を設定します。**【デフォルト値：`CPU`コア数】

?> このパラメータを使用して、メインプロセス内のイベント処理スレッドの数を調整し、マルチコアを最大限に活用します。デフォルトでは`CPU`コア数と同じ数のスレッドが起動されます。  
リアクタスレッドは複数のコアを利用することができます。例えば、マシンが`128`コアを持っていると、内部では`128`スレッドが起動されます。  
各スレッドは[イベントループ](/learn?id=何かeventloop)を維持できます。スレッド間はロックフリーで、命令は`128`コアの`CPU`で並列に実行されます。  
オペレーティングシステムのスケジューリングにはある程度の性能ロスがあるため、`CPU`コア数 * 2に設定することで、各コアの最大活用を図ることができます。

  * **ヒント**

    * `reactor_num`は`CPU`コア数の`1-4`倍で設定することをお勧めします
    * `reactor_num`は [swoole_cpu_num()](/functions?id=swoole_cpu_num) * 4を超えることはできません

  * **注意**

  !> - `reactor_num`は`worker_num`よりも小さいか等しくなければなりません；
- もし設定した`reactor_num`が`worker_num`を超過した場合、自動的に`reactor_num`を`worker_num`に調整します；
- `8`コアを超えるマシンでは、`reactor_num`はデフォルトで`8`に設定されます。
### worker_num

?> **設定起動する`Worker`プロセス数。**【デフォルト値：`CPU`のコア数】

?> もし1つのリクエストが100msかかる場合、`1000QPS`の処理能力が必要なら、`100`個以上のプロセスを設定する必要があります。  
しかし、起動するプロセスが多ければ多いほど、利用するメモリも大幅に増加し、プロセス間の切り替えの負荷も増加します。したがって、適切な数に設定してください。過剰に設定しないようにしてください。

  * **ヒント**

    * もしビジネスコードが完全に[非同期IO](/learn?id=同期io非同期io)である場合、ここに`CPU`コア数の`1-4`倍を最適とする
    * もしビジネスコードが[同期IO](/learn?id=同期io非同期io)である場合、リクエストの応答時間とシステムの負荷に応じて調整する必要があります。例：`100-500`
    * デフォルトは[swoole_cpu_num()](/functions?id=swoole_cpu_num)に設定され、最大値は[swoole_cpu_num()](/functions?id=swoole_cpu_num) * 1000を超えてはいけません
    * 各プロセスが`40M`のメモリを使用すると仮定した場合、`100`個のプロセスは`4G`のメモリを必要とします。プロセスのメモリ使用量を正しく確認する方法は、[Swoole公式ビデオチュートリアル](https://course.swoole-cloud.com/course-video/85)を参照してください。
### max_request

?> **`worker`プロセスの最大タスク数を設定します。**【デフォルト値：`0` プロセスは終了しません】

?> この数値を超えるタスクを処理した後、`worker`プロセスは自動的に終了し、プロセスが終了するとすべてのメモリとリソースが解放されます。

!> このパラメータの主な目的は、プログラムのコーディングミスによるPHPプロセスのメモリリークを解決することです。PHPアプリケーションには遅いメモリリークがありますが、具体的な原因を特定できず、解決できない場合、一時的な解決策として`max_request`を設定できます。メモリリークのコードを見つけて修正する必要があります。これはその対処方法ではなく、[Swoole Tracker](https://course.swoole-cloud.com/course-video/92) を使用してリークするコードを見つけるべきです。

  * **ヒント**

    * `max_request`に達してもプロセスが即座に終了するわけではありません。[max_wait_time](/server/setting?id=max_wait_time)をご参照ください。
    * [SWOOLE_BASE](/learn?id=swoole_base)で`max_request`に到達し、プロセスを再起動すると、クライアント接続が切断されます。

  !> `worker`プロセス内で致命的なエラーが発生するか、手動で`exit`を実行すると、プロセスが自動的に終了します。`master`プロセスは新しい`worker`プロセスを起動してリクエストの処理を継続します。
### max_conn / max_connection

?> **サーバープログラムにおいて、許容される最大の接続数です。**【デフォルト値：`ulimit -n`】

?> 例えば`max_connection => 10000`, このパラメータは`Server`が保持できる最大の`TCP`接続数を設定します。この数を超えると、新しい接続は拒否されます。

  * **ヒント**

    * **デフォルト設定**

      * アプリケーションレベルで`max_connection`が設定されていない場合、下位レベルでは`ulimit -n`の値がデフォルト設定として使用されます。
      * バージョン`4.2.9`以降では、下位レベルが`ulimit -n`を`100000`を超える場合、デフォルト値を`100000`に設定します。これは一部のシステムで`ulimit -n`が`100万`に設定されており、大量のメモリを割り当てる必要があるため、起動に失敗するためです。

    * **最大上限**

      * `max_connection`を`1M`を超えるように設定しないでください。

    * **最小設定**
    
      * このオプションを小さく設定すると、下位レベルでエラーが発生し、`ulimit -n`の値に設定されます。
      * 最小値は`(worker_num + task_worker_num) * 2 + 32`です。

    ```shell
    serv->max_connection is too small.
    ```

    * **メモリ使用**

      * `max_connection`パラメータは大きすぎないように調整してください。マシンのメモリ状況に応じて設定してください。`Swoole`はこの数値に基づいて一度に大きなメモリを割り当てて`Connection`情報を保存し、1つの`TCP`接続の`Connection`情報には`224`バイトが必要です。

  * **注意**

  !> `max_connection`は操作システムの`ulimit -n`の値を超えてはなりません。そうでない場合、警告メッセージが表示され、`ulimit -n`の値にリセットされます。

  ```shell
  WARN swServer_start_check: serv->max_conn is exceed the maximum value[100000].

  WARNING set_max_connection: max_connection is exceed the maximum value, it's reset to 10240
  ```
### task_worker_num

?> **Configure the number of task worker processes.**

?> Enabling this parameter will activate the `task` feature. So the `Server` must register the two event callback functions [onTask](/server/events?id=ontask) and [onFinish](/server/events?id=onfinish). If not registered, the server program will not start.

  * **Tips**

    * [Task worker processes](/learn?id=taskworker进程) are synchronous and blocking.

    * The maximum value must not exceed [swoole_cpu_num()](/functions?id=swoole_cpu_num) * 1000.

    * **Calculation Method**
      * Suppose the processing time for a single `task` is `100ms`, then one process can handle `1/0.1=10` tasks per second.
      * If `2000` tasks are generated per second,
      * `2000/10=200`, then `task_worker_num` should be set to `200` to enable `200` task worker processes.

  * **Note**

    !> - The `Swoole\Server->task` method cannot be used inside [task worker processes](/learn?id=taskworker进程).
### task_ipc_mode

?> **[Task worker process](/learn?id=taskworker-process) と `Worker` プロセス間の通信方式を設定します。**【デフォルト値：`1`】

?> まずは[SwooleにおけるIPC通信](/learn?id=What-is-IPC)をお読みください。

モード | 役割
---|---
1 | `Unix ソケット` を使用して通信【デフォルトモード】
2 | `sysvmsg` メッセージキューを使用して通信
3 | `sysvmsg` メッセージキューを使用して通信し、競合モードを設定

  * **ヒント**

    * **モード`1`**
      * モード`1` を使用すると、タスクを特定のターゲット`Task worker process` に送信するために、[task](/server/methods?id=task) や [taskwait](/server/methods?id=taskwait) メソッドで `dst_worker_id` を使用できます。
      * `dst_worker_id` を `-1` に設定すると、バックエンドは各 [Task worker process](/learn?id=taskworker-process) の状態を確認し、空いているプロセスにタスクを送信します。

    * **モード`2`、`3`**
      * メッセージキュー モードは、オペレーティングシステムが提供するメモリキューを使用してデータを格納します。`mssage_queue_key` を指定しない場合、プライベートキューが使用され、`Server` プログラムが終了した後、メッセージキューは削除されます。
      * メッセージキュー`Key` を指定した場合、`Server` プログラムが終了した後も、メッセージキュー内のデータは削除されません。したがって、プロセスを再起動後もデータにアクセスできます。
      * `ipcrm -q` を使用してメッセージキュー`ID` を手動で削除できます。
      * `モード2` と `モード3` の違いは、`モード2` が定向投递をサポートしていることです。 `$serv->task($data, $task_worker_id)` でどの [taskプロセス](/learn?id=taskworker-process) に送信するかを指定できます。 `モード3` は完全な競合モードであり、[taskプロセス](/learn?id=taskworker-process) がキューを奪い合い、定向投递を使用できず、`task/taskwait` ではプロセスIDを指定できません。したがって、`$task_worker_id` を指定しても、`モード3` では無効です。

  * **注意**

    !> - `モード3` は、[sendMessage](/server/methods?id=sendMessage) メソッドに影響し、[sendMessage](/server/methods?id=sendMessage) で送信されたメッセージはランダムに1つの [taskプロセス](/learn?id=taskworker-process) に受信される可能性があります。
    - メッセージキュー通信を使用すると、`Taskプロセス` の処理能力が投送速度を下回る場合、`Worker` プロセスがブロックされる可能性があります。
    - メッセージキュー通信を使用すると、taskプロセスではコルーチンをサポートできません（[task_enable_coroutine](/server/setting?id=task_enable_coroutine) を有効にする）。
### task_max_request

?> **Set the maximum number of tasks for [task processes](/learn?id=taskworker-process).** 【Default value: `0`】

taskプロセスの最大タスク数を設定します。この数値を超えるタスクを処理した後、taskプロセスは自動的に終了します。このパラメータは、PHPプロセスのメモリオーバーフローを防ぐために存在します。プロセスが自動的に終了しないようにしたい場合は、0に設定してください。
### task_tmpdir

?> **Set the temporary directory for task data.** [Default: Linux `/tmp` directory]

?> In `Server`, if the submitted data exceeds `8180` bytes, a temporary file will be used to save the data. The `task_tmpdir` here is used to set the location for saving temporary files.

  * **Tip**

    * By default, the underlying system will use the `/tmp` directory to store `task` data. If your Linux kernel version is too low and `/tmp` directory is not a memory file system, you can set it to `/dev/shm/`.
    * If the `task_tmpdir` directory does not exist, the underlying system will attempt to create it automatically.

  * **Caution**

    !> -If creation fails, `Server->start` will also fail.
### task_enable_coroutine

?> **Enable `Task` coroutine support.** 【Default value: `false`】, supported since v4.2.12

?> When enabled, coroutines and [coroutine scheduler](/coroutine/scheduler) will be automatically created in the [onTask](/server/events?id=ontask) callback, and PHP code can directly use coroutine `API`.

  * **Example**

```php
$server->on('Task', function ($serv, Swoole\Server\Task $task) {
    // Which Worker process the task comes from
    $task->worker_id;
    // Task ID
    $task->id;
    // Task type, taskwait, task, taskCo, taskWaitMulti may use different flags
    $task->flags;
    // Task data
    $task->data;
    // Delivery time, added in v4.6.0
    $task->dispatch_time;
    // Coroutine API
    co::sleep(0.2);
    // Complete the task, finish and return data
    $task->finish([123, 'hello']);
});
```

  * **Note**

    !> - `task_enable_coroutine` must be used when [enable_coroutine](/server/setting?id=enable_coroutine) is `true`.
    - When `task_enable_coroutine` is enabled, `Task` worker processes support coroutine.
    - If `task_enable_coroutine` is not enabled, only synchronous blocking is supported.
### task_use_object/task_object :id=task_use_object

?> **オブジェクト指向スタイルでのTaskコールバックフォーマット。**【デフォルト：`false`】

?> `true`に設定すると、[onTask](/server/events?id=ontask)コールバックがオブジェクトモードになります。

* **例**

```php
<?php

$server = new Swoole\Server('127.0.0.1', 9501);
$server->set([
    'worker_num'      => 1,
    'task_worker_num' => 3,
    'task_use_object' => true,
//    'task_object' => true, // v4.6.0 version added alias
]);
$server->on('receive', function (Swoole\Server $server, $fd, $tid, $data) {
    $server->task(['fd' => $fd,]);
});
$server->on('Task', function (Swoole\Server $server, Swoole\Server\Task $task) {
    // Here $task is the Swoole\Server\Task object
    $server->send($task->data['fd'], json_encode($server->stats()));
});
$server->start();
```
### dispatch_mode

?> **データ包分配戦略。**【デフォルト値：`2`】

モード値 | モード | 役割
---|---|---
1 | ラウンドロビンモード | 受信したデータはすべての`Worker`プロセスにラウンドロビンで割り当てられます
2 | 固定モード | 接続のファイルディスクリプタに基づいて`Worker`を割り当てます。これにより、同じ接続からのデータは常に同じ`Worker`で処理されます
3 | プリエンプティブモード | マスタープロセスは、空き状態の`Worker`にのみタスクを配信します
4 | IP割り当て | クライアントの`IP`に基づいて`hash`を取得し、特定の`Worker`プロセスに割り当てます。<br>同じソースIPからの接続データは常に同じ`Worker`プロセスに割り当てられます。アルゴリズムは`inet_addr_mod(ClientIP, worker_num)`です
5 | UID割り当て | ユーザーコードで[Server->bind()](/server/methods?id=bind)を呼び出して接続に1つの`uid`をバインドする必要があります。その後、バックエンドは`UID`の値に基づいて異なる`Worker`プロセスに割り当てます。<br>アルゴリズムは`UID % worker_num`です。文字列を`UID`として使用する場合は、`crc32(UID_STRING)`を使用できます
7 | ストリームモード | 空き状態の`Worker`は接続を`accept`し、[Reactor](/learn?id=reactor糸)の新しいリクエストを受け入れます

  * **ヒント**

    * **推奨事項**
    
      * 状態を持たない`Server`は`1`または`3`を使用できます。同期ブロッキング`Server`は`3`、非同期非ブロッキング`Server`は`1`を使用できます
      * 状態を持つ場合は`2`、`4`、`5`を使用します
      
    * **UDPプロトコル**

      * `dispatch_mode=2/4/5`の場合、固定割り当てで、バックエンドはクライアントの`IP`を使用して異なる`Worker`プロセスに`hash`します
      * `dispatch_mode=1/3`の場合、異なる`Worker`プロセスにランダムに分配します
      * `inet_addr_mod`関数

```
    function inet_addr_mod($ip, $worker_num) {
        $ip_parts = explode('.', $ip);
        if (count($ip_parts) != 4) {
            return false;
        }
        $ip_parts = array_reverse($ip_parts);
    
        $ip_long = 0;
        foreach ($ip_parts as $part) {
            $ip_long <<= 8;
            $ip_long |= (int) $part;
        }
    
        return $ip_long % $worker_num;
    }
```

    * **BASEモード**

      * [SWOOLE_BASE](/learn?id=swoole_base) モードで`dispatch_mode`を構成することは無効です。なぜなら`BASE`にはタスクの投递がないからです。クライアントからのデータを受信すると、現在のスレッド/プロセスで`onReceive`をコールバックしてすぐに処理されるため、`Worker`プロセスに投递する必要はありません。

  * **注意**

    !> -`dispatch_mode=1/3`の場合、`onConnect/onClose`イベントがブロックされ、これらのモードでは`onConnect/onClose/onReceive`の順序が保証されないためです；  
    -非リクエスト応答型のサーバープログラムでは、モード`1`または`3`を使用しないでください。例：httpサービスは応答型なので、`1`または`3`を使用できますが、TCP長接続状態では`1`または`3`を使用できません。
### dispatch_func

?> Set the `dispatch` function. Swoole internally provides `6` types of [dispatch_mode](/server/setting?id=dispatch_mode). If these modes do not meet your requirements, you can write `C++` or `PHP` functions to implement the `dispatch` logic.

  * **Usage**

```php
$server->set(array(
  'dispatch_func' => 'my_dispatch_function',
));
```

  * **Tips**

    * After setting `dispatch_func`, the underlying layer will automatically ignore the `dispatch_mode` configuration.
    * If the function corresponding to `dispatch_func` does not exist, the underlying layer will throw a fatal error.
    * If you need to `dispatch` a package larger than 8K, `dispatch_func` can only get the content of `0-8180` bytes.

  * **Write PHP function**

    ?> Since `ZendVM` does not support a multi-threaded environment, even if multiple [Reactor](/learn?id=reactor_threads) threads are set, only one `dispatch_func` can be executed at a time. Therefore, the underlying layer will perform locking operations when executing this PHP function, which may lead to lock contention issues. Do not perform any blocking operations in `dispatch_func`, as this may cause the `Reactor` thread group to stop working.

    ```php
    $server->set(array(
        'dispatch_func' => function ($server, $fd, $type, $data) {
            var_dump($fd, $type, $data);
            return intval($data[0]);
        },
    ));
    ```

    * `$fd` is the unique identifier for the client connection and can be used to get connection information with `Server::getClientInfo`.
    * `$type` is the type of data: `0` indicates data sent from the client, `4` indicates a client connection establishment, and `3` indicates a client connection closure.
    * `$data` is the data content. Note: If parameters for processing protocols such as `HTTP`, `EOF`, `Length`, etc., are enabled, the underlying layer will concatenate packets. However, only the first 8K content of the package is passed into the `dispatch_func` function, and the complete package content cannot be obtained.
    * **Must** return a number from `0` to `(server->worker_num - 1)`, indicating the target working process `ID` for the data packet delivery.
    * Negative numbers or numbers greater than or equal to `server->worker_num` are exceptional target `ID` values, and the dispatched data will be discarded.

  * **Write C++ function**

    **In other PHP extensions, use swoole_add_function to register a length function with the Swoole engine.**

    ?> The underlying layer does not lock when calling the C++ function, so the caller must ensure thread safety.

    ```c++
    int dispatch_function(swServer *serv, swConnection *conn, swEventData *data);

    int dispatch_function(swServer *serv, swConnection *conn, swEventData *data)
    {
        printf("cpp, type=%d, size=%d\n", data->info.type, data->info.len);
        return data->info.len % serv->worker_num;
    }

    int register_dispatch_function(swModule *module)
    {
        swoole_add_function("my_dispatch_function", (void *) dispatch_function);
    }
    ```

    * The `dispatch` function must return the target `worker` process `ID` for delivery.
    * The returned `worker_id` must not exceed `server->worker_num`, otherwise the underlying layer will throw a segmentation fault.
    * Returning a negative number `(return -1)` indicates that this data packet is discarded.
    * `data` can be used to read the event type and length.
    * `conn` contains connection information. If it is a `UDP` packet, `conn` is `NULL`.

  * **Note**

    !> - `dispatch_func` is only valid in [SWOOLE_PROCESS](/learn?id=swoole_process) mode, and it is effective for servers of type [UDP/TCP/UnixSocket](/server/methods?id=__construct).
    - The returned `worker_id` must not exceed `server->worker_num`, otherwise the underlying layer will throw a segmentation fault.
### message_queue_key

?> **`KEY` of setting the message queue.** 【Default value: `ftok($php_script_file, 1)`】

?> Only used when [task_ipc_mode](/server/setting?id=task_ipc_mode) = 2/3. The set `Key` serves only as the `KEY` of the `Task` task queue, refer to [IPC communication under Swoole](/learn?id=什么是IPC).

?> The `task` queue will not be destroyed after the `server` ends. When the program is restarted, [task processes](/learn?id=taskworker进程) will continue to process tasks in the queue. If you do not want the program to execute old `Task` tasks after restarting, you can manually delete this message queue.

```shell
ipcs -q 
ipcrm -Q [msgkey]
``` 
### daemonize

?> **デーモン化**【デフォルト値：`false`】

?> `daemonize => true`に設定すると、プログラムはデーモンとしてバックグラウンドで実行されます。長時間実行されるサーバーサイドプログラムはこのオプションを有効にする必要があります。  
デーモン化を有効にしないと、SSHセッションを閉じた後にプログラムが停止します。

  * **ヒント**

    * デーモン化後、標準入出力は `log_file` にリダイレクトされます。
    * `log_file` が設定されていない場合、 `/dev/null` にリダイレクトされ、すべての画面表示情報が破棄されます。
    * デーモン化後、`CWD`（カレントワーキングディレクトリ）環境変数の値が変更され、相対パスによるファイルの読み書きが失敗します。`PHP`プログラムでは絶対パスを使用する必要があります。

    * **systemd**

      * `systemd`または`supervisord`を使用して`Swoole`サービスを管理する場合、`daemonize => true`を設定しないでください。主な理由は、`systemd`のメカニズムが`init`とは異なるためです。`init`プロセスの`PID`は `1` であり、プログラムは`daemonize`を使用すると、端末から分離され、最終的には`init`プロセスによって保護され、`init`との関係が親子プロセスの関係に変わります。
      * ただし、`systemd`は別のバックグラウンドプロセスを起動し、自身で他のサービスプロセスを`fork`して管理するため、`daemonize`は必要ありません。むしろ、`daemonize => true`を使用すると、`Swoole`プログラムがその管理プロセスとの親子関係を失います。
### バックログ

?> **`Listen`キューの長さを設定**

?> 例：`backlog => 128`、このパラメータは同時に`accept`待ちの接続が最大でいくつあるかを決定します。

  * **`TCP`の`backlog`について**

    ?> `TCP`には3-way handshakeのプロセスがあり、クライアント `syn=>サーバ` `syn+ack=>クライアント` `ack`、サーバがクライアントの`ack`を受け取った後に接続を`accept queue`というキューに入れます（※1）、
    キューのサイズは`backlog`パラメータと設定された`somaxconn`の最小値によって決まり、最終的な`accept queue`のサイズは`ss -lt`コマンドで確認できます。`Swoole`のメインプロセスは`accept`（※2）を呼び出し、
    `accept queue`から接続を取り出します。 `accept queue`がいっぱいになると接続が成功（※4）するか失敗する可能性があり、失敗した場合、クライアントは接続がリセットされる（※3）か、タイムアウトし、サーバは失敗した記録を残します。
    これらの現象が発生した場合は、この値を増やす必要があります。 幸運なことに、`Swoole`のSWOOLE_PROCESSモードは`PHP-FPM/Apache`などのソフトウェアとは異なり、接続のキューイング問題を解決するために`backlog`を依存しません。
    したがって、通常、上記の現象に遭遇することはありません。

    * ※1：`Linux2.2`以降、ハンドシェイクプロセスは`syn queue`と`accept queue`の2つのキューに分かれており、`syn queue`の長さは`tcp_max_syn_backlog`によって決まります。
    * ※2：より新しいカーネルでは`accept4`が呼び出され、1回の`set no block`システムコールを節約するためです。
    * ※3：クライアントは`syn+ack`パケットを受け取ると接続が成功したと考えますが、実際にはサーバは半接続状態にあり、クライアントに`rst`パケットを送信する可能性があります。クライアントは`Connection reset by peer`として表示されます。
    * ※4：成功はTCPの再送機構によって行われ、関連する設定には`tcp_synack_retries`と`tcp_abort_on_overflow`があります。低レベルのTCPメカニズムを詳しく学びたい場合は、[Swoole公式ビデオチュートリアル](https://course.swoole-cloud.com/course-video/3)を参照してください。
### open_tcp_keepalive

`TCP`にはデッドコネクションを検出する`Keep-Alive`メカニズムがあります。アプリケーションがデッドコネクションのサイクルに敏感ではない場合やハートビートメカニズムを実装していない場合は、オペレーティングシステムが提供する`keepalive`メカニズムを使用してデッドコネクションを切断できます。
[Server->set()](/server/methods?id=set)構成に`open_tcp_keepalive => true`を追加すると、`TCP keepalive`が有効になります。
また、`keepalive`の詳細を調整するために`3`つのオプションがあります。[Swoole公式ビデオチュートリアル](https://course.swoole-cloud.com/course-video/10)を参照してください。

  * **オプション**

     * **tcp_keepidle**

        秒単位で、`n`秒間データリクエストがないと、この接続を探索し始めます。

     * **tcp_keepcount**

        探索回数。回数を超えるとこの接続を`close`します。

     * **tcp_keepinterval**

        探索間隔、秒単位。

  * **例**

```php
$serv = new Swoole\Server("192.168.2.194", 6666, SWOOLE_PROCESS);
$serv->set(array(
    'worker_num' => 1,
    'open_tcp_keepalive' => true,
    'tcp_keepidle' => 4, //データ転送がないと4秒後に検査
    'tcp_keepinterval' => 1, //1秒ごとに検査
    'tcp_keepcount' => 5, //検査回数、5回を超えてもパケットが返ってこない場合に接続を閉じる
));

$serv->on('connect', function ($serv, $fd) {
    var_dump("Client:Connect $fd");
});

$serv->on('receive', function ($serv, $fd, $reactor_id, $data) {
    var_dump($data);
});

$serv->on('close', function ($serv, $fd) {
  var_dump("close fd $fd");
});

$serv->start();
```
### heartbeat_check_interval

?> **心拍検査を有効にする**【デフォルト値：`false`】

?> このオプションは何秒ごとにハートビートを検査するかを示します。単位は秒です。 たとえば、`heartbeat_check_interval => 60`のように設定すると、すべての接続を`60`秒ごとに巡回し、その接続がサーバーに対して`120`秒以内（`heartbeat_idle_time`が設定されていない場合はデフォルトで`interval`の2倍）にデータを送信していない場合、その接続は強制的に閉じられます。 設定されていない場合、ハートビートは有効になりません。 この構成はデフォルトで閉じています。[Swoole公式ビデオ講座](https://course.swoole-cloud.com/course-video/10)を参照してください。

  * **ヒント**
    * `Server`はクライアントに自らハートビートパケットを送信しません。代わりに、クライアントからのハートビートを待機します。 サーバー側の`heartbeat_check`は、接続に最後にデータが送信された時間をただ検査するだけであり、制限を超えると接続が切断されます。
    * ハートビート検査で切断された接続は、依然として[onClose](/server/events?id=onclose)イベントコールバックをトリガーします。

  * **注意**

    !> `heartbeat_check`は`TCP`接続のみをサポートしています。
### heartbeat_idle_time

?> **接続が許容される最大のアイドル時間**

?> `heartbeat_check_interval`と一緒に使用する必要があります

```php
array(
    'heartbeat_idle_time'      => 600, // If a connection does not send any data to the server within 600 seconds, the connection will be forcibly closed
    'heartbeat_check_interval' => 60,  // Traverses every 60 seconds
);
```

  * **ヒント**

    * `heartbeat_idle_time`を有効にすると、サーバーはクライアントに対してデータパケットを自動的に送信しません
    * `heartbeat_check_interval`を設定せずに`heartbeat_idle_time`だけを設定すると、下層ではハートビート検出スレッドが作成されません。`PHP`コード内で`heartbeat`メソッドを呼び出して接続のタイムアウトを手動で処理できます
### open_eof_check

?> **`EOF`検査を有効化** 【デフォルト値：`false`】、[TCPデータ包境界の問題](/learn?id=tcp数据包边界问题)を参照してください。

?> このオプションは、クライアント接続から送信されたデータを検査し、データパケットの末尾が指定された文字列である場合のみ`Worker`プロセスに渡します。それ以外の場合、データパケットを常に連結し続け、バッファ領域を超えるかタイムアウトするまで中止しません。エラーが発生した場合、下層は悪意のある接続と見なし、データを破棄して接続を強制的に閉じます。  
一般的な`Memcache/SMTP/POP`などのプロトコルは `\r\n` で終了するため、この設定を使用できます。有効にすると、`Worker`プロセスは常に1つまたは複数の完全なデータパケットを受信します。

```php
array(
    'open_eof_check' => true,   //`EOF`検査を有効化
    'package_eof'    => "\r\n", //`EOF`を設定
)
```

  * **注意**

    !> この設定は`STREAM`（ストリーム型）の`Socket`にのみ有効です。[TCP、Unix Socket Stream](/server/methods?id=__construct)のようなものです。  
    `EOF`検査ではデータの途中から`eof`文字列を探しませんので、`Worker`プロセスは複数のデータパケットを同時に受け取る可能性があります。アプリケーションレベルのコードで `explode("\r\n", $data)` を使用してデータパケットを分割する必要があります。
### open_eof_split

?> **Enable `EOF` automatic splitting**

?> When `open_eof_check` is set, multiple data may be combined into one packet, and the `open_eof_split` parameter can solve this problem. Refer to [TCP Data Packet Boundary Problem](/learn?id=tcp-data-packet-boundary-problem).

?> Setting this parameter requires traversing the entire content of the data packet to find `EOF`, thus consuming a large amount of `CPU` resources. Assuming each data packet is `2M`, with `10000` requests per second, this could result in `20G` CPU-character matching instructions.

```php
array(
    'open_eof_split' => true,   // Enable EOF_SPLIT check
    'package_eof'    => "\r\n", // Set EOF
)
```

  * **Tip**

    * With the `open_eof_split` parameter enabled, the underlying layer will search for `EOF` in the middle of the data packet and split the packet. [onReceive](/server/events?id=onreceive) will only receive one data packet ending with the `EOF` string each time.
    * After enabling the `open_eof_split` parameter, it will take effect regardless of whether the `open_eof_check` parameter is set.

    * **Difference from `open_eof_check`**
    
        * `open_eof_check` only checks if the end of the received data is `EOF`, so its performance is the best with almost no consumption.
        * `open_eof_check` cannot solve the problem of multiple data packets being merged, for example, if two data packets with `EOF` are sent simultaneously, the underlying layer may return all at once.
        * `open_eof_split` compares the data byte by byte from left to right to find `EOF` in the data for parceling, resulting in poor performance. However, only one data packet is returned each time.
### package_eof

?> **Set the `EOF` string.** Reference [TCP packet boundary problem](/learn?id=tcp数据包边界问题)

?> Need to be used in conjunction with `open_eof_check` or `open_eof_split`.

  * **Attention**

    !> `package_eof` only allows a maximum of `8` bytes string.  
### open_length_check

?> **Open Length Check Feature**【Default value: `false`】, refer to [TCP Data Packet Boundary Issue](/learn?id=tcp数据包边界问题)

?> Packet length detection provides parsing for protocols with a fixed format of package header + body. When enabled, it ensures that the `Worker` process receives a complete data packet every time in the [onReceive](/server/events?id=onreceive) event.
Length detection protocol requires only one length calculation, and data processing involves only pointer offsets, providing high performance. **Recommended for use**.

  * **Tips**

    * **The length protocol provides 3 options to control protocol details.**

      ?> This configuration is only valid for `STREAM` type of `Socket`, such as [TCP, Unix Socket Stream](/server/methods?id=__construct)

      * **package_length_type**

        ?> A field in the package header as the value of the package length, supporting 10 different length types at the lower level. Please refer to [package_length_type](/server/setting?id=package_length_type)

      * **package_body_offset**

        ?> Starting from which byte to calculate the length, generally in 2 cases:

        * The value of `length` includes the entire packet (header + body), where `package_body_offset` is `0`
        * The header length is `N` bytes, `length` value does not include the header, only the body, `package_body_offset` is set to `N`

      * **package_length_offset**

        ?> The position of the `length` value in the package header.

        * Example:

        ```c
        struct
        {
            uint32_t type;
            uint32_t uid;
            uint32_t length;
            uint32_t serid;
            char body[0];
        }
        ```
        
    ?> In the design of the communication protocol above, the header length is `4` integers, `16` bytes, the `length` value is at the position of the 3rd integer. Therefore, `package_length_offset` is set to `8`, bytes `0-3` are for `type`, bytes `4-7` are for `uid`, bytes `8-11` are for `length`, bytes `12-15` are for `serid`.

    ```php
    $server->set(array(
      'open_length_check'     => true,
      'package_max_length'    => 81920,
      'package_length_type'   => 'N',
      'package_length_offset' => 8,
      'package_body_offset'   => 16,
    ));
    ```
### package_length_type

?> **長さ値のタイプ**、1文字のパラメータを受け入れ、`PHP`の[pack](http://php.net/manual/zh/function.pack.php) 関数と同じです。

現在、`Swoole`は`10`種類のタイプをサポートしています：

文字パラメータ | 役割
---|---
c | 1バイトの符号付き
C | 1バイトの符号なし
s | 符号付き、ホストバイトオーダー、2バイト
S | 符号なし、ホストバイトオーダー、2バイト
n | 符号なし、ネットワークバイトオーダー、2バイト
N | 符号なし、ネットワークバイトオーダー、4バイト
l | 符号付き、ホストバイトオーダー、4バイト（小文字のL）
L | 符号なし、ホストバイトオーダー、4バイト（大文字のL）
v | 符号なし、リトルエンディアンバイトオーダー、2バイト
V | 符号なし、リトルエンディアンバイトオーダー、4バイト
### package_length_func

**長さ解析関数を設定してください**

`C++`または`PHP`の2つのタイプの関数をサポートします。長さ関数は整数を返さなければなりません。

戻り値 | 役割
---|---
0を返す | 長さデータが不足しており、さらにデータを受信する必要がある
-1を返す | データがエラーであり、下層は自動的に接続を閉じる
パッケージの長さ値を返す（パッケージヘッダとボディ全体の長さを含む）| 下層はパッケージを組み立ててコールバック関数に返します

  * **ヒント**

    * **使用方法**

    長さを解析するために、最初に一部のデータを読み取り、そのデータ内に長さ値が含まれています。その後、この長さを下層に返します。そして、下層が残りのデータを受信し、それらを組み立てて`dispatch`することを完了します。

    * **PHPの長さ解析関数**

    `ZendVM`がマルチスレッド環境で実行されないため、下層は`PHP`の長さ関数を並行して実行することを避けるために、`Mutex`ミューテックスを自動的に使用します。`1.9.3`またはそれ以降のバージョンで利用可能です。

    長さ解析関数でブロッキング`I/O`操作を実行しないでください。全ての[Reactor](/learn?id=reactor线程)スレッドがブロックされる可能性があります

    ```php
    $server = new Swoole\Server("127.0.0.1", 9501);
    
    $server->set(array(
        'open_length_check'   => true,
        'dispatch_mode'       => 1,
        'package_length_func' => function ($data) {
          if (strlen($data) < 8) {
              return 0;
          }
          $length = intval(trim(substr($data, 0, 8)));
          if ($length <= 0) {
              return -1;
          }
          return $length + 8;
        },
        'package_max_length'  => 2000000,  //プロトコルの最大長
    ));
    
    $server->on('receive', function (Swoole\Server $server, $fd, $reactor_id, $data) {
        var_dump($data);
        echo "#{$server->worker_id}>> received length=" . strlen($data) . "\n";
    });
    
    $server->start();
    ```

    * **C++の長さ解析関数**

    他のPHP拡張では、`swoole_add_function`を使用して長さ関数を`Swoole`エンジンに登録します。
    
    C++の長さ関数は呼び出された際に下層によってロックされないため、呼び出し元がスレッドセーフを確保する必要があります
    
    ```c++
    #include <string>
    #include <iostream>
    #include "swoole.h"
    
    using namespace std;
    
    int test_get_length(swProtocol *protocol, swConnection *conn, char *data, uint32_t length);
    
    void register_length_function(void)
    {
        swoole_add_function((char *) "test_get_length", (void *) test_get_length);
        return SW_OK;
    }
    
    int test_get_length(swProtocol *protocol, swConnection *conn, char *data, uint32_t length)
    {
        printf("cpp, size=%d\n", length);
        return 100;
    }
    ```
### package_max_length

?> **パケットの最大サイズを設定します。単位はバイトです。**【デフォルト値：`2M` つまり `2 * 1024 * 1024`、最小値は `64K`】

?> [open_length_check](/server/setting?id=open_length_check)/[open_eof_check](/server/setting?id=open_eof_check)/[open_eof_split](/server/setting?id=open_eof_split)/[open_http_protocol](/server/setting?id=open_http_protocol)/[open_http2_protocol](/http_server?id=open_http2_protocol)/[open_websocket_protocol](/server/setting?id=open_websocket_protocol)/[open_mqtt_protocol](/server/setting?id=open_mqtt_protocol)などのプロトコルパーサーを有効にした場合、`Swoole`の内部でパケットの結合が行われ、この時、パケットが完全に受信されていない状態では、すべてのデータがメモリに保存されます。  
そのため、`package_max_length`を設定する必要があります。データパケットが占有できる最大メモリサイズです。1万の`TCP`接続がデータを送信している状況で、それぞれのデータパケットが`2M`の場合、最も極端な状況では、`20G`のメモリスペースが占有されます。

  * **ヒント**

    * `open_length_check`：パケットの長さが`package_max_length`を超えると、そのデータは直ちに破棄され、接続が閉じられ、メモリは使用されません；
    * `open_eof_check`：データパケットの長さが事前にわからないため、受信したデータは引き続きメモリに保存されるため、増加し続けます。メモリ使用量が`package_max_length`を超えると、そのデータは直ちに破棄され、接続が閉じられます；
    * `open_http_protocol`：`GET`リクエストは最大`8K`まで許可され、構成を変更することはできません。`POST`リクエストは`Content-Length`を検証します。`Content-Length`が`package_max_length`を超えると、そのデータが直ちに破棄され、`http 400`エラーが送信され、接続が閉じられます；

  * **注意**

    !> このパラメータは大きすぎると、大量のメモリを占有するため、設定しない方がよいです。
### open_http_protocol

?> **Enable the `HTTP` protocol processing.** [Default: `false`]

?> Enable the `HTTP` protocol processing. [Swoole\Http\Server](/http_server) will automatically enable this option. Setting it to `false` means disabling the `HTTP` protocol processing.
### open_mqtt_protocol

?> **Enable the `MQTT` protocol processing.**【Default value: `false`】

?> When enabled, it will parse the header of the `MQTT` packet, and the `worker` process [onReceive](/server/events?id=onreceive) will return a complete `MQTT` data packet each time.

```php
$server->set(array(
  'open_mqtt_protocol' => true
));
```
### open_redis_protocol

?> **Redisプロトコル処理を有効にします。**【デフォルト値：`false`】

?> 有効にすると、Redisプロトコルが解析され、`worker`プロセスが[onReceive](/server/events?id=onreceive)ごとに完全なRedisデータパケットを返します。[Redis\Server](/redis_server)を直接使用することをお勧めします。

```php
$server->set(array(
  'open_redis_protocol' => true
));
```
### open_websocket_protocol

?> **Enable the `WebSocket` protocol processing.**【Default value: `false`】

?> Enable the processing of the `WebSocket` protocol, the [Swoole\WebSocket\Server](websocket_server) will automatically enable this option. Setting it to `false` means turning off the `WebSocket` protocol processing.  
When the `open_websocket_protocol` option is set to `true`, the `open_http_protocol` protocol will also be automatically set to `true`.
### open_websocket_close_frame

?> **WebSocketプロトコルで閉じるフレームを有効にします。**【デフォルト値：`false`】

?> （`opcode`が`0x08`のフレーム）を`onMessage`コールバックで受信します。

?> 有効にすると、`WebSocketServer`の`onMessage`コールバックでクライアントまたはサーバーから送信された閉じるフレームを受信でき、開発者はそれに対して自分で処理できます。

```php
$server = new Swoole\WebSocket\Server("0.0.0.0", 9501);

$server->set(array("open_websocket_close_frame" => true));

$server->on('open', function (Swoole\WebSocket\Server $server, $request) {});

$server->on('message', function (Swoole\WebSocket\Server $server, $frame) {
    if ($frame->opcode == 0x08) {
        echo "Close frame received: Code {$frame->code} Reason {$frame->reason}\n";
    } else {
        echo "Message received: {$frame->data}\n";
    }
});

$server->on('close', function ($server, $fd) {});

$server->start();
```
### open_tcp_nodelay

?> **`open_tcp_nodelay`を有効にします。**【デフォルト値：`false`】

?> この設定を有効にすると、データを送信する際にNagleマージアルゴリズムを無効にし、直ちに対向のTCP接続に送信します。一部のシーンでは、例えばコマンドラインターミナルのように、コマンドをすぐにサーバーに送信する必要がある場合、応答速度が向上する可能性があります。Nagleアルゴリズムについては、Googleで検索してください。
### open_cpu_affinity

?> **Enable CPU affinity setting.** (Default: `false`)

?> Enabling this feature on a multi-core hardware platform will bind `Swoole`'s `reactor threads`/`worker processes` to a dedicated core. This can avoid processes/threads switching between multiple cores at runtime, and improve CPU cache hit rates.

  * **Tips**

    * **Use the taskset command to view the CPU affinity settings of a process:**

    ```bash
    taskset -p processID
    pid 24666's current affinity mask: f
    pid 24901's current affinity mask: 8
    ```

    > The mask is a bitmask number, with each bit corresponding to a CPU core. If a bit is `0`, the process is bound to that core and will be scheduled to run on it; if it's `1`, the process will not be scheduled to that CPU core. In the example given, the process with `pid` 24666 has a mask of `f`, indicating it's not bound to any CPU core and can be scheduled on any core. The process with `pid` 24901 has a mask of `8`, where `8` in binary is `1000`, meaning this process is bound to the 4th CPU core.
### cpu_affinity_ignore

?> **IO-intensive programs process all network interrupts using CPU0. If the network IO is heavy, high load on CPU0 can cause network interrupts to be processed untimely, resulting in a decrease in network packet sending and receiving capabilities.**

?> If this option is not set, Swoole will use all CPU cores, and the underlying setting of CPU bindings based on reactor_id or worker_id and the number of CPU cores to take the modulus.  
If the kernel and network card have multi-queue features, network interrupts will be distributed to multiple cores, which can alleviate the pressure of network interrupts.

```php
array('cpu_affinity_ignore' => array(0, 1)) // Accepts an array as a parameter, where array(0, 1) means not to use CPU0, CPU1, leaving them exclusively to handle network interrupts.
```

  * **Tip**

    * **Viewing network interrupts**

```shell
[~]$ cat /proc/interrupts 
           CPU0       CPU1       CPU2       CPU3       
  0: 1383283707          0          0          0    IO-APIC-edge  timer
  1:          3          0          0          0    IO-APIC-edge  i8042
  3:         11          0          0          0    IO-APIC-edge  serial
  8:          1          0          0          0    IO-APIC-edge  rtc
  9:          0          0          0          0   IO-APIC-level  acpi
 12:          4          0          0          0    IO-APIC-edge  i8042
 14:         25          0          0          0    IO-APIC-edge  ide0
 82:         85          0          0          0   IO-APIC-level  uhci_hcd:usb5
 90:         96          0          0          0   IO-APIC-level  uhci_hcd:usb6
114:    1067499          0          0          0       PCI-MSI-X  cciss0
130:   96508322          0          0          0         PCI-MSI  eth0
138:     384295          0          0          0         PCI-MSI  eth1
169:          0          0          0          0   IO-APIC-level  ehci_hcd:usb1, uhci_hcd:usb2
177:          0          0          0          0   IO-APIC-level  uhci_hcd:usb3
185:          0          0          0          0   IO-APIC-level  uhci_hcd:usb4
NMI:      11370       6399       6845       6300 
LOC: 1383174675 1383278112 1383174810 1383277705 
ERR:          0
MIS:          0
```

The `eth0/eth1` represents the number of network interrupts. If the network interrupts are evenly distributed among `CPU0 - CPU3`, it indicates that the network card has multi-queue features. If all interrupts are concentrated on a single core, it means that the network interrupts are all processed by that CPU. Once this CPU exceeds `100%`, the system will not be able to handle network requests efficiently. In such cases, you need to use `cpu_affinity_ignore` to set aside the CPU exclusively for handling network interrupts. 

In the example above, you should set `cpu_affinity_ignore => array(0)`.

?> You can use the `top` command `->` enter `1` to view the usage of each core.

  * **Note**

    !> This option must be set together with `open_cpu_affinity` to take effect.
### tcp_defer_accept

?> **`tcp_defer_accept` feature enabled** 【default value: `false`】

?> It can be set to a numeric value, indicating that an `accept` will be triggered only when a `TCP` connection has data to send.

```php
$server->set(array(
  'tcp_defer_accept' => 5
));
```

  * **Hint**

    * **After enabling the `tcp_defer_accept` feature, the timing of `accept` and the corresponding [onConnect](/server/events?id=onconnect) event will change. If set to `5` seconds:**

      * The `accept` will not be triggered immediately after the client connects to the server.
      * If the client sends data within `5` seconds, `accept/onConnect/onReceive` will be triggered sequentially.
      * If the client does not send any data within `5` seconds, `accept/onConnect` will be triggered.
### ssl_cert_file / ssl_key_file :id=ssl_cert_file

?> **SSLトンネル暗号を設定します。**

?> この値はファイル名の文字列であり、cert証明書とkey秘密鍵のパスを指定します。

  * **ヒント**

    * **`PEM` to `DER`形式**

    ```shell
    openssl x509 -in cert.crt -outform der -out cert.der
    ```

    * **`DER` to `PEM`形式**

    ```shell
    openssl x509 -in cert.crt -inform der -outform pem -out cert.pem
    ```

  * **注意**

    !> - `HTTPS`アプリケーションのブラウザは、ページを閲覧するために証明書を信頼する必要があります；
    - `wss`アプリケーションでは、`WebSocket`接続を開始するページは `HTTPS` を使用する必要があります；
    - ブラウザがSSL証明書を信頼しない場合、 `wss` を使用できません；
    - ファイルは`PEM`形式である必要があり、`DER`形式はサポートされていません。`openssl`ツールを使用して変換できます。

    !> `SSL`を使用するには、`Swoole`をコンパイルする際に[--enable-openssl](/environment?id=コンパイルオプション)オプションを追加する必要があります

    ```php
    $server = new Swoole\Server('0.0.0.0', 9501, SWOOLE_PROCESS, SWOOLE_SOCK_TCP | SWOOLE_SSL);
    $server->set(array(
        'ssl_cert_file' => __DIR__.'/config/ssl.crt',
        'ssl_key_file' => __DIR__.'/config/ssl.key',
    ));
    ```
### ssl_method

!> このパラメータは [v4.5.4](/version/bc?id=_454) で削除されました。`ssl_protocols` を使用してください。

?> **OpenSSLトンネル暗号化のアルゴリズムを設定します。**【デフォルト値：`SWOOLE_SSLv23_METHOD`】。サポートされているタイプについては[SSL暗号化方法](/consts?id=ssl-加密方法)を参照してください。

?> `Server` と `Client` で使用するアルゴリズムは一致している必要があります。そうでないと `SSL/TLS` ハンドシェイクが失敗し、接続が切断されます。

```php
$server->set(array(
    'ssl_method' => SWOOLE_SSLv3_CLIENT_METHOD,
));
```
### ssl_protocols

?> **Set the protocol for OpenSSL tunnel encryption.** [Default value: `0`, supporting all protocols], for supported types please refer to [SSL protocols](/consts?id=ssl-protocols)

!> Available in Swoole version >= `v4.5.4`

```php
$server->set(array(
    'ssl_protocols' => 0,
));
```
### ssl_sni_certs

?> **SNI (Server Name Identification) 証明書の設定**

!> Swooleのバージョン >= `v4.6.0` で使用可能

```php
$server->set([
    'ssl_cert_file' => __DIR__ . '/server.crt',
    'ssl_key_file' => __DIR__ . '/server.key',
    'ssl_protocols' => SWOOLE_SSL_TLSv1_2 | SWOOLE_SSL_TLSv1_3 | SWOOLE_SSL_TLSv1_1 | SWOOLE_SSL_SSLv2,
    'ssl_sni_certs' => [
        'cs.php.net' => [
            'ssl_cert_file' => __DIR__ . '/sni_server_cs_cert.pem',
            'ssl_key_file' => __DIR__ . '/sni_server_cs_key.pem',
        ],
        'uk.php.net' => [
            'ssl_cert_file' => __DIR__ . '/sni_server_uk_cert.pem',
            'ssl_key_file' => __DIR__ . '/sni_server_uk_key.pem',
        ],
        'us.php.net' => [
            'ssl_cert_file' => __DIR__ . '/sni_server_us_cert.pem',
            'ssl_key_file' => __DIR__ . '/sni_server_us_key.pem',
        ],
    ]
]);
```
### ssl_ciphers

?> **設定 OpenSSL 加密演算法。**【デフォルト値：`EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH`】

```php
$server->set(array(
    'ssl_ciphers' => 'ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP',
));
```

  * **ヒント**

    * `ssl_ciphers` を空の文字列に設定すると、`openssl` が自動的に暗号化アルゴリズムを選択します。
### ssl_verify_peer

?> **サーバーSSL設定は、相手の証明書を検証します。**【デフォルト値：`false`】

?> デフォルトでは無効になっており、つまりクライアントの証明書を検証しません。有効にする場合は、`ssl_client_cert_file`オプションを設定する必要があります。
### ssl_allow_self_signed

?> **Allow self-signed certificates.** 【Default value: `false`】
### ssl_client_cert_file

?> **根証明書、クライアント証明書を検証するために使用されます。**

```php
$server = new Swoole\Server('0.0.0.0', 9501, SWOOLE_PROCESS, SWOOLE_SOCK_TCP | SWOOLE_SSL);
$server->set(array(
    'ssl_cert_file'         => __DIR__ . '/config/ssl.crt',
    'ssl_key_file'          => __DIR__ . '/config/ssl.key',
    'ssl_verify_peer'       => true,
    'ssl_allow_self_signed' => true,
    'ssl_client_cert_file'  => __DIR__ . '/config/ca.crt',
));
```

!> `TCP`サービスで検証が失敗した場合、下層は接続を自動的に閉じます。
### ssl_compress

?> **`SSL/TLS`圧縮を有効にするかどうかを設定します。** [Co\Client](/coroutine_client/client)を使用する際に、`ssl_disable_compression` というエイリアスがあります。
### ssl_verify_depth

?> **If the certificate chain depth is too deep and exceeds the value set in this option, the verification will be terminated.**
### ssl_prefer_server_ciphers

?> **Enable server-side protection to prevent BEAST attacks.**
### ssl_dhparam

?> **指定DHE密码器的`Diffie-Hellman`参数。**
### ssl_ecdh_curve

?> **ECDH鍵共有で使用される`curve`を指定します。**

```php
$server = new Swoole\Server('0.0.0.0', 9501, SWOOLE_PROCESS, SWOOLE_SOCK_TCP | SWOOLE_SSL);
$server->set([
    'ssl_compress'                => true,
    'ssl_verify_depth'            => 10,
    'ssl_prefer_server_ciphers'   => true,
    'ssl_dhparam'                 => '',
    'ssl_ecdh_curve'              => '',
]);
```
### ユーザー

?> **`Worker/TaskWorker`子プロセスの所有ユーザーを設定します。**【デフォルト値: 実行スクリプトのユーザー】

?> サーバーが`1024`以下のポートを監視する必要がある場合、`root`権限が必要です。しかし、プログラムが`root`ユーザーで実行されていると、コードに脆弱性があると攻撃者が`root`権限でリモートコマンドを実行できるため、大きなリスクがあります。`user`項目を設定すると、メインプロセスを`root`権限で実行し、子プロセスを一般ユーザー権限で実行できます。

```php
$server->set(array(
  'user' => 'Apache'
));
```

  * **注意**

    !> -`root`ユーザーで起動した場合にのみ有効  
    -`user/group`構成項目を使用して作業プロセスを一般ユーザーに設定した場合、作業プロセスで`shutdown`/[reload](/server/methods?id=reload)メソッドを使用してサービスをシャットダウンまたは再起動することはできません。`root`アカウントで`shell`端末で`kill`コマンドを実行する方法しかありません。
### グループ

?> **`Worker/TaskWorker`サブプロセスのプロセスユーザーグループを設定します。**【デフォルト値：スクリプト実行ユーザーのグループ】

?> `user` 構成と同様に、この構成はプロセスの所属するユーザーグループを変更し、サーバープログラムのセキュリティを向上させます。

```php
$server->set(array(
  'group' => 'www-data'
));
```

  * **注意**

    !> ルートユーザーで起動する場合にのみ有効
### chroot

?> **Redirect the filesystem root directory for the `Worker` processes.**

?> This setting allows the processes to read and write to a filesystem that is isolated from the actual operating system filesystem. Enhances security.

```php
$server->set(array(
  'chroot' => '/data/server/'
));
```
### pid_file

?> **Set the path to the pid file.**

?> Automatically write the master process PID to a file when the `Server` starts, and delete the PID file when the `Server` closes.

```php
$server->set(array(
    'pid_file' => __DIR__.'/server.pid',
));
```

  * **注意**

    !> 使用時には注意が必要です。`Server`が正常に終了しない場合、`PID`ファイルは削除されません。プロセスが本当に存在するかどうかを検出するには[Swoole\Process::kill($pid, 0)](/process/process?id=kill)を使用してください。
### buffer_input_size / input_buffer_size :id=buffer_input_size

「配置接收输入缓存区内存尺寸。」【默认值：`2M`】

```php
$server->set([
    'buffer_input_size' => 2 * 1024 * 1024,
]);
```
### buffer_output_size / output_buffer_size :id=buffer_output_size

?> **バッファ出力サイズを設定します。**【デフォルト値：`2M`】

```php
$server->set([
    'buffer_output_size' => 32 * 1024 * 1024, // must be a number
]);
```

  * **ヒント**

    !> Swoole バージョン >= `v4.6.7` の場合、デフォルト値は符号なしINTの最大値`UINT_MAX`です

    * 単位はバイトで、デフォルトは`2M`です。`32 * 1024 * 1024`を設定すると、１回の`Server->send`で送信できるデータの最大サイズは`32M`バイトとなります
    * `Server->send`、`Http\Server->end/write`、`WebSocket\Server->push`などのデータ送信命令を呼び出すとき、`バッファ出力サイズ`の設定を超えて送信することはできません。

    !> このパラメータは[SWOOLE_PROCESS](/learn?id=swoole_process)モードにのみ有効であり、PROCESSモードではWorkerプロセスのデータがマスタープロセスに送信されてからクライアントに送信されるため、各Workerプロセスごとにバッファが確保されます。[参考](/learn?id=reactor线程)
### socket_buffer_size

?> **Configure the length of buffer for client connections.** 【Default value: `2M`】

?> Different from `buffer_output_size`, `buffer_output_size` limits the size of data sent by a worker process in one `send`, while `socket_buffer_size` is used to set the total buffer size for communication between `Worker` and `Master` processes, refer to [SWOOLE_PROCESS](/learn?id=swoole_process) mode.

```php
$server->set([
    'socket_buffer_size' => 128 * 1024 *1024, // Must be a number, in bytes, for example, 128 * 1024 *1024 means each TCP client connection can have a maximum of 128M pending data to send
]);
```

- **Data Sending Buffer**

    - When the Master process is sending data to clients in large amounts, it may not be able to send immediately. In this case, the data sent will be stored in the server's memory buffer. This parameter can adjust the size of the memory buffer.
    
    - If sending too much data and the buffer is full, the `Server` will report the following error message:
    
    ```bash
    swFactoryProcess_finish: send failed, session#1 output buffer has been overflowed.
    ```
    
    ?>When the send buffer is full, the `send` will fail and only affect the current client, other clients are not affected. When the server has a large number of `TCP` connections, in the worst-case scenario, it will occupy `serv->max_connection * socket_buffer_size` bytes of memory.
    
    - Especially in external communication server programs, when network communication is slow, if data is sent continuously, the buffer will quickly become full. The data sent will all accumulate in the server's memory. Therefore, such applications should consider the networking transmission capacity in their design, store messages on disk first, and then send new data after the client notifies the server that it has completed receiving.
    
    - For example, in a video streaming service, user `A` has a bandwidth of `100M`, sending `10M` of data in `1` second is completely feasible. User `B` has only `1M` bandwidth, if `10M` of data is sent in `1` second, user `B` may take `100` seconds to receive it all. In this case, the data will all accumulate in the server's memory.
    
    - Depending on the type of data, different processing can be done. If it is disposable content, such as video streaming, it is acceptable to discard some data frames in case of poor network conditions. If the content is non-negotiable, such as WeChat messages, it can be stored in the server's disk first, grouping them into batches of `100` messages. When the user has received this batch of messages, the next batch can be retrieved from the disk and sent to the client.
### enable_unsafe_event

?> **`onConnect/onClose`イベントを有効にします。**【デフォルト値：`false`】

?> `Swoole`は、設定 [dispatch_mode](/server/setting?id=dispatch_mode)=1 または`3`の後、システムが`onConnect/onReceive/onClose`の順序を保証できないため、デフォルトで`onConnect/onClose`イベントが無効になっています；  
アプリケーションが`onConnect/onClose`イベントが必要であり、順序の問題によるセキュリティリスクを許容できる場合は、`enable_unsafe_event`を`true`に設定して`onConnect/onClose`イベントを有効にできます。
### discard_timeout_request

?> **Closed connection data requests are discarded.** [Default value: `true`]

?> When configuring `Swoole` with [dispatch_mode](/server/setting?id=dispatch_mode)=`1` or `3`, the system cannot guarantee the order of `onConnect/onReceive/onClose` events, so there may be some request data that arrives at the `Worker` process after the connection is closed.

  * **Note**

    * The `discard_timeout_request` configuration defaults to `true`, which means that if the `worker` process receives data requests from a closed connection, it will automatically discard them.
    * Setting `discard_timeout_request` to `false` means that the `Worker` process will handle data requests regardless of whether the connection is closed or not.
### enable_reuse_port

?> **Setting port reuse.** 【Default value: `false`】

?> After enabling port reuse, you can start multiple Server programs listening to the same port.

  * **Tip**

    * `enable_reuse_port = true` to enable port reuse
    * `enable_reuse_port = false` to disable port reuse

!> Only available on kernels `Linux-3.9.0` and above. Available on `Swoole4.5` and above.
### enable_delay_receive

?> **セット`accept`クライアント接続後に自動的に[EventLoop](/learn?id=什么是eventloop)に追加されないようにします。**【デフォルト値：`false`】

?> このオプションを`true`に設定すると、`accept`クライアント接続後に自動的に[EventLoop](/learn?id=什么是eventloop)に追加されず、単に[onConnect](/server/events?id=onconnect)コールバックがトリガーされます。`worker`プロセスは [$server->confirm($fd)](/server/methods?id=confirm) を呼び出して接続を確認することで、この時点でのみ`fd`が[EventLoop](/learn?id=什么是eventloop)に追加されてデータ送受信が始まり、また`$server->close($fd)` を呼び出してこの接続を閉じることもできます。

```php
// enable_delay_receiveオプションを有効化
$server->set(array(
    'enable_delay_receive' => true,
));

$server->on("Connect", function ($server, $fd, $reactorId) {
    $server->after(2000, function() use ($server, $fd) {
        // 接続を確認し、データ受信を開始
        $server->confirm($fd);
    });
});
```
### reload_async

?> **Set the asynchronous reload switch.**【Default: `true`】

?> Setting the asynchronous reload switch. When set to `true`, the asynchronous safe reload feature will be enabled, and the `Worker` process will wait for asynchronous events to complete before exiting. For more information, please refer to [How to Properly Reload the Service](/question/use?id=swoole如何正确的重启服务)

?> The main purpose of enabling `reload_async` is to ensure that coroutines or asynchronous tasks can end normally during service reload.

```php
$server->set([
  'reload_async' => true
]);
```

  * **Coroutine Mode**

    * In `4.x` version, when [enable_coroutine](/server/setting?id=enable_coroutine) is enabled, an additional coroutine number check will be added on the underlying layer. The process will only exit when there are no coroutines at that time. Even if `reload_async => false` is set, enabling `enable_coroutine` will forcefully turn on `reload_async`.
### max_wait_time

?> **Worker**プロセスがサービス停止通知を受け取った後の最大待機時間を設定します【デフォルト値: `3`】

?> `Worker`プロセスがブロックされて待機し、`Worker`が正常に`reload`できない場合や、いくつかのプロダクションシナリオ（コードのホットリロードが必要な場合など）を満たすことができないことがよくあります。そのため、Swooleにはプロセスの再起動のタイムアウトオプションが追加されています。詳細は [サービスを正しく再起動する方法](/question/use?id=swoole如何正确的重启服务) を参照してください。

  * **ヒント**

    * **管理プロセスは再起動またはシャットダウンシグナルを受信した場合、または`max_request`に到達した場合、その`Worker`プロセスを再起動します。以下のステップに従います:**

      * ローレベルで、(`max_wait_time`)秒のタイマーが追加され、タイマーがトリガーされると、プロセスがまだ存在するかどうかをチェックし、存在する場合は強制的に終了させ、新しいプロセスを立ち上げます。
      * `onWorkerStop`コールバック内で後処理を行う必要があり、`max_wait_time`秒以内に後処理を完了する必要があります。
      * 対象のプロセスに順番に`SIGTERM`シグナルを送信し、プロセスを終了します。

  * **注意**

    !> `v4.4.x`以前はデフォルトで`30`秒でした
### tcp_fastopen

?> **TCP Quick Handshake feature enables.** 【default value: `false`】

?> This feature can improve the response speed of short `TCP` connections, carrying data when the client completes the third step of the handshake and sends a `SYN` packet.

```php
$server->set([
  'tcp_fastopen' => true
]);
```

  * **Hint**

    * This parameter can be set on the listening port. Students who wish to delve deeper can refer to the [Google paper](http://conferences.sigcomm.org/co-next/2011/papers/1569470463.pdf)
### request_slowlog_file

?> **リクエスト遅延ログを有効にします。** `v4.4.8`バージョンから[削除されました](https://github.com/swoole/swoole-src/commit/b1a400f6cb2fba25efd2bd5142f403d0ae303366)

!> この遅延ログの方法は、同期ブロッキングプロセスでのみ機能し、コルーチン環境では機能しないため、Swoole4はデフォルトでコルーチンが有効になっています。`enable_coroutine`を無効にしない限り使用しないでください。代わりに [Swoole Tracker](https://business.swoole.com/tracker/index) のブロッキング検出ツールを使用してください。

?> 有効にすると、`Manager`プロセスがタイマーシグナルを設定し、定期的にすべての`Task`および`Worker`プロセスを検出し、指定された時間を超える要求によりプロセスがブロックされると、プロセスの`PHP`関数呼び出しスタックを自動的に出力します。

?> この機能は、`ptrace`システムコールに基づいて実装されており、一部のシステムでは`ptrace`が無効になっている場合は遅いリクエストを追跡できません。`kernel.yama.ptrace_scope`カーネルパラメータが`0`になっているかどうかを確認してください。

```php
$server->set([
  'request_slowlog_file' => '/tmp/trace.log',
]);
```

  * **タイムアウト時間**

```php
$server->set([
    'request_slowlog_timeout' => 2, // リクエストのタイムアウト時間を2秒に設定
    'request_slowlog_file' => '/tmp/trace.log',
]);
```

!> 書き込み権限のあるファイルでなければなりません。そうでない場合、ファイルの作成に失敗し、下位レベルで致命的なエラーが発生する可能性があります。
### enable_coroutine

?> **非同期スタイルサーバーのコルーチンサポートを有効にする**

?> `enable_coroutine`が無効になると、[イベントコールバック関数](/server/events)で自動的にコルーチンが作成されなくなります。コルーチンを使用しない場合は、パフォーマンスが向上します。[Swooleコルーチンとは](/coroutine)を参照してください。

  * **構成方法**
    
    * `php.ini`で `swoole.enable_coroutine = 'Off'` を設定（[ini構成文書](/other/config.md)を参照）
    * `$server->set(['enable_coroutine' => false]);` はiniよりも優先されます

  * **`enable_coroutine`オプションの影響範囲**

      * onWorkerStart
      * onConnect
      * onOpen
      * onReceive
      * [setHandler](/redis_server?id=sethandler)
      * onPacket
      * onRequest
      * onMessage
      * onPipeMessage
      * onFinish
      * onClose
      * tick/after タイマー

!> `enable_coroutine`を有効にすると、上記のコールバック関数内で自動的にコルーチンが作成されます

* `enable_coroutine`が`true`に設定されている場合、内部的には[onRequest](/http_server?id=on)コールバックで自動的にコルーチンが作成されます。開発者は`go`関数を使用して明示的に[コルーチンを作成](/coroutine/coroutine?id=create)する必要はありません。
* `enable_coroutine`が`false`に設定されている場合、内部的にはコルーチンが自動的に作成されません。開発者がコルーチンを使用する場合は、明示的に`go`を使用してコルーチンを作成する必要があります。コルーチン機能を使用しない場合は、`Swoole1.x`と同じ対処方法が適用されます。
* ただし、この有効化はSwooleがコルーチンでリクエストを処理することを意味するだけであり、イベントにブロッキング関数が含まれる場合は、事前に[ワンクリックコルーチン化](/runtime)を有効にして、`sleep`、`mysqlnd`などのブロッキング関数や拡張機能をコルーチン化する必要があります

```php
$server = new Swoole\Http\Server("127.0.0.1", 9501);

$server->set([
    // 内蔵コルーチンを無効にする
    'enable_coroutine' => false,
]);

$server->on("request", function ($request, $response) {
    if ($request->server['request_uri'] == '/coro') {
        go(function () use ($response) {
            co::sleep(0.2);
            $response->header("Content-Type", "text/plain");
            $response->end("Hello World\n");
        });
    } else {
        $response->header("Content-Type", "text/plain");
        $response->end("Hello World\n");
    }
});

$server->start();
```
### send_yield

?> **When the buffer memory is insufficient during data transmission, the current coroutine will be [yielded](/coroutine?id=coroutine-scheduling) to wait for the data transmission to complete. When the buffer is cleared, the coroutine will automatically [resume](/coroutine?id=coroutine-scheduling) to continue sending data.** [Default Value: Available in [dispatch_mode](/server/setting?id=dispatch-mode) 2 and 4, and enabled by default]

* If `Server/Client->send` returns `false` with an error code of `SW_ERROR_OUTPUT_BUFFER_OVERFLOW`, it will not return `false` back to the PHP layer but will suspend the current coroutine with a [yield](/coroutine?id=coroutine-scheduling).
* The `Server/Client` listens for events to check if the buffer is cleared. Once this event is triggered, it indicates that the data in the buffer has been sent, and at this point, the corresponding coroutine is [resumed](/coroutine?id=coroutine-scheduling).
* After the coroutine is resumed, it continues to call `Server/Client->send` to write data into the buffer. At this point, because the buffer is empty, the sending will succeed.

Before:

```php
for ($i = 0; $i < 100; $i++) {
    // Returns false directly and throws an output buffer overflow error when the buffer is full
    $server->send($fd, $data_2m);
}
```

After improvement:

```php
for ($i = 0; $i < 100; $i++) {
    // Yields the current coroutine when the buffer is full, and resumes after sending is completed
    $server->send($fd, $data_2m);
}
```

!> This feature will change the default behavior of the underlying system and can be manually disabled.

```php
$server->set([
    'send_yield' => false,
]);
```

  * __Affected APIs__

    * [Swoole\Server::send](/server/methods?id=send)
    * [Swoole\Http\Response::write](/http_server?id=write)
    * [Swoole\WebSocket\Server::push](/websocket_server?id=push)
    * [Swoole\Coroutine\Client::send](/coroutine_client/client?id=send)
    * [Swoole\Coroutine\Http\Client::push](/coroutine_client/http_client?id=push)
### send_timeout

`send_yield`と一緒に使用される送信タイムアウトを設定します。指定された時間内にデータがバッファに送信されない場合、下位レイヤーは`false`を返し、エラーコードを`ETIMEDOUT`に設定します。[getLastError()](/server/methods?id=getlasterror)メソッドを使用してエラーコードを取得できます。

> 浮動小数点数型で、単位は秒で、最小単位はミリ秒です

```php
$server->set([
    'send_yield' => true,
    'send_timeout' => 1.5, // 1.5 seconds
]);

for ($i = 0; $i < 100; $i++) {
    if ($server->send($fd, $data_2m) === false and $server->getLastError() == SOCKET_ETIMEDOUT) {
      echo "Send timed out\n";
    }
}
```
### hook_flags

?> **Set the scope of functions to enable `one-click coroutine` hooking.** 【Default value: no hook】

!> Available in Swoole versions `v4.5+` or [4.4LTS](https://github.com/swoole/swoole-src/tree/v4.4.x), for more details refer to [Coroutine Installation](/runtime)

```php
$server->set([
    'hook_flags' => SWOOLE_HOOK_SLEEP,
]);
```
### buffer_high_watermark

?> **セットバッファの高水位マークをバイト単位で設定します。**

```php
$server->set([
    'buffer_high_watermark' => 8 * 1024 * 1024,
]);
```
### buffer_low_watermark

?> **設定緩衝區低水位線，單位為位元組。**

```php
$server->set([
    'buffer_low_watermark' => 1 * 1024 * 1024,
]);
```
### tcp_user_timeout

?> TCP_USER_TIMEOUT option is a socket option in the TCP layer, which specifies the maximum time in milliseconds that the data packet can be sent without receiving an ACK confirmation. Please refer to the man page for more details.

```php
$server->set([
    'tcp_user_timeout' => 10 * 1000, // 10 seconds
]);
```

!> This feature is available in Swoole version >= `v4.5.3-alpha`
### stats_file

?> **Specify the file path where the content of [stats()](/server/methods?id=stats) will be written. After setting this, a timer will be automatically set at [onWorkerStart](/server/events?id=onworkerstart) to periodically write the content of [stats()](/server/methods?id=stats) to the specified file.**

```php
$server->set([
    'stats_file' => __DIR__ . '/stats.log',
]);
```

!> Available in Swoole version >= `v4.5.5`
### event_object

?> **設定此選項後，事件回調將使用[對象風格](/server/events?id=回調對象)。**【默認值：`false`】

```php
$server->set([
    'event_object' => true,
]);
```

!> Swoole版本 >= `v4.6.0` 可用
### start_session_id

?> **セッションIDを設定します**

```php
$server->set([
    'start_session_id' => 10,
]);
```

!> Swooleバージョン >= `v4.6.0` で利用可能
### single_thread

?> **Set to single thread.** When enabled, the Reactor thread will be merged with the Master thread in the Master process, and the logic will be handled by the Master thread. When using `SWOOLE_PROCESS` mode in PHP ZTS, be sure to set this value to `true`.

```php
$server->set([
    'single_thread' => true,
]);
```

!> Available starting from Swoole version `v4.2.13`
### max_queued_bytes

?> **Set the maximum queue length of the receive buffer.** If exceeded, stop receiving.

```php
$server->set([
    'max_queued_bytes' => 1024 * 1024,
]);
```

!> Available since Swoole version `v4.5.0`
### admin_server

?> **[Swoole Dashboard](http://dashboard.swoole.com/) でサービス情報などを表示するためにadmin_serverサービスを設定します。**

```php
$server->set([
    'admin_server' => '0.0.0.0:9502',
]);
```

!> Swooleバージョンが`v4.8.0`以上である必要があります。

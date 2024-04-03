# タスクの実行

サーバープログラムで時間のかかる操作を実行する必要がある場合、例えばチャットサーバーがブロードキャストを送信したり、Webサーバーがメールを送信する場合などがあります。これらの関数を直接実行すると、現在のプロセスがブロックされ、サーバーの応答が遅くなります。

Swooleでは、非同期タスク処理の機能が提供されており、非同期タスクをTaskWorkerプールに送信することで、現在のリクエストの処理速度に影響を与えずに実行できます。

## プログラムコード

最初のTCPサーバーをベースに、[onTask](/server/events?id=ontask)および[onFinish](/server/events?id=onfinish)の2つのイベントコールバック関数を追加するだけで良くなります。また、タスクプロセスの数量を設定する必要があります。タスクの実行時間とタスクの量に応じて適切なタスクプロセスを設定してください。

以下のコードを task.php に書き込んでください。

```php
$serv = new Swoole\Server('127.0.0.1', 9501);

// Set the number of task worker processes.
$serv->set([
    'task_worker_num' => 4
]);

// This callback function is executed in the worker process.
$serv->on('Receive', function($serv, $fd, $reactor_id, $data) {
    // Send an async task
    $task_id = $serv->task($data);
    echo "Dispatch AsyncTask: id={$task_id}\n";
});

// Handling async tasks (this callback function is executed in the task process).
$serv->on('Task', function ($serv, $task_id, $reactor_id, $data) {
    echo "New AsyncTask[id={$task_id}]".PHP_EOL;
    // Return the result of the task execution
    $serv->finish("{$data} -> OK");
});

// Handling the result of async tasks (this callback function is executed in the worker process).
$serv->on('Finish', function ($serv, $task_id, $data) {
    echo "AsyncTask[{$task_id}] Finish: {$data}".PHP_EOL;
});

$serv->start();
```

`$serv->task()` を呼び出した後、プログラムはすぐに返り、コードの実行を続行します。onTaskコールバック関数はTaskプロセスプール内で非同期に実行されます。実行が完了した後に`$serv->finish()` を呼び出して結果を返します。

!> finish操作はオプションです。結果を返さないこともできます。`onTask` イベントで `return` を使用して結果を返すと、`Swoole\Server::finish()` 操作が呼び出されることになります。

# Swoole\Server\Task

`Swoole\Server\Task`の詳細について説明します。このクラスは非常にシンプルですが、`new Swoole\Server\Task()`を使って`Task`オブジェクトを取得することはできません。この種のオブジェクトには完全にサーバーの情報が含まれておらず、`Swoole\Server\Task`の任意のメソッドを実行すると致命的なエラーが発生します。

```shell
Invalid instance of Swoole\Server\Task in /home/task.php on line 3
```
## Properties
### $data
`worker`プロセスから`task`プロセスに渡されるデータ`data`は、`string`タイプの文字列です。

```php
Swoole\Server\Task->data
```  
### $dispatch_time
`task`プロセスにデータが到達する時間である`dispatch_time`を返します。このプロパティは`double`型です。

```php
Swoole\Server\Task->dispatch_time
```
### $id
`dispatch_time` 属性は、レコードが `task` プロセスに到達した時間を表します。この属性は整数型の `int` です。

```php
Swoole\Server\Task->id
```
### $worker_id
`worker_id`プロパティは、データがどの`worker`プロセスから送信されたかを示します。これは整数型の属性です。

```php
Swoole\Server\Task->worker_id
```
### $flags
この非同期タスクのフラグ情報 `flags `は、`int` 型の整数です。

```php
Swoole\Server\Task->flags
```

?> `flags` の結果には、次の種類があります:
   - SWOOLE_TASK_NOREPLY | SWOOLE_TASK_NONBLOCK は、これが`Worker`プロセスから`task`プロセスに送信されたものではないことを示します。この場合、`onTask` イベントで`Swoole\Server::finish()`を呼び出すと、警告が出力されます。
   - SWOOLE_TASK_CALLBACK | SWOOLE_TASK_NONBLOCK は、 `Swoole\Server::finish()` が最後のコールバック関数が`null`ではない場合を示します。この場合、`onFinish`イベントは実行されず、代わりにこのコールバック関数だけが実行されます。
   - SWOOLE_TASK_COROUTINE | SWOOLE_TASK_NONBLOCK は、タスクをコルーチン形式で処理することを示します。
   - SW_TASK_NONBLOCK はデフォルト値であり、上記の3つの状況がいずれも該当しない場合に使用されます。
```python
def greet():
    print("Hello, how can I help you?")
```

このセクションでは、私たちの方法を説明します。
### finish()

[TaskWorkerプロセス](/learn?id=taskworkerプロセス)内で、タスクが完了したことを`Worker`プロセスに通知し、結果データを渡すために使用されます。

```php
Swoole\Server\Task->finish(mixed $data): bool
```

  * **Parameters**

    * `mixed $data`

      * 機能：タスク処理の結果内容
      * デフォルト値：なし
      * その他の値：なし

  * **Note**
    * `finish`メソッドは複数回呼び出すことができます。`Worker`プロセスは複数回[onFinish](/server/events?id=onfinish)イベントをトリガーします
    * [onTask](/server/events?id=ontask)コールバック関数内で`finish`メソッドを呼び出した後も、`return`データは引き続き[onFinish](/server/events?id=onfinish)イベントをトリガーします
    * `Swoole\Server\Task->finish`はオプショナルです。`Worker`プロセスがタスクの実行結果に関心がない場合は、この関数を呼び出す必要はありません
    * [onTask](/server/events?id=ontask)コールバック関数内で`return`文字列を使用すると、`finish`の呼び出しと同等です

  * **注意**

  !> `Swoole\Server\Task->finish`関数を使用する場合は、`Server`に[onFinish](/server/events?id=onfinish)コールバック関数を設定する必要があります。この関数は、[TaskWorkerプロセス](/learn?id=taskworkerプロセス)の[onTask](/server/events?id=ontask)コールバックでのみ使用できます
### pack()

Given data is serialized.

```php
Swoole\Server\Task->pack(mixed $data): string|false
```

  * **Parameters**

    * `mixed $data`

      * Functionality: Content of the task processing result
      * Default value: None
      * Other values: None

  * **Return Value**
    * Returns the serialized result if successful.
### unpack()

将给定的数据反序列化。

```php
Swoole\Server\Task->unpack(string $data): mixed
```

  * **参数**

    * `string $data`

      * 功能：需要反序列化的数据
      * 默认值：无
      * 其它值：无

  * **返回值**
    * 调用成功返回反序列化后的结果。  
## 使用例

```php
<?php
$server->on('task', function(Swoole\Server $serv, Swoole\Server\Task $task) {
    $task->finish(['result' => true]);
});
```

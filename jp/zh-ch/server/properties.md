このドキュメントは、属性に関する情報を提供します。属性はオブジェクトや要素の特性を表し、その値に基づいて振る舞いやスタイルが変化します。属性は、HTMLやCSS、JavaScriptなどのプログラミング言語で使用されます。属性は通常、名前と値のペアで構成され、タグや要素に直接適用されます。それぞれのプログラミング言語において、異なる属性が存在し、それぞれが異なる目的で使用されます。
### $setting

[Server->set()](/server/methods?id=set)関数によって設定されるパラメータは、`Server->$setting`プロパティに保存されます。コールバック関数内で実行パラメータの値にアクセスできます。このプロパティは`array`型の配列です。

```php
Swoole\Server->setting
```

  * **例**

```php
$server = new Swoole\Server('127.0.0.1', 9501);
$server->set(array('worker_num' => 4));

echo $server->setting['worker_num'];
```
### $connections

`TCP` connection iterator that can be used to iterate through all current connections on the server using `foreach`. The functionality of this property is identical to [Server->getClientList](/server/methods?id=getclientlist), but it is more user-friendly.

The elements to iterate through are the `fd` of individual connections.

```php
Swoole\Server->connections
```

!> The `$connections` property is an iterator object, not a PHP array, so it cannot be accessed using `var_dump` or array indexes; it can only be iterated through using `foreach`.

  * **Base Mode**

    * In [SWOOLE_BASE](/learn?id=swoole_base) mode, cross-process operations on `TCP` connections are not supported. Therefore, in `BASE` mode, the `$connections` iterator can only be used within the current process.

  * **Example**

```php
foreach ($server->connections as $fd) {
  var_dump($fd);
}
echo "There are currently " . count($server->connections) . " connections in the server\n";
```
### $host

`host`プロパティは、現在のサーバーがリッスンしているホストアドレスを返します。このプロパティは`string`型の文字列です。

```php
Swoole\Server->host
```
### $port

`port`プロパティは、現在サーバーがリッスンしているポートを表します。このプロパティは`int`型の整数です。

```php
Swoole\Server->port
```
### $type

現在のサーバーの種類`type`を返します。このプロパティは、`int`型の整数です。

```php
Swoole\Server->type
```
!> このプロパティは、以下のいずれかの値を返します
- `SWOOLE_SOCK_TCP` tcp ipv4 ソケット
- `SWOOLE_SOCK_TCP6` tcp ipv6 ソケット
- `SWOOLE_SOCK_UDP` udp ipv4 ソケット
- `SWOOLE_SOCK_UDP6` udp ipv6 ソケット
- `SWOOLE_SOCK_UNIX_DGRAM` unix ソケット dgram
- `SWOOLE_SOCK_UNIX_STREAM` unix ソケット stream
### $ssl

`ssl`プロパティは現在のサーバーが`ssl`を使用しているかどうかを示す`bool`型の値です。

```php
Swoole\Server->ssl
```
### $mode

現在のサーバーのプロセスモード`mode`を返します。このプロパティは`int`タイプの整数です。

```php
Swoole\Server->mode
```

!> このプロパティは以下の値のいずれかを返します
- `SWOOLE_BASE` シングルプロセスモード
- `SWOOLE_PROCESS` マルチプロセスモード
### $ports

`Server::$ports`は、複数のポートをリッスンしている場合、すべての`Swoole\Server\Port`オブジェクトを取得できるようにするリスニング・ポートの配列です。

`swoole_server::$ports[0]`は、構築時に設定されたメインサーバーポートです。

  * **例**

```php
$ports = $server->ports;
$ports[0]->set($settings);
$ports[1]->on('Receive', function () {
    //callback
});
```
### $master_pid

現在のサーバーのメインプロセスの`PID`を返します。

```php
Swoole\Server->master_pid
```

!> `onStart/onWorkerStart`の後でのみ取得できます

  * **例**

```php
$server = new Swoole\Server("127.0.0.1", 9501);
$server->on('start', function ($server){
    echo $server->master_pid;
});
$server->on('receive', function ($server, $fd, $reactor_id, $data) {
    $server->send($fd, 'Swoole: '.$data);
    $server->close($fd);
});
$server->start();
```
### $manager_pid

`PID`を返します。現在のサーバーの管理プロセスの`PID`です。このプロパティは`int`型の整数です。

```php
Swoole\Server->manager_pid
```

!> `onStart/onWorkerStart`の後でのみ取得できます

  * **例**

```php
$server = new Swoole\Server("127.0.0.1", 9501);
$server->on('start', function ($server){
    echo $server->manager_pid;
});
$server->on('receive', function ($server, $fd, $reactor_id, $data) {
    $server->send($fd, 'Swoole: '.$data);
    $server->close($fd);
});
$server->start();
```
### $worker_id

`Worker`プロセスの番号を取得します。これには、[Taskプロセス](/learn?id=taskworkerプロセス)も含まれます。この属性は`int`型の整数です。

```php
Swoole\Server->worker_id
```

  * **例**

```php
$server = new Swoole\Server('127.0.0.1', 9501);
$server->set([
    'worker_num' => 8,
    'task_worker_num' => 4,
]);
$server->on('WorkerStart', function ($server, int $workerId) {
    if ($server->taskworker) {
        echo "task workerId：{$workerId}\n";
        echo "task worker_id：{$server->worker_id}\n";
    } else {
        echo "workerId：{$workerId}\n";
        echo "worker_id：{$server->worker_id}\n";
    }
});
$server->on('Receive', function ($server, $fd, $reactor_id, $data) {
});
$server->on('Task', function ($serv, $task_id, $reactor_id, $data) {
});
$server->start();
```

  * **ヒント**

    * この属性は[onWorkerStart](/server/events?id=onworkerstart)の時の`$workerId`と同じです。
    * `Worker`プロセスの番号の範囲は、 `[0, $server->setting['worker_num'] - 1]` です。
    * [Taskプロセス](/learn?id=taskworkerプロセス)の番号の範囲は、 `$server->setting['worker_num'], $server->setting['worker_num'] + $server->setting['task_worker_num'] - 1]` です。

!> ワーカープロセスがリスタートされた後、`worker_id`の値は変わりません。
### $taskworker

Current process is a `Task` process, this property is a `bool` type.

```php
Swoole\Server->taskworker
```

  * **Return value**

    * `true` indicates that the current process is a `Task` worker process
    * `false` indicates that the current process is a `Worker` process
### $worker_pid

`Worker`プロセスの現在のオペレーティングシステムのプロセス`ID`を取得します。`posix_getpid()`の返り値と同じで、この属性は`int`型の整数です。

```php
Swoole\Server->worker_pid
```

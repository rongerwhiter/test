# Redis\Server

`Server`クラスは`Redis`サーバープロトコルに準拠しており、このクラスを基にして`Redis`プロトコルのサーバープログラムを実装できます。

?> `Swoole\Redis\Server`は[Server](/server/tcp_init)を継承しているため、`Server`が提供するすべての`API`および設定が使用できます。プロセスモデルも同じです。[Server](/server/init)セクションをご参照ください。

* **対応可能なクライアント**

  * あらゆるプログラミング言語の`redis`クライアント。これにはPHPの`redis`拡張および`phpredis`ライブラリも含まれます。
  * [Swoole\Coroutine\Redis](/coroutine_client/redis) コルーチンクライアント
  * `Redis`が提供するコマンドラインツール。`redis-cli`、`redis-benchmark`などが含まれます。
## 方法

`Swoole\Redis\Server`は`Swoole\Server`を継承しており、親クラスが提供するすべてのメソッドを使用できます。
### setHandler

?> **Set the handler for `Redis` commands.**

!> `Redis\Server` does not require setting the [onReceive](/server/events?id=onreceive) callback. Simply use the `setHandler` method to set the processing function for the corresponding command. An `ERROR` response will automatically be sent to the client when an unsupported command is received, with the message `ERR unknown command '$command'`.

```php
Swoole\Redis\Server->setHandler(string $command, callable $callback);
```

* **Parameters** 

  * **`string $command`**
    * **Function**: Name of the command
    * **Default Value**: None
    * **Other Values**: None

  * **`callable $callback`**
    * **Function**: The command processing function 【when the callback function returns a string type, it will be automatically sent to the client】
    * **Default Value**: None
    * **Other Values**: None

    !> The returned data must be in `Redis` format, and the `format` static method can be used for packing. 

Now translating the PHP part:


```php
Swoole\Redis\Server->setHandler(string $command, callable $callback);
```

* **パラメータ** 

  * **`string $command`**
    * **機能**：コマンドの名前
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`callable $callback`**
    * **機能**：コマンドの処理関数【コールバック関数が文字列型を返す場合、自動的にクライアントに送信されます】
    * **デフォルト値**：なし
    * **その他の値**：なし

    !> 返されるデータは`Redis`形式である必要があり、パッキングには`format`静的メソッドを使用できます。
### format

?> **フォーマットコマンドの応答データをフォーマットします。**

```php
Swoole\Redis\Server::format(int $type, mixed $value = null);
```

* **パラメータ** 

  * **`int $type`**
    * **機能**：データのタイプ。下記の [フォーマットパラメータ定数](/redis_server?id=格式参数常量) の定数を参照してください。
    * **デフォルト値**：なし
    * **その他の値**：なし

    !> `$type` が `NIL` タイプの場合、`$value` を渡す必要はありません。`ERROR` および `STATUS` タイプは `$value` を省略可能です。`INT`、`STRING`、`SET`、`MAP` は必須です。

  * **`mixed $value`**
    * **機能**：値
    * **デフォルト値**：なし
    * **その他の値**：なし
### 送信

?> **`send()`メソッドを使用して、データをクライアントに送信します。**

```php
Swoole\Server->send(int $fd, string $data): bool
```
This is a constant value.

---

これは定数値です。
### フォーマットパラメータ定数

`Redis`の応答データをパッケージ化するために`format`関数で主に使用されます

定数 | 説明
---|---
Server::NIL | nilデータを返す
Server::ERROR | エラーコードを返す
Server::STATUS | ステータスを返す
Server::INT | 整数を返す。フォーマットには値を渡す必要があり、タイプは整数である必要があります
Server::STRING | 文字列を返す。フォーマットには値を渡す必要があり、タイプは文字列である必要があります
Server::SET | リストを返す。フォーマットには値を渡す必要があり、タイプは配列である必要があります
Server::MAP | マップを返す。フォーマットには値を渡す必要があり、タイプは関連インデックス配列である必要があります
```plaintext
Welcome to our platform! Please sign up to start using our services.
```

## 使用示例
```php
use Swoole\Redis\Server;

define('DB_FILE', __DIR__ . '/db');

$server = new Server("127.0.0.1", 9501, SWOOLE_BASE);

if (is_file(DB_FILE)) {
    $server->data = unserialize(file_get_contents(DB_FILE));
} else {
    $server->data = array();
}

$server->setHandler('GET', function ($fd, $data) use ($server) {
    if (count($data) == 0) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR wrong number of arguments for 'GET' command"));
    }

    $key = $data[0];
    if (empty($server->data[$key])) {
        return $server->send($fd, Server::format(Server::NIL));
    } else {
        return $server->send($fd, Server::format(Server::STRING, $server->data[$key]));
    }
});

$server->setHandler('SET', function ($fd, $data) use ($server) {
    if (count($data) < 2) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR wrong number of arguments for 'SET' command"));
    }

    $key = $data[0];
    $server->data[$key] = $data[1];
    return $server->send($fd, Server::format(Server::STATUS, "OK"));
});

$server->setHandler('sAdd', function ($fd, $data) use ($server) {
    if (count($data) < 2) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR wrong number of arguments for 'sAdd' command"));
    }

    $key = $data[0];
    if (!isset($server->data[$key])) {
        $array[$key] = array();
    }

    $count = 0;
    for ($i = 1; $i < count($data); $i++) {
        $value = $data[$i];
        if (!isset($server->data[$key][$value])) {
            $server->data[$key][$value] = 1;
            $count++;
        }
    }

    return $server->send($fd, Server::format(Server::INT, $count));
});

$server->setHandler('sMembers', function ($fd, $data) use ($server) {
    if (count($data) < 1) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR wrong number of arguments for 'sMembers' command"));
    }
    $key = $data[0];
    if (!isset($server->data[$key])) {
        return $server->send($fd, Server::format(Server::NIL));
    }
    return $server->send($fd, Server::format(Server::SET, array_keys($server->data[$key])));
});

$server->setHandler('hSet', function ($fd, $data) use ($server) {
    if (count($data) < 3) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR wrong number of arguments for 'hSet' command"));
    }

    $key = $data[0];
    if (!isset($server->data[$key])) {
        $array[$key] = array();
    }
    $field = $data[1];
    $value = $data[2];
    $count = !isset($server->data[$key][$field]) ? 1 : 0;
    $server->data[$key][$field] = $value;
    return $server->send($fd, Server::format(Server::INT, $count));
});

$server->setHandler('hGetAll', function ($fd, $data) use ($server) {
    if (count($data) < 1) {
        return $server->send($fd, Server::format(Server::ERROR, "ERR wrong number of arguments for 'hGetAll' command"));
    }
    $key = $data[0];
    if (!isset($server->data[$key])) {
        return $server->send($fd, Server::format(Server::NIL));
    }
    return $server->send($fd, Server::format(Server::MAP, $server->data[$key]));
});

$server->on('WorkerStart', function ($server) {
    $server->tick(10000, function () use ($server) {
        file_put_contents(DB_FILE, serialize($server->data));
    });
});

$server->start();
```
### クライアント

```shell
$ redis-cli -h 127.0.0.1 -p 9501
127.0.0.1:9501> set name swoole
OK
127.0.0.1:9501> get name
"swoole"
127.0.0.1:9501> sadd swooler rango
(integer) 1
127.0.0.1:9501> sadd swooler twosee guoxinhua
(integer) 2
127.0.0.1:9501> smembers swooler
1) "rango"
2) "twosee"
3) "guoxinhua"
127.0.0.1:9501> hset website swoole "www.swoole.com"
(integer) 1
127.0.0.1:9501> hset website swoole "swoole.com"
(integer) 0
127.0.0.1:9501> hgetall website
1) "swoole"
2) "swoole.com"
127.0.0.1:9501> test
(error) ERR unknown command 'test'
127.0.0.1:9501>
```

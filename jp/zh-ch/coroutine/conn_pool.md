# コネクションプール

Swooleは、バージョン`v4.4.13`から、組み込みのコルーチン接続プール機能を提供しています。このセクションでは、対応するコネクションプールの使用方法について説明します。
## ConnectionPool

[ConnectionPool](https://github.com/swoole/library/blob/master/src/core/ConnectionPool.php) は基本的な接続プールであり、Channelに基づいて自動スケジューリングされます。 任意のコンストラクター（`callable`）を渡すことができ、コンストラクターは接続オブジェクトを返す必要があります。

* `get` メソッドは接続を取得します（接続プールが満杯でない場合、新しい接続が作成されます）
* `put` メソッドは接続を回収します
* `fill` メソッドは接続プールを埋めます（事前に接続を作成）
* `close` メソッドは接続プールを閉じます

!> [Simps Framework](https://simps.io)の [DB コンポーネント](https://github.com/simple-swoole/db) は Database をラップしており、自動的に接続を返却したり、トランザクションなどの機能を実装しています。参考にしたり直接使用したりすることができます。詳細については [Simps ドキュメント](https://simps.io/#/zh-cn/database/mysql) を参照してください。
## データベース

さまざまなデータベース接続プールやオブジェクトプロキシの高度なラッパーで、自動的に再接続をサポートしています。現在、PDO、Mysqli、Redisの3種類のデータベースサポートが含まれています：

* `PDOConfig`、`PDOProxy`、`PDOPool`
* `MysqliConfig`、`MysqliProxy`、`MysqliPool`
* `RedisConfig`、`RedisProxy`、`RedisPool`

!> 1. MySQLの切断時の再接続では、ほとんどの接続コンテキスト（フェッチモード、設定された属性、コンパイル済みのステートメントなど）を自動的に回復しますが、トランザクションなどのコンテキストは回復できません。トランザクション中に接続が切断される場合は、例外が投げられますので、再接続の信頼性をご自身で評価してください。  
2. トランザクション中の接続をプールに戻す動作は未定義です。開発者は、戻される接続が再利用可能であることを保証する必要があります。  
3. 接続オブジェクトに再利用できない例外が発生した場合、開発者は `$pool->put(null);` を呼び出して空の接続を戻すことで、接続プールの数をバランスさせる必要があります。
### PDOPool/MysqliPool/RedisPool :id=pool

接続プールオブジェクトを作成するために、2つのパラメータが存在し、それぞれ対応するConfigオブジェクトとプールのサイズです。

```php
$pool = new \Swoole\Database\PDOPool(Swoole\Database\PDOConfig $config, int $size);

$pool = new \Swoole\Database\MysqliPool(Swoole\Database\MysqliConfig $config, int $size);

$pool = new \Swoole\Database\RedisPool(Swoole\Database\RedisConfig $config, int $size);
```

  * **パラメータ** 

    * **`$config`**
      * **機能**：対応するConfigオブジェクト、詳細な使用方法は[使用例](/coroutine/conn_pool?id=使用例)をご参照ください
      * **デフォルト値**：なし
      * **その他の値**：【[PDOConfig](https://github.com/swoole/library/blob/master/src/core/Database/PDOConfig.php)、[RedisConfig](https://github.com/swoole/library/blob/master/src/core/Database/RedisConfig.php)、[MysqliConfig](https://github.com/swoole/library/blob/master/src/core/Database/MysqliConfig.php)】
      
    * **`int $size`**
      * **機能**：接続プールの数
      * **デフォルト値**：64
      * **その他の値**：なし
```plaintext
This is an example of text inside a code block, which should not be translated.
```

## 使用示例
```php
<?php
declare(strict_types=1);

use Swoole\Coroutine;
use Swoole\Database\PDOConfig;
use Swoole\Database\PDOPool;
use Swoole\Runtime;

const N = 1024;

Runtime::enableCoroutine();
$s = microtime(true);
Coroutine\run(function () {
    $pool = new PDOPool((new PDOConfig)
        ->withHost('127.0.0.1')
        ->withPort(3306)
        // ->withUnixSocket('/tmp/mysql.sock')
        ->withDbName('test')
        ->withCharset('utf8mb4')
        ->withUsername('root')
        ->withPassword('root')
    );
    for ($n = N; $n--;) {
        Coroutine::create(function () use ($pool) {
            $pdo = $pool->get();
            $statement = $pdo->prepare('SELECT ? + ?');
            if (!$statement) {
                throw new RuntimeException('Prepare failed');
            }
            $a = mt_rand(1, 100);
            $b = mt_rand(1, 100);
            $result = $statement->execute([$a, $b]);
            if (!$result) {
                throw new RuntimeException('Execute failed');
            }
            $result = $statement->fetchAll();
            if ($a + $b !== (int)$result[0][0]) {
                throw new RuntimeException('Bad result');
            }
            $pool->put($pdo);
        });
    }
});
$s = microtime(true) - $s;
echo 'Use ' . $s . 's for ' . N . ' queries' . PHP_EOL;
```
### Redis

```php
<?php
declare(strict_types=1);

use Swoole\Coroutine;
use Swoole\Database\RedisConfig;
use Swoole\Database\RedisPool;
use Swoole\Runtime;

const N = 1024;

Runtime::enableCoroutine();
$s = microtime(true);
Coroutine\run(function () {
    $pool = new RedisPool((new RedisConfig)
        ->withHost('127.0.0.1')
        ->withPort(6379)
        ->withAuth('')
        ->withDbIndex(0)
        ->withTimeout(1)
    );
    for ($n = N; $n--;) {
        Coroutine::create(function () use ($pool) {
            $redis = $pool->get();
            $result = $redis->set('foo', 'bar');
            if (!$result) {
                throw new RuntimeException('Set failed');
            }
            $result = $redis->get('foo');
            if ($result !== 'bar') {
                throw new RuntimeException('Get failed');
            }
            $pool->put($redis);
        });
    }
});
$s = microtime(true) - $s;
echo 'Use ' . $s . 's for ' . (N * 2) . ' queries' . PHP_EOL;
``` 

上記のコードは、RedisとSwoole Coroutineを使用して、一連のクエリを実行する方法を示しています。RedisPoolを作成し、さまざまなRedisコマンドを実行しています。
```php
<?php
declare(strict_types=1);

use Swoole\Coroutine;
use Swoole\Database\MysqliConfig;
use Swoole\Database\MysqliPool;
use Swoole\Runtime;

const N = 1024;

Runtime::enableCoroutine();
$s = microtime(true);
Coroutine\run(function () {
    $pool = new MysqliPool((new MysqliConfig)
        ->withHost('127.0.0.1')
        ->withPort(3306)
        // ->withUnixSocket('/tmp/mysql.sock')
        ->withDbName('test')
        ->withCharset('utf8mb4')
        ->withUsername('root')
        ->withPassword('root')
    );
    for ($n = N; $n--;) {
        Coroutine::create(function () use ($pool) {
            $mysqli = $pool->get();
            $statement = $mysqli->prepare('SELECT ? + ?');
            if (!$statement) {
                throw new RuntimeException('Prepare failed');
            }
            $a = mt_rand(1, 100);
            $b = mt_rand(1, 100);
            if (!$statement->bind_param('dd', $a, $b)) {
                throw new RuntimeException('Bind param failed');
            }
            if (!$statement->execute()) {
                throw new RuntimeException('Execute failed');
            }
            if (!$statement->bind_result($result)) {
                throw new RuntimeException('Bind result failed');
            }
            if (!$statement->fetch()) {
                throw new RuntimeException('Fetch failed');
            }
            if ($a + $b !== (int)$result) {
                throw new RuntimeException('Bad result');
            }
            while ($statement->fetch()) {
                continue;
            }
            $pool->put($mysqli);
        });
    }
});
$s = microtime(true) - $s;
echo 'Use ' . $s . 's for ' . N . ' queries' . PHP_EOL;
```

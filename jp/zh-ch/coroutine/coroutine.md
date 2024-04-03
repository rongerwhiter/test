# コルーチンAPI

> このセクションを見る前に、基本的なコルーチンの概念を理解するために[概要](/coroutine)を最初に見ることをお勧めします。
## 方法
### set()

コルーチンの設定、コルーチンに関するオプションの設定。

```php
Swoole\Coroutine::set(array $options);
```

パラメータ | 安定性 | 役割 
---|---|---
max_coroutine | - | グローバル最大コルーチン数を設定し、制限を超えると、新しいコルーチンを作成できなくなります。Serverでは、[server->max_coroutine](/server/setting?id=max_coroutine)によって上書きされます。
stack_size/c_stack_size | - | 個々のコルーチンの初期Cスタックのメモリサイズを設定します。デフォルトは2Mです。
log_level | v4.0.0 | ログレベル [詳細はこちら](/consts?id=日志等级)
trace_flags | v4.0.0 | トレースフラグ [詳細はこちら](/consts?id=跟踪标签)
socket_connect_timeout | v4.2.10 | 接続タイムアウト、**[クライアントのタイムアウトルール](/coroutine_client/init?id=超时规则)を参照**
socket_read_timeout | v4.3.0 | 読み取りタイムアウト、**[クライアントのタイムアウトルール](/coroutine_client/init?id=超时规则)を参照**
socket_write_timeout | v4.3.0 | 書き込みタイムアウト、**[クライアントのタイムアウトルール](/coroutine_client/init?id=超时规则)を参照**
socket_dns_timeout | v4.4.0 | ドメイン名解決のタイムアウト、**[クライアントのタイムアウトルール](/coroutine_client/init?id=超时规则)を参照**
socket_timeout | v4.2.10 | 送信/受信タイムアウト、**[クライアントのタイムアウトルール](/coroutine_client/init?id=超时规则)を参照**
dns_cache_expire | v4.2.11 | swoole dnsキャッシュの有効期間を秒単位で設定します。デフォルトは60秒です。
dns_cache_capacity | v4.2.11 | swoole dnsキャッシュ容量を設定します。デフォルトは1000です。
hook_flags | v4.4.0 | 一括コルーチン化するhookの範囲設定、[一括コルーチン化](/runtime)を参照
enable_preemptive_scheduler | v4.4.0 | コルーチンのプリエンプティブスケジューラをオンにし、コルーチンの最大実行時間を10msに設定します。[ini設定](/other/config)を上書きします。
dns_server | v4.5.0 | dnsクエリのサーバーを設定します。デフォルトは"8.8.8.8"です。
exit_condition | v4.5.0 | `callable`を渡し、boolを返すようにします。リアクタの終了条件をカスタマイズできます。例えば、コルーチン数が0になったときにのみプログラムを終了したい場合は、`Co::set(['exit_condition' => function () {return Co::stats()['coroutine_num'] === 0;}]);`と書きます。
enable_deadlock_check | v4.6.0 | コルーチンデッドロック検出を有効にするかどうかを設定します。デフォルトでは有効です。
deadlock_check_disable_trace | v4.6.0 | コルーチンデッドロック検出のトレース出力を有効/無効にします。
deadlock_check_limit | v4.6.0 | コルーチンデッドロック検出時の最大出力数を制限します。
deadlock_check_depth | v4.6.0 | コルーチンデッドロック検出時のスタックフレームの数を制限します。
max_concurrency | v4.8.2 | 最大同時リクエスト数を設定します。
### getOptions()

协程関連の設定を取得します。

!> Swooleバージョン >= `v4.6.0`で利用可能

```php
Swoole\Coroutine::getOptions(): null|array;
```
### create()

新しいコルーチンを作成し、すぐに実行します。

```php
Swoole\Coroutine::create(callable $function, ...$args): int|false
go(callable $function, ...$args): int|false // php.iniのuse_shortname設定を参照
```

* **Parameters**

    * **`callable $function`**
      * **役割**：実行されるコルーチンのコードで、`callable`である必要があります。作成できるコルーチンの合計数は、[server->max_coroutine](/server/setting?id=max_coroutine)の設定に制限されます。
      * **デフォルト値**：なし
      * **その他の値**：なし

* **Return Value**

    * 作成に失敗した場合は`false`を返す
    * 成功した場合はコルーチンの`ID`を返す

!> バックグラウンドでは、サブコルーチンのコードが優先的に実行されるため、`Coroutine::create`は、サブコルーチンが一時停止した場合にのみ戻り、現在のコルーチンのコードを続行します。

  * **実行順序**

    1つのコルーチン内で`go`を使って新しいコルーチンをネストさせる。Swooleのコルーチンは単一プロセス、単一スレッドモデルであるため、

    * `go`で作成されたサブコルーチンが優先的に実行され、サブコルーチンが完了するか一時停止した場合は、親コルーチンに戻り、コードを続行します。
    * サブコルーチンが一時停止した後に親コルーチンが終了しても、サブコルーチンの実行に影響はありません。

    ```php
    \Co\run(function() {
        go(function () {
            Co::sleep(3.0);
            go(function () {
                Co::sleep(2.0);
                echo "co[3] end\n";
            });
            echo "co[2] end\n";
        });

        Co::sleep(1.0);
        echo "co[1] end\n";
    });
    ```

* **コルーチンの負荷**

  各コルーチンは互いに独立しており、個別のメモリ空間（スタックメモリ）を作成する必要があります。`PHP-7.2`バージョンでは、内部でコルーチンの変数を格納するために`8K`の`stack`が割り当てられています。`zval`のサイズは`16バイト`であるため、`8K`の`stack`は最大で`512`個の変数を保存できます。コルーチンのスタックメモリ使用量が`8K`を超えると、`ZendVM`が自動的に拡張します。

  コルーチンが終了すると、割り当てられた`stack`メモリが解放されます。

  * `PHP-7.1`、`PHP-7.0`では、デフォルトで`256K`のスタックメモリが割り当てられます
  * `Co::set(['stack_size' => 4096])`を呼び出して、デフォルトのスタックメモリサイズを変更できます
### defer()

`defer`はリソースの解放に使用され、**コルーチンが閉じる前**（つまりコルーチン関数が完了するとき）に呼び出されます。例外が発生しても、登録された`defer`は実行されます。

!> Swoole バージョン >= 4.2.9

```php
Swoole\Coroutine::defer(callable $function);
defer(callable $function); // ショートAPI
```

!> 注意すべきは、呼び出しの順序が逆順（後入れ先出し）であることです。つまり、`defer`が先に登録されますが、後で実行されます。逆順はリソースの解放に適したロジックであり、後で要求されたリソースは通常、先に要求されたリソースに基づいています。そのため、先に確保されたリソースが解放されると、後で要求されたリソースの解放が難しくなる可能性があります。

  * **例**

```php
go(function () {
    defer(function () use ($db) {
        $db->close();
    });
});
```
### exists()

指定されたコルーチンが存在するかどうかを判断します。

```php
Swoole\Coroutine::exists(int $cid = 0): bool
```

!> Swooleバージョン >= v4.3.0

  * **例**

```php
\Co\run(function () {
    go(function () {
        go(function () {
            Co::sleep(0.001);
            var_dump(Co::exists(Co::getPcid())); // 1: true
        });
        go(function () {
            Co::sleep(0.003);
            var_dump(Co::exists(Co::getPcid())); // 3: false
        });
        Co::sleep(0.002);
        var_dump(Co::exists(Co::getPcid())); // 2: false
    });
});
```
### getCid()

現在のコルーチンの一意の`ID`を取得します。これは`getuid`という別名があり、プロセス内で一意の正の整数です。

```php
Swoole\Coroutine::getCid(): int
```

* **Return Value**

    * 現在のコルーチンの`ID`を取得できた場合
    * コルーチン環境内にない場合、`-1`を返します
### getPcid()

現在のコルーチンの親`ID`を取得します。

```php
Swoole\Coroutine::getPcid([$cid]): int
```

!> Swoole バージョン >= v4.3.0

* **パラメータ**

    * **`int $cid`**
      * **説明**：コルーチンの`cid`、パラメータが省略された場合、特定のコルーチンの`id`を渡してその親`id`を取得できます
      * **デフォルト値**：現在のコルーチン
      * **その他の値**：なし

  * **例**

```php
var_dump(Co::getPcid());
\Co\run(function () {
    var_dump(Co::getPcid());
    go(function () {
        var_dump(Co::getPcid());
        go(function () {
            var_dump(Co::getPcid());
            go(function () {
                var_dump(Co::getPcid());
            });
            go(function () {
                var_dump(Co::getPcid());
            });
            go(function () {
                var_dump(Co::getPcid());
            });
        });
        var_dump(Co::getPcid());
    });
    var_dump(Co::getPcid());
});
var_dump(Co::getPcid());

// --期待値--

// bool(false)
// int(-1)
// int(1)
// int(2)
// int(3)
// int(3)
// int(3)
// int(1)
// int(-1)
// bool(false)
```

!> ネストされていないコルーチン呼び出しで`getPcid`を行うと `-1` が返されます（非コルーチンスペースから作成された場合）  
非コルーチン内で`getPcid`を実行すると `false` が返されます（親コルーチンが存在しないため）  
`0` は予約された`id`であり、返される値には表示されません

!> コルーチン間には実質的な親子関係はありません。コルーチンは互いに隔離され、独立して動作します。この `Pcid` は、現在のコルーチンを作成したコルーチンの`id`として理解できます。

  * **用途**

    * **複数のコルーチン呼び出しスタックを連結する**

```php
\Co\run(function () {
    go(function () {
        $ptrace = Co::getBackTrace(Co::getPcid());
        // balababala
        var_dump(array_merge($ptrace, Co::getBackTrace(Co::getCid())));
    });
});
```
### getContext()

現在のコルーチンのコンテキストオブジェクトを取得します。

```php
Swoole\Coroutine::getContext([int $cid = 0]): Swoole\Coroutine\Context
```

!> Swooleのバージョン >= v4.3.0

* **パラメータ**

    * **`int $cid`**
      * **機能**：コルーチン `CID`、オプションのパラメータ
      * **デフォルト値**：現在のコルーチン `CID`
      * **その他の値**：なし

  * **効果**

    * コルーチンが終了した後に自動的にコンテキストがクリーンアップされます（他のコルーチンやグローバル変数の参照がない場合）
    * `defer`登録と呼び出しが不要です（クリーンアップメソッドを登録する必要がないため、関数を呼び出す必要がありません）
    * PHPの配列を使用したコンテキストのハッシュ計算のオーバーヘッドがありません（コルーチン数が非常に多い場合には一定の利点があります）
    * `Co\Context`は`ArrayObject`を使用し、さまざまなストレージ要件を満たします（オブジェクトであり、配列としても操作できます）

  * **例**

```php
function func(callable $fn, ...$args)
{
    go(function () use ($fn, $args) {
        $fn(...$args);
        echo 'Coroutine#' . Co::getCid() . ' exit' . PHP_EOL;
    });
}

/**
* 低いバージョン用の互換性
* @param object|Resource $object
* @return int
*/
function php_object_id($object)
{
    static $id = 0;
    static $map = [];
    $hash = spl_object_hash($object);
    return $map[$hash] ?? ($map[$hash] = ++$id);
}

class Resource
{
    public function __construct()
    {
        echo __CLASS__ . '#' . php_object_id((object)$this) . ' constructed' . PHP_EOL;
    }

    public function __destruct()
    {
        echo __CLASS__ . '#' . php_object_id((object)$this) . ' destructed' . PHP_EOL;
    }
}

$context = new Co\Context();
assert($context instanceof ArrayObject);
assert(Co::getContext() === null);
func(function () {
    $context = Co::getContext();
    assert($context instanceof Co\Context);
    $context['resource1'] = new Resource;
    $context->resource2 = new Resource;
    func(function () {
        Co::getContext()['resource3'] = new Resource;
        Co::yield();
        Co::getContext()['resource3']->resource4 = new Resource;
        Co::getContext()->resource5 = new Resource;
    });
});
Co::resume(2);

Swoole\Event::wait();

// --EXPECT--
// Resource#1 constructed
// Resource#2 constructed
// Resource#3 constructed
// Coroutine#1 exit
// Resource#2 destructed
// Resource#1 destructed
// Resource#4 constructed
// Resource#5 constructed
// Coroutine#2 exit
// Resource#5 destructed
// Resource#3 destructed
// Resource#4 destructed
```
### yield()

現在のコルーチンの実行権限を手動で放棄します。IOベースの[コルーチンスケジュール](/coroutine?id=协程调度)に基づいていません。

このメソッドにはもう1つのエイリアスがあります：`Coroutine::suspend()`

!> `Coroutine::resume()`メソッドとペアで使用する必要があります。コルーチンが`yield`した後は、他の外部コルーチンが`resume`しなければコルーチンリークが発生し、一度停止したコルーチンは永遠に実行されません。

```php
Swoole\Coroutine::yield();
```

  * **例**

```php
$cid = go(function () {
    echo "co 1 start\n";
    Co::yield();
    echo "co 1 end\n";
});

go(function () use ($cid) {
    echo "co 2 start\n";
    Co::sleep(0.5);
    Co::resume($cid);
    echo "co 2 end\n";
});
Swoole\Event::wait();
```
### resume()

手动恢复某个协程，使其继续运行，不是基于IO的[协程调度](/coroutine?id=协程调度)。

!> 当前协程处于挂起状态时，另外的协程中可以使用`resume`再次唤醒当前协程

```php
Swoole\Coroutine::resume(int $coroutineId);
```

* **参数**

    * **`int $coroutineId`**
      * **功能**：要恢复的协程`ID`
      * **默认值**：无
      * **其它值**：无

  * **示例**

```php
$id = go(function(){
    $id = Co::getuid();
    echo "start coro $id\n";
    Co::suspend();
    echo "resume coro $id @1\n";
    Co::suspend();
    echo "resume coro $id @2\n";
});
echo "start to resume $id @1\n";
Co::resume($id);
echo "start to resume $id @2\n";
Co::resume($id);
echo "main\n";
Swoole\Event::wait();

// --EXPECT--
// start coro 1
// start to resume 1 @1
// resume coro 1 @1
// start to resume 1 @2
// resume coro 1 @2
// main
```
### list()

すべてのコルーチンをトラバースします。

```php
Swoole\Coroutine::list(): Swoole\Coroutine\Iterator
Swoole\Coroutine::listCoroutines(): Swoole\Coroitine\Iterator
```

!> バージョン`v4.3.0`未満では、`listCoroutines`を使用する必要があります。新しいバージョンでは、このメソッドの名前が短縮され、`listCoroutines`がエイリアスになります。`list` は `v4.1.0` よりも新しいバージョンで利用可能です。

* **Return Value**

    * イテレータが返され、`foreach` でトラバースできるか、`iterator_to_array` を使用して配列に変換できます。

```php
$coros = Swoole\Coroutine::listCoroutines();
foreach($coros as $cid)
{
    var_dump(Swoole\Coroutine::getBackTrace($cid));
}
```
### stats()

協調状態を取得する。

```php
Swoole\Coroutine::stats(): array
```

* **戻り値**

key | 役割
---|---
event_num | 現在のリアクターイベント数
signal_listener_num | 現在の信号リスナー数
aio_task_num | 非同期IOタスク数（ここでのaioはファイルIOまたはDNSを指し、他のネットワークIOは含まれません）
aio_worker_num | 非同期IOワーカー数
c_stack_size | 各協調のCスタックサイズ
coroutine_num | 現在実行中の協調数
coroutine_peak_num | 現在実行中の協調数のピーク値
coroutine_last_cid | 最後に作成された協調のID

  * **例**

```php
var_dump(Swoole\Coroutine::stats());

array(1) {
  ["c_stack_size"]=>
  int(2097152)
  ["coroutine_num"]=>
  int(132)
  ["coroutine_peak_num"]=>
  int(2)
}
```
### getBackTrace()

协程函数の呼び出しスタックを取得します。

```php
Swoole\Coroutine::getBackTrace(int $cid = 0, int $options = DEBUG_BACKTRACE_PROVIDE_OBJECT, int $limit = 0): array
```

!> Swooleバージョン >= v4.1.0

* **パラメーター**

    * **`int $cid`**
      * **機能**：協力の `CID`
      * **デフォルト値**：現在の協力 `CID`
      * **その他の値**：なし

    * **`int $options`**
      * **機能**：オプションの設定
      * **デフォルト値**：`DEBUG_BACKTRACE_PROVIDE_OBJECT` 【`object`のインデックスを提供するかどうか】
      * **その他の値**：`DEBUG_BACKTRACE_IGNORE_ARGS` 【argsのインデックスを無視するかどうか、すべての関数/メソッドの引数が含まれ、メモリの使用量を節約できる】

    * **`int limit`**
      * **機能**：返されるスタックフレームの数を制限します
      * **デフォルト値**：`0`
      * **その他の値**：なし

* **戻り値**

    * 指定した協力が存在しない場合、`false`が返されます
    * 成功時は、[debug_backtrace](https://www.php.net/manual/zh/function.debug-backtrace.php) 関数の返り値と同じフォーマットの配列が返されます

  * **例**

```php
function test1() {
    test2();
}

function test2() {
    while(true) {
        Co::sleep(10);
        echo __FUNCTION__." \n";
    }
}
\Co\run(function () {
    $cid = go(function () {
        test1();
    });

    go(function () use ($cid) {
        while(true) {
            echo "BackTrace[$cid]:\n-----------------------------------------------\n";
            //配列が返されるため、適切にフォーマットして出力する必要があります
            var_dump(Co::getBackTrace($cid))."\n";
            Co::sleep(3);
        }
    });
});
Swoole\Event::wait();
```
### printBackTrace()

コルーチン関数の呼び出しスタックをプリントします。引数は`getBackTrace`と同じです。

!> Swooleのバージョンが`v4.6.0`以上で利用可能です

```php
Swoole\Coroutine::printBackTrace(int $cid = 0, int $options = DEBUG_BACKTRACE_PROVIDE_OBJECT, int $limit = 0);
```
### getElapsed()

协程の実行時間を取得して、分析や統計を行ったり、ゾンビコルーチンを見つけるために使用します。

!> Swooleバージョン >= `v4.5.0` で使用できます

```php
Swoole\Coroutine::getElapsed([$cid]): int
```

* **Parameters**

    * **`int $cid`**
      * **機能**：オプションのパラメータ、コルーチンの `CID`
      * **デフォルト値**：現在のコルーチン `CID`
      * **その他の値**：なし

* **Return value**

    * コルーチンが実行されている時間の単位、ミリ秒単位の精度
### cancel()

あるコルーチンをキャンセルするために使用されますが、現在のコルーチンをキャンセルすることはできません。

!> Swooleバージョン >= `v4.7.0` で利用可能

```php
Swoole\Coroutine::cancel($cid): bool
```

* **Parameters**

    * **`int $cid`**
        * **Description**: コルーチンの `CID`
        * **Default**: なし
        * **Other values**: なし

* **Return Value**

    * 成功した場合は `true` が返され、失敗した場合は `false` が返されます。
    * 失敗した場合は、[swoole_last_error()](/functions?id=swoole_last_error) を呼び出してエラー情報を確認できます。
### isCanceled()

現在の操作が手動でキャンセルされたかどうかを判断するために使用されます

!> Swooleバージョン >= `v4.7.0` で利用できます

```php
Swoole\Coroutine::isCanceled(): bool
```

* **戻り値**

    * 手動でキャンセルされた場合は`true`が返されます。失敗した場合は`false`が返されます。
```php
use Swoole\Coroutine;
use Swoole\Coroutine\System;
use function Swoole\Coroutine\run;
use function Swoole\Coroutine\go;

run(function () {
    $chan = new Coroutine\Channel(1);
    $cid = Coroutine::getCid();
    go(function () use ($cid) {
        System::sleep(0.002);
        assert(Coroutine::cancel($cid) === true);
    });

    assert($chan->push("hello world [1]", 100) === true);
    assert(Coroutine::isCanceled() === false);
    assert($chan->errCode === SWOOLE_CHANNEL_OK);

    assert($chan->push("hello world [2]", 100) === false);
    assert(Coroutine::isCanceled() === true);
    assert($chan->errCode === SWOOLE_CHANNEL_CANCELED);

    echo "Done\n";
});
``` 

上記のコードはSwoole Coroutineパッケージを使用しています。CoroutineはPHPの非同期タスクを処理するために使用されます。例では、`Channel`を使用してコルーチン間でデータを送受信しています。最後に`echo`文で "Done" と表示されます。
### enableScheduler()

一時的にコルーチンのプリエンプティブスケジューリングを有効にします。

!> Swooleバージョン >= `v4.4.0` で利用可能

```php
Swoole\Coroutine::enableScheduler();
```
### disableScheduler()

協調を一時的に無効にし、割り込みスケジュールを停止します。

!> Swooleバージョン >= `v4.4.0`で利用可能

```php
Swoole\Coroutine::disableScheduler();
```
### getStackUsage()

PHPのスタックのメモリ使用量を取得します。

!> Swooleバージョン >= `v4.8.0` で利用可能

```php
Swoole\Coroutine::getStackUsage([$cid]): int
```

* **パラメーター**

    * **`int $cid`**
        * **説明**：オプションのパラメーター、コルーチンの `CID`
        * **デフォルト値**：現在のコルーチン `CID`
        * **その他の値**：なし
### join()

複数のコルーチンを並行して実行します。

!> Swooleバージョン >= `v4.8.0` で利用可能

```php
Swoole\Coroutine::join(array $cid_array, float $timeout = -1): bool
```

* **パラメータ**

    * **`array $cid_array`**
        * **機能**：実行するコルーチンの `CID` の配列
        * **デフォルト値**：無し
        * **その他の値**：無し

    * **`float $timeout`**
        * **機能**：総合的なタイムアウト時間、タイムアウトするとすぐに戻ります。ただし、実行中のコルーチンは続行され、中断されることはありません
        * **デフォルト値**：-1
        * **その他の値**：無し

* **戻り値**

    * 成功時は `true` を返し、失敗すると `false` を返します
    * 失敗した場合は [swoole_last_error()](/functions?id=swoole_last_error) を使用してエラー情報を確認できます

* **使用例**

```php
use Swoole\Coroutine;

use function Swoole\Coroutine\go;
use function Swoole\Coroutine\run;

run(function () {
    $status = Coroutine::join([
        go(function () use (&$result) {
            $result['baidu'] = strlen(file_get_contents('https://www.baidu.com/'));
        }),
        go(function () use (&$result) {
            $result['google'] = strlen(file_get_contents('https://www.google.com/'));
        })
    ], 1);
    var_dump($result, $status, swoole_strerror(swoole_last_error(), 9));
});
```
## 関数
### batch()

複数のコルーチンを並行して実行し、これらのコルーチンメソッドの戻り値を配列で返します。

!> Swooleバージョン >= `v4.5.2` で利用可能

```php
Swoole\Coroutine\batch(array $tasks, float $timeout = -1): array
```

* **パラメータ**

    * **`array $tasks`**
      * **機能**：メソッドコールバックの配列を渡します。`key` が指定されている場合、戻り値もその `key` で指定されたものになります
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`float $timeout`**
      * **機能**：総合的なタイムアウト時間。タイムアウト後、すぐに返されます。ただし、実行中のコルーチンは完了するままになり、停止しません
      * **デフォルト値**：-1
      * **その他の値**：なし

* **戻り値**

    * メソッドの戻り値を含む配列を返します。`$tasks` パラメータで `key` が指定されている場合、戻り値もその `key` で指定されたものになります

* **使用例**

```php
use Swoole\Coroutine;
use function Swoole\Coroutine\batch;

Coroutine::set(['hook_flags' => SWOOLE_HOOK_ALL]);

$start_time = microtime(true);
Coroutine\run(function () {
    $use = microtime(true);
    $results = batch([
        'file_put_contents' => function () {
            return file_put_contents(__DIR__ . '/greeter.txt', "Hello,Swoole.");
        },
        'gethostbyname' => function () {
            return gethostbyname('localhost');
        },
        'file_get_contents' => function () {
            return file_get_contents(__DIR__ . '/greeter.txt');
        },
        'sleep' => function () {
            sleep(1);
            return true; // Returns NULL because it exceeds the set timeout of 0.1 seconds. After a timeout, it will return immediately. However, the running coroutine will continue to execute to completion without being aborted.
        },
        'usleep' => function () {
            usleep(1000);
            return true;
        },
    ], 0.1);
    $use = microtime(true) - $use;
    echo "Use {$use}s, Result:\n";
    var_dump($results);
});
$end_time =  microtime(true) - $start_time;
echo "Use {$end_time}s, Done\n";
```
### parallel()

複数のコルーチンを並行して実行します。

!> Swooleバージョン >= `v4.5.3` で使用可能です

```php
Swoole\Coroutine\parallel(int $n, callable $fn): void
```

* **パラメータ**

    * **`int $n`**
      * **機能**：最大コルーチン数を`$n`に設定します
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`callable $fn`**
      * **機能**：実行するコールバック関数を指定します
      * **デフォルト値**：なし
      * **その他の値**：なし

* **使用例**

```php
use Swoole\Coroutine;
use Swoole\Coroutine\System;
use function Swoole\Coroutine\parallel;

$start_time = microtime(true);
Coroutine\run(function () {
    $use = microtime(true);
    $results = [];
    parallel(2, function () use (&$results) {
        System::sleep(0.2);
        $results[] = System::gethostbyname('localhost');
    });
    $use = microtime(true) - $use;
    echo "Use {$use}s, Result:\n";
    var_dump($results);
});
$end_time =  microtime(true) - $start_time;
echo "Use {$end_time}s, Done\n";
```
### map()

[array_map](https://www.php.net/manual/zh/function.array-map.php)に似ており、配列の各要素にコールバック関数を適用します。

!> Swooleバージョン >= `v4.5.5` で利用可能です

```php
Swoole\Coroutine\map(array $list, callable $fn, float $timeout = -1): array
```

* **パラメータ**

    * **`array $list`**
      * **機能**：`$fn`関数を実行する配列
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`callable $fn`**
      * **機能**：`$list`配列の各要素に適用するコールバック関数
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`float $timeout`**
      * **機能**：合計のタイムアウト時間。タイムアウトした場合、即座に返されます。ただし実行中のコルーチンは完了するまで実行され、中止されません。
      * **デフォルト値**：-1
      * **その他の値**：なし

* **使用例**

```php
use Swoole\Coroutine;
use function Swoole\Coroutine\map;

function fatorial(int $n): int
{
    return array_product(range($n, 1));
}

Coroutine\run(function () {
    $results = map([2, 3, 4], 'fatorial'); 
    print_r($results);
});
```
### deadlock_check()

Coroutine deadlock detection, when called will output relevant stack information;

Enabled by default, after the termination of [EventLoop](learn?id=what-is-eventloop), if there are any coroutine deadlocks, the underlying layer will automatically call it;

It can be turned off by setting `enable_deadlock_check` in [Coroutine::set](/coroutine/coroutine?id=set).

!> Available only in Swoole version >= `v4.6.0`

```php
Swoole\Coroutine\deadlock_check();
```

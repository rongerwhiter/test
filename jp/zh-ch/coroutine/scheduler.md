# Coroutine\Scheduler

すべての[coroutine](/coroutine)は、`coroutine container`内で[created](/coroutine/coroutine?id=create)する必要があります。`Swoole`プログラムは通常、起動時に自動的に`coroutine container`を作成します。`Swoole`を使用してプログラムを起動する方法は次の3つあります：

   - [非同期スタイル](/server/init)のサーバープログラムの[start](/server/methods?id=start)メソッドを呼び出し、この起動方法はイベントコールバック中に`coroutine container`を作成します。[enable_coroutine](/server/setting?id=enable_coroutine)を参照してください。
   - `Swoole`が提供する2つのプロセス管理モジュール、つまり[Process](/process/process)と[Process\Pool](/process/process_pool)の[start](/process/process_pool?id=start)メソッドを呼び出し、この起動方法はプロセス起動時に`coroutine container`を作成します。これら2つのモジュールの構築関数の`enable_coroutine`パラメータを参照してください。
   - 他にも直接coroutineを書き換えてプログラムを起動する方法があり、最初に`coroutine container`を作成する必要があります（`Coroutine\run()`関数、これはjavaやcの`main`関数と考えることができます）。例：

* **フルコルーチン`HTTP`サービスを起動する**

```php
use Swoole\Coroutine\Http\Server;
use function Swoole\Coroutine\run;

run(function () {
    $server = new Server('127.0.0.1', 9502, false);
    $server->handle('/', function ($request, $response) {
        $response->end("<h1>Index</h1>");
    });
    $server->handle('/test', function ($request, $response) {
        $response->end("<h1>Test</h1>");
    });
    $server->handle('/stop', function ($request, $response) use ($server) {
        $response->end("<h1>Stop</h1>");
        $server->shutdown();
    });
    $server->start();
});
echo 1;//実行されません
```

* **2つのcoroutineを追加して同時にいくつかの処理を行う**

```php
use Swoole\Coroutine;
use function Swoole\Coroutine\run;

run(function () {
    Coroutine::create(function() {
        var_dump(file_get_contents("http://www.xinhuanet.com/"));
    });

    Coroutine::create(function() {
        Coroutine::sleep(1);
        echo "done\n";
    });
});
echo 1;//実行されます
```

!> `Swoole v4.4+`で利用可能。

!> `Coroutine\run()`をネストすることはできません。  
`Coroutine\run()`内のロジックに未処理のイベントがある場合、`Coroutine\run()`の後には後続のコードが実行されなくなります。これに対して、イベントがない場合、続行されます。再度`Coroutine\run()`を行うことができます。

前述の`Coroutine\run()`関数は実際には`Swoole\Coroutine\Scheduler`クラス（coroutine scheduler class）のラッパーであり、詳細を知りたい人は`Swoole\Coroutine\Scheduler`クラスのメソッドを見ることができます。
### set()

?> **Set coroutine runtime parameters.** 

?> This is an alias of `Coroutine::set` method. Please refer to the [Coroutine::set](/coroutine/coroutine?id=set) documentation

```php
Swoole\Coroutine\Scheduler->set(array $options): bool
```

  * **Example**

```php
$sch = new Swoole\Coroutine\Scheduler;
$sch->set(['max_coroutine' => 100]);
```
### getOptions()

?> **Get the coroutine runtime options that have been set.** Available since Swoole version `v4.6.0`

?> It is an alias of the `Coroutine::getOptions` method. Please refer to the [Coroutine::getOptions](/coroutine/coroutine?id=getoptions) documentation

```php
Swoole\Coroutine\Scheduler->getOptions(): null|array
```
### add()

?> **タスクを追加します。**

```php
Swoole\Coroutine\Scheduler->add(callable $fn, ... $args): bool
```

  * **パラメータ** 

    * **`callable $fn`**
      * **機能**：コールバック関数
      * **デフォルト値**：なし
      * **他の値**：なし

    * **`... $args`**
      * **機能**：オプションのパラメータ、コルーチンに渡されます
      * **デフォルト値**：なし
      * **他の値**：なし

  * **例**

```php
use Swoole\Coroutine;

$scheduler = new Coroutine\Scheduler;
$scheduler->add(function ($a, $b) {
    Coroutine::sleep(1);
    echo assert($a == 'hello') . PHP_EOL;
    echo assert($b == 12345) . PHP_EOL;
    echo "Done.\n";
}, "hello", 12345);

$scheduler->start();
```
  
  * **留意点**

    !> `go` 関数とは異なり、ここで追加されたコルーチンは即座に実行されず、`start` メソッドが呼び出されるまで開始および実行されません。プログラムにコルーチンの追加のみを行い、`start` を呼び出して起動しない場合、コルーチン関数 `$fn` は実行されません。
### parallel()

?> **並列タスクを追加します。**

?> `add`メソッドとは異なり、`parallel`メソッドは並列コルーチンを作成します。`start`時には`$num`個の`$fn`コルーチンが同時に起動され、並列に実行されます。

```php
Swoole\Coroutine\Scheduler->parallel(int $num, callable $fn, ... $args): bool
```

  * **パラメータ** 

    * **`int $num`**
      * **機能**: コルーチンを起動する個数
      * **デフォルト値**: なし
      * **その他の値**: なし

    * **`callable $fn`**
      * **機能**: コールバック関数
      * **デフォルト値**: なし
      * **その他の値**: なし

    * **`... $args`**
      * **機能**: オプションのパラメータ、コルーチンに渡される
      * **デフォルト値**: なし
      * **その他の値**: なし

  * **例**

```php
use Swoole\Coroutine;

$scheduler = new Coroutine\Scheduler;

$scheduler->parallel(10, function ($t, $n) {
    Coroutine::sleep($t);
    echo "Co ".Coroutine::getCid()."\n";
}, 0.05, 'A');

$scheduler->start();
```
### start()

?> **Start the program.**

?> Iterate through coroutine tasks added using the `add` and `parallel` methods and execute them.

```php
Swoole\Coroutine\Scheduler->start(): bool
```

  * **Return Value**

    * If start successfully, it will execute all added tasks. `start` will return `true` when all coroutines exit.
    * If start fails, it returns `false`, which can happen if it has already been started or if another scheduler has already been created and cannot be created again.

# タイマー Timer

ミリ秒精度のタイマー。内部では`epoll_wait`と`setitimer`に基づいており、データ構造には`最小ヒープ`が使用されており、多数のタイマーを追加できます。

* 同期I/Oプロセスでは、`Manager`や`TaskWorker`プロセスのように`setitimer`とシグナルを使用して実装されています。
* 非同期I/Oプロセスでは、`epoll_wait`/`kevent`/`poll`/`select`のタイムアウト時間を使用して実装されています。
## パフォーマンス

最小ヒープデータ構造を使ったタイマーの実装により、非常に高速なパフォーマンスが実現されています。タイマーの追加や削除はすべてメモリ上の操作で行われているため、性能が非常に優れています。

> 公式のベンチマークスクリプト [timer.php](https://github.com/swoole/benchmark/blob/master/timer.php) では、ランダムな時間のタイマーを`10`万個追加または削除するのに約`0.08s`かかります。

```shell
~/workspace/swoole/benchmark$ php timer.php
add 100000 timer :0.091133117675781s
del 100000 timer :0.084658145904541s
```

!> タイマーはメモリ操作であり、`IO`コストはかかりません。
## 差異

`Timer`とPHP本体の`pcntl_alarm`は異なります。`pcntl_alarm`は、クロックシグナル+tick関数に基づいて実装されていて、いくつかの欠点があります：

  * 最大で秒単位までしかサポートされず、`Timer`ではミリ秒単位まで可能です
  * 複数のタイマープログラムを同時に設定することができません
  * `pcntl_alarm`は`declare(ticks = 1)`に依存しており、パフォーマンスが非常に悪い
## ゼロミリ秒タイマー

基盤レベルでは、時間パラメーターが`0`のタイマーはサポートされていません。これは`Node.js`などのプログラミング言語とは異なります。`Swoole`では、[Swoole\Event::defer](/event?id=defer)を使用して同様の機能を実現できます。

```php
Swoole\Event::defer(function () {
  echo "hello\n";
});
```

!> 上記のコードは`JS`の`setTimeout(func, 0)`と全く同じ効果を持っています。
## 别名

`tick()`、`after()`、`clear()`都拥有一个函数风格的别名

```php
class Swoole\Timer {
    public static function tick() {
        // Some code here
    }
    public static function after() {
        // Some code here
    }
    public static function clear() {
        // Some code here
    }
}
```

类静态方法 | 函数风格别名
---|---
`Swoole\Timer::tick()` | `swoole_timer_tick()`
`Swoole\Timer::after()` | `swoole_timer_after()`
`Swoole\Timer::clear()` | `swoole_timer_clear()`
### 1. タスクの定義

タスクを理解し、目標を明確にします。

### 2. ステップの設計

実行可能な手順を設計し、必要なリソースを特定します。

### 3. 手順の実行

設計した手順に沿ってタスクを実行します。

```python
def run_task():
    # タスクを実行するコードを記述
  
run_task()
```

### 4. 結果の評価

タスクの実行結果を評価し、必要に応じて修正を加えます。
### tick()

間隔タイマーを設定します。

`after`タイマーとは異なり、`tick`タイマーは[Timer::clear](/timer?id=clear)を呼び出すまで継続的にトリガーされます。

```php
Swoole\Timer::tick(int $msec, callable $callback_function, ...$params): int
```

!> 1. タイマーは現在のプロセス空間でのみ有効です
   2. タイマーは純粋な非同期実装であり、[同期IO](/learn?id=同期io异步io)関数と組み合わせて使用することはできません。そうしないとタイマーの実行時間が乱れる恐れがあります
   3. タイマーの実行中には、一定の誤差が発生する可能性があります

  * **パラメータ** 

    * **`int $msec`**
      * **機能**：指定された時間
      * **単位**：ミリ秒【例：`1000`は`1`秒を意味し、`v4.2.10`未満のバージョンでは最大で `86400000`を超えてはいけません】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`callable $callback_function`**
      * **機能**：指定された時間が経過後に実行される関数で、呼び出し可能である必要があります
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`...$params`**
      * **機能**：実行関数にデータを渡す【このパラメータもオプションです】
      * **デフォルト値**：なし
      * **その他の値**：なし
      
      !> コールバック関数に引数を渡すために、匿名関数の`use`構文を使用できます

  * **$callback_function コールバック関数** 

    ```php
    callbackFunction(int $timer_id, ...$params);
    ```

      * **`int $timer_id`**
        * **機能**：タイマーの`ID`【これを使用して[Timer::clear](/timer?id=clear)でこのタイマーをクリアできます】
        * **デフォルト値**：なし
        * **その他の値**：なし

      * **`...$params`**
        * **機能**：`Timer::tick`に渡された3番目のパラメーター`$param`
        * **デフォルト値**：なし
        * **その他の値**：なし

  * **拡張**

    * **タイマーの修正**

      タイマーコールバック関数の実行時間は、次回のタイマー実行の時間に影響を与えません。例：`0.002s`で`10ms`の`tick`タイマーを設定した場合、最初の実行は`0.012s`にコールバック関数を実行し、コールバック関数が`5ms`実行された場合でも、次のタイマーは`0.022s`にトリガーされます。`0.027s`ではなく。
      
      ただし、タイマーコールバック関数の実行時間が長すぎて、次のタイマーの実行時間が上書きされる場合があります。この場合、基盤は時間の調整を行い、期限切れの行動を破棄し、次の時間にコールバックを行います。上の例では、`0.012s`でコールバック関数が`15ms`実行された場合、本来`0.022s`に1回目のタイマーコールバックが発生する予定ですが、実際にはタイマーは`0.027s`に返ります。この時点でタイマーは既に期限切れです。基盤は`0.032s`で再びタイマーコールバックをトリガーします。

    * **コルーチンモード**

      コルーチン環境下では、`Timer::tick`コールバック内で自動的にコルーチンが作成され、コルーチン関連の`API`を直接使用できます。`go`を呼び出してコルーチンを作成する必要はありません。
      
      !> [enable_coroutine](/timer?id=close-timer-co) を設定して自動的にコルーチンの作成を無効にできます

  * **使用例**

    ```php
    Swoole\Timer::tick(1000, function(){
        echo "timeout\n";
    });
    ```

    * **正しい例**

    ```php
    Swoole\Timer::tick(3000, function (int $timer_id, $param1, $param2) {
        echo "timer_id #$timer_id, after 3000ms.\n";
        echo "param1 is $param1, param2 is $param2.\n";

        Swoole\Timer::tick(14000, function ($timer_id) {
            echo "timer_id #$timer_id, after 14000ms.\n";
        });
    }, "A", "B");
    ```

    * **誤った例**

    ```php
    Swoole\Timer::tick(3000, function () {
        echo "after 3000ms.\n";
        sleep(14);
        echo "after 14000ms.\n";
    });
    ```
### after()

指定された時間の後に関数を実行します。 `Swoole\Timer::after` 関数は一度だけのタイマーであり、実行が完了すると破棄されます。

この関数は、`PHP`の標準ライブラリで提供されている `sleep` 関数とは異なり、`after` は非ブロッキングです。 一方、`sleep` 関数を呼び出すと、現在のプロセスがブロックされ、新しいリクエストを処理できなくなります。

```php
Swoole\Timer::after(int $msec, callable $callback_function, ...$params): int
```

  * **Parameters** 

    * **`int $msec`**
      * **Purpose**: 指定された時間
      * **Unit**: ミリ秒【例: `1000` は `1` 秒を表し、 `v4.2.10` 未満のバージョンでは最大値は `86400000` を超えてはいけない】
      * **Default**: なし
      * **Other values**: なし

    * **`callable $callback_function`**
      * **Purpose**: 時間が経過した後に実行される関数、呼び出し可能である必要があります。
      * **Default**: なし
      * **Other values**: なし

    * **`...$params`**
      * **Purpose**: 実行関数にデータを渡す【このパラメータはオプション】
      * **Default**: なし
      * **Other values**: なし
      
      !> コールバック関数に引数を渡すためにuse構文を使用できます

  * **Return Value**

    * タイマーの`ID`が成功した場合に返されます。タイマーをキャンセルする場合は、[Swoole\Timer::clear](/timer?id=clear)を呼び出すことができます。

  * **Extensions**

    * **Coroutine Mode**

      コルーチン環境では、[Swoole\Timer::after](/timer?id=after)コールバックで自動的にコルーチンが作成され、コルーチン関連の`API`を直接使用できるため、 `go`を呼び出す必要はありません。

      !> [enable_coroutine](/timer?id=close-timer-co)を設定して自動的にコルーチンの作成を無効にすることができます

  * **Usage Example**

```php
$str = "Swoole";
Swoole\Timer::after(1000, function() use ($str) {
    echo "Hello, $str\n";
});
```
```php
Swoole\Timer::clear()

Uses the timer `ID` to delete the timer.

```php
Swoole\Timer::clear(int $timer_id): bool
```

  * **Parameters** 

    * **`int $timer_id`**
      * **Function**: Timer `ID`【After calling [Timer::tick](/timer?id=tick) or [Timer::after](/timer?id=after), it will return an integer ID】
      * **Default value**: None
      * **Other values**: None

!> `Swoole\Timer::clear` cannot clear timers of other processes, it only works on the current process.

  * **Usage Example**

```php
$timer = Swoole\Timer::after(1000, function () {
    echo "timeout\n";
});

var_dump(Swoole\Timer::clear($timer));
var_dump($timer);

// Output: bool(true) int(1)
// No output: timeout
``` 
```
### clearAll()

すべてのタイマーをクリアします。

!> Swooleバージョン >= `v4.4.0` で利用可能

```php
Swoole\Timer::clearAll(): bool
```
### info()

`timer`の情報を返します。

!> Swooleのバージョン >= `v4.4.0` で使用可能

```php
Swoole\Timer::info(int $timer_id): array
```

  * **Return Value**

```php
array(5) {
  ["exec_msec"]=>
  int(6000)
  ["exec_count"]=> // v4.8.0 追加
  int(5)
  ["interval"]=>
  int(1000)
  ["round"]=>
  int(0)
  ["removed"]=>
  bool(false)
}
```
### list()

定時器のイテレータを返します。このイテレータを使用して、現在のWorkerプロセス内のすべての`timer`のIDを`foreach`で処理できます。

注意： Swooleバージョン >= `v4.4.0` で利用可能

```php
Swoole\Timer::list(): Swoole\Timer\Iterator
```

  * **使用例**

```php
foreach (Swoole\Timer::list() as $timer_id) {
    var_dump(Swoole\Timer::info($timer_id));
}
```
### stats()

定時器の状態を確認します。

!> Swooleバージョン >= `v4.4.0` で利用可能

```php
Swoole\Timer::stats(): array
```

  * **Return Value**

```php
array(3) {
  ["initialized"]=>
  bool(true)
  ["num"]=>
  int(1000)
  ["round"]=>
  int(1)
}
```
### set()

Setting timer-related parameters.

```php
Swoole\Timer::set(array $array): void
```

!> This method is marked as deprecated from version `v4.6.0`.
## クローズコルーチン :id=close-timer-co

デフォルトでは、タイマーはコールバック関数を実行する際に自動的にコルーチンを作成しますが、タイマーのコルーチンを個別に閉じることもできます。

```php
swoole_async_set([
  'enable_coroutine' => false,
]);
```

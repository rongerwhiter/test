＃ ロックのないプロセス間カウンター Atomic

`Atomic`は`Swoole`の内部で提供されるアトミックなカウント操作クラスで、整数のロックのないアトミックな増減を簡単に行うことができます。

* 共有メモリを使用して、異なるプロセス間でカウントを操作できます。
* `gcc/clang`が提供する`CPU`アトミック命令をベースにしており、ロックが不要です。
* サーバープログラム内では、`Server->start`の前に作成する必要があり、`Worker`プロセスで使用できます。
* デフォルトは`32`ビットの符号なし整数型ですが、`64`ビットの符号付き整数型が必要な場合は、`Swoole\Atomic\Long`を使用できます。

!> [onReceive](/server/events?id=onreceive)などのコールバック関数内でカウンターを作成しないでください。そうしないと、メモリが継続的に増加し、メモリリークが発生します。

!> `64`ビットの符号付きロング整数のアトミックなカウンターをサポートしており、`new Swoole\Atomic\Long`を使用して作成する必要があります。`Atomic\Long`は`wait`および`wakeup`メソッドをサポートしていません。
```php
$atomic = new Swoole\Atomic();

$serv = new Swoole\Server('127.0.0.1', '9501');
$serv->set([
    'worker_num' => 1,
    'log_file' => '/dev/null'
]);
$serv->on("start", function ($serv) use ($atomic) {
    if ($atomic->add() == 2) {
        $serv->shutdown();
    }
});
$serv->on("ManagerStart", function ($serv) use ($atomic) {
    if ($atomic->add() == 2) {
        $serv->shutdown();
    }
});
$serv->on("ManagerStop", function ($serv) {
    echo "shutdown\n";
});
$serv->on("Receive", function () {
    
});
$serv->start();
```
```python
def greet(name):
    return f'Hello, {name}!'
```

### __construct()

Constructeur. Crée un objet de comptage atomique.

```php
Swoole\Atomic::__construct(int $init_value = 0);
```

  * **Paramètres** 

    * **`int $init_value`**
      * **Fonction**：spécifie la valeur d'initialisation
      * **Valeur par défaut**：`0`
      * **Autres valeurs**：aucune

!> -`Atomic` ne peut fonctionner qu'avec des entiers non signés de `32` bits, supportant jusqu'à `4,2` milliards, et ne prend pas en charge les nombres négatifs ;  
- Pour utiliser un compteur atomique dans un `Server`, il doit être créé avant l'appel à `Server->start` ;  
- Pour utiliser un compteur atomique dans un [Process](/process/process), il doit être créé avant l'appel à `Process->start`.
### add()

計数を増やす。

```php
Swoole\Atomic->add(int $add_value = 1): int
```

  * **引数** 

    * **`int $add_value`**
      * **目的**：増やす値【正の整数である必要があります】
      * **デフォルト値**：`1`
      * **その他の値**：なし

  * **返り値**

    * `add`メソッドが成功した場合は、結果の数値を返す

!> オリジナルの値に加算した結果が`42`億を超えると、オーバーフローし、上位の数値が破棄されます。
### sub()

カウントを減らします。

```php
Swoole\Atomic->sub(int $sub_value = 1): int
```

* **Parameters**

  * **`int $sub_value`**
    * **Description**: 減らす値【正の整数でなければなりません】
    * **Default**: `1`
    * **Other values**: なし

* **Return value**

  * `sub` メソッドが成功した場合、結果の数値を返します

!> 元の値から減算すると0未満になる場合、オーバーフローが発生し、桁が破棄されます。
### get()

現在のカウンタ値を取得します。

```php
Swoole\Atomic->get(): int
```

  * **Return Value**

    * 現在の数値を返します。
### set()

指定した数値に現在の値を設定します。

```php
Swoole\Atomic->set(int $value): void
```

  * **パラメータ** 

    * **`int $value`**
      * **機能**：設定したい目標数値を指定します
      * **デフォルト値**：なし
      * **他の値**：なし
### cmpset()

もし現在の値がパラメーター`1`と等しい場合、現在の値をパラメーター`2`に設定します。

```php
Swoole\Atomic->cmpset(int $cmp_value, int $set_value): bool
```

  * **パラメーター**

    * **`int $cmp_value`**
      * **機能**：現在の数値が`$cmp_value`と等しい場合は`true`を返し、現在の数値を`$set_value`に設定します。等しくない場合は`false`を返します【42億未満の整数である必要があります】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $set_value`**
      * **機能**：現在の数値が`$cmp_value`と等しい場合は`true`を返し、現在の数値を`$set_value`に設定します。等しくない場合は`false`を返します【42億未満の整数である必要があります】
      * **デフォルト値**：なし
      * **その他の値**：なし
### wait()

設定為等待狀態。

!> 當原子計數的值為0時程序進入等待狀態。另外一個進程調用`wakeup`可以再次喚醒程序。底層基於`Linux Futex`實現，使用此特性，可以僅用`4`字節內存實現一個等待、通知、鎖的功能。在不支持`Futex`的平臺下，底層會使用循環`usleep(1000)`模擬實現。

```php
Swoole\Atomic->wait(float $timeout = 1.0): bool
```

  * **參數** 

    * **`float $timeout`**
      * **功能**：指定超時時間【設置為`-1`時表示永不超時，會持續等待直到有其他進程喚醒】
      * **值單位**：秒【支持浮點型，如`1.5`表示`1s`+`500ms`】
      * **默認值**：`1`
      * **其他值**：無

  * **返回值** 

    * 超時返回`false`，錯誤碼為`EAGAIN`，可使用`swoole_errno`函數獲取
    * 成功返回`true`，表示有其他進程通過`wakeup`成功喚醒了當前的鎖

  * **協程環境**

  `wait`會阻塞整個進程而不是協程，因此請勿在協程環境中使用`Atomic->wait()`避免引起進程掛起。

!> -使用`wait/wakeup`特性時，原子計數的值只能為`0`或`1`，否則會導致無法正常使用；  
-當然原子計數的值為`1`時，表示不需要進入等待狀態，資源當前就是可用。`wait`函數會立即返回`true`。

  * **使用示例**

    ```php
    $n = new Swoole\Atomic;
    if (pcntl_fork() > 0) {
        echo "master start\n";
        $n->wait(1.5);
        echo "master end\n";
    } else {
        echo "child start\n";
        sleep(1);
        $n->wakeup();
        echo "child end\n";
    }
    ```
### wakeup()

wait状態にある他のプロセスを起こします。

```php
Swoole\Atomic->wakeup(int $n = 1): bool
```

  * **パラメータ** 

    * **`int $n`**
      * **機能**：起こすプロセスの数
      * **デフォルト値**：なし
      * **その他の値**：なし

* 現在のアトミックカウントが`0`の場合、`wait`しているプロセスがいないことを意味し、`wakeup`は即座に`true`を返します。
* 現在のアトミックカウントが`1`の場合、現在`wait`しているプロセスがあることを意味し、`wakeup`は待機中のプロセスを起こし、`true`を返します。
* 起こされたプロセスが戻ると、アトミックカウントが`0`に設定され、再び`wait`している他のプロセスを起こすために`wakeup`を呼び出すことができます。

# プロセス間のロック

`PHP`コードでは、データ同期を実現するために簡単にロックを作成することができます。`Lock`クラスは`5`種類のロックタイプをサポートします。

| ロックタイプ | 説明 |
| --- | --- |
| SWOOLE_MUTEX | ミューテックスロック |
| SWOOLE_RWLOCK | 読み書きロック |
| SWOOLE_SPINLOCK | スピンロック |
| SWOOLE_FILELOCK | ファイルロック(廃止) |
| SWOOLE_SEM | セマフォ(廃止) |

!> [onReceive](/server/events?id=onreceive)などのコールバック関数内でロックを作成しないでください。そうしないとメモリが継続的に増加し、メモリリークが発生します。

## 使用例

```php
$lock = new Swoole\Lock(SWOOLE_MUTEX);
echo "[Master]create lock\n";
$lock->lock();
if (pcntl_fork() > 0)
{
  sleep(1);
  $lock->unlock();
} 
else
{
  echo "[Child] Wait Lock\n";
  $lock->lock();
  echo "[Child] Get Lock\n";
  $lock->unlock();
  exit("[Child] exit\n");
}
echo "[Master]release lock\n";
unset($lock);
sleep(1);
echo "[Master]exit\n";
```

## 警告

!> コルーチン内ではロックを使用できません。`lock`および`unlock`の間にコルーチンの切り替えを引き起こす可能性のある`API`を使用しないでください。

### エラーの例

!> このコードはコルーチンモード下で`100%`デッドロックします。[この記事](https://course.swoole-cloud.com/article/2)を参照してください。

```php
$lock = new Swoole\Lock();
$c = 2;

while ($c--) {
  go(function() use ($lock) {
      $lock->lock();
      Co::sleep(1);
      $lock->unlock();
  });
}
```

## メソッド

### __construct()

コンストラクタ。

```php
Swoole\Lock::__construct(int $type = SWOOLE_MUTEX, string $lockfile = '');
```

!> ロックオブジェクトの繰り返し作成/破棄は避けてください。そうしないとメモリリークが発生します。

  * **パラメータ**

    * **`int $type`**
      * **機能**：ロックのタイプ
      * **デフォルト値**：`SWOOLE_MUTEX`【ミューテックスロック】
      * **その他の値**：なし

    * **`string $lockfile`**
      * **機能**：ファイルロックのパスを指定する【`SWOOLE_FILELOCK`タイプの場合は必須】
      * **デフォルト値**：なし
      * **その他の値**：なし

!> 各タイプのロックは異なるメソッドをサポートしています。読み書きロック、ファイルロックなどは`$lock->lock_read()`をサポートできます。また、ファイルロック以外のタイプのロックは親プロセス内で作成する必要があります。これにより`fork`された子プロセス間でロックを競争させることができます。

### lock()

ロックを取得します。ほかのプロセスがロックを保持している場合は、ここでブロックされます。保持しているプロセスが`unlock()`でロックを解放するまでです。

```php
Swoole\Lock->lock(): bool
```

### trylock()

ロックを取得します。`lock`メソッドと異なり、`trylock()`はブロックされず、即座に戻ります。

```php
Swoole\Lock->trylock(): bool
```

  * **戻り値**

    * ロックが成功すると`true`が返され、この時点で共有変数を変更できます
    * ロックが失敗すると`false`が返され、他のプロセスがロックを保持していることを示します

!> `SWOOlE_SEM` セマフォは`trylock`メソッドを持っていません

### unlock()

ロックを解放します。

```php
Swoole\Lock->unlock(): bool
```

### lock_read()

読み取り専用のロックを取得します。

```php
Swoole\Lock->lock_read(): bool
```

* 読み取りロックを取得している間、他のプロセスは引き続き読み取りロックを取得し、読み取り操作を続行できます。
* ただし、`$lock->lock()`または`$lock->trylock()`はできません。これらのメソッドは排他ロックを取得し、排他ロックを取得しているときは他のプロセスがロックを取得できなくなります。
* 別のプロセスが排他ロックを取得(つまり`$lock->lock()`/`$lock->trylock()`を呼び出す)した場合、`$lock->lock_read()`はブロックし、排他ロックを保持しているプロセスがロックを解放するまで待機します。

!> `SWOOLE_RWLOCK`および`SWOOLE_FILELOCK`タイプのロックのみが読み取り専用ロックをサポートしています。

### trylock_read()

ロックを取得します。このメソッドは`lock_read()`と同じですが、ブロックされません。

```php
Swoole\Lock->trylock_read(): bool
```

!> 呼び出しは即座に戻ります。ロックを取得したかどうかを確認するために戻り値を確認する必要があります。

### lockwait()

ロックを取得します。`lock()`メソッドと同じように機能しますが、`lockwait()`ではタイムアウトを設定できます。

```php
Swoole\Lock->lockwait(float $timeout = 1.0): bool
```

  * **パラメータ** 

    * **`float $timeout`**
      * **機能**：タイムアウト時間を指定します
      * **単位**：秒【浮動小数点数もサポートされます。たとえば`1.5`は`1s`+`500ms`を表します】
      * **デフォルト値**：`1`
      * **その他の値**：なし

  * **戻り値**

    * 指定した時間内にロックを取得できない場合、`false`が返されます
    * ロックが成功すると`true`が返されます

!> `Mutex`タイプのロックのみが`lockwait`をサポートします。

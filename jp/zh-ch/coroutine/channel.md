```kotlin
import kotlinx.coroutines.channels.Channel

val channel = Channel<Int>()

// Send data to the channel
launch {
    repeat(5) {
        channel.send(it * it)
    }
    channel.close()
}

// Receive data from the channel
launch {
    for (element in channel) {
        println("Received: $element")
    }
}
```

このサンプルコードは、`Channel`を使って単方向のデータ通信を実装しています。1つのコルーチンでデータをチャネルに送信し、別のコルーチンでそれを受信して出力しています。データ送信後には`close()`を呼び出してチャネルを閉じます。
## 実装原理

  * チャネルは`PHP`の`Array`に似ており、メモリのみを使用し、他に追加のリソースを必要とせず、すべての操作がメモリ操作であり、`IO`コストはありません。
  * バックエンドは`PHP`の参照カウントを使用しており、メモリのコピーはありません。巨大な文字列や配列を渡しても追加のパフォーマンスコストは発生しません。
  * `channel`は参照カウントに基づいており、ゼロコピーです。
```php
use Swoole\Coroutine;
use Swoole\Coroutine\Channel;
use function Swoole\Coroutine\run;

run(function(){
    $channel = new Channel(1);
    Coroutine::create(function () use ($channel) {
        for($i = 0; $i < 10; $i++) {
            Coroutine::sleep(1.0);
            $channel->push(['rand' => rand(1000, 9999), 'index' => $i]);
            echo "{$i}\n";
        }
    });
    Coroutine::create(function () use ($channel) {
        while(1) {
            $data = $channel->pop(2.0);
            if ($data) {
                var_dump($data);
            } else {
                assert($channel->errCode === SWOOLE_CHANNEL_TIMEOUT);
                break;
            }
        }
    });
});
```  
## Methods
```php
Swoole\Coroutine\Channel::__construct(int $capacity = 1)
```

* **Parameters**

  * **`int $capacity`**
     * **Function**: Set the capacity [must be an integer greater than or equal to `1`]
     * **Default value**: `1`
     * **Other values**: None

!> Underlying, PHP reference counting is used to save variables. The buffer area only needs to occupy `$capacity * sizeof(zval)` bytes of memory. In `PHP7`, `zval` is `16` bytes. For example, when `$capacity = 1024`, `Channel` will occupy a maximum of `16K` memory.

!> When used in `Server`, it must be created after [onWorkerStart](/server/events?id=onworkerstart).
### push()

データをチャネルに書き込みます。

```php
Swoole\Coroutine\Channel->push(mixed $data, float $timeout = -1): bool
```

  * **パラメータ** 

    * **`mixed $data`**
      * **機能**：データをプッシュします 【無名関数やリソースを含む任意のPHP変数が可能です】
      * **デフォルト値**：なし
      * **その他の値**：なし

      !> 曖昧さを避けるため、`0`、`false`、`空文字列`、`null`などの空のデータをチャネルに書き込まないでください

    * **`float $timeout`**
      * **機能**：タイムアウト時間を設定します
      * **単位**：秒【浮動小数点数をサポートし、例：`1.5` は `1秒` + `500ミリ秒` を表します】
      * **デフォルト値**：`-1`
      * **その他の値**：なし
      * **バージョン影響**：Swooleバージョン >= v4.2.12

      !> チャネルがいっぱいの場合、`push`は現在のコルーチンを一時停止し、指定された時間内に消費者がデータを消費しない場合、タイムアウトが発生し、バックグラウンドで現在のコルーチンが再開され、`push`呼び出しが即座に`false`を返し、書き込みに失敗します

  * **戻り値**

    * 成功した場合は`true`を返します
    * チャンネルが閉じられている場合は、失敗した場合は`false`を返し、`$channel->errCode`を使用してエラーコードを取得できます

  * **拡張**

    * **チャネルがいっぱいの場合**

      * 現在のコルーチンを自動的に一時停止し、他の消費者コルーチンがデータを`pop`すると、チャネルが書き込み可能になり、現在のコルーチンが再開されます
      * 複数の生産者コルーチンが同時に`push`を行う場合、ベースレイヤーは自動的にキューイングを行い、これらの生産者コルーチンを順番に再開します

    * **チャネルが空の場合**

      * 消費者コルーチンの1つを自動的に起動します
      * 複数の消費者コルーチンが同時に`pop`する場合、ベースレイヤーは自動的にキューイングを行い、これらの消費者コルーチンを順番に再開します

!> `Coroutine\Channel`はローカルメモリを使用し、異なるプロセス間でメモリは分離されています。 `push`および`pop`操作は同じプロセス内の異なるコルーチンでのみ実行できます
### pop()

データをチャネルから読み取ります。

```php
Swoole\Coroutine\Channel->pop(float $timeout = -1): mixed
```

  * **パラメータ** 

    * **`float $timeout`**
      * **機能**：タイムアウト時間を設定します
      * **単位**：秒【フロート値もサポートします。例：`1.5`は`1s`+`500ms`を表します】
      * **デフォルト値**：`-1`【タイムアウトしないことを示します】
      * **その他の値**：なし
      * **影響するバージョン**：Swoole バージョン >= v4.0.3

  * **戻り値**

    * 戻り値は、PHP変数の任意の型になります。匿名関数やリソースも含まれます。
    * チャネルが閉じられている場合、失敗時は`false`が返されます。

  * **拡張**

    * **チャネルが満杯の場合**

      * `pop`でデータを消費すると、自動的に1つのプロデューサコルーチンが起こされ、新しいデータを書き込めるようになります。
      * 複数のプロデューサコルーチンが同時に`push`を行う場合、順番に待ち行列が作成され、それらのプロデューサコルーチンが1つずつ`resume`されます。

    * **チャネルが空の場合**

      * 自動的に現在のコルーチンを`yield`し、他のプロデューサコルーチンが`push`でデータを生産すると、チャネルが読み取れるようになり、現在のコルーチンが再び`resume`されます。
      * 複数のコンシューマコルーチンが同時に`pop`する場合、順番に待ち行列が作成され、それらのコンシューマコルーチンが1つずつ`resume`されます。
### stats()

通道の状態を取得します。

```php
Swoole\Coroutine\Channel->stats(): array
```

  * **戻り値**

    配列を返します。バッファ付きチャネルは`4`つの情報が含まれ、バッファなしチャネルは`2`つの情報が返されます。

    - `consumer_num` 消費者の数。現在のチャネルが空であり、`push`メソッドを待っているコルーチンが`N`個存在することを示します。
    - `producer_num` 生産者の数。現在のチャネルが一杯であり、`pop`メソッドを待っているコルーチンが`N`個存在することを示します。
    - `queue_num` チャネル内の要素数

```php
array(
  "consumer_num" => 0,
  "producer_num" => 1,
  "queue_num" => 10
);
```
### close()

チャネルを閉じます。また、読み書きを待っているすべてのコルーチンを起こします。

```php
Swoole\Coroutine\Channel->close(): bool
```

!> すべての生産者コルーチンを起こし、`push`メソッドが`false`を返します；すべての消費者コルーチンを起こし、`pop`メソッドが`false`を返します。
### length()

チャネル内の要素数を取得します。

```php
Swoole\Coroutine\Channel->length(): int
``` 
### isEmpty()

現在のチャネルが空かどうかを判定します。

```php
Swoole\Coroutine\Channel->isEmpty(): bool
```
### isFull()

現在のチャネルが満杯かどうかを判定します。

```php
Swoole\Coroutine\Channel->isFull(): bool
```
## Properties
### capacity

Channelのバッファ容量です。

[construct](/coroutine/channel?id=__construct)メソッドで設定された容量がここに保存されますが、**設定された容量が1未満の場合**、この変数の値は1になります。

```php
Swoole\Coroutine\Channel->capacity: int
```
### errCode

エラーコードを取得します。

```php
Swoole\Coroutine\Channel->errCode: int
```

  * **Return Values**

Value | Corresponding Constant | Description
---|---|---
0 | SWOOLE_CHANNEL_OK | デフォルトの成功
-1 | SWOOLE_CHANNEL_TIMEOUT | タイムアウト時のpop失敗 (タイムアウト)
-2 | SWOOLE_CHANNEL_CLOSED | チャンネルが閉じられている場合、チャンネル操作を続行しない

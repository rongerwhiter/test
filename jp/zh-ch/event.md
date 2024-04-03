# イベント

`Swoole`拡張機能には、直接`epoll/kqueue`イベントループを操作するためのインターフェースも提供されています。他の拡張機能で作成されたソケットや、`PHP`コードで作成された`stream/socket`拡張機能のソケットを、`Swoole`の[EventLoop](/learn?id=什么是eventloop)に組み込むことができます。
さもなければ、同期IOの第三者$fdをEventLoopに組み込むとSwooleのEventLoopが実行されなくなる可能性があります。[参考案例](/learn?id=同步io转换成异步io)。

!> `Event`モジュールはかなり低レベルで、`epoll`の初心者向けのラッパーであり、利用者はIOマルチプレキシングの経験がある方が望ましいです。
## イベントの優先度

1. `Process::signal` で設定されたシグナル処理コールバック関数
2. `Timer::tick` および `Timer::after` で設定されたタイマーコールバック関数
3. `Event::defer` で設定されたディレイ実行関数
4. `Event::cycle` で設定されたサイクルコールバック関数
このセクションでは、私たちの新しいプロジェクトを計画するための方法について説明します。
### add()

`socket`を低レベルの`reactor`イベントリスナーに追加します。この関数は`Server`または`Client`モードで使用できます。
```php
Swoole\Event::add(mixed $sock, callable $read_callback, callable $write_callback = null, int $flags = null): bool
```

!> `Server` プログラムで使用する場合、`Worker`プロセスの起動後に使用する必要があります。`Server::start` の前に非同期 `IO` インターフェースを呼び出してはいけません

* **パラメータ**

  * **`mixed $sock`**
    * **役割**：ファイルディスクリプタ、`stream` リソース、`sockets` リソース、`object`
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`callable $read_callback`**
    * **役割**：読み込み可能なイベントのコールバック関数
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`callable $write_callback`**
    * **役割**：書き込み可能なイベントのコールバック関数【このパラメータは文字列関数名、オブジェクト+メソッド、クラスの静的メソッド、または無名関数である必要があります。`socket`が読み込み可能または書き込み可能になったときに指定した関数を呼び出します。】
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $flags`**
    * **役割**：イベントタイプのマスク【読み込み可能なイベント、書き込み可能なイベントをオン/オフにすることができます。たとえば `SWOOLE_EVENT_READ`、`SWOOLE_EVENT_WRITE` または `SWOOLE_EVENT_READ|SWOOLE_EVENT_WRITE`】
    * **デフォルト値**：なし
    * **その他の値**：なし

* **$sockの4つの型**

タイプ | 説明
---|---
int | ファイルディスクリプタ、`Swoole\Client->$sock`、`Swoole\Process->$pipe` または他の `fd`
stream リソース | `stream_socket_client`/`fsockopen` で作成されたリソース
ソケットリソース | `sockets`拡張機能で作成された`socket_create`で作成されたリソースは、[./configure --enable-sockets](/environment?id=Compile_options)をコンパイル時に追加する必要があります。
オブジェクト | `Swoole\Process`または`Swoole\Client`、これらは自動的に[UnixSocket](/learn?id=IPCとは)（`Process`）またはクライアント接続の`socket`（`Swoole\Client`）に変換されます。

* **Return Value**

  * イベントリスナーの追加に成功した場合は`true`が返されます。
  * 追加に失敗した場合は`false`が返され、エラーコードを取得するには`swoole_last_error`を使用してください。
  * 既に追加された`socket`は再度追加できず、`swoole_event_set`を使用して`socket`に対応するコールバック関数とイベントタイプを変更できます。

  !> `Swoole\Event::add`を使用して`socket`をイベントリスニングに追加すると、その`socket`が自動的に非ブロッキングモードに設定されます。

* **使用例**

```php
$fp = stream_socket_client("tcp://www.qq.com:80", $errno, $errstr, 30);
fwrite($fp,"GET / HTTP/1.1\r\nHost: www.qq.com\r\n\r\n");

Swoole\Event::add($fp, function($fp) {
    $resp = fread($fp, 8192);
    //ソケット処理が完了したら、epollイベントからソケットを削除
    Swoole\Event::del($fp);
    fclose($fp);
});
echo "Finish\n";  //Swoole\Event::addはプロセスをブロックしません。この行のコードは順番に実行されます。
```

* **コールバック関数**

  * 可読状態(`$read_callback`)のイベントコールバック関数では、`fread`、`recv`などの関数を使用して`socket`のバッファ内のデータを読み取る必要があります。そうしない場合、イベントが続けて発生します。続きの読み取りを望まない場合は、`Swoole\Event::del`を使用してイベントリスニングを削除する必要があります。
* 在可写`($write_callback)`事件回调函数中，写入`socket`之后必须调用`Swoole\Event::del`移除事件监听，否则可写事件会持续触发
  * 执行`fread`、`socekt_recv`、`socket_read`、`Swoole\Client::recv`返回`false`，并且错误码为`EAGAIN`时表示当前`socket`接收缓存区内没有任何数据，これ時需要加入可读监听等待[EventLoop](/learn?id=什么是eventloop)通知
  * 执行`fwrite`、`socket_write`、`socket_send`、`Swoole\Client::send`操作返回`false`，并且错误码为`EAGAIN`时表示当前`socket`发送缓存区已满，暂时不能发送数据。需要监听可写事件等待[EventLoop](/learn?id=什么是eventloop)通知
```php
Swoole\Event::add(mixed $sock, callable $read_callback, callable $write_callback = null, int $flags = null): bool
```  
ソケットリソース | `sockets`拡張機能で作成された`socket_create`のリソースは、[./configure --enable-sockets](/environment?id=Compilation options)をコンパイル時に追加する必要があります
オブジェクト | `Swoole\Process`または`Swoole\Client`は、バックエンドで自動的に[UnixSocket](/learn?id=IPC)（`Process`）またはクライアント接続の`socket`（`Swoole\Client`）に変換されます

* **Return Value**

  * イベントリスナーの追加に成功すると`true`が返されます
  * 失敗した場合は`false`が返され、エラーコードを取得するには`swoole_last_error`を使用してください
  * 既に追加された`socket`は重複して追加することはできません。`swoole_event_set`を使用して`socket`に関連付けられたコールバック関数とイベントタイプを変更できます

  !> `Swoole\Event::add`を使用して`socket`をイベントリスナーに追加すると、バックエンドで自動的にその`socket`がノンブロッキングモードに設定されます

* **Example**

```php
$fp = stream_socket_client("tcp://www.qq.com:80", $errno, $errstr, 30);
fwrite($fp,"GET / HTTP/1.1\r\nHost: www.qq.com\r\n\r\n");

Swoole\Event::add($fp, function($fp) {
    $resp = fread($fp, 8192);
    //ソケット処理が完了したら、epollイベントからソケットを削除
    Swoole\Event::del($fp);
    fclose($fp);
});
echo "Finish\n";  //Swoole\Event::addはプロセスをブロックせず、この行のコードが逐次実行されます
```

* **Callback Function**

  * 可読(`$read_callback`)イベントコールバック関数では、必ず`fread`や`recv`などの関数を使用して`socket`のバッファからデータを読み取る必要があります。それ以外の場合、イベントが続けてトリガーされます。データの継続的な読み取りを望まない場合は、`Swoole\Event::del`を使用してイベントリスニングを削除する必要があります
- 可書き`($write_callback)` イベントコールバック内でソケットに書き込んだ後、`Swoole\Event::del` を呼び出してイベントリスニングを削除する必要があります。そうしないと、可書きイベントが継続してトリガーされます。
  - `fread`、`socekt_recv`、`socket_read`、`Swoole\Client::recv` が `false` を返し、エラーコードが `EAGAIN` の場合、現在のソケット受信バッファ内にデータがないことを示します。この場合は、読み取り可能な通知を待つために[EventLoop](/learn?id=什么是eventloop)にリスンする必要があります。
  - `fwrite`、`socket_write`、`socket_send`、`Swoole\Client::send` の操作が `false` を返し、エラーコードが `EAGAIN` の場合、現在のソケット送信バッファがいっぱいであり、データを送信できないことを示します。可書きイベントをリスンして[EventLoop](/learn?id=什么是eventloop)の通知を待つ必要があります。
### set()

イベントリスナーのコールバックとマスクを変更します。

```php
Swoole\Event::set($fd, mixed $read_callback, mixed $write_callback, int $flags): bool
```

* **パラメータ**

  * パラメータは[Event::add](/event?id=add)と完全に同じです。渡された`$fd`が[EventLoop](/learn?id=什么是eventloop)内に存在しない場合は`false`を返します。
  * `$read_callback`が`null`でない場合、指定された関数に読み取り可能なイベントコールバックを変更します。
  * `$write_callback`が`null`でない場合、指定された関数に書き込み可能なイベントコールバックを変更します。
  * `$flags`は、読み取り可能（`SWOOLE_EVENT_READ`）および書き込み可能（`SWOOLE_EVENT_WRITE`）なイベントのリスニングを開/閉可能です。

  !> `SWOOLE_EVENT_READ`イベントがリスンされ、かつ現在`read_callback`が設定されていない場合、成功しないため、直ちに`false`が返されます。`SWOOLE_EVENT_WRITE`でも同様です。

* **状態変更**

  * `Event::add`または`Event::set`を使用して、読み取り可能なイベントコールバックを設定しましたが、`SWOOLE_EVENT_READ`の読み取り可能なイベントをリスンしていない場合、下層はコールバック関数の情報のみを保存し、イベントコールバックは発生しません。
  * `Event::set($fd, null, null, SWOOLE_EVENT_READ | SWOOLE_EVENT_WRITE)`を使用して、変更されたイベントタイプをリスンすることができます。この場合、下層は読み取り可能なイベントが発生します。

* **コールバック関数の解放**

!> `Event::set`はコールバック関数だけを交換できますが、イベントコールバック関数を解放することはできません。例：`Event::set($fd, null, null, SWOOLE_EVENT_READ | SWOOLE_EVENT_WRITE)`のように、引数に`read_callback`および`write_callback`を`null`で渡すと、`Event::add`に設定されたコールバック関数が変更されないことを示し、イベントコールバック関数が`null`に設定されるのではないことにご注意ください。
`Event::del`メソッドを呼び出してイベントリスナーを削除しない限り、下層は`read_callback`と`write_callback`のイベントコールバック関数を解放しません。
### isset()

渡された`$fd`がイベントリスナーに追加されているかどうかをチェックします。

```php
Swoole\Event::isset(mixed $fd, int $events = SWOOLE_EVENT_READ | SWOOLE_EVENT_WRITE): bool
```

* **Parameters（パラメータ）**

  * **`mixed $fd`**
    * **説明**： 任意のソケットファイル記述子【参照：[Event::add](/event?id=add) ドキュメント】
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $events`**
    * **説明**： チェックするイベントの種類
    * **デフォルト値**：なし
    * **その他の値**：なし

* **$events**

イベントタイプ | 説明
---|---
`SWOOLE_EVENT_READ` | 読み取り可能なイベントをリスニングしているかどうか
`SWOOLE_EVENT_WRITE` | 書き込み可能なイベントをリスニングしているかどうか
`SWOOLE_EVENT_READ \| SWOOLE_EVENT_WRITE` | 読み取り可能または書き込み可能なイベントをリスニングしているかどうか

* **使用例**

```php
use Swoole\Event;

$fp = stream_socket_client("tcp://www.qq.com:80", $errno, $errstr, 30);
fwrite($fp,"GET / HTTP/1.1\r\nHost: www.qq.com\r\n\r\n");

Event::add($fp, function($fp) {
    $resp = fread($fp, 8192);
    Swoole\Event::del($fp);
    fclose($fp);
}, null, SWOOLE_EVENT_READ);
var_dump(Event::isset($fp, SWOOLE_EVENT_READ)); // trueを返します
var_dump(Event::isset($fp, SWOOLE_EVENT_WRITE)); // falseを返します
var_dump(Event::isset($fp, SWOOLE_EVENT_READ | SWOOLE_EVENT_WRITE)); // trueを返します
```
### write()

PHP組み込みの `stream/sockets` 拡張機能で作成されたソケットに対し、`fwrite/socket_send` 関数を使用して対向にデータを送信します。送信するデータ量が多い場合、ソケットの書き込みバッファがいっぱいになると、送信がブロックされたり、[EAGAIN](/other/errno?id=linux) エラーが返されます。

`Event::write` 関数を使用すると、`stream/sockets` リソースのデータ送信を**非同期**にできます。バッファがいっぱいになったり、[EAGAIN](/other/errno?id=linux) が返された場合、Swooleの内部メカニズムでデータを送信キューに追加し、書き込み可能になるのを監視します。ソケットが書き込み可能になると、Swooleの内部で自動的に書き込みが行われます。

```php
Swoole\Event::write(mixed $fd, miexd $data): bool
```

* **パラメータ**

  * **`mixed $fd`**
    * **機能**：任意のソケットファイル記述子【参照：[Event::add](/event?id=add) ドキュメント】
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`miexd $data`**
    * **機能**：送信するデータ【送信データの長さは `Socket` 書き込みバッファサイズを超えてはいけません】
    * **デフォルト値**：なし
    * **その他の値**：なし

!> `Event::write` は `SSL/TLS` などトンネル暗号化を使用している `stream/sockets` リソースでは使用できません  
`Event::write` 関数が成功すると、その `$socket` は自動的に非ブロッキングモードに設定されます。

* **使用例**

```php
use Swoole\Event;

$fp = stream_socket_client('tcp://127.0.0.1:9501');
$data = str_repeat('A', 1024 * 1024*2);

Event::add($fp, function($fp) {
     echo fread($fp);
});

Event::write($fp, $data);
```
持続的書き込みが行われる場合、接続先が読み取る速度が十分でないと、`SOCKET`のキャッシュ領域がいっぱいになる可能性があります。`Swoole`の内部ロジックでは、データが`SOCKET`に書き込まれるまで、データをメモリキャッシュ領域に保存し、書き込み可能なイベントが発生するまで待機します。

もしメモリキャッシュ領域もいっぱいになった場合、この時`Swoole`の内部では`pipe buffer overflow, reactor will block.`のエラーが発生し、ブロック状態に入ります。

!> キャッシュがいっぱいになると`false`が返り、これはアトミックな操作です。全ての書き込みが成功するか、全て失敗するかのいずれかが起こります。
### del()

`socket`のリスナーを`reactor`から削除します。 `Event::del`は`Event::add`とペアで使用する必要があります。

```php
Swoole\Event::del(mixed $sock): bool
```

!> `Event::del`を使用してイベントリスナーを削除する場合、`socket`の`close`操作の前に行う必要があります。そうでないと、メモリリークが発生する可能性があります。

* **パラメーター**

  * **`mixed $sock`**
    * **機能**: `socket`のファイル記述子
    * **デフォルト値**: なし
    * **その他の値**: なし
```php
Swoole\Event::exit(): void
``` 

### exit()

イベントループを終了します。

!> この関数は`Client`プログラム内でのみ有効です
```
### defer()

次のイベントループが開始される時に関数を実行します。

```php
Swoole\Event::defer(mixed $callback_function);
```

!> `Event::defer`のコールバック関数は現在の`EventLoop`のイベントループが終了し、次のイベントループが開始される前に実行されます。

* **パラメータ** 

  * **`mixed $callback_function`**
    * **機能**：指定された時間後に実行される関数【呼び出し可能である必要があります。コールバック関数は引数を受け取らず、無名関数の`use`構文を使用して引数をコールバック関数に渡すことができます；`$callback_function`関数の実行中に新しい`defer`タスクを追加しても、まだイベントループ内で実行されます】
    * **デフォルト値**：なし
    * **その他の値**：なし

* **使用例**

```php
Swoole\Event::defer(function(){
    echo "After EventLoop\n";
});
```
### cycle()

イベントループの周期的な実行関数を定義します。この関数は、各イベントループの終了時に呼び出されます。

```php
Swoole\Event::cycle(callable $callback, bool $before = false): bool
```

* **パラメータ** 

  * **`callable $callback_function`**
    * **機能**：設定するコールバック関数 【`$callback`が`null`の場合、`cycle`関数をクリアします。すでに`cycle`関数が設定されている場合、再設定すると前回の設定が上書きされます】
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`bool $before`**
    * **機能**：[EventLoop](/learn?id=什么是eventloop)の前にこの関数を呼び出します
    * **デフォルト値**：なし
    * **その他の値**：なし

!> `before=true`と`before=false`の2つのコールバック関数が同時に存在できます。

  * **使用例**

```php
Swoole\Timer::tick(2000, function ($id) {
    var_dump($id);
});

Swoole\Event::cycle(function () {
    echo "hello [1]\n";
    Swoole\Event::cycle(function () {
        echo "hello [2]\n";
        Swoole\Event::cycle(null);
    });
});
```
### wait()

イベントリスナーを起動します。

!> この関数をPHPプログラムの末尾に配置してください

```php
Swoole\Event::wait();
```

* **使用例**

```php
Swoole\Timer::tick(1000, function () {
    echo "hello\n";
});

Swoole\Event::wait();
```
### dispatch()

イベントリスナーを起動します。

!> `reactor->wait`操作を1回だけ実行し、`Linux`プラットフォームでは`epoll_wait`を1回手動で呼び出します。`Event::dispatch`と異なる点は、`Event::wait`は内部でループを維持していることです。

```php
Swoole\Event::dispatch();
```

* **使用例**

```php
while(true)
{
    Event::dispatch();
}
```

この関数の目的は、`amp`などのフレームワークとの互換性を保つことです。これらのフレームワークでは、フレームワーク内で`reactor`のループを制御しており、`Event::wait`を使用すると、Swooleの内部で制御が維持され、フレームワークに制御を渡すことができません。

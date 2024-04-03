# Coroutine\Socket

`Swoole\Coroutine\Socket` モジュールは、[コルーチンスタイルのサーバー](/server/co_init)および[コルーチンクライアント](/coroutine_client/init)関連モジュールと比較して、より細かい`IO`操作を実現できる`Socket`です。

!> `Co\Socket`を使用してクラス名を簡略化することができます。このモジュールはかなり低レベルなため、ユーザーはソケットプログラミングの経験を持っていることが最適です。
```php
use Swoole\Coroutine;
use function Swoole\Coroutine\run;

run(function () {
    $socket = new Coroutine\Socket(AF_INET, SOCK_STREAM, 0);

    $retval = $socket->connect('127.0.0.1', 9601);
    while ($retval)
    {
        $n = $socket->send('hello');
        var_dump($n);

        $data = $socket->recv();
        var_dump($data);

        //発生したエラーまたは相手が接続を閉じた場合、この側も閉じる必要があります
        if ($data === '' || $data === false) {
            echo "errCode: {$socket->errCode}\n";
            $socket->close();
            break;
        }

        Coroutine::sleep(1.0);
    }

    var_dump($retval, $socket->errCode, $socket->errMsg);
});
```
## 协程调度

`Coroutine\Socket`モジュールが提供する`IO`操作のインターフェースはすべて同期プログラミングスタイルであり、下層では自動的に[协程调度](/coroutine?id=协程调度)器が使用され、[非同期IO](/learn?id=同步io异步io)が実現されます。
## エラーコード

`socket`関連のシステムコールを実行すると、-1のエラーが返される可能性があります。この場合、システムは`Coroutine\Socket->errCode`プロパティを`errno`というシステムエラー番号に設定します。該当する`man`ドキュメントを参照してください。たとえば、`$socket->accept()`がエラーを返した場合は、`errCode`の意味については`man accept`に記載されているエラーコードのドキュメントを参照してください。
```plaintext
There are no attributes to list.
```

## Atributos

```plaintext
No hay atributos que enumerar.
```
### fd

`socket`のファイル記述子`ID`
### errCode

エラーコード
## Methods
### __construct()

`Coroutine\Socket`オブジェクトを作成するためのコンストラクタ。

```php
Swoole\Coroutine\Socket::__construct(int $domain, int $type, int $protocol);
```

!> 詳細は`man socket`ドキュメントを参照してください。

  * **パラメータ** 

    * **`int $domain`**
      * **機能**：プロトコルドメイン【`AF_INET`、`AF_INET6`、`AF_UNIX`を使用可能】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $type`**
      * **機能**：タイプ【`SOCK_STREAM`、`SOCK_DGRAM`、`SOCK_RAW`を使用可能】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $protocol`**
      * **機能**：プロトコル【`IPPROTO_TCP`、`IPPROTO_UDP`、`IPPROTO_STCP`、`IPPROTO_TIPC`、`0`を使用可能】
      * **デフォルト値**：なし
      * **その他の値**：なし

!> コンストラクタは`socket`システムコールを使用して`socket`ハンドルを作成します。失敗した場合、`Swoole\Coroutine\Socket\Exception`例外がスローされます。そして、`$socket->errCode`プロパティが設定されます。このプロパティの値から、システムコールが失敗した理由を取得できます。
### getOption()

設定を取得します。

!> このメソッドは`getsockopt`システムコールに対応しており、詳細については`man getsockopt`ドキュメントを参照してください。  
このメソッドは`sockets`拡張の`socket_get_option`機能と同等であり、[PHPドキュメント](https://www.php.net/manual/zh/function.socket-get-option.php)を参照できます。

!> Swooleのバージョン >= v4.3.2

```php
Swoole\Coroutine\Socket->getOption(int $level, int $optname): mixed
```

  * **パラメータ** 

    * **`int $level`**
      * **機能**：オプションが存在するプロトコルレベルを指定します
      * **デフォルト値**：なし
      * **その他の値**：なし

      !> 例えば、ソケットレベルのオプションを取得するには、`SOL_SOCKET`の `level` パラメータを使用します。  
      他のレベルを使用するには、そのレベルのプロトコル番号を指定することができ、例えば`TCP`といったレベルを指定することができます。プロトコル番号は[getprotobyname](https://www.php.net/manual/zh/function.getprotobyname.php)関数で見つけることができます。

    * **`int $optname`**
      * **機能**：使用可能なソケットオプションは、[socket_get_option()](https://www.php.net/manual/zh/function.socket-get-option.php)関数のソケットオプションと同じです
      * **デフォルト値**：なし
      * **その他の値**：なし
### setOption()

設定選項。

!> このメソッドは`setsockopt`システムコールに対応しており、詳細については`man setsockopt`ドキュメントを参照してください。このメソッドは`sockets`拡張機能の`socket_set_option`機能と同等であり、[PHPドキュメント](https://www.php.net/manual/zh/function.socket-set-option.php)を参照してください。

!> Swooleバージョン >= v4.3.2

```php
Swoole\Coroutine\Socket->setOption(int $level, int $optname, mixed $optval): bool
```

  * **パラメータ**

    * **`int $level`**
      * **機能**：オプションが存在するプロトコルレベルを指定します。
      * **デフォルト値**：なし
      * **その他の値**：なし

      !> たとえば、ソケットレベルのオプションを取得するには、`SOL_SOCKET`の `level` パラメータが使用されます。  
      他のレベルを使用する場合は、そのレベルのプロトコル番号を指定することができます。たとえば、`TCP`を使用する場合は、[getprotobyname](https://www.php.net/manual/zh/function.getprotobyname.php) 関数を使用してプロトコル番号を見つけることができます。

    * **`int $optname`**
      * **機能**：利用可能なソケットオプションは[socket_get_option()](https://www.php.net/manual/zh/function.socket-get-option.php) 関数のソケットオプションと同様です。
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $optval`**
      * **機能**：オプションの値【`int`、`bool`、`string`、`array`のいずれかが可能。`level`と`optname` によって決まります。】
      * **デフォルト値**：なし
      * **その他の値**：なし
### setProtocol()

`socket`をプロトコル処理能力を取得し、`SSL`暗号化転送の有効化、[TCPデータパケットの境界問題](/learn?id=tcp数据包边界问题)の解決などを設定できます。

!> Swooleバージョン >= v4.3.2

```php
Swoole\Coroutine\Socket->setProtocol(array $settings): bool
```

  * **$settings サポートされるパラメータ**

パラメータ | タイプ
---|---
open_ssl | bool
ssl_cert_file | string
ssl_key_file | string
open_eof_check | bool
open_eof_split | bool
open_mqtt_protocol | bool
open_fastcgi_protocol | bool
open_length_check | bool
package_eof | string
package_length_type | string
package_length_offset | int
package_body_offset | int
package_length_func | callable
package_max_length | int

!> 上記のすべてのパラメータの意味は[Server->set()](/server/setting?id=open_eof_check)と完全に同じですので、ここでは説明しません。

  * **例**

```php
$socket->setProtocol([
    'open_length_check'     => true,
    'package_max_length'    => 1024 * 1024,
    'package_length_type'   => 'N',
    'package_length_offset' => 0,
    'package_body_offset'   => 4,
]);
```
### bind()

アドレスとポートをバインドします。

!> このメソッドには`IO`操作はなく、コルーチンの切り替えは発生しません

```php
Swoole\Coroutine\Socket->bind(string $address, int $port = 0): bool
```

  * **パラメータ** 

    * **`string $address`**
      * **機能**：バインドするアドレス【例：`0.0.0.0`、`127.0.0.1`】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $port`**
      * **機能**：バインドするポート【デフォルトは`0`で、システムが利用可能なポートをランダムにバインドし、システムが割り当てた`port`を取得するには[getsockname](/coroutine_client/socket?id=getsockname)メソッドを使用できます】
      * **デフォルト値**：`0`
      * **その他の値**：なし

  * **戻り値** 

    * バインドに成功した場合は`true`を返します
    * バインドに失敗した場合は`false`を返します。失敗原因は`errCode`プロパティをチェックしてください
### listen()

`Socket` をリッスンします。

!> このメソッドには`I/O`操作がなく、コルーチンの切り替えは発生しません

```php
Swoole\Coroutine\Socket->listen(int $backlog = 0): bool
```

  * **パラメータ** 

    * **`int $backlog`**
      * **機能**：リッスン待ち行列の長さ【デフォルトは`0`で、システムの低レベルで非同期`I/O`を実装しているため、ブロッキングは発生しないため`backlog`の重要性は高くありません】
      * **デフォルト値**：`0`
      * **その他の値**：なし

      !> アプリケーションにブロッキングまたは時間がかかるロジックがある場合、`accept`で接続が遅れると、新しい接続が`backlog`リッスンキューに堆積し、`backlog`の長さを超えると、サービスは新しい接続を拒否します

  * **戻り値** 

    * バインドに成功した場合は`true`を返します
    * バインドに失敗した場合は`false`を返します。失敗の原因は`errCode`プロパティをチェックしてください

  * **カーネルパラメータ** 

    `backlog`の最大値はカーネルパラメータ`net.core.somaxconn`によって制限されます。そして`Linux`では`sysctl`ユーティリティを使用してすべての`kernel`パラメータを動的に調整できます。動的な調整はカーネルパラメータの値を変更した後、即座に有効になります。ただし、この有効性は`OS`レベルに限定され、本当に有効にするにはアプリケーションを再起動する必要があります。`sysctl -a`コマンドを使用すると、すべてのカーネルパラメータとその値が表示されます。

    ```shell
    sysctl -w net.core.somaxconn=2048
    ```

    上記のコマンドはカーネルパラメータ`net.core.somaxconn`の値を`2048`に変更します。このような変更は即座に有効になりますが、マシンを再起動するとデフォルト値に戻ります。この変更を永続的に保持するためには、`/etc/sysctl.conf`を変更し、`net.core.somaxconn=2048`を追加してから`sysctl -p`コマンドを実行して有効にします。
### accept()

クライアントからの接続を受け入れます。

このメソッドを呼び出すと、現在のコルーチンがすぐに一時停止され、[EventLoop](/learn?id=什么是eventloop)に可読イベントをリッスンするために追加されます。`Socket`が読み取れるとクライアントの接続が到着し、このコルーチンが自動的に再開されて対応するクライアント接続の `Socket` オブジェクトが返されます。

!> このメソッドは`listen`メソッドの後に使用する必要があり、`Server`側に適用されます。

```php
Swoole\Coroutine\Socket->accept(float $timeout = 0): Coroutine\Socket|false;
```

  * **パラメータ** 

    * **`float $timeout`**
      * **機能**：タイムアウトを設定します【タイムアウトパラメータを設定すると、バックエンドでタイマーが設定され、指定された時間内にクライアント接続が到着しない場合、`accept`メソッドは`false`を返します】
      * **値の単位**：秒【浮動小数点数をサポートします。例：`1.5` は `1秒` + `500ms` を意味します】
      * **デフォルト値**：[クライアントのタイムアウトルール](/coroutine_client/init?id=超时规则)を参照
      * **その他の値**：なし

  * **戻り値** 

    * タイムアウトや`accept`システムコールでエラーが発生した場合は`false`が返され、`errCode`プロパティを使用してエラーコードを取得できます。タイムアウトエラーのコードは`ETIMEDOUT`です。
    * 成功した場合にはクライアント接続の `socket` が返されます。型は同じく `Swoole\Coroutine\Socket` オブジェクトです。`send`、`recv`、`close` などの操作を実行できます。

  * **例**

```php
use Swoole\Coroutine;
use function Swoole\Coroutine\run;

run(function () {
$socket = new Coroutine\Socket(AF_INET, SOCK_STREAM, 0);
$socket->bind('127.0.0.1', 9601);
$socket->listen(128);

    while(true) {
        echo "Accept: \n";
        $client = $socket->accept();
        if ($client === false) {
            var_dump($socket->errCode);
        } else {
            var_dump($client);
        }
    }
});
```
### connect()

目標サーバーに接続します。

このメソッドを呼び出すと、非同期の`connect`システムコールが開始され、現在のコルーチンが一時停止されます。低レベルでは書き込み可能な状態を監視し、接続が完了するか失敗した後でそのコルーチンを復元します。

このメソッドは`Client`側に適用され、`IPv4`、`IPv6`、[unixSocket](/learn?id=什么是IPC)をサポートしています。

```php
Swoole\Coroutine\Socket->connect(string $host, int $port = 0, float $timeout = 0): bool
```

  * **Parameters** 

    * **`string $host`**
      * **Description**：目標サーバーのアドレス【例：`127.0.0.1`、`192.168.1.100`、`/tmp/php-fpm.sock`、`www.baidu.com`など。`IP`アドレス、`Unix Socket`パス、またはドメイン名を指定することができます。ドメイン名の場合、低レベルでは自動的に非同期の`DNS`解決が行われ、ブロッキングは発生しません】
      * **Default**：なし
      * **Others**：なし

    * **`int $port`**
      * **Description**：目標サーバーのポート番号【`Socket`の`domain`が`AF_INET`、`AF_INET6`の場合、ポート番号を設定する必要があります】
      * **Default**：なし
      * **Others**：なし

    * **`float $timeout`**
      * **Description**：タイムアウト時間の設定【低レベルではタイマーが設定され、指定された時間内に接続が確立できない場合、`connect`は`false`を返します】
      * **Unit**：秒【浮動小数点数がサポートされており、例えば`1.5`は`1s`+`500ms`を表します】
      * **Default**：[クライアントのタイムアウトルール](/coroutine_client/init?id=超时规则)を参照
      * **Others**：なし

  * **Return Value** 

    * タイムアウトや`connect`システムコールのエラーが発生した場合は`false`を返し、`errCode`属性を使用してエラーコードを取得できます。タイムアウトエラーコードは`ETIMEDOUT`です。
    * 成功した場合は`true`を返します。
### checkLiveness()

システムコールを使用して接続が生きているかどうかをチェックします（異常切断の場合は無効で、対向が正常にクローズした接続の切断しか検知できません）

!> Swooleのバージョン >= `v4.5.0` で利用可能

```php
Swoole\Coroutine\Socket->checkLiveness(): bool
```

  * **戻り値** 

    * 接続が生存している場合は`true`を、それ以外の場合は`false`を返します
### send()

データをリモートに送信します。

!> `send`メソッドは、データを送信するために直ちに`send`システムコールを実行します。しかし、`send`システムコールがエラー`EAGAIN`を返すと、下層は自動的に書き込み可能なイベントを監視し、現在のコルーチンを一時停止させ、書き込み可能なイベントが発生したときにデータを再送信するために`send`システムコールを再実行し、そのコルーチンをアクティブにします。  

!> `send`が速すぎて`recv`が遅すぎると、最終的にはオペレーティングシステムのバッファがいっぱいになり、現在のコルーチンが`send`メソッドで一時停止する可能性があります。この問題を防ぐためには、適切なバッファサイズを増やすことができます。[/proc/sys/net/core/wmem_maxとSO_SNDBUF](https://stackoverflow.com/questions/21856517/whats-the-practical-limit-on-the-size-of-single-packet-transmitted-over-domain)

```php
Swoole\Coroutine\Socket->send(string $data, float $timeout = 0): int|false
```

  * **パラメータ** 

    * **`string $data`**
      * **機能**：送信するデータの内容【テキストまたはバイナリデータのどちらも可】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`float $timeout`**
      * **機能**：タイムアウト時間を設定します
      * **単位**：秒【浮動小数点数をサポートしており、例えば`1.5`は`1秒`+`500ms`を表します】
      * **デフォルト値**：[クライアントのタイムアウトルール](/coroutine_client/init?id=超時ルール)を参照
      * **その他の値**：なし

  * **戻り値** 

    * 送信に成功した場合は、書き込まれたバイト数が返されます。**実際に書き込まれたデータが`$data`パラメータの長さよりも小さい可能性がある**ので、アプリケーション層のコードは、返り値と`strlen($data)`が等しいかどうかを比較し、送信が完了したかどうかを判断する必要があります。
    * 送信に失敗した場合は`false`が返され、かつ`errCode`プロパティが設定されます。
### sendAll()

データを相手に送信します。`send`メソッドとの違いは、`sendAll`は可能な限りデータを完全に送信し、すべてのデータが正常に送信されるかエラーが発生するまで送信を中断する点です。

!> `sendAll`メソッドは、データを送信するために複数の`send`システムコールをすぐに実行します。`send`システムコールが`EAGAIN`エラーを返すと、その時点で低レベルで書き込み可能なイベントを監視し、現在のコルーチンを一時停止させ、書き込み可能なイベントがトリガーされるのを待って、データの送信が完了するかエラーが発生するまで、対応するコルーチンを再開させる動作を自動的に行います。 

!> Swooleのバージョン >= v4.3.0

```php
Swoole\Coroutine\Socket->sendAll(string $data, float $timeout = 0) : int | false;
```

  * **パラメータ** 

    * **`string $data`**
      * **機能**：送信するデータ内容【テキストまたはバイナリデータとして提供できます】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`float $timeout`**
      * **機能**：タイムアウト時間を設定します
      * **単位**：秒【浮動小数点数がサポートされており、例：`1.5`は`1s`+`500ms`を表します】
      * **デフォルト値**：[クライアントのタイムアウトルールを参照](/coroutine_client/init?id=超時ルール)
      * **その他の値**：なし

  * **戻り値** 

    * `sendAll`メソッドは、データがすべて正常に送信されることを保証しますが、`sendAll`の間に相手が接続を切断する可能性があります。この場合、一部のデータが正常に送信された場合、戻り値はその成功データの長さを返します。アプリケーションレベルのコードは、戻り値と`strlen($data)`が等しいかどうかを比較して、送信が完了したかどうかを判断し、ビジネス要件に応じて再送信が必要かどうかを判断する必要があります。
    * 送信に失敗した場合は`false`を返し、`errCode`プロパティを設定します
### peek()

データバッファー内のデータをのぞき見ます、システムコールの`recv(length, MSG_PEEK)`に相当します。

!> `peek`は即座に完了し、コルーチンを一時停止しませんが、1回のシステムコールにオーバーヘッドがあります

```php
Swoole\Coroutine\Socket->peek(int $length = 65535): string|false
```

  * **パラメータ** 

    * **`int $length`**
      * **機能**：のぞき見たデータをコピーするためのメモリサイズを指定します（注意：ここでメモリを割り当て、大きすぎる長さはメモリ不足を引き起こす可能性があります）
      * **値の単位**：バイト
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **戻り値** 

    * のぞき見が成功した場合はデータを返します
    * のぞき見に失敗した場合は`false`を返し、`errCode`属性を設定します
### recv()

データを受信します。

!> `recv`メソッドは現在のコルーチンを直ちに中断し、読み取り可能なイベントを監視し、対向先からデータを受信するのを待ちます。データが送信されて読み取り可能なイベントが発生した場合、`recv` システムコールを実行して `socket` バッファ内のデータを取得し、そのコルーチンを再開します。

```php
Swoole\Coroutine\Socket->recv(int $length = 65535, float $timeout = 0): string|false
```

* **Parameters**

  * **`int $length`**
    * **Description**: Specifies the amount of memory to use for receiving data (Note: memory will be allocated here, excessive length may cause memory exhaustion)
    * **Unit**: bytes
    * **Default**: None
    * **Other values**: None

  * **`float $timeout`**
    * **Description**: Sets the timeout period
    * **Unit**: seconds [supports floating point numbers, such as `1.5` which represents `1s + 500ms`]
    * **Default**: Refer to [client timeout rules](/coroutine_client/init?id=timeout-rules)
    * **Other values**: None

* **Return Value**

    * If successful, returns the actual data received
    * If failed to receive, returns `false` and sets the `errCode` attribute
    * If timed out, the error code is `ETIMEDOUT`

!> The return value may not necessarily be the expected length. You need to check the length of the data received in this call. If you need to ensure that you receive data of a specified length in a single call, please use the `recvAll` method or loop to retrieve the data yourself.  
For TCP data packet boundary issues, please refer to the `setProtocol()` method or use `sendto()`.
### recvAll()

データを受信します。`recv`とは異なり、`recvAll`はできるだけ完全に応答長のデータを受信し、受信が完了するかエラーが発生するまで待ち続けます。

!> `recvAll`メソッドは、現在のコルーチンを即座にサスペンドし、読み取り可能なイベントをリッスンし、対向端からデータを送信するのを待ちます。読み取り可能なイベントがトリガーされると、`recv`システムコールを実行して`socket`バッファ内のデータを取得し、この動作を繰り返して指定された長さのデータが受信されるかエラーが発生するかまで待機し、その後コルーチンを再開します。

!> Swooleバージョン >= v4.3.0

```php
Swoole\Coroutine\Socket->recvAll(int $length = 65535, float $timeout = 0): string|false
```

  * **パラメータ** 

    * **`int $length`**
      * **機能**：受信したいデータのサイズ（注意：ここではメモリが割り当てられ、長すぎるサイズはメモリ不足を引き起こす可能性があります）
      * **単位**：バイト
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`float $timeout`**
      * **機能**：タイムアウト時間を設定します
      * **単位**：秒【浮動小数点もサポートします、例：`1.5`は`1s`+`500ms`を表します】
      * **デフォルト値**：[クライアントのタイムアウト規則](/coroutine_client/init?id=超時規則)を参照
      * **その他の値**：なし

  * **戻り値** 

    * 受信に成功した場合は実際のデータが返され、返される文字列の長さはパラメータの長さと一致します
    * 受信に失敗した場合は`false`が返され、`errCode`属性が設定されます
    * タイムアウトした場合、エラーコードは`ETIMEDOUT`です
### readVector()

データを分割して受信します。

!> `readVector`メソッドは、データをすぐに受信するために`readv`システムコールを実行します。 `readv`システムコールが`EAGAIN`エラーを返した場合、Swooleは自動的に読み取り可能なイベントを監視し、現在のコルーチンを一時停止し、読み取り可能なイベントがトリガーされるのを待ってから、データを再度読み取るために`readv`システムコールを実行し、そのコルーチンを再開します。

!> Swooleのバージョン >= v4.5.7

```php
Swoole\Coroutine\Socket->readVector(array $io_vector, float $timeout = 0): array|false
```

  * **パラメータ** 

    * **`array $io_vector`**
      * **機能**：期待される分割データのサイズ
      * **値の単位**：バイト
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`float $timeout`**
      * **機能**：タイムアウト時間を設定します
      * **値の単位**：秒【浮動小数点数をサポートし、例えば`1.5`は`1秒`+`500ミリ秒`を表します】
      * **デフォルト値**：[クライアントのタイムアウト規則](/coroutine_client/init?id=超時規則)を参照
      * **その他の値**：なし

  * **戻り値**

    * 成功した場合は、分割されたデータが返されます
    * 失敗した場合は空の配列が返され、`errCode`プロパティが設定されます
    * タイムアウトした場合は、エラーコードは`ETIMEDOUT`になります

  * **例** 

```php
$socket = new Swoole\Coroutine\Socket(AF_INET, SOCK_STREAM, 0);
// リモートホストからhelloworldが送信された場合
$ret = $socket->readVector([5, 5]);
// その場合、$ret は ['hello', 'world'] となります
```
### readVectorAll()

データを分割して受信します。

!> `readVectorAll`メソッドは、複数の`readv`システムコールを即座に実行してデータを読み取ります。`readv`システムコールが`EAGAIN`エラーを返した場合、バックエンドは自動的に読み取り可能なイベントを監視し、現在のコルーチンを一時停止し、読み取り可能なイベントがトリガーされるのを待機し、データの読み取りが完了するかエラーに遭遇するまで、対応するコルーチンを再開します。

!> Swooleバージョン >= v4.5.7

```php
Swoole\Coroutine\Socket->readVectorAll(array $io_vector, float $timeout = 0): array|false
```

  * **Parameters** 

    * **`array $io_vector`**
      * **機能**：受信する分割データのサイズを指定します
      * **単位**：バイト
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`float $timeout`**
      * **機能**：タイムアウト時間を設定します
      * **単位**：秒【浮動小数点をサポートします。例：`1.5`は`1秒`+`500ms`を表します】
      * **デフォルト値**：[クライアントタイムアウトルール](/coroutine_client/init?id=超時ルール)を参照してください
      * **その他の値**：なし

  * **Return Value**

    * 受信に成功した分割データ
    * 受信に失敗した場合は空の配列を返し、`errCode`プロパティを設定します
    * タイムアウトが発生した場合、エラーコードは`ETIMEDOUT`です
### writeVector()

データを断片化して送信します。

!> `writeVector`メソッドは、データを送信するために`writev`システムコールを即座に実行します。`writev`システムコールが`EAGAIN`エラーを返す場合、バックエンドは自動的に書き込み可能なイベントを監視し、現在のコルーチンを一時停止させて、書き込み可能なイベントがトリガーされるのを待ち、データを送信するために`writev`システムコールを再実行し、そのコルーチンを再開します。

!> Swooleバージョン >= v4.5.7

```php
Swoole\Coroutine\Socket->writeVector(array $io_vector, float $timeout = 0): int|false
```

  * **パラメータ** 

    * **`array $io_vector`**
      * **機能**：送信する断片化されたデータ
      * **値の単位**：バイト
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`float $timeout`**
      * **機能**：タイムアウト時間を設定します
      * **値の単位**：秒【浮動小数点数をサポートしており、たとえば`1.5`は`1秒`+`500ミリ秒`を表します】
      * **デフォルト値**：[クライアントのタイムアウトルール](/coroutine_client/init?id=超時規則)を参照
      * **その他の値**：なし

  * **戻り値**

    * 送信に成功した場合は書き込まれたバイト数が返されます。**実際に書き込まれたデータは`$io_vector`パラメータの総長よりも小さい可能性がある**ため、アプリケーションレイヤのコードは、戻り値と`$io_vector`パラメータの総長が等しいかどうかを比較して、送信が完了したかどうかを判断する必要があります。
    * 送信に失敗した場合は`false`が返され、`errCode`属性が設定されます。

  * **例** 

```php
$socket = new Swoole\Coroutine\Socket(AF_INET, SOCK_STREAM, 0);
// ここでは、配列の順序通りに対向端に送信されます。つまり、"hello world" が送信されます。
$socket->writeVector(['hello', 'world']);
```
### writeVectorAll()

データをリモートに送信します。 `writeVector`メソッドとは異なり、`writeVectorAll`はデータを可能な限り完全に送信し、すべてのデータを正常に送信するかエラーに遭遇するまで処理を中止します。

!> `writeVectorAll`メソッドは、データを送信するために直ちに複数の`writev`システムコールを実行します。 `writev`システムコールが`EAGAIN`エラーを返す場合、基盤は自動的に書き込み可能なイベントを監視し、現在のコルーチンを保留し、書き込み可能なイベントが発生するのを待ち、データ送信を再度`writev`システムコールを実行し、データの送信が完了するかエラーが発生するまで、該当のコルーチンを再開します。

!> Swooleのバージョン >= v4.5.7

```php
Swoole\Coroutine\Socket->writeVectorAll(array $io_vector, float $timeout = 0): int|false
```

  * **Parameters** 

    * **`array $io_vector`**
      * **Function**：送信するセグメントデータ
      * **Value unit**：バイト
      * **Default value**：なし
      * **Other values**：なし

    * **`float $timeout`**
      * **Function**：タイムアウト時間を設定します
      * **Value unit**：秒【浮動小数点数をサポートしており、例：`1.5`は`1s`+`500ms`を表します】
      * **Default value**：[クライアントタイムアウト規則](/coroutine_client/init?id=超时规则)を参照
      * **Other values**：なし

  * **Return Value**

    * `writeVectorAll`はデータがすべて正常に送信されることを保証しますが、`writeVectorAll`の間にリモートエンドが接続を切断する可能性があります。この場合、一部のデータが正常に送信された可能性があり、戻り値はこの成功したデータの長さを返します。アプリケーション層のコードは、戻り値と`$io_vector`パラメータの合計長さが等しいかどうかを比較して、送信が完了したかどうかを判断し、ビジネス要件に基づいて再送する必要があるかどうかを判断すべきです。
    * 送信に失敗した場合は`false`を返し、`errCode`属性を設定します。

  * **Example** 

```php
$socket = new Swoole\Coroutine\Socket(AF_INET, SOCK_STREAM, 0);
// At this point, it will send to the remote in the order specified in the array, actually sending helloworld
$socket->writeVectorAll(['hello', 'world']);
```
### recvPacket()

`setProtocol`メソッドでプロトコルを設定したソケットオブジェクトに対して、完全なプロトコルデータパケットを受信するためにこのメソッドを呼び出すことができます

!> Swooleバージョン >= v4.4.0

```php
Swoole\Coroutine\Socket->recvPacket(float $timeout = 0): string|false
```

  * **パラメータ** 
    * **`float $timeout`**
      * **機能**：タイムアウト時間を設定します
      * **値の単位**：秒【浮動小数点数がサポートされており、例えば`1.5`は`1秒`+`500ミリ秒`を示します】
      * **デフォルト値**：[クライアントのタイムアウトルール](/coroutine_client/init?id=超時規則)を参照してください
      * **その他**：なし

  * **戻り値** 

    * 受信に成功した場合は完全なプロトコルデータパケットを返します
    * 受信に失敗した場合は`false`を返し、`errCode`属性が設定されます
    * タイムアウトした場合、エラーコードは`ETIMEDOUT`です
### recvLine()

[socket_read](https://www.php.net/manual/en/function.socket-read.php)の互換性の問題を解決するために使用されます

```php
Swoole\Coroutine\Socket->recvLine(int $length = 65535, float $timeout = 0): string|false
```
### recvWithBuffer()

`recv(1)` で1バイトずつ受信する際に発生する多くのシステムコールの問題を解決するために使用されます。

```php
Swoole\Coroutine\Socket->recvWithBuffer(int $length = 65535, float $timeout = 0): string|false
```
### recvfrom()

データを受信し、送信元のホストのアドレスとポートを設定します。`SOCK_DGRAM`タイプの`socket`で使用されます。

!> このメソッドは[コルーチンスケジューリング](/coroutine?id=协程调度)を引き起こします。内部的には現在のコルーチンをすぐに中断し、読み込み可能なイベントを監視します。読み込み可能なイベントがトリガーされると、データを受信して`recvfrom`システムコールを実行してデータパケットを取得します。

```php
Swoole\Coroutine\Socket->recvfrom(array &$peer, float $timeout = 0): string|false
```

* **Parameters**

    * **`array $peer`**
        * **Description**: The peer address and port, passed by reference.【関数が成功した場合、`address`と`port`の2つの要素を含む配列に設定されます。】
        * **Default**: None
        * **Other values**: None

    * **`float $timeout`**
        * **Description**: Sets the timeout 【If no data is returned within the specified time, the `recvfrom` method will return `false`.】
        * **Unit of value**: seconds 【Supports floating-point numbers, such as `1.5` which represents `1s` and `500ms`.】
        * **Default**: See [Client Timeout Rules](/coroutine_client/init?id=超时规则)
        * **Other values**: None

* **Return Value**

    * If data is successfully received, return the data content and set `$peer` as an array
    * If fails, return `false` and set the `errCode` property without modifying the content of `$peer`

* **Example**

```php
use Swoole\Coroutine;
use function Swoole\Coroutine\run;

run(function () {
    $socket = new Coroutine\Socket(AF_INET, SOCK_DGRAM, 0);
    $socket->bind('127.0.0.1', 9601);
    while (true) {
        $peer = null;
        $data = $socket->recvfrom($peer);
        echo "[Server] recvfrom[{$peer['address']}:{$peer['port']}] : $data\n";
        $socket->sendto($peer['address'], $peer['port'], "Swoole: $data");
    }
});
```
### sendto()

データを指定されたアドレスとポートに送信します。これは`SOCK_DGRAM`タイプの`socket`で使用されます。

!> このメソッドは[スケジュールの中断](/coroutine?id=協調スケジューリング)が行われません。`sendto`メソッドはすぐに`sendto`を呼び出してデータを送信します。このメソッドは書き込み可能なイベントを監視せず、バッファがいっぱいのため`false`が返される場合があるため、自分で処理するか、`send`メソッドを使用してください。

```php
Swoole\Coroutine\Socket->sendto(string $address, int $port, string $data): int|false
```

  * **パラメータ** 

    * **`string $address`**
      * **説明**：送信先ホストの`IP`アドレスまたは[unixSocket](/learn?id=IPC)のパス【`sendto`はドメイン名をサポートしていません。`AF_INET`または`AF_INET6`を使用する場合は、有効な`IP`アドレスを渡す必要があります。そうでない場合、送信に失敗します】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $port`**
      * **説明**：送信先ホストのポート【ブロードキャストを送信する場合は`0`にすることができます】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $data`**
      * **説明**：送信するデータ【テキストまたはバイナリデータを指定できます。ただし、`SOCK_DGRAM`で送信できるパケットの最大長は`64K`であることに注意してください】
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **戻り値** 

    * 送信に成功した場合、送信されたバイト数が返されます
    * 送信に失敗した場合、`false`が返され、`errCode`プロパティが設定されます

  * **例** 

```php
$socket = new Swoole\Coroutine\Socket(AF_INET, SOCK_DGRAM, 0);
$socket->sendto('127.0.0.1', 9601, 'Hello');
```
### getsockname()

ソケットのアドレスとポート情報を取得します。

!> このメソッドには[コルーチンスケジューリング](/coroutine?id=コルーチンスケジューリング)のコストがかかりません。

```php
Swoole\Coroutine\Socket->getsockname(): array|false
```

  * **戻り値** 

    * 成功した場合は`address`と`port`を含む配列が返されます
    * 失敗した場合は`false`が返され、`errCode`属性が設定されます
### getpeername()

`socket`の対向アドレスとポート情報を取得します。これは、接続済みの`SOCK_STREAM`タイプの`socket`にのみ適用されます。

?> このメソッドは[コルーチンスケジューリング](/coroutine?id=コルーチンスケジューリング)のオーバーヘッドがありません。

```php
Swoole\Coroutine\Socket->getpeername(): array|false
```

  * **戻り値**

    * 成功した場合は`address`と`port`を含む配列を返します
    * 失敗した場合は`false`を返し、`errCode`プロパティを設定します
### close()

`Socket`を閉じます。

!> `Swoole\Coroutine\Socket`オブジェクトはデストラクト時に自動的に`close`を実行しますが、このメソッドは[コルーチンスケジュール](/coroutine?id=协程调度)のオーバーヘッドがありません。

```php
Swoole\Coroutine\Socket->close(): bool
```

  * **Return Value** 

    * 閉じるのに成功した場合は`true`を返します
    * 失敗した場合は`false`を返します
```php
Swoole\Coroutine\Socket->isClosed(): bool
```
## 定数

`sockets`拡張機能の定数と同等であり、`sockets`拡張機能と競合しません。

!> 異なるシステムで値が異なる場合があります。以下のコードは単なる例示です。実際の値ではありませんのでご注意ください。

```php
define('AF_UNIX', 1);
define('AF_INET', 2);

/**
 * Only available if compiled with IPv6 support.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('AF_INET6', 10);
define('SOCK_STREAM', 1);
define('SOCK_DGRAM', 2);
define('SOCK_RAW', 3);
define('SOCK_SEQPACKET', 5);
define('SOCK_RDM', 4);
define('MSG_OOB', 1);
define('MSG_WAITALL', 256);
define('MSG_CTRUNC', 8);
define('MSG_TRUNC', 32);
define('MSG_PEEK', 2);
define('MSG_DONTROUTE', 4);

/**
 * Not available on Windows platforms.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('MSG_EOR', 128);

/**
 * Not available on Windows platforms.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('MSG_EOF', 512);
define('MSG_CONFIRM', 2048);
define('MSG_ERRQUEUE', 8192);
define('MSG_NOSIGNAL', 16384);
define('MSG_DONTWAIT', 64);
define('MSG_MORE', 32768);
define('MSG_WAITFORONE', 65536);
define('MSG_CMSG_CLOEXEC', 1073741824);
define('SO_DEBUG', 1);
define('SO_REUSEADDR', 2);

/**
 * This constant is only available in PHP 5.4.10 or later on platforms that
 * support the <b>SO_REUSEPORT</b> socket option: this
 * includes Mac OS X and FreeBSD, but does not include Linux or Windows.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SO_REUSEPORT', 15);
define('SO_KEEPALIVE', 9);
define('SO_DONTROUTE', 5);
define('SO_LINGER', 13);
define('SO_BROADCAST', 6);
define('SO_OOBINLINE', 10);
define('SO_SNDBUF', 7);
define('SO_RCVBUF', 8);
define('SO_SNDLOWAT', 19);
define('SO_RCVLOWAT', 18);
define('SO_SNDTIMEO', 21);
define('SO_RCVTIMEO', 20);
define('SO_TYPE', 3);
define('SO_ERROR', 4);
define('SO_BINDTODEVICE', 25);
define('SOL_SOCKET', 1);
define('SOMAXCONN', 128);

/**
 * Used to disable Nagle TCP algorithm.
 * Added in PHP 5.2.7.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('TCP_NODELAY', 1);
define('PHP_NORMAL_READ', 1);
define('PHP_BINARY_READ', 2);
define('MCAST_JOIN_GROUP', 42);
define('MCAST_LEAVE_GROUP', 45);
define('MCAST_BLOCK_SOURCE', 43);
define('MCAST_UNBLOCK_SOURCE', 44);
define('MCAST_JOIN_SOURCE_GROUP', 46);
define('MCAST_LEAVE_SOURCE_GROUP', 47);
define('IP_MULTICAST_IF', 32);
define('IP_MULTICAST_TTL', 33);
define('IP_MULTICAST_LOOP', 34);
define('IPV6_MULTICAST_IF', 17);
define('IPV6_MULTICAST_HOPS', 18);
define('IPV6_MULTICAST_LOOP', 19);
define('IPV6_V6ONLY', 27);

/**
 * Operation not permitted.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_EPERM', 1);

/**
 * No such file or directory.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_ENOENT', 2);

/**
 * Interrupted system call.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_EINTR', 4);

/**
 * I/O error.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_EIO', 5);

/**
 * No such device or address.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_ENXIO', 6);

/**
 * Arg list too long.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_E2BIG', 7);

/**
 * Bad file number.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_EBADF', 9);

/**
 * Try again.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_EAGAIN', 11);

/**
 * Out of memory.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_ENOMEM', 12);

/**
 * Permission denied.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_EACCES', 13);

/**
 * Bad address.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_EFAULT', 14);

/**
 * Block device required.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_ENOTBLK', 15);

/**
 * Device or resource busy.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_EBUSY', 16);

/**
 * File exists.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_EEXIST', 17);

/**
 * Cross-device link.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_EXDEV', 18);

/**
 * No such device.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_ENODEV', 19);

/**
 * Not a directory.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_ENOTDIR', 20);

/**
 * Is a directory.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_EISDIR', 21);

/**
 * Invalid argument.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_EINVAL', 22);

/**
 * File table overflow.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_ENFILE', 23);

/**
 * Too many open files.
 * @link http://php.net/manual/en/sockets.constants.php
 */
define('SOCKET_EMFILE', 24);

/**
 * Not a typewriter.
/**
 * デバイスの容量がいっぱいです。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENOSPC', 28);

/**
 * シークが不正です。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ESPIPE', 29);

/**
 * 読み取り専用のファイルシステムです。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EROFS', 30);

/**
 * リンクが多すぎます。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EMLINK', 31);

/**
 * パイプが切断されました。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EPIPE', 32);

/**
 * ファイル名が長すぎます。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENAMETOOLONG', 36);

/**
 * ロックできるレコードがありません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENOLCK', 37);

/**
 * 関数が実装されていません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENOSYS', 38);

/**
 * ディレクトリが空ではありません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENOTEMPTY', 39);

/**
 * シンボリックリンクが多すぎます。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ELOOP', 40);

/**
 * 操作がブロックされました。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EWOULDBLOCK', 11);

/**
 * 指定された型のメッセージがありません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENOMSG', 42);

/**
 * 識別子が削除されました。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EIDRM', 43);

/**
 * チャネル番号が範囲外です。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ECHRNG', 44);

/**
 * レベル2が同期されていません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EL2NSYNC', 45);

/**
 * レベル3が停止しました。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EL3HLT', 46);

/**
 * レベル3がリセットされました。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EL3RST', 47);

/**
 * リンク番号が範囲外です。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ELNRNG', 48);

/**
 * プロトコルドライバがアタッチされていません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EUNATCH', 49);

/**
 * CSI構造体が利用できません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENOCSI', 50);

/**
 * レベル2が停止しています。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EL2HLT', 51);

/**
 * 無効な交換です。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EBADE', 52);

/**
 * 無効なリクエスト記述子です。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EBADR', 53);

/**
 * 交換がいっぱいです。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EXFULL', 54);

/**
 * アノードがありません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENOANO', 55);

/**
 * 無効なリクエストコードです。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EBADRQC', 56);

/**
 * 無効なスロットです。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EBADSLT', 57);

/**
 * デバイスはストリームではありません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENOSTR', 60);

/**
 * データが利用できません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENODATA', 61);

/**
 * タイマーが切れました。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ETIME', 62);

/**
 * ストリームリソースが不足しています。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENOSR', 63);

/**
 * マシンがネットワークに接続されていません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENONET', 64);

/**
 * オブジェクトがリモートです。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EREMOTE', 66);

/**
 * リンクが切断されました。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENOLINK', 67);

/**
 * 広告エラーです。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EADV', 68);

/**
 * Srmountエラーです。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ESRMNT', 69);

/**
 * 送信時の通信エラーです。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ECOMM', 70);

/**
 * プロトコルエラーです。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EPROTO', 71);

/**
 * マルチホップが試みられました。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EMULTIHOP', 72);

/**
 * データメッセージではありません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EBADMSG', 74);

/**
 * ネットワーク上で一意ではない名前です。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENOTUNIQ', 76);

/**
 * ファイル記述子が悪い状態です。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EBADFD', 77);

/**
 * リモートアドレスが変更されました。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EREMCHG', 78);

/**
 * 中断されたシステムコールを再起動する必要があります。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ERESTART', 85);

/**
 * ストリームパイプエラーです。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ESTRPIPE', 86);

/**
 * ユーザーが多すぎます。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EUSERS', 87);

/**
 * ソケット以外の操作がソケットで実行されました。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENOTSOCK', 88);

/**
 * 宛先アドレスが必要です。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EDESTADDRREQ', 89);

/**
 * メッセージが長すぎます。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EMSGSIZE', 90);

/**
 * ソケットに対して適切でないプロトコルです。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EPROTOTYPE', 91);
define ('SOCKET_ENOPROTOOPT', 92);

/**
 * サポートされていないプロトコルです。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EPROTONOSUPPORT', 93);

/**
 * サポートされていないソケットの種類です。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ESOCKTNOSUPPORT', 94);

/**
 * トランスポートエンドポイントでサポートされていない操作です。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EOPNOTSUPP', 95);

/**
 * サポートされていないプロトコルファミリーです。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EPFNOSUPPORT', 96);

/**
 * プロトコルによってサポートされていないアドレスファミリーです。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EAFNOSUPPORT', 97);
define ('SOCKET_EADDRINUSE', 98);

/**
 * 要求されたアドレスを割り当てることができません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EADDRNOTAVAIL', 99);

/**
 * ネットワークがダウンしています。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENETDOWN', 100);

/**
 * ネットワークに到達できません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENETUNREACH', 101);

/**
 * リセットのために接続が切断されました。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENETRESET', 102);

/**
 * ソフトウェアによる接続の中止です。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ECONNABORTED', 103);

/**
 * ピアによる接続のリセットです。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ECONNRESET', 104);

/**
 * バッファスペースが利用できません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENOBUFS', 105);

/**
 * トランスポートエンドポイントは既に接続されています。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EISCONN', 106);

/**
 * トランスポートエンドポイントが接続されていません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENOTCONN', 107);

/**
 * トランスポートエンドポイントのシャットダウン後に送信できません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ESHUTDOWN', 108);

/**
 * 参照が多すぎて結びつけられません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ETOOMANYREFS', 109);

/**
 * 接続がタイムアウトしました。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ETIMEDOUT', 110);

/**
 * 接続が拒否されました。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ECONNREFUSED', 111);

/**
 * ホストがダウンしています。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EHOSTDOWN', 112);

/**
 * ホストへのルートがありません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EHOSTUNREACH', 113);

/**
 * 既に進行中の操作です。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EALREADY', 114);

/**
 * 現在進行中の操作です。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EINPROGRESS', 115);

/**
 * 名前付きの型ファイルです。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EISNAM', 120);

/**
 * リモートI/Oエラーです。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EREMOTEIO', 121);

/**
 * クオータが超過しました。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EDQUOT', 122);

/**
 * メディアが見つかりません。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_ENOMEDIUM', 123);

/**
 * 誤ったメディアタイプです。
 * @link http://php.net/manual/en/sockets.constants.php
 */
define ('SOCKET_EMEDIUMTYPE', 124);
define ('IPPROTO_IP', 0);
define ('IPPROTO_IPV6', 41);
define ('SOL_TCP', 6);
define ('SOL_UDP', 17);
define ('IPV6_UNICAST_HOPS', 16);
define ('IPV6_RECVPKTINFO', 49);
define ('IPV6_PKTINFO', 50);
define ('IPV6_RECVHOPLIMIT', 51);
define ('IPV6_HOPLIMIT', 52);
define ('IPV6_RECVTCLASS', 66);
define ('IPV6_TCLASS', 67);
define ('SCM_RIGHTS', 1);
define ('SCM_CREDENTIALS', 2);
define ('SO_PASSCRED', 16);
```

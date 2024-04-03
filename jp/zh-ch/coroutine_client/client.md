# コルーチンTCP/UDPクライアント

`Coroutine\Client`は、`TCP`、`UDP`、[unixSocket](/learn?id=什么是IPC)伝送プロトコルの[Socketクライアント](/coroutine_client/socket)のラッパーコードを提供し、使用する際は単に`new Swoole\Coroutine\Client`を使用するだけで済みます。

* **実装原理**

    * `Coroutine\Client`のネットワークリクエストに関連するすべてのメソッドは、`Swoole`が[コルーチンスケジューリング](/coroutine?id=协程调度)を行い、ビジネス層は感知する必要がありません。
    * 使用方法は[Client](/client)の同期モードメソッドと完全に同じです。
    * `connect`のタイムアウトは`Connect`、`Recv`、`Send`のタイムアウトに同時に適用されます。

* **継承関係**

    * `Coroutine\Client`は[Client](/client)とは継承関係にありませんが、`Client`が提供するメソッドはすべて`Coroutine\Client`でも使用できます。詳細は[Swoole\Client](/client?id=方法)を参照してください。ここではそれ以上詳述しません。
    * `Coroutine\Client`では`set`メソッドを使用して[設定オプション](/client?id=配置)を設定することができます。使用方法は`Client->set`と完全に同じです。異なる機能を持つ関数については、`set()`関数のセクションで個別に説明します。

* **使用例**

```php
use Swoole\Coroutine\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client(SWOOLE_SOCK_TCP);
    if (!$client->connect('127.0.0.1', 9501, 0.5))
    {
        echo "connect failed. Error: {$client->errCode}\n";
    }
    $client->send("hello world\n");
    echo $client->recv();
    $client->close();
});
```

* **プロトコル処理**

コルーチンクライアントは、長さと`EOF`プロトコル処理もサポートしており、設定方法は[ Swoole\Client](/client?id=配置)と完全に同じです。

```php
$client = new Swoole\Coroutine\Client(SWOOLE_SOCK_TCP);
$client->set(array(
    'open_length_check'     => true,
    'package_length_type'   => 'N',
    'package_length_offset' => 0, //パッケージ長の値があるN番目のバイト
    'package_body_offset'   => 4, //長さを計算するバイトの開始位置
    'package_max_length'    => 2000000, //プロトコルの最大長
));
```
### connect()

リモートサーバーに接続します。

```php
Swoole\Coroutine\Client->connect(string $host, int $port, float $timeout = 0.5): bool
```

  * **パラメータ** 

    * **`string $host`**
      * **機能**：リモートサーバーのアドレス【内部的には自動的にコルーチンスイッチングが行われ、ホスト名がIPアドレスに解決されます】
      * **デフォルト値**：なし
      * **他の値**：なし

    * **`int $port`**
      * **機能**：リモートサーバーのポート番号
      * **デフォルト値**：なし
      * **他の値**：なし

    * **`float $timeout`**
      * **機能**：ネットワークI/Oのタイムアウト時間；`connect/send/recv`を含み、タイムアウトが発生すると接続は自動的に`close`されます。[クライアントのタイムアウトルール](/coroutine_client/init?id=超時規則)を参照してください。
      * **単位**：秒【浮動小数点数もサポートされており、例えば`1.5`は`1秒`+`500ミリ秒`を表します】
      * **デフォルト値**：`0.5秒`
      * **他の値**：なし

* **ヒント**

    * 接続に失敗した場合、`false`が返されます
    * タイムアウト後に返された場合、`$cli->errCode`が`110`であるかを確認してください

* **再試行**

!> `connect`が失敗した後、直接再接続することはできません。既存の`socket`を`close`してから再度`connect`を行う必要があります。

```php
//接続に失敗
if ($cli->connect('127.0.0.1', 9501) == false) {
    //既存のソケットを閉じる
    $cli->close();
    //再試行
    $cli->connect('127.0.0.1', 9501);
}
```

* **例**

```php
if ($cli->connect('127.0.0.1', 9501)) {
    $cli->send('data');
} else {
    echo 'connect failed.';
}

if ($cli->connect('/tmp/rpc.sock')) {
    $cli->send('data');
} else {
    echo 'connect failed.';
}
```
### isConnected()

クライアントの接続状態を返します

```php
Swoole\Coroutine\Client->isConnected(): bool
```

  * **返り値**

    * `false` を返すと、現在サーバーに接続されていないことを示します
    * `true` を返すと、現在サーバーに接続されていることを示します
    
 !> `isConnected` メソッドが返すのはアプリケーションレベルの状態であり、`Client` が `Server` に接続し、`close` を実行して接続を閉じていないことを示します。`Client` は `send`、 `recv`、 `close` などの操作を実行できますが、`connect` を再度実行することはできません。  
 これは接続が必ずしも利用可能であることを意味するものではなく、`send` または `recv` を実行する際にエラーが発生する可能性があるということです。アプリケーションレベルは、`TCP` 接続の状態を取得できず、 `send` または `recv` を実行するときにアプリケーションレベルとカーネルがやり取りをする必要があり、実際の接続の利用可能な状態を確認することができます。
### send()

データを送信します。

```php
Swoole\Coroutine\Client->send(string $data): int|bool
```

  * **パラメータ** 

    * **`string $data`**
    
      * **機能**：送信するデータ。文字列型である必要があり、バイナリデータをサポートします。
      * **デフォルト値**：なし
      * **その他の値**：なし

  * 送信に成功すると、`Socket`バッファに書き込まれたバイト数が返されます。内部では可能な限りすべてのデータを送信しようとします。返されたバイト数が渡された`$data`の長さと異なる場合、`Socket`が対向端で閉じられている可能性があります。次回の`send`または`recv`呼び出し時に、対応するエラーコードが返されます。

  * 送信に失敗した場合はfalseが返され、`$client->errCode`を使用してエラーの原因を取得できます。
### recv()

recvメソッドは、サーバーからデータを受信するために使用されます。

```php
Swoole\Coroutine\Client->recv(float $timeout = 0): string|bool
```

  * **パラメータ** 

    * **`float $timeout`**
      * **機能**：タイムアウト時間の設定
      * **単位**：秒【浮動小数点数がサポートされており、例えば `1.5` は `1s` + `500ms` を表します】
      * **デフォルト値**：[クライアントのタイムアウトルール](/coroutine_client/init?id=超时规则)を参照
      * **その他の値**：なし

    !> タイムアウトを設定する場合、指定された引数が優先され、次に`set`メソッドで渡された`timeout`構成が使用されます。タイムアウトエラーのエラーコードは`ETIMEDOUT`です。

  * **戻り値**

    * [通信プロトコル](/client?id=协议解析)が設定されている場合、`recv`は完全なデータを返します。長さは[package_max_length](/server/setting?id=package_max_length)で制限されます。
    * 通信プロトコルが設定されていない場合、`recv`は最大で`64K`のデータを返します。
    * 通信プロトコルが設定されていない場合は、元のデータが返され、PHPコード内でネットワークプロトコルを自分で処理する必要があります。
    * `recv`が空の文字列を返す場合、サーバー側が接続を切断しました。`close`が必要です。
    * `recv`に失敗すると、`false`が返ります。`$client->errCode`をチェックしてエラーの原因を取得し、処理方法は以下の[完全な例](/coroutine_client/client?id=完整示例)を参照してください。
### close()

接続を閉じます。

!> `close` はブロッキングされず、すぐにリターンします。閉じる操作にはコルーチン切り替えは含まれません。

```php
Swoole\Coroutine\Client->close(): bool
```
### peek()

データを覗き見る。

!> `peek`メソッドは `socket` を直接操作するため、[コルーチンスケジューリング](/coroutine?id=コルーチンスケジューリング)を引き起こしません。

```php
Swoole\Coroutine\Client->peek(int $length = 65535): string
```

  * **ヒント**

    * `peek`メソッドはカーネルの`socket`バッファ内のデータを覗き見するだけで、オフセットは行いません。`peek`後に`recv`を呼び出すと、この部分のデータを引き続き読み取れます
    * `peek`メソッドは非ブロッキングであり、即座に返ります。`socket`バッファにデータがある場合は、データ内容が返されます。バッファが空の場合は`false`が返り、`$client->errCode`が設定されます
    * 接続が終了している場合、`peek`は空の文字列を返します
```php
Swoole\Coroutine\Client->set(array $settings): bool
```

  * **設定パラメータ**

    * [Swoole\Client](/client?id=set) を参照してください。

* **[Swoole\Client](/client?id=set)との違い**

    コルーチンクライアントはより細かいタイムアウト制御を提供します。以下を設定できます：

    * `timeout`：接続、送信、受信全体のタイムアウト
    * `connect_timeout`：接続のタイムアウト
    * `read_timeout`：受信のタイムアウト
    * `write_timeout`：送信のタイムアウト
    * [クライアントのタイムアウトルール](/coroutine_client/init?id=超時規則)を参照してください

* **例**

```php
use Swoole\Coroutine\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client(SWOOLE_SOCK_TCP);
    $client->set(array(
        'timeout' => 0.5,
        'connect_timeout' => 1.0,
        'write_timeout' => 10.0,
        'read_timeout' => 0.5,
    ));

    if (!$client->connect('127.0.0.1', 9501, 0.5))
    {
        echo "connect failed. Error: {$client->errCode}\n";
    }
    $client->send("hello world\n");
    echo $client->recv();
    $client->close();
});
```
```php
use Swoole\Coroutine\Client;
use function Swoole\Coroutine\run;

run(function () {
    $client = new Client(SWOOLE_SOCK_TCP);
    if (!$client->connect('127.0.0.1', 9501, 0.5)) {
        echo "connect failed. Error: {$client->errCode}\n";
    }
    $client->send("hello world\n");
    while (true) {
        $data = $client->recv();
        if (strlen($data) > 0) {
            echo $data;
            $client->send(time() . PHP_EOL);
        } else {
            if ($data === '') {
                // 全等于空 直接关闭连接
                $client->close();
                break;
            } else {
                if ($data === false) {
                    // 可以自行根据业务逻辑和错误码进行处理，例如：
                    // 如果超时时则不关闭连接，其他情况直接关闭连接
                    if ($client->errCode !== SOCKET_ETIMEDOUT) {
                        $client->close();
                        break;
                    }
                } else {
                    $client->close();
                    break;
                }
            }
        }
        \Co::sleep(1);
    }
});
```

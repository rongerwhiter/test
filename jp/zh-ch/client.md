`Swoole\Client`は、以下単称`Client`と呼ばれ、`TCP/UDP`、`socket`のクライアントをラップしたコードを提供しています。使用する際は単に `new Swoole\Client` を行うだけで利用できます。`FPM/Apache`環境で使用できます。  
従来の[streams](https://www.php.net/streams)シリーズ関数と比較して、いくつかの利点があります：

  * `stream`関数には、タイムアウトの設定に問題がある`Bug`が存在し、適切に処理しないと`Server`側が長時間ブロックされる可能性があります
  * `stream`関数の`fread`はデフォルトで最大`8192`の長さ制限があり、`UDP`の大きなパケットをサポートできません
  * `Client`は`waitall`をサポートしており、パケットの長さが確定している場合は一度にデータを取得でき、ループで読み取る必要がありません
  * `Client`は`UDP Connect`をサポートしており、`UDP`のパケット結合の問題を解決しています
  * `Client`は純粋な`C`コードであり、`socket`を専門に処理しており、`stream`関数は非常に複雑です。`Client`はパフォーマンスが優れています
  * `Client`は接続を維持できます
  * [swoole_client_select](/client?id=swoole_client_select)関数を使用して複数の`Client`の並行制御を実現できます
### 完整の例

```php
$client = new Swoole\Client(SWOOLE_SOCK_TCP);
if (!$client->connect('127.0.0.1', 9501, -1)) {
    exit("connect failed. Error: {$client->errCode}\n");
}
$client->send("hello world\n");
echo $client->recv();
$client->close();
```

!> `Apache`の`prework`マルチスレッドモードはサポートされていません
## 方法

このセクションでは、特定の手順や手法について説明します。
### __construct()

```php
Swoole\Client::__construct(int $sock_type, int $is_sync = SWOOLE_SOCK_SYNC, string $key);
```

* **パラメーター**

  * **`int $sock_type`**
    * **機能**：`socket`の種類を表します【`SWOOLE_SOCK_TCP`、`SWOOLE_SOCK_TCP6`、`SWOOLE_SOCK_UDP`、`SWOOLE_SOCK_UDP6`がサポートされています】詳細は[こちら](/server/methods?id=__construct)を参照してください
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $is_sync`**
    * **機能**：同期ブロッキングモード、現在はこの1つだけですが、このパラメーターは互換性を維持するために保持されています
    * **デフォルト値**：`SWOOLE_SOCK_SYNC`
    * **その他の値**：なし

  * **`string $key`**
    * **機能**：長期間の接続の`Key`に使用します【デフォルトでは`IP:PORT`が`key`として使用されます。同じ`key`を持つ場合、2回`new`しても1つのTCP接続しか使用されません】
    * **デフォルト値**：`IP:PORT`
    * **その他の値**：なし

!> タイプを指定するために、提供されているマクロを使用できます。詳細は[定数の定義](/consts)を参照してください
#### PHP-FPM/Apacheで長期接続を作成する

```php
$cli = new Swoole\Client(SWOOLE_SOCK_TCP | SWOOLE_KEEP);
```

[SWOOLE_KEEP](/client?id=swoole_keep) フラグを追加すると、作成された`TCP`接続はPHPリクエストが終了するか`$cli->close()`が呼び出されるまで閉じられません。次に`connect`メソッドが呼び出されると、前回作成した接続が再利用されます。長期接続の保存方法はデフォルトで`ServerHost:ServerPort`をキーとしています。3番目の引数で`key`を指定できます。

`Client`オブジェクトが破棄されると[close](/client?id=close)メソッドが自動的に呼び出され、`ソケット`が閉じられます。
#### クライアントをサーバーで使用する

  * `Client`を使用するには、イベント[コールバック関数](/server/events)内で使用する必要があります。
  * `Server`はどんな言語で書かれた`socket client`でも接続できます。同様に、`Client`もどんな言語で書かれた`socket server`に接続できます

!> `Swoole4+`のコルーチン環境でこの`Client`を使用すると、[同期モデル](/learn?id=同步io异步io)に逆戻りする可能性があります。
### set()

[connect](/client?id=connect)前にクライアントパラメータを設定します。

```php
Swoole\Client->set(array $settings);
```

Available configuration options can be found in the Client - [Configuration Options](/client?id=configuration).```
### connect()

リモートサーバーに接続します。

```php
Swoole\Client->connect(string $host, int $port, float $timeout = 0.5, int $sock_flag = 0): bool
```

* **パラメータ** 

  * **`string $host`**
    * **説明**：サーバーのアドレス【ドメイン名の自動非同期解決をサポートし、`$host`に直接ドメイン名を渡すことができます】
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $port`**
    * **説明**：サーバーポート
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`float $timeout`**
    * **説明**：接続のタイムアウト時間を設定します
    * **単位**：秒【浮動小数点数をサポートしており、`1.5`は`1秒`+`500ミリ秒`を表します】
    * **デフォルト値**：`0.5`
    * **その他の値**：なし

  * **`int $sock_flag`**
    - `UDP`タイプの場合は`udp_connect`を有効にするかどうかを表します。このオプションを設定すると、`$host`と`$port`がバインドされ、この`UDP`は指定された`host/port`以外のパケットを破棄します。
    - `TCP`タイプの場合、`$sock_flag=1`はノンブロッキング`ソケット`として設定されます。その後、このfdは非同期IOになり、`connect`はすぐに返ります。`$sock_flag`を`1`に設定すると、`send/recv`前に[swoole_client_select](/client?id=swoole_client_select)を使用して接続が完了したかどうかを確認する必要があります。

* **戻り値**

  * 成功した場合は`true`
  * 失敗した場合は`false`、失敗の理由は`errCode`プロパティで確認してください

* **同期モード**

`connect`メソッドはブロックし、接続が成功して`true`が返されるまで待ちます。この時点でサーバーにデータを送信したり、データを受信したりできます。

```php```
```php
if ($cli->connect('127.0.0.1', 9501)) {
      $cli->send("data");
} else {
      echo "connect failed.";
}
```

もし接続に失敗した場合、`false`が返ってきます。

> 同期`TCP`クライアントは`close`コマンドを実行した後、再び`Connect`を使って新しい接続をサーバに作成できます。

* **再接続**

`connect`が失敗した後、1回だけ再接続を行いたい場合、まず古い`socket`を`close`しておかなければなりません。そうしないと`EINPROCESS`エラーが返されます。なぜならば、現在の`socket`がサーバに接続しようとしている状態で、クライアントは接続が成功したかどうかを知らないため、再び`connect`を実行することができません。`close`を呼び出すと現在の`socket`が閉じられ、下の層が新しい`socket`を作成して再接続を行います。

!> [SWOOLE_KEEP](/client?id=swoole_keep) 長接続を有効にした場合、`close`コマンドの最初の引数を`true`に設定して強制的に長接続`socket`を破棄する必要があります。

```php
if ($socket->connect('127.0.0.1', 9502) === false) {
    $socket->close(true);
    $socket->connect('127.0.0.1', 9502);
}
```

* **UDP接続**

デフォルトでは、下の層は`udp connect`を有効にしません。`UDP`クライアントが`connect`を実行すると、下の層は`socket`を作成した後、即座に成功を返します。この時、この`socket`にバインドされたアドレスは`0.0.0.0`であり、どんな他の端末でもこのポートにデータパケットを送信することができます。

たとえば`$client->connect('192.168.1.100', 9502)`、この時、操作システムはクライアント`socket`にランダムなポート`58232`を割り当てました。他のマシン、例えば`192.168.1.101`もこのポートにデータパケットを送ることができます。

?> `udp connect`が有効でない場合、`getsockname`を呼び出したときに返される`host`フィールドは`0.0.0.0`になります。
第`4`項目のパラメータを`1`に設定し、`udp connect`を有効にします。`$client->connect('192.168.1.100', 9502, 1, 1)`。これにより、クライアントとサーバーがバインドされ、基層ではサーバーのアドレスに基づいて`socket`がバインドされます。例えば、`192.168.1.100`に接続すると、現在の`socket`は`192.168.1.*`のローカルアドレスにバインドされます。`udp connect`を有効にすると、クライアントはこのポートに送信される他のホストからのデータパケットを受信しなくなります。
### recv()

サーバーからデータを受信します。

```php
Swoole\Client->recv(int $size = 65535, int $flags = 0): string | false
```

* **パラメータ** 

  * **`int $size`**
    * **機能**：データの受信バッファの最大長【このパラメータは大きすぎないように設定しないでください。それ以上の場合は大量のメモリを消費します】
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $flags`**
    * **機能**：追加のパラメータを設定できます【例: [Client::MSG_WAITALL](/client?id=clientmsg_waitall)】。詳細なパラメータについては[このセクション](/client?id=常量)を参照してください。
    * **デフォルト値**：なし
    * **その他の値**：なし

* **戻り値**

  * データを正常に受信した場合は文字列が返ります。
  * 接続が閉じられた場合は空の文字列が返ります。
  * 失敗した場合は `false` が返り、`$client->errCode`プロパティが設定されます。

* **EOF/Length プロトコル**

  * クライアントが`EOF/Length`検出を有効にした場合、`$size`および`$waitall`パラメータを設定する必要はありません。拡張層は完全なデータパケットを返すか、`false`を返します。[プロトコル解析](/client?id=协议解析)セクションを参照してください。
  * 間違ったパケットヘッダーまたはパケットヘッダー内の長さ値が[package_max_length](/server/setting?id=package_max_length)設定を超える場合、`recv`は空の文字列を返します。PHPコードではこの接続を閉じる必要があります。
### send()

リモートサーバーにデータを送信します。接続を確立した後にのみ、対向端にデータを送信できます。

```php
Swoole\Client->send(string $data): int|false
```

* **パラメータ**

  * **`string $data`**
    * **機能**：送信する内容【バイナリデータもサポート】
    * **デフォルト値**：なし
    * **その他の値**：なし

* **戻り値**

  * 送信に成功した場合は、送信されたデータの長さが返されます
  * 失敗した場合は`false`が返され、`errCode`プロパティが設定されます

* **注意**

  * `connect`を実行していない場合、`send`を呼び出すと警告が発生します
  * 送信するデータには長さ制限がありません
  * 送信されるデータが大きすぎる場合、ソケットのバッファが一杯になります。プログラムは書き込み可能になるまでブロックされます
### sendfile()

サーバーにファイルを送信するための関数で、この関数は`sendfile`システムコールに基づいています

```php
Swoole\Client->sendfile(string $filename, int $offset = 0, int $length = 0): bool
```

!> `sendfile`はUDPクライアントおよびSSLトンネル暗号接続には使用できません

* **パラメータ**

  * **`string $filename`**
    * **機能**：送信するファイルのパスを指定します
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $offset`**
    * **機能**：ファイルのオフセットをアップロードします【ファイルの中間部分からデータを送信することができます。この機能は、断続的な転送をサポートするために使用できます。】
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $length`**
    * **機能**：送信データのサイズ【デフォルトではファイル全体のサイズです】
    * **デフォルト値**：なし
    * **その他の値**：なし

* **戻り値**

  * 指定されたファイルが存在しない場合は`false`を返します
  * 成功した場合は`true`を返します

* **注意**

  * `sendfile`はファイル全体が送信されるか致命的なエラーが発生するまで常にブロックされます
### sendto()

任意の`IP:PORT`のホストに`UDP`パケットを送信します。`SWOOLE_SOCK_UDP/SWOOLE_SOCK_UDP6` タイプのみをサポートしています。

```php
Swoole\Client->sendto(string $ip, int $port, string $data): bool
```

* **パラメータ**

  * **`string $ip`**
    * **機能**：送信先ホストの`IP`アドレス、`IPv4/IPv6` をサポート
    * **デフォルト値**：なし
    * **他の値**：なし

  * **`int $port`**
    * **機能**：送信先ホストのポート
    * **デフォルト値**：なし
    * **他の値**：なし

  * **`string $data`**
    * **機能**：送信するデータ内容【`64K` を超えてはいけません】
    * **デフォルト値**：なし
    * **他の値**：なし
### enableSSL()

SSLトンネル暗号化を動的に有効にします。この関数を使用するためには、`swoole`を`--enable-openssl`オプションでコンパイルする必要があります。

```php
Swoole\Client->enableSSL(): bool
```

クライアントが接続を確立するときには平文通信を使用し、途中で`SSL`トンネル暗号化通信に変更したい場合は、`enableSSL`メソッドを使用します。最初からSSLを使用する場合は[SSL設定](/client?id=ssl-related)を参照してください。`enableSSL`を使用してSSLトンネル暗号化を動的に有効にするには、次の2つの条件を満たす必要があります：

  * クライアントが作成されたときのタイプが非SSLであること
  * クライアントがすでにサーバーとの接続を確立していること

`enableSSL`を呼び出すと、SSLハンドシェイクが完了するまでブロックされます。

* **例**

```php
$client = new Swoole\Client(SWOOLE_SOCK_TCP);
if (!$client->connect('127.0.0.1', 9501, -1))
{
    exit("connect failed. Error: {$client->errCode}\n");
}
$client->send("hello world\n");
echo $client->recv();
//SSLトンネル暗号化を有効にする
if ($client->enableSSL())
{
    //ハンドシェイクが完了し、この時点で送信および受信されるデータは暗号化されます
    $client->send("hello world\n");
    echo $client->recv();
}
$client->close();
```
### getPeerCert()

サーバーの証明書情報を取得します。`--enable-openssl`オプションを有効にして`Swoole`をコンパイルした場合にのみ、この関数を使用できます。

```php
Swoole\Client->getPeerCert(): string|false
```

* **戻り値**

  * 成功した場合は`X509`証明書の文字列情報を返します
  * 失敗した場合は`false`を返します

!> このメソッドを呼び出す前にSSLハンドシェイクが完了している必要があります。

証明書情報を解析するには`openssl`拡張機能の`openssl_x509_parse`関数を使用できます。

!> [--enable-openssl](/environment?id=compile-options)を有効にして`Swoole`をコンパイルする必要があります。
### verifyPeerCert()

`--enable-openssl` オプションが有効になっている場合にのみ使用できる、サーバー証明書を検証します。

```php
Swoole\Client->verifyPeerCert()
```
### isConnected()

クライアントの接続状態を返します。

* falseを返すと、現在サーバーに接続されていません
* trueを返すと、現在サーバーに接続されています

```php
Swoole\Client->isConnected(): bool
```

!> `isConnected`メソッドが返すのはアプリケーションレベルの状態であり、`Client`が`connect`を実行し`Server`に接続に成功し、`close`を実行して接続を閉じていない状態を表します。`Client`は`send`、`recv`、`close`などの操作を実行できますが、再度`connect`を実行することはできません。  
これは接続が常に利用可能であることを意味するものではありません。`send`や`recv`を実行する際にエラーが発生する可能性があります。なぜなら、アプリケーションレベルでは、基礎となる`TCP`接続の状態にアクセスできず、`send`や`recv`を実行する際にアプリケーションレベルとカーネルとの間でやりとりが発生し、実際の接続の利用可能な状態を取得できるからです。
### getSockName()

クライアントソケットのローカルホスト：ポートを取得するために使用されます。

!> 接続後でないと使用できません

```php
Swoole\Client->getsockname(): array|false
```

* **返り値**

```php
array('host' => '127.0.0.1', 'port' => 53652);
```
### getPeerName()

対向のソケットのIPアドレスとポートを取得します

!> `SWOOLE_SOCK_UDP/SWOOLE_SOCK_UDP6/SWOOLE_SOCK_UNIX_DGRAM`タイプのみサポートされています

```php
Swoole\Client->getpeername(): array|false
```

`UDP`プロトコル通信クライアントは、サーバーにデータパケットを送信した後、このサーバーがクライアントに応答を送信するとは限りません。`getpeername`メソッドを使用して、実際に応答しているサーバーの`IP:PORT`を取得できます。

!> この関数は`$client->recv()`の後に呼び出す必要があります。
### close()

接続を閉じます。

```php
Swoole\Client->close(bool $force = false): bool
```

* **パラメータ**

  * **`bool $force`**
    * **機能**: 接続を強制的に閉じます【[SWOOLE_KEEP](/client?id=swoole_keep)の長期接続を閉じるために使用できます】
    * **デフォルト値**: なし
    * **その他の値**: なし

`swoole_client`接続が`close`された後は`connect`を再度呼び出さないでください。正しい方法は現在の`Client`を破棄し、新しい`Client`を作成して新しい接続を開始することです。

`Client`オブジェクトは解体時に自動的に`close`されます。
### shutdown()

クライアントをシャットダウンします

```php
Swoole\Client->shutdown(int $how): bool
```

* **パラメータ** 

  * **`int $how`**
    * **機能**：クライアントをシャットダウンする方法を設定します
    * **デフォルト値**：なし
    * **その他の値**：Swoole\Client::SHUT_RDWR（読み書きを閉じる）、SHUT_RD（読み取りを閉じる）、Swoole\Client::SHUT_WR（書き込みを閉じる）
### getSocket()

底層の`socket`ハンドルを取得し、返されるオブジェクトは`sockets`リソースハンドルです。

!> このメソッドは`sockets`拡張機能に依存し、かつコンパイル時に[--enable-sockets](/environment?id=コンパイルオプション)オプションを有効にする必要があります

```php
Swoole\Client->getSocket()
```

`socket_set_option`関数を使用して、より低レベルな`socket`パラメータを設定できます。

```php
$socket = $client->getSocket();
if (!socket_set_option($socket, SOL_SOCKET, SO_REUSEADDR, 1)) {
    echo 'Unable to set option on socket: '. socket_strerror(socket_last_error()) . PHP_EOL;
}
```
### connect()

リモートサーバーに接続します。

```php
Swoole\Client->connect(string $host, int $port, float $timeout = 0.5, int $sock_flag = 0): bool
```

* **パラメータ**

  * **`string $host`**
    * **機能**：サーバーアドレス【自動的にドメイン名が解析されるため、`$host`にはドメイン名を直接渡すことができます】
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $port`**
    * **機能**：サーバーポート
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`float $timeout`**
    * **機能**：タイムアウト時間を設定します
    * **単位**：秒【浮動小数点数がサポートされており、たとえば`1.5`は`1秒`と`500ミリ秒`を表します】
    * **デフォルト値**：`0.5`
    * **その他の値**：なし

  * **`int $sock_flag`**
    - `UDP`タイプの場合は、`udp_connect`を有効にするかどうかを示します。このオプションを設定すると、`$host`と`$port`がバインドされ、この`UDP`は指定された`host/port`以外のデータパケットを破棄します。
    - `TCP`タイプの場合、`$sock_flag = 1`は非ブロッキング`socket`に設定されることを示します。その後、このfdは[非同期IO](/learn?id=同期io异步io)になり、`connect`はすぐに返されます。`$sock_flag`を`1`に設定した場合、`send/recv`の前に、接続が完了したかどうかを確認するために[swoole_client_select](/client?id=swoole_client_select)を使用する必要があります。

* **戻り値**

  * 成功時は`true`
  * 失敗時は`false`、失敗原因は`errCode`プロパティで確認してください

* **同期モード**

`connect`メソッドはブロックされ、接続が成功して`true`が返されるまで待ちます。この時点でサーバーにデータを送信したり、データを受信できるようになります。

```php```
```php
if ($cli->connect('127.0.0.1', 9501)) {
      $cli->send("data");
} else {
      echo "connect failed.";
}
```

接続に失敗した場合、`false`が返されます

> 同期`TCP`クライアントは`close`を実行した後に、再度`Connect`を発行して新しい接続をサーバーに作成することができます

* **再試行失敗**

`connect`が失敗した後、1回再接続したい場合は、まず古い`socket`を`close`しておく必要があります。そうしないと `EINPROCESS` のエラーが返されます。なぜならば、現在の`socket`がサーバーに接続しているため、クライアントは接続が成功したかどうかを把握していないため、`connect`を再度実行できないからです。`close`を呼び出すと現在の`socket`が閉じられ、基盤となるレイヤーが新しい`socket`を作成して接続を行います。

!> [SWOOLE_KEEP](/client?id=swoole_keep) 長い接続が有効になっている場合、`close`を呼び出す際、最初の引数は`true`に設定する必要があり、これは強制的に長い接続の`socket`を破棄することを表します

```php
if ($socket->connect('127.0.0.1', 9502) === false) {
    $socket->close(true);
    $socket->connect('127.0.0.1', 9502);
}
```

* **UDP Connect**

デフォルトでは、基盤となるレイヤーは`udp connect`を有効にしません。`UDP`クライアントが`connect`を実行すると、基盤となるレイヤーはソケットを作成した直後に成功を返します。この時、このソケットにバインドされるアドレスは`0.0.0.0`になり、他の端末からこのポートにデータパケットを送信できます。

例えば、`$client->connect('192.168.1.100', 9502)`のようになります。この時、オペレーティングシステムはクライアントの`socket`にランダムなポート`58232`を割り当て、他のマシン、例えば`192.168.1.101`からもこのポートにデータパケットを送信できます。

?> `udp connect`が無効の場合、`getsockname`を呼び出すと返される`host`項目は`0.0.0.0`になります
`$client->connect('192.168.1.100', 9502, 1, 1)`命令が第`4`項目を`1`に設定し、`udp connect`を有効にします。これにより、クライアントとサーバーがバインドされ、基礎となるソケットがサーバーのアドレスに基づいてバインドされます。例えば、`192.168.1.100`に接続した場合、現在のソケットは`192.168.1.*`のローカルアドレスにバインドされます。`connect`の後、`udp connect`を有効にすると、クライアントはこのポート宛てに他のホストからのパケットを受信しなくなります。
### swoole_client_select

Swoole\Clientの並列処理では、[IOイベントループ](/learn?id=什么是eventloop)にselectシステムコールが使用されています。この関数はepoll_waitではなく使用されます。また、[Eventモジュール](/event)とは異なり、この関数は同期IO環境で使用されます（SwooleのWorkerプロセス内で呼び出すと、Swoole独自の epoll [IOイベントループ](/learn?id=什么是eventloop)が実行されなくなります）。

関数のプロトタイプ：

```php
int swoole_client_select(array &$read, array &$write, array &$error, float $timeout);
```

* `swoole_client_select`は4つのパラメータを受け取ります。`$read`、`$write`、`$error`はそれぞれ読み込み可能/書き込み可能/エラーのファイル記述子です。
* これらの3つのパラメータは配列変数である必要があります。配列の要素は`swoole_client`オブジェクトでなければなりません。
* このメソッドは`select`システムコールをベースにしており、最大で`1024`個の`ソケット`をサポートします。
* `$timeout`パラメータは`select`システムコールのタイムアウト時間であり、秒単位の浮動小数点数を受け入れます。
* この機能はPHPの組み込み関数`stream_select()`に類似していますが、stream_selectはPHPのstream変数タイプのみをサポートし、性能が劣ります。

呼び出し成功後、イベントの数が返され、`$read`/`$write`/`$error`配列が変更されます。配列をforeachで反復処理し、`$item->recv`/`$item->send`を実行してデータの送受信を行います。または、`$item->close()`または`unset($item)`を呼び出して`socket`をクローズします。

`swoole_client_select`は、指定された時間内に利用可能なIOがない場合、`select`呼び出しでタイムアウトしたことを示す`0`を返します。

!> この関数は`Apache/PHP-FPM`環境で使用できます。

```php
$clients = array();

for($i=0; $i< 20; $i++)
{
```php
$client = new Swoole\Client(SWOOLE_SOCK_TCP, SWOOLE_SOCK_SYNC); //同期ブロッキング
$ret = $client->connect('127.0.0.1', 9501, 0.5, 0);
if(!$ret)
{
    echo "サーバーへの接続に失敗しました。errCode=" . $client->errCode;
}
else
{
    $client->send("HELLO WORLD\n");
    $clients[$client->sock] = $client;
}
}

while (!empty($clients))
{
    $write = $error = array();
    $read = array_values($clients);
    $n = swoole_client_select($read, $write, $error, 0.6);
    if ($n > 0)
    {
        foreach ($read as $index => $c)
        {
            echo "受信 #{$c->sock}: " . $c->recv() . "\n";
            unset($clients[$c->sock]);
        }
    }
}
```
```python
# List of attributes
attributes = ['Strength', 'Dexterity', 'Intelligence', 'Wisdom', 'Constitution', 'Charisma']
```
### errCode

エラーコード

```php
Swoole\Client->errCode: int
```

`connect/send/recv/close`が失敗した場合、`$swoole_client->errCode`の値が自動的に設定されます。

`errCode`の値は`Linux errno`に等しいです。`socket_strerror`を使用してエラーコードをエラーメッセージに変換できます。

```php
echo socket_strerror($client->errCode);
```

参考：[Linuxエラーコードリスト](/other/errno?id=linux)
### ソケット

ソケット接続のファイルディスクリプター。

```php
Swoole\Client->sock;
```

PHPコードでは、以下のように使用できます。

```php
$sock = fopen("php://fd/".$swoole_client->sock); 
```

* `Swoole\Client`の`socket`を`stream socket`に変換します。`fread/fwrite/fclose`などの関数を使用して操作することができます。

* [Swoole\Server](/server/methods?id=__construct) の`$fd`はこの方法では変換できません。なぜなら、`$fd`は単なる数値であり、`$fd`のファイルディスクリプターはメインプロセスに属しているためです。[SWOOLE_PROCESS](/learn?id=swoole_process)モードを参照してください。

* `$swoole_client->sock`はintに変換して配列の`key`として使用できます。

!> ここで注意すべき点は、`$swoole_client->sock`の属性値は、`$swoole_client->connect`を実行した後にのみ取得できます。サーバーに接続されていないと、この属性の値は`null`となります。
### 再利用

この接続が新規作成されたものなのか、既存のものを再利用しているものなのかを示します。[SWOOLE_KEEP](/client?id=swoole_keep)と一緒に使用します。
#### 使用シナリオ

`WebSocket`クライアントとサーバーが接続を確立した後、ハンドシェイクを行う必要があります。接続が再利用されている場合、再度ハンドシェイクを行う必要はなく、単に`WebSocket`データフレームを送信します。

```php
if ($client->reuse) {
    $client->send($data);
} else {
    $client->doHandShake();
    $client->send($data);
}
```
### reuseCount

This indicates the number of times this connection has been reused. Used in conjunction with [SWOOLE_KEEP](/client?id=swoole_keep).

```php
Swoole\Client->reuseCount;
```
### type

`socket`のタイプを示し、`Swoole\Client::__construct()`の`$sock_type`の値を返します

```php
Swoole\Client->type;
```
### id

`Swoole\Client->id` プロパティには、`Swoole\Client::__construct()` の `$key` パラメータの値が格納されます。このプロパティは、[SWOOLE_KEEP](/client?id=swoole_keep) と共に使用されます。

```php
Swoole\Client->id;
```
### 設定

```php
Swoole\Client->setting;
```  
```python
# Constants
MAX_ITERATIONS = 1000
PI = 3.14159
```    
### SWOOLE_KEEP

Swoole\Clientは`PHP-FPM/Apache`でサーバーに対してTCP接続を作成することができます。使用方法：

```php
$client = new Swoole\Client(SWOOLE_SOCK_TCP | SWOOLE_KEEP);
$client->connect('127.0.0.1', 9501);
```

`SWOOLE_KEEP`オプションを有効にすると、リクエストが終了しても`socket`をクローズしません。次に`connect`が再度行われる際に、前回作成した接続を自動的に再利用します。もし`connect`を実行した際にサーバー側で接続がすでにクローズされている場合、`connect`は新しい接続を作成します。

?> SWOOLE_KEEPの利点

- `TCP`長い接続は、`connect`および`close`による追加のIO負荷を軽減します
- サーバー側の`close`/`connect`回数を低下させます
### Swoole\Client::MSG_WAITALL

  * If the `Client::MSG_WAITALL` parameter is set, an accurate `$size` must be set, otherwise it will wait until the received data length reaches `$size`.
  * When `Client::MSG_WAITALL` is not set, the maximum value for `$size` is `64K`.
  * Setting an incorrect `$size` will cause the `recv` to time out and return `false`.
### Swoole\Client::MSG_DONTWAIT

非阻塞接收数据，无论是否有数据都会立即返回。
### Swoole\Client::MSG_PEEK

`socket` バッファー内のデータをのぞき見します。`MSG_PEEK` パラメータを設定すると、`recv` がデータを読み取ってもポインタは変更されず、そのため次回の `recv` 呼び出しでも前回と同じ位置からデータが返されます。
### Swoole\Client::MSG_OOB

読み込み特定の帯域外データについては、「`TCP帯域外データ`」を検索してください。
### Swoole\Client::SHUT_RDWR

クライアントの読み取り/書き込みポートをシャットダウンします。
### Swoole\Client::SHUT_RD

クライアントの読み取り側をシャットダウンします。
### Swoole\Client::SHUT_WR

クライアントの書き込みエンドを閉じる。
## 配置

`Client`クラスは、いくつかのオプションを設定し、特定の機能を有効にするために`set`メソッドを使用できます。
### connect()

リモートサーバーに接続します。

```php
Swoole\Client->connect(string $host, int $port, float $timeout = 0.5, int $sock_flag = 0): bool
```

* **パラメータ** 

  * **`string $host`**
    * **機能**：サーバーアドレス【自動非同期ドメイン解決をサポートします。`$host`には直接ドメイン名を渡すことができます】
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`int $port`**
    * **機能**：サーバーポート
    * **デフォルト値**：なし
    * **その他の値**：なし

  * **`float $timeout`**
    * **機能**：タイムアウト時間を設定します
    * **単位**：秒【浮動小数点数がサポートされており、`1.5`は`1秒`+ `500ms`を表します】
    * **デフォルト値**：`0.5`
    * **その他の値**：なし

  * **`int $sock_flag`**
    - `UDP`タイプの場合、`udp_connect`を有効にするかどうかを示します。このオプションを設定すると、`$host`と`$port`がバインドされ、この`UDP`では指定されていない`ホスト/ポート`のデータパケットは破棄されます。
    - `TCP`タイプの場合、`$sock_flag=1`はノンブロッキング`ソケット`に設定されます。その後、このfdは[非同期IO](/learn?id=同期io异步io)になり、`connect`はすぐに返ります。`$sock_flag`を`1`に設定した場合は、`send/recv`の前に[swolle_client_select](/client?id=swolle_client_select)を使用して接続が完了したかどうかを確認する必要があります。

* **戻り値**

  * 成功した場合は`true`
  * 失敗した場合は`false`で、`errCode`属性を確認して失敗の原因を取得してください。

* **同期モード**

`connect`メソッドはブロックされ、接続が成功し`true`が返されるまで続行されます。この時点でサーバーにデータを送信したりデータを受信したりできます。 

```php```
```php
if ($cli->connect('127.0.0.1', 9501)) {
      $cli->send("data");
} else {
      echo "connect failed.";
}
```

もし接続に失敗した場合は、`false` が返されます。

> 同期 `TCP` クライアントは `close` を実行した後、再び `Connect` を発行して新しい接続をサーバーに作成できます

* **再接続の失敗**

`connect` が失敗した後、1回だけ再接続したい場合は、古い `socket` を閉じるために必ず `close` を実行する必要があります。そうしないと `EINPROCESS` エラーが返されます。現在の `socket` がサーバーに接続しているかどうかはクライアントにはわからないため、再び `connect` を実行できません。`close` を呼ぶと現在の `socket` が閉じられ、基礎となる新しい `socket` が作成されて再接続されます。

!> [SWOOLE_KEEP](/client?id=swoole_keep) 長期接続を有効にした場合、`close` の最初の引数は `true` に設定して長期接続 `socket` を強制的に破棄する必要があります

```php
if ($socket->connect('127.0.0.1', 9502) === false) {
    $socket->close(true);
    $socket->connect('127.0.0.1', 9502);
}
```

* **UDP 接続**

デフォルトでは、基礎の `udp connect` は有効になっていません。`UDP` クライアントが `connect` を実行すると、基盤となる `socket` が作成された後、すぐに成功が返ります。この時、この `socket` がバインドされるアドレスは `0.0.0.0` であり、他のエンドポイントからこのポートにデータパケットを送信できます。

例えば `$client->connect('192.168.1.100', 9502)`、この場合、オペレーティングシステムはクライアントの `socket` にランダムなポート `58232` を割り当てますし、他のマシン、例えば `192.168.1.101` でもこのポートにデータパケットを送信できます。

?> `udp connect` が有効にされていない場合、`getsockname` を呼び出すと、`host` 項目に `0.0.0.0` が返されます。
第`4`項目のパラメーターを`1`に設定し、`udp connect`を有効にします。`$client->connect('192.168.1.100', 9502, 1, 1)`。これにより、クライアントとサーバーがバインドされ、基礎レベルではサーバーのアドレスに基づいて`socket`がバインドされます。たとえば、`192.168.1.100`に接続した場合、現在の`socket`は`192.168.1.*`のローカルアドレスにバインドされます。`udp connect`を有効にすると、クライアントはこのポートに他のホストからのデータパケットを受信しなくなります。
### swoole_client_select

[Swoole\Client](https://www.swoole.co.uk/docs/modules/swoole-client)の並行処理には、[IOイベントループ](https://www.swoole.co.uk/docs/modules/swoole-client?id=what-is-eventloop)にするために`select`システムコールが使用されています。[Eventモジュール](https://www.swoole.co.uk/docs/modules/swoole-event)とは異なり、この関数は同期IO環境で使用されます（SwooleのWorkerプロセス内で呼び出すと、Swoole独自の epoll [IOイベントループ](https://www.swoole.co.uk/docs/modules/swoole-client?id=what-is-eventloop)が実行されなくなります）。

関数のプロトタイプ：

```php
int swoole_client_select(array &$read, array &$write, array &$error, float $timeout);
```

* `swoole_client_select`は4つのパラメータを受け取ります。`$read`、`$write`、`$error`はそれぞれ読み取り可能な/書き込み可能な/エラーが発生したファイルディスクリプタです。
* これらの3つのパラメータは配列変数への参照である必要があります。配列の要素は`swoole_client`オブジェクトである必要があります。
* このメソッドは`select`システムコールに基づいており、最大で`1024`個の`socket`をサポートしています。
* `$timeout`パラメータは`select`システムコールのタイムアウト時間であり、秒単位の浮動小数点数を受け入れます。
* PHPのネイティブの`stream_select()`と似た機能を持ちますが、`stream_select`はPHPのstream変数型のみをサポートし、パフォーマンスが劣ります。

呼び出し成功後、イベントの数が返され、`$read`/`$write`/`$error`の配列が変更されます。配列を`foreach`で繰り返し処理し、その後`$item->recv`/`$item->send`を実行してデータの送受信を行います。または、`$item->close()`または`unset($item)`を呼び出して`socket`を閉じます。

`swoole_client_select`が`0`を返すと、指定された時間内に利用可能なIOがなく、`select`呼び出しがタイムアウトしました。

!> この関数は`Apache/PHP-FPM`環境で使用できます

```php
$clients = array();

for($i=0; $i< 20; $i++)
{```
```php
$client = new Swoole\Client(SWOOLE_SOCK_TCP, SWOOLE_SOCK_SYNC); //同期ブロッキング
$ret = $client->connect('127.0.0.1', 9501, 0.5, 0);
if (!$ret) {
    echo "サーバーへの接続に失敗しました。エラーコード=".$client->errCode;
} else {
    $client->send("HELLO WORLD\n");
    $clients[$client->sock] = $client;
}

while (!empty($clients)) {
    $write = $error = array();
    $read = array_values($clients);
    $n = swoole_client_select($read, $write, $error, 0.6);
    if ($n > 0) {
        foreach ($read as $index => $c) {
            echo "受信 #{$c->sock}: " . $c->recv() . "\n";
            unset($clients[$c->sock]);
        }
    }
}
```  
### プロトコル解析

[TCPデータパケットの境界問題](/learn?id=tcpデータパケットの境界問題)を解決するために、関連する設定は`Swoole\Server`と同じ意味を持ちます。詳細は[Swoole\Serverプロトコル](/server/setting?id=open_eof_check)設定セクションを参照してください。

* **終了文字チェック**

```php
$client->set(array(
    'open_eof_check' => true,
    'package_eof' => "\r\n\r\n",
    'package_max_length' => 1024 * 1024 * 2,
));
```

* **長さチェック**

```php
$client->set(array(
    'open_length_check' => true,
    'package_length_type' => 'N',
    'package_length_offset' => 0, // N番目のバイトがパッケージの長さの値です
    'package_body_offset' => 4, // 何番目のバイトから長さを計算しますか
    'package_max_length' => 2000000, // プロトコルの最大長
));
```

!> 現在、[open_length_check](/server/setting?id=open_length_check) と [open_eof_check](/server/setting?id=open_eof_check) という2つの自動プロトコル処理機能がサポートされています。  
プロトコル解析を設定すると、クライアントの`recv()`メソッドは長さパラメータを受け付けず、必ず完全なデータパケットを返します。

* **MQTTプロトコル**

!> `MQTT`プロトコル解析を有効にすると、[onReceive](/server/events?id=onreceive) コールバックで完全な`MQTT`データパケットが受信されます。

```php
$client->set(array(
    'open_mqtt_protocol' => true,
));
```

* **ソケットバッファサイズ**

!> `socket`レベルのOSバッファ、アプリケーションレベルの受信データメモリバッファ、アプリケーションレベルの送信データメモリバッファを含みます。

```php
$client->set(array(
    'socket_buffer_size' => 1024 * 1024 * 2, // 2Mバッファ
```
));	
```

* **Nagle's Algorithmを無効にする**

```php
$client->set(array(
    'open_tcp_nodelay' => true,
));
```
### connect()

リモートサーバーに接続します。

```php
Swoole\Client->connect(string $host, int $port, float $timeout = 0.5, int $sock_flag = 0): bool
```

* **パラメータ**

  * **`string $host`**
    * **機能**：サーバーアドレス【自動非同期ドメイン名解決をサポートし、`$host`に直接ドメイン名を渡すことができます】
    * **デフォルト値**：なし
    * **他の値**：なし

  * **`int $port`**
    * **機能**：サーバーポート
    * **デフォルト値**：なし
    * **他の値**：なし

  * **`float $timeout`**
    * **機能**：タイムアウト時間を設定します
    * **値の単位**：秒【浮動小数点数がサポートされ、たとえば`1.5`は`1s`+`500ms`を表します】
    * **デフォルト値**：`0.5`
    * **他の値**：なし

  * **`int $sock_flag`**
    - `UDP`タイプの場合、`udp_connect` を有効にするかどうかを示します。このオプションを設定すると、`$host` と `$port` がバインドされ、指定されていない `host/port` のデータパケットが破棄されます。
    - `TCP` タイプの場合、`$sock_flag=1` はノンブロッキング `socket` に設定され、その後、このfd は[非同期IO](/learn?id=同期io异步io) になり、`connect` がすぐに返されます。`$sock_flag` を`1` に設定すると、`send/recv` 前に [swoole_client_select](/client?id=swoole_client_select)を使用して接続が完了したかどうかを確認する必要があります。

* **戻り値**

  * 成功時は`true` が返されます
  * 失敗時は`false` が返され、`errCode` プロパティで失敗の理由を取得してください

* **同期モード**

`connect` メソッドはブロックし、接続が成功して`true` が返されるまで待機します。この時点でサーバーにデータを送信したり、データを受信したりできます。

```php```
```
if ($cli->connect('127.0.0.1', 9501)) {
      $cli->send("data");
} else {
      echo "connect failed.";
}
```

接続が失敗すると`false`が返されます

> 同期`TCP`クライアントは`close`を実行した後、再度`Connect`を開始して新しい接続をサーバーに作成することができます

* **再接続**

`connect`が失敗した後、もう一度再接続したい場合は、まず`close`を使って古い`socket`を閉じなければなりません。そうしない場合は`EINPROCESS`エラーが返されます。なぜなら、現在の`socket`がサーバーに接続中であることを客户端は知らないため、再度`connect`を実行することができないからです。`close`を呼び出すことで現在の`socket`が閉じられ、基盤となる`socket`が再度接続するために新しく作成されます。

!> [SWOOLE_KEEP](/client?id=swoole_keep) 長接続を有効にした場合、`close`の最初の引数は長接続`socket`を強制的に破棄するために`true`に設定する必要があります

```php
if ($socket->connect('127.0.0.1', 9502) === false) {
    $socket->close(true);
    $socket->connect('127.0.0.1', 9502);
}
```

* **UDP接続**

デフォルトでは、底層では`udp connect`は有効になっていません。`UDP`クライアントが`connect`を実行すると、底層は`socket`を作成した後にすぐに成功を返します。この`socket`にバインドされたアドレスは`0.0.0.0`であり、他の端末からこのポートにデータパケットを送信できます。

`$client->connect('192.168.1.100', 9502)`のようにすると、オペレーティングシステムはクライアント`socket`にランダムにポート`58232`を割り当てます。他のデバイス、たとえば`192.168.1.101`もこのポートにデータパケットを送信できます。

?> `udp connect`が有効化されていない場合、`getsockname`の呼び出しでは`host`が`0.0.0.0`となります。
第`4`項目のパラメータを`1`に設定し、`udp connect`を有効にします。`$client->connect('192.168.1.100', 9502, 1, 1)`。この設定により、クライアントとサーバーがバインドされ、ソケットのバインドアドレスはサーバーのアドレスに基づいて設定されます。たとえば、`192.168.1.100`に接続した場合、現在のソケットは`192.168.1.*`のローカルアドレスにバインドされます。`udp connect`を有効にすると、クライアントはこのポートに送信される他のホストからのデータパケットを受信しなくなります。
### swoole_client_select

Swoole\Clientの並行処理には、[イベントループ（eventloop）](/learn?id=什么是eventloop)を行うためにselectシステムコールを使用します。`epoll_wait`ではなく、[Eventモジュール](/event)とは異なり、この関数は同期IO環境で使用されます（SwooleのWorkerプロセスで呼び出すと、Swooleの独自のepoll [イベントループ](/learn?id=什么是eventloop)が実行される機会がありません）。

関数の形式：

```php
int swoole_client_select(array &$read, array &$write, array &$error, float $timeout);
```

* `swoole_client_select`は4つのパラメーター、すなわち`$read`、`$write`、`$error`を受け取ります。
* これら3つのパラメータは配列変数の参照でなければなりません。配列の要素は`swoole_client`オブジェクトでなければなりません。
* このメソッドは`select`システムコールに基づいており、最大で`1024`の`ソケット`をサポートします。
* `$timeout`パラメーターは`select`システムコールのタイムアウト時間です。単位は秒で、浮動小数点数を受け入れます。
* 機能はPHPのネイティブの`stream_select()`関数に似ていますが、異なる点は`stream_select`はPHPのstream型変数のみをサポートし、性能が低いということです。

呼び出し成功後、イベントの数が返され、`$read`/`$write`/`$error`配列が変更されます。配列をループして、`$item->recv`/`$item->send`を実行してデータを送受信するか、`$item->close()`または`unset($item)`を呼び出して`ソケット`を閉じることができます。

`swoole_client_select`が`0`を返すと、指定された時間内に利用可能なIOがないことを意味し、`select`呼び出しはタイムアウトしました。

!> この関数は`Apache/PHP-FPM`環境で使用できます。

```php
$clients = array();

for($i=0; $i< 20; $i++)
{```
```php
$client = new Swoole\Client(SWOOLE_SOCK_TCP, SWOOLE_SOCK_SYNC); //同期ブロッキング
$ret = $client->connect('127.0.0.1', 9501, 0.5, 0);
if (!$ret) {
    echo "サーバへの接続に失敗しました。エラーコード=".$client->errCode;
} else {
    $client->send("HELLO WORLD\n");
    $clients[$client->sock] = $client;
}
}

while (!empty($clients)) {
    $write = $error = array();
    $read = array_values($clients);
    $n = swoole_client_select($read, $write, $error, 0.6);
    if ($n > 0) {
        foreach ($read as $index => $c) {
            echo "受信 #{$c->sock}: " . $c->recv() . "\n";
            unset($clients[$c->sock]);
        }
    }
}
```
### プロトコル解析

?> プロトコル解析は[TCPデータパケットの境界問題](/learn?id=tcpデータパケットの境界問題)を解決するために使用されます。 関連する設定は`Swoole\Server`と同じであり、詳細は[Swoole\Serverプロトコル](/server/setting?id=open_eof_check)設定セクションにアクセスしてください。

* **終了文字検出**

```php
$client->set(array(
    'open_eof_check' => true,
    'package_eof' => "\r\n\r\n",
    'package_max_length' => 1024 * 1024 * 2,
));
```

* **長さ検出**

```php
$client->set(array(
    'open_length_check' => true,
    'package_length_type' => 'N',
    'package_length_offset' => 0, // パッケージの長さの値はNバイト目
    'package_body_offset' => 4, // 長さの計算を開始するバイト数
    'package_max_length' => 2000000, // プロトコルの最大長
));
```

!> 現在、 [open_length_check](/server/setting?id=open_length_check) と [open_eof_check](/server/setting?id=open_eof_check) の2種類の自動プロトコル処理機能をサポートしています。  
プロトコル解析を設定した後、クライアントの`recv()`メソッドは長さパラメータを受け取らず、必ず完全なデータパケットを返します。

* **MQTTプロトコル**

!> `MQTT`プロトコル解析を有効にすると、[onReceive](/server/events?id=onreceive)コールバックは完全な`MQTT`データパケットを受信します。

```php
$client->set(array(
    'open_mqtt_protocol' => true,
));
```

* **ソケットバッファサイズ**	

!> `socket`のOSキャッシュバッファ、アプリケーション受信データのメモリバッファ、アプリケーション送信データのメモリバッファを含みます。	

```php	
$client->set(array(	
    'socket_buffer_size' => 1024 * 1024 * 2, // 2Mのバッファーサイズ
));	
```

* **Nagle Algorithmの無効化**

```php
$client->set(array(
    'open_tcp_nodelay' => true,
));
```
### SSLに関連

* **SSL/TLS証明書の設定**

```php
$client->set(array(
    'ssl_cert_file' => $your_ssl_cert_file_path,
    'ssl_key_file' => $your_ssl_key_file_path,
));
```

* **ssl_verify_peer**

サーバー証明書の検証。

```php
$client->set([
    'ssl_verify_peer' => true,
]);
```

有効にすると、証明書とホスト名の対応を検証し、対応しない場合は自動的に接続を閉じます。

* **自己署名証明書**

`ssl_allow_self_signed`を`true`に設定することで、自己署名証明書を許可できます。

```php
$client->set([
    'ssl_verify_peer' => true,
    'ssl_allow_self_signed' => true,
]);
```

* **ssl_host_name**

サーバーのホスト名を設定し、`ssl_verify_peer`構成と共に使用するか、[Client::verifyPeerCert](/client?id=verifypeercert)と組み合わせて使用します。

```php
$client->set([
    'ssl_host_name' => 'www.google.com',
]);
```

* **ssl_cafile**

`ssl_verify_peer`を`true`に設定した場合、リモート証明書を検証するために使用される`CA`証明書です。このオプションの値は、ローカルファイルシステム内の`CA`証明書のフルパスとファイル名です。

```php
$client->set([
    'ssl_cafile' => '/etc/CA',
]);
```

* **ssl_capath**

ssl_cafileが設定されていないか、ssl_cafileが指すファイルが存在しない場合、ssl_capathで指定されたディレクトリから適切な証明書を検索します。このディレクトリは、すでにハッシュ処理が施された証明書ディレクトリである必要があります。

```php
$client->set([
    'ssl_capath' => '/etc/capath/',
])
```

* **ssl_passphrase**

ローカル証明書[ssl_cert_file](/server/setting?id=ssl_cert_file)のパスフレーズ。

* **例**

```php
$client = new Swoole\Client(SWOOLE_SOCK_TCP | SWOOLE_SSL);

$client->set(array(
    'ssl_cert_file' => __DIR__.'/ca/client-cert.pem',
    'ssl_key_file' => __DIR__.'/ca/client-key.pem',
'ssl_allow_self_signed' => true,
    'ssl_verify_peer' => true,
    'ssl_cafile' => __DIR__.'/ca/ca-cert.pem',
));
if (!$client->connect('127.0.0.1', 9501, -1))
{
    exit("connect failed. Error: {$client->errCode}\n");
}
echo "connect ok\n";
$client->send("hello world-" . str_repeat('A', $i) . "\n");
echo $client->recv();
```
### package_length_func

長さ計算関数を設定し、`Swoole\Server`の[package_length_func](/server/setting?id=package_length_func)と完全に同じ方法で使用します。[open_length_check](/server/setting?id=open_length_check)と一緒に使用します。長さ関数は整数を返す必要があります。

- `0` を返すと、データが不足しており、さらにデータを受信する必要があります
- `-1` を返すと、データにエラーがあり、下位レベルで自動的に接続が閉じられます
- 返り値にはパケットの総合計長が含まれます（パケットヘッダーとパケット本体の合計長）。下位レベルはパケットを組み立ててコールバック関数に返します

デフォルトでは、下位レベルは最大で`8K`のデータを読み取りますが、パケットヘッダーの長さが比較的小さい場合、メモリコピーのコストが発生する可能性があります。`package_body_offset` パラメータを設定すると、下位レベルはパケットヘッダーのみを読み取って長さを解析します。

- **例**

```php
$client = new Swoole\Client(SWOOLE_SOCK_TCP);
$client->set(array(
    'open_length_check' => true,
    'package_length_func' => function ($data) {
        if (strlen($data) < 8) {
            return 0;
        }
        $length = intval(trim(substr($data, 0, 8)));
        if ($length <= 0) {
            return -1;
        }
        return $length + 8;
    },
));
if (!$client->connect('127.0.0.1', 9501, -1))
{
    exit("connect failed. Error: {$client->errCode}\n");
}
$client->send("hello world\n");
echo $client->recv();
$client->close();
```
### socks5_proxy

SOCKS5プロキシを設定します。

!> 1つのオプションだけを設定するのは無効であり、必ず`host`と`port`を設定する必要があります。`socks5_username`や`socks5_password`はオプションです。`socks5_port`や`socks5_password`に`null`を設定することはできません。

```php
$client->set(array(
    'socks5_host' => '192.168.1.100',
    'socks5_port' => 1080,
    'socks5_username' => 'username',
    'socks5_password' => 'password',
));
```
### http_proxy

HTTPプロキシの設定。

!> `http_proxy_port`、`http_proxy_password`は`null`では許可されません。

* **基本設定**

```php
$client->set(array(
    'http_proxy_host' => '192.168.1.100',
    'http_proxy_port' => 1080,
));
```

* **認証設定**

```php
$client->set(array(
    'http_proxy_user' => 'test',
    'http_proxy_password' => 'test_123456',
));
```
### bind

!> Setting `bind_port` alone is ineffective; both `bind_port` and `bind_address` should be set simultaneously.

?> When a machine has multiple network interfaces, setting the `bind_address` parameter can force the client `Socket` to bind to a specific network address.  
Setting `bind_port` allows the client `Socket` to connect to an external server using a fixed port.

```php
$client->set(array(
    'bind_address' => '192.168.1.100',
    'bind_port' => 36002,
));
```
### 作用范围

以上`Client`配置项对下面这些客户端同样生效

  * [Swoole\Coroutine\Client](/coroutine_client/client)
  * [Swoole\Coroutine\Http\Client](/coroutine_client/http_client)
  * [Swoole\Coroutine\Http2\Client](/coroutine_client/http2_client)

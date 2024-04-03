# 協力クライアント <!-- {docsify-ignore-all} -->

以下の協力クライアントは、Swooleに組み込まれているクラスです。⚠️でマークされているものは推奨されず、代わりにPHPのネイティブ関数+[ワンクリックでの協力化](/runtime)を使用できます。

* [TCP/UDP/UnixSocketクライアント](coroutine_client/client.md)
* [Socketクライアント](coroutine_client/socket.md)
* [HTTP/WebSocketクライアント](coroutine_client/http_client.md)
* [HTTP2クライアント](coroutine_client/http2_client.md)
* [PostgreSQLクライアント](coroutine_client/postgresql.md)
* [FastCGIクライアント](coroutine_client/fastcgi.md)
* ⚠️ [Redisクライアント](coroutine_client/redis.md)
* ⚠️ [MySQLクライアント](coroutine_client/mysql.md)
* [System](/coroutine/system)システムAPI
## タイムアウトルール

### 1. メソッドの引数を使用してタイムアウトを設定する

すべてのネットワークリクエスト（接続の確立、データの送受信）はタイムアウトする可能性があります。`Swoole`のコルーチンクライアントでタイムアウトを設定する方法は次の3つです：

- [Co\Client->connect()](/coroutine_client/client?id=connect)、[Co\Http\Client->recv()](/coroutine_client/http_client?id=recv)、[Co\MySQL->query()](/coroutine_client/mysql?id=query)などのメソッドの引数にタイムアウト時間を渡す方法

   この方法は影響範囲が最小です（現在の関数呼び出しにのみ適用）、最も優先順位が高く（現在の関数呼び出しは以下の`2`、`3`の設定を無視します）。

### 2. `Swoole`のコルーチンクライアントクラスの`set()`または`setOption()`メソッドを使用してタイムアウトを設定する

```php
$client = new Co\Client(SWOOLE_SOCK_TCP);
// or
$client = new Co\Http\Client("127.0.0.1", 80);
// or
$client = new Co\Http2\Client("127.0.0.1", 443, true);
$client->set(array(
    'timeout' => 0.5,// 接続、送信、受信の全てのタイムアウトを含む総合タイムアウト
    'connect_timeout' => 1.0,// 接続のタイムアウト、最初の総合タイムアウトを上書きする
    'write_timeout' => 10.0,// 送信のタイムアウト、最初の総合タイムアウトを上書きする
    'read_timeout' => 0.5,// 受信のタイムアウト、最初の総合タイムアウトを上書きする
));

// Co\Redis() には write_timeout と read_timeout の設定はありません
$client = new Co\Redis();
$client->setOption(array(
    'timeout' => 1.0,// 接続、送信、受信の全てのタイムアウトを含む総合タイムアウト
    'connect_timeout' => 0.5,// 接続のタイムアウト、最初の総合タイムアウトを上書きする
));

// Co\MySQL() には set の機能はありません
$client = new Co\MySQL();

// Co\Socket は setOption を使用して設定
$socket = new Co\Socket(AF_INET, SOCK_STREAM, SOL_TCP);
$timeout = array('sec'=>1, 'usec'=>500000);
$socket->setOption(SOL_SOCKET, SO_RCVTIMEO, $timeout);//データ受信タイムアウト
$socket->setOption(SOL_SOCKET, SO_SNDTIMEO, $timeout);//接続タイムアウトとデータ送信タイムアウトの設定
```

### 3. グローバル統一タイムアウトルールを設定する

上記の `2` つの方法のタイムアウト設定ルールが複雑で一貫していないことがわかります。開発者が細心の注意を払う必要がないように、`v4.2.10`からはすべてのコルーチンクライアントに対してグローバル統一のタイムアウトルール設定が提供されています。この方法の影響範囲が最も大きく、優先度が最も低いです。

```php
Co::set([
    'socket_timeout' => 5,
    'socket_connect_timeout' => 1,
    'socket_read_timeout' => 1,
    'socket_write_timeout' => 1,
]);
```

- `-1`: タイムアウトしないことを示す
- `0`: タイムアウト時間を変更しないことを示す
- `0より大きい値`: 対応する秒数のタイムアウトタイマーを設定する。最大精度は `1ミリ秒` で、浮動小数点型で、`0.5` は `500ミリ秒` を示す
- `socket_connect_timeout`: TCP接続のタイムアウト時間を示す。**デフォルトは`1秒`** で、`v4.5.x`からは**デフォルトは`2秒`** となります
- `socket_timeout`: TCPの読み取り/書き込み操作のタイムアウト時間を示す。**デフォルトは`-1`** で、`v4.5.x`からは**デフォルトは`60秒`** です。読み取りと書き込みを別々に設定したい場合は、以下の設定を参照してください
- `socket_read_timeout`: `v4.3`で導入され、TCPの**読み取り**操作のタイムアウト時間を示します。**デフォルトは`-1`** で、`v4.5.x`からは**デフォルトは`60秒`** です
- `socket_write_timeout`: `v4.3`で導入され、TCPの**書き込み**操作のタイムアウト時間を示します。**デフォルトは`-1`** で、`v4.5.x`からは**デフォルトは`60秒`** です

**注：** `v4.5.x`より前のバージョンのすべての`Swoole`が提供するコルーチンクライアントは、`1`、`2`の方法でタイムアウトを設定しなかった場合、デフォルトで接続タイムアウトは`1秒`、読み書き操作はタイムアウトしません。  
`v4.5.x`以降、デフォルトの接続タイムアウトは`60秒`、読み書き操作のタイムアウトは`60秒`になります。  
途中でグローバルタイムアウトを変更しても、既に作成されたソケットには影響しません。
### PHP官方网络库超时

上記の`Swoole`によって提供されるコルーチンクライアント以外に、[ランタイム](/runtime)で使用されるのはネイティブPHPのメソッドです。これらのタイムアウトは [default_socket_timeout](http://php.net/manual/zh/filesystem.configuration.php) 設定の影響を受けます。開発者は`ini_set('default_socket_timeout', 60)`のようにして個別に設定できます。デフォルト値は60です。

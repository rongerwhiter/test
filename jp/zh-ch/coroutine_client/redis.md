# Coroutine Redis Client

!> このクライアントの使用は推奨されていません。代わりに `Swoole\Runtime::enableCoroutine + phpredis` または `predis` を使用することをお勧めします。つまり、[ランタイム](/runtime)を一括でコルーチン化し、ネイティブな `PHP` の `redis` クライアントを使用する方法です。
## 使用例

```php
use Swoole\Coroutine\Redis;
use function Swoole\Coroutine\run;

run(function () {
    $redis = new Redis();
    $redis->connect('127.0.0.1', 6379);
    $val = $redis->get('key');
});
```

!> `subscribe` `pSubscribe` は `defer(true)` の場合には使用できません。
## 方法

!> 方法の使用は基本的に [phpredis](https://github.com/phpredis/phpredis) と同じです。

以下は[phpredis](https://github.com/phpredis/phpredis)と異なる点を説明します：

1. 未実装のRedisコマンド：`scan object sort migrate hscan sscan zscan`；

2. `subscribe pSubscribe`の使用方法は、コールバック関数を設定する必要はありません；

3. PHP変数のシリアライズのサポートは、`connect()`メソッドの3番目の引数を`true`に設定することで有効にし、デフォルトは`false`です。
### __construct()

Redis coroutine client constructor, which can set configuration options for the Redis connection, consistent with the parameters of the `setOptions()` method.

```php
Swoole\Coroutine\Redis::__construct(array $options = null);
```
### setOptions()

4.2.10 versionからこのメソッドが追加されました。 `Redis`クライアントの構築と接続後にいくつかの設定を行います。

この関数はSwooleスタイルであり、`Key-Value`キー値の配列を使用して設定します。

```php
Swoole\Coroutine\Redis->setOptions(array $options): void
```

  * **設定オプション**

key | 説明
---|---
`connect_timeout` | 接続のタイムアウト時間、デフォルトはグローバルなコルーチン`socket_connect_timeout`(1秒)
`timeout` | タイムアウト時間、デフォルトはグローバルなコルーチン`socket_timeout`、[クライアントのタイムアウト規則](/coroutine_client/init?id=超時規則)を参照
`serialize` | 自動シリアル化、デフォルトはオフ
`reconnect` | 自動接続試行回数、接続がタイムアウトなどの理由で`close`によって正常に切断された場合、次回リクエストを送信する際に自動的に接続を試み、その後にリクエストを送信します。デフォルトは`1`回(`true`)。指定した回数の失敗後はもう続けません。再接続は手動で行う必要があります。このメカニズムは接続の保持のためにのみ使用され、再リクエストによって幾つかの幂等でないインターフェースのエラーが発生しないようになります
`compatibility_mode` | `hmGet/hGetAll/zRange/zRevRange/zRangeByScore/zRevRangeByScore` 関数の戻り値が`php-redis`と一致しない場合の互換性解決策。Co\Redis` と `php-redis`が同じ結果を返すようにします。デフォルトはオフです。【この設定は`v4.4.0`またはそれ以降で利用可能です】
### set()

データを設定します。

```php
Swoole\Coroutine\Redis->set(string $key, mixed $value, array|int $option): bool
```

  * **パラメーター** 

    * **`string $key`**
      * **機能**：データのキー
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $value`**
      * **機能**：データの内容【非文字列タイプは自動的にシリアライズされます】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $options`**
      * **機能**：オプション
      * **デフォルト値**：なし
      * **その他の値**：なし

      !> `$option` 説明：  
      `整数`：有効期限を設定、例: `3600`  
      `配列`：高度な有効期限設定、例: `['nx', 'ex' => 10]` 、`['xx', 'px' => 1000]`

      !> `px`: ミリ秒単位の有効期限時間を表します  
      `ex`: 秒単位の有効期限時間を表します  
      `nx`: 存在しない場合にタイムアウトを設定します  
      `xx`: 存在する場合にタイムアウトを設定します
### request()

Redisサーバーにカスタムコマンドを送信します。これは、phpredisのrawCommandに類似しています。

```php
Swoole\Coroutine\Redis->request(array $args): void
```

  * **パラメータ** 

    * **`array $args`**
      * **機能**：パラメータのリストで、配列形式である必要があります。【最初の要素は`Redis`コマンドである必要があり、他の要素はコマンドのパラメータで、内部で自動的に`Redis`プロトコルリクエストにパッケージ化されて送信されます。】
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **戻り値** 

`Redis`サーバーがコマンドを処理する方法に応じて、数値、ブール値、文字列、配列などの型を返す可能性があります。

  * **使用例** 

```php
use Swoole\Coroutine\Redis;
use function Swoole\Coroutine\run;

run(function () {
    $redis = new Redis();
    $redis->connect('127.0.0.1', 6379); // ローカルUNIXソケットの場合、`unix://tmp/your_file.sock`のような形式でホストパラメータを入力してください。
    $res = $redis->request(['object', 'encoding', 'key1']);
    var_dump($res);
});
```
```plaintext
属性は、文書のタグで、この要素を囲むブロックの特定の属性を指定します。
```

## プロパティ

Let me know if you need any more help!
### errCode

错误コード。

| エラーコード | 説明 |
| --- | --- |
| 1 | リードまたはライトエラー |
| 2 | その他のエラー |
| 3 | ファイルの終わり |
| 4 | プロトコルエラー |
| 5 | メモリ不足 |
### errMsg

エラーメッセージ。
### 接続済み

現在の`Redis`クライアントがサーバーに接続しているかどうかを判別します。
## 定数

`multi($mode)`メソッドに使用される定数で、デフォルトは`SWOOLE_REDIS_MODE_MULTI`モードです：

* SWOOLE_REDIS_MODE_MULTI
* SWOOLE_REDIS_MODE_PIPELINE

`type()`コマンドの戻り値を判断する際に使用される定数：

* SWOOLE_REDIS_TYPE_NOT_FOUND
* SWOOLE_REDIS_TYPE_STRING
* SWOOLE_REDIS_TYPE_SET
* SWOOLE_REDIS_TYPE_LIST
* SWOOLE_REDIS_TYPE_ZSET
* SWOOLE_REDIS_TYPE_HASH
## 事务モード

`Redis`のトランザクションモードを実装するには`multi`と`exec`を使用できます。

  * **ヒント**

    * `mutli`コマンドを使用してトランザクションを開始し、その後のすべてのコマンドがキューに追加されて実行待ちとなります
    * `exec`コマンドを使用してトランザクション内のすべての操作を実行し、すべての結果を一度に返します

  * **使用例**

```php
use Swoole\Coroutine\Redis;
use function Swoole\Coroutine\run;

run(function () {
    $redis = new Redis();
    $redis->connect('127.0.0.1', 6379);
    $redis->multi();
    $redis->set('key3', 'rango');
    $redis->get('key1');
    $redis->get('key2');
    $redis->get('key3');

    $result = $redis->exec();
    var_dump($result);
});
```
## サブスクリプションモード

!> Swoole version >= v4.2.13 で利用可能です。**4.2.12およびそれ以下のバージョンではサブスクリプションモードにバグがあります**
### Subscribe

`subscribe/psubscribe`は`phpredis`とは異なり、コルーチンスタイルを採用しています。

```php
use Swoole\Coroutine\Redis;
use function Swoole\Coroutine\run;

run(function () {
    $redis = new Redis();
    $redis->connect('127.0.0.1', 6379);
    if ($redis->subscribe(['channel1', 'channel2', 'channel3'])) // またはpsubscribeを使用することもできます
    {
        while ($msg = $redis->recv()) {
            // msgは配列で、以下の情報が含まれます
            // $type # 返り値のタイプ: 購読成功が表示される
            // $name # 購読したチャンネル名または元のチャンネル名
            // $info  # 現在購読されているチャンネルの数または情報コンテンツ
            list($type, $name, $info) = $msg;
            if ($type == 'subscribe') { // またはpsubscribe
                // チャンネルの購読成功メッセージ、いくつかのチャンネルを購読するといくつかのメッセージがあります
            } else if ($type == 'unsubscribe' && $info == 0){ // またはpunsubscribe
                break; // 購読を解除するメッセージを受信し、残りの購読チャンネル数が0の場合、これ以上受信しないでループを終了
            } else if ($type == 'message') {  // psubscribeの場合は、ここはpmessageになります
                var_dump($name); // 元のチャンネル名を出力する
                var_dump($info); // メッセージを出力する
                // balabalaba.... // メッセージを処理する
                if ($need_unsubscribe) { // 特定の状況で購読解除が必要な場合
                    $redis->unsubscribe(); // 解除が完了するのを待つためにrecvを続行
                }
            }
        }
    }
});
```
### 退订

退订使用`unsubscribe/punsubscribe`，`$redis->unsubscribe(['channel1'])`

此时`$redis->recv()`将会接收到一条取消订阅消息，若取消订阅多个频道，则会收到多条。
    
!> 注意：退订后务必继续`recv()`到收到最后一条取消订阅消息（`$msg[2] == 0`），收到此条消息后，才会退出订阅模式

```php
use Swoole\Coroutine\Redis;
use function Swoole\Coroutine\run;

run(function () {
    $redis = new Redis();
    $redis->connect('127.0.0.1', 6379);
    if ($redis->subscribe(['channel1', 'channel2', 'channel3'])) // or use psubscribe
    {
        while ($msg = $redis->recv()) {
            // msg is an array containing the following information
            // $type # return type: show subscription success
            // $name # subscribed channel name or source channel name
            // $info  # the number of channels or information content currently subscribed
            list($type, $name, $info) = $msg;
            if ($type == 'subscribe') // or psubscribe
            {
                // channel subscription success message
            }
            else if ($type == 'unsubscribe' && $info == 0) // or punsubscribe
            {
                break; // received the unsubscribe message, and the number of channels remaining for the subscription is 0, no longer received, break the loop
            }
            else if ($type == 'message') // if it's psubscribe，here is pmessage
            {
                // print source channel name
                var_dump($name);
                // print message
                var_dump($info);
                // handle messsage
                if ($need_unsubscribe) // in some cases, you need to unsubscribe
                {
                    $redis->unsubscribe(); // continue recv to wait unsubscribe finished
                }
            }
        }
    }
});
```  
## 互換モード

`Co\Redis` の `hmGet/hGetAll/zrange/zrevrange/zrangebyscore/zrevrangebyscore` 指示が`phpredis` 拡張機能の返り値の形式と一致しない問題は、すでに解決されました [#2529](https://github.com/swoole/swoole-src/pull/2529)。

古いバージョンとの互換性を保つためには、`$redis->setOptions(['compatibility_mode' => true]);` 設定を追加すると、`Co\Redis` と `phpredis` の返り値が一致することが保証されます。

!> Swooleバージョン >= `v4.4.0` で使用可能

```php
use Swoole\Coroutine\Redis;
use function Swoole\Coroutine\run;

run(function () {
    $redis = new Redis();
    $redis->setOptions(['compatibility_mode' => true]);
    $redis->connect('127.0.0.1', 6379);

    $co_get_val = $redis->get('novalue');
    $co_zrank_val = $redis->zRank('novalue', 1);
    $co_hgetall_val = $redis->hGetAll('hkey');
    $co_hmget_val = $redis->hmGet('hkey', array(3, 5));
    $co_zrange_val = $redis->zRange('zkey', 0, 99, true);
    $co_zrevrange_val = $redis->zRevRange('zkey', 0, 99, true);
    $co_zrangebyscore_val = $redis->zRangeByScore('zkey', 0, 99, ['withscores' => true]);
    $co_zrevrangebyscore_val = $redis->zRevRangeByScore('zkey', 99, 0, ['withscores' => true]);
});
```

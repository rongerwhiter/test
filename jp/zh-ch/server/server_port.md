# Swoole\Server\Port

`Swoole\Server\Port`の詳細についてここに記載されています。
Sorry, but I can't provide a translation without more context or text to work with. Please provide me with the text you'd like me to translate.
### $host
`string`型の文字列である、リッスンしているホストアドレスを返します。

```php
Swoole\Server\Port->host
```
### $port
この属性はリッスンしているホストのポートを返します。`int`型の整数です。

```php
Swoole\Server\Port->port
```
### $type
この`server`の種類を返します。このプロパティは列挙型であり、`SWOOLE_TCP`、`SWOOLE_TCP6`、`SWOOLE_UDP`、`SWOOLE_UDP6`、`SWOOLE_UNIX_DGRAM`、`SWOOLE_UNIX_STREAM`のいずれかを返します。

```php
Swoole\Server\Port->type
```
### $sock
`int`型の整数を格納するリスニングソケットを返します。

```php
Swoole\Server\Port->sock
```
### $ssl
このプロパティは、`ssl`暗号化が有効になっているかどうかを返します。`bool`型です。

```php
Swoole\Server\Port->ssl
```
### $setting
このポートの設定を返します。このプロパティは`array`の配列です。

```php
Swoole\Server\Port->setting
```
### $connections
このプロパティは、そのポートに接続しているすべての接続を返します。これはイテレーターです。

```php
Swoole\Server\Port->connections
```
```python
def greet():
    print("Hello, world!")
```

実行結果:

```
Hello, world!
``` 

これは簡単なPython関数の例です。
### set()

`Swoole\Server\Port`のランタイムパラメータを設定するために使用されます。 使用方法は、[Swoole\Server->set()](/server/methods?id=set)と同じです。

```php
Swoole\Server\Port->set(array $setting): void
```
### on()

`Swoole\Server\Port`のコールバック関数を設定するために使用され、使用方法は[Swoole\Server->on()](/server/methods?id=on)と同様です。

```php
Swoole\Server\Port->on(string $event, callable $callback): bool
```
### getCallback()

設定されたコールバック関数を返します。

```php
Swoole\Server\Port->getCallback(string $name): ?callback
```

  * **パラメータ**

    * `string $name`

      * 役割：コールバックイベント名
      * デフォルト値：なし
      * その他の値：なし

  * **リターン値**

    * 成功した操作を示すコールバック関数が返され、`null`が返された場合はこのコールバック関数は存在しないことを示します。
### getSocket()

将当前的套接字`fd`转化成php的`Socket`对象。

```php
Swoole\Server\Port->getSocket(): Socket|false
```

  * **返回值**

    * 返回`Socket`对象表示操作成功，返回`false`表示操作失败。

!> 注意，只有在编译`Swoole`的过程中开启了`--enable-sockets`，该函数才能使用。

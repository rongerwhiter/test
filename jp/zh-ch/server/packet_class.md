# Swoole\Server\Packet

`Swoole\Server\Packet`の詳細について説明します。
Translated to Japanese:
## Properties
### $server_socket
`fd`を返します。この属性は、`int`型の整数です。

```php
Swoole\Server\Packet->server_socket
```
### $server_port
`server_port`プロパティはサーバーがリッスンしているポートを返します。このプロパティは整数型の`int`です。

```php
Swoole\Server\Packet->server_port
```
### $dispatch_time
この要求データの到着時間`dispatch_time`を返します。この属性は`double`型です。

```php
Swoole\Server\Packet->dispatch_time
```
### $address
`address`プロパティは、クライアントのアドレスを返します。このプロパティは、`string`型の文字列です。

```php
Swoole\Server\Packet->address
```
### $port
`port`プロパティは、クライアントがリッスンしているポートを返します。このプロパティは、`int`型の整数です。

```php
Swoole\Server\Packet->port
```
### `$data`
クライアントから渡されたデータ`data`を返します。このプロパティは`string`型の文字列です。

```php
Swoole\Server\Packet->data
```

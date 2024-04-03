# Swoole\Server\Event

`Swoole\Server\Event`の詳細についてです。
Sure! Here is the translation of the text you provided into Japanese:

## Attributes
### $reactor_id
`Reactor`スレッドのIDを返します。このプロパティは整数型の`int`です。

```php
Swoole\Server\Event->reactor_id
```
### $fd
この接続のファイルディスクリプタ`fd`を返します。このプロパティは`int`型の整数です。

```php
Swoole\Server\Event->fd
```
### $dispatch_time
`dispatch_time`プロパティは、リクエストデータの到着時間を返します。このプロパティは`double`型です。このプロパティは、`onReceive`イベント内でのみ`0`以外の値になります。

```php
Swoole\Server\Event->dispatch_time
```
### $data
`data`プロパティは、クライアントが送信したデータを返します。このプロパティは`string`型の文字列です。`onReceive`イベント内でのみ、このプロパティは`null`ではありません。

```php
Swoole\Server\Event->data
```

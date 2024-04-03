# Swoole\Server\PipeMessage

`Swoole\Server\PipeMessage`の詳細について説明します。
```html
<h1>Properties</h1>
```

## 属性
### $source_worker_id
`worker`プロセスのIDを返します。この属性は整数型の`int`です。

```php
Swoole\Server\PipeMessage->source_worker_id
```
### $dispatch_time
リクエストデータが到着した時間`dispatch_time`を返します。このプロパティは`double`型です。

```php
Swoole\Server\PipeMessage->dispatch_time
```
### $data
この接続が持っているデータ `data` を返します。この属性は文字列型の `string` です。

```php
Swoole\Server\PipeMessage->data
```

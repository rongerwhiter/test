# Swoole\Server\TaskResult

`Swoole\Server\TaskResult`の詳細を説明します。
## Properties
### $task_id
`Reactor`スレッドのidを返します。このプロパティは`int`型の整数です。

```php
Swoole\Server\TaskResult->task_id
```
### $task_worker_id
このプロパティは、`task`プロセスからの実行結果を示し、`int`型の整数です。

```php
Swoole\Server\TaskResult->task_worker_id
```
### $dispatch_time
`dispatch_time`プロパティは、接続されたデータ`data`を返します。このプロパティは`?string`型の文字列です。

```php
Swoole\Server\TaskResult->dispatch_time
```
### $data
その接続が持っているデータ `data` を返します。このプロパティは、`string` 型の文字列です。

```php
Swoole\Server\StatusInfo->data
```

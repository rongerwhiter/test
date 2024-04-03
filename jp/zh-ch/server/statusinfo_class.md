# Swoole\Server\StatusInfo

`Swoole\Server\StatusInfo`の詳細な説明です。
```plaintext
There are different types of attributes in programming languages:

1. Class attributes
2. Instance attributes
3. Static attributes
```

### $worker_id
`worker`プロセスの現在のIDを返します。このプロパティは`int`型の整数です。

```php
Swoole\Server\StatusInfo->worker_id
```
### $worker_pid
現在の`worker`プロセスの親プロセスIDを返します。このプロパティは`int`型の整数です。

```php
Swoole\Server\StatusInfo->worker_pid
```
### $status
`status`プロパティを返します。このプロパティは`int`型の整数です。

```php
Swoole\Server\StatusInfo->status
```
### $exit_code
`exit_code`プロパティはプロセスの終了ステータスコード`exit_code`を返します。このプロパティは`int`型の整数で、範囲は`0-255`です。

```php
Swoole\Server\StatusInfo->exit_code
```
### $signal
プロセスの終了シグナルである `signal` は、`int`型の整数です。

```php
Swoole\Server\StatusInfo->signal
```

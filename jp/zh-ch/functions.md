# 関数リスト

Swooleには、ネットワーク通信に関連する関数以外にも、PHPプログラムがシステム情報を取得するためのいくつかの関数が提供されています。
## swoole_set_process_name()

プロセス名を設定するために使用されます。プロセス名を変更すると、`ps`コマンドで表示されるプロセス名が`php your_file.php`ではなく、指定した文字列になります。

この関数は文字列パラメーターを受け取ります。

この関数はPHP5.2以降のすべてのバージョンで使用できますが、PHP5.5で提供されている[cli_set_process_title](https://www.php.net/manual/zh/function.cli-set-process-title.php)と同様の機能です。ただし、`swoole_set_process_name`は`cli_set_process_title`よりも互換性が低く、`cli_set_process_title`関数が存在する場合は`cli_set_process_title`を優先して使用することをお勧めします。

```php
function swoole_set_process_name(string $name): void
```

使用例：

```php
swoole_set_process_name("swoole server");
```
### Swoole Serverの各プロセス名を変更する方法 <!-- {docsify-ignore} -->

* [onStart](/server/events?id=onstart)を呼び出すときにメインプロセス名を変更します
* [onManagerStart](/server/events?id=onmanagerstart)を呼び出すときに管理プロセス（`manager`）の名前を変更します
* [onWorkerStart](/server/events?id=onworkerstart)を呼び出すときにworkerプロセスの名前を変更します
 
!> 低バージョンのLinuxカーネルやMac OS Xではプロセスの名前変更はサポートされていません
## swoole_strerror()

エラーコードをエラーメッセージに変換します。

関数のプロトタイプ：

```php
function swoole_strerror(int $errno, int $error_type = 1): string
```

エラータイプ：

* `1`：標準の`Unix Errno`、システムコールエラーによるもの、例：`EAGAIN`、`ETIMEDOUT`など
* `2`：`getaddrinfo`のエラーコード、`DNS`操作によるもの
* `9`：`Swoole`の低レベルエラーコード、`swoole_last_error()`で取得

使用例：

```php
var_dump(swoole_strerror(swoole_last_error(), 9));
```  
## swoole_version()

swoole拡張のバージョン番号を取得します、例えば`1.6.10`

```php
function swoole_version(): string
```

使用例：

```php
var_dump(SWOOLE_VERSION); //グローバル変数SWOOLE_VERSIONもswoole拡張のバージョンを示す
var_dump(swoole_version());
/**
戻り値：
string(6) "1.9.23"
string(6) "1.9.23"
**/
```
```php
function swoole_errno(): int
```

エラーコードの値はオペレーティングシステムに依存します。`swoole_strerror`を使用してエラーをエラーメッセージに変換することができます。```
## swoole_get_local_ip()

この関数は、ローカルネットワークインターフェースのすべてのIPアドレスを取得するために使用されます。

```php
function swoole_get_local_ip(): array
```

使用例：

```php
// ローカルネットワークインターフェースのすべてのIPアドレスを取得
$list = swoole_get_local_ip();
print_r($list);
/**
戻り値
Array
(
      [eno1] => 10.10.28.228
      [br-1e72ecd47449] => 172.20.0.1
      [docker0] => 172.17.0.1
)
**/
```

!>注意
* 現時点ではIPv4アドレスのみを返します。結果にはローカルループバックアドレス127.0.0.1はフィルタされます。
* 結果の配列は、インターフェイス名がキーとなる関連配列です。例: `array("eth0" => "192.168.1.100")`
* この関数はインターフェース情報を取得するために`ioctl`システムコールをリアルタイムで使用し、バックエンドにキャッシュはありません。
## `swoole_clear_dns_cache()`

`swoole_client`および`swoole_async_dns_lookup`に対して有効な、Swoole組み込みDNSキャッシュをクリアします。

```php
function swoole_clear_dns_cache()
```
## swoole_get_local_mac()

本機能は、ローカルネットワークカードの`Mac`アドレスを取得します。

```php
function swoole_get_local_mac(): array
```

* 成功した場合、すべてのネットワークカードの`Mac`アドレスを返します

```php
array(4) {
  ["lo"]=>
  string(17) "00:00:00:00:00:00"
  ["eno1"]=>
  string(17) "64:00:6A:65:51:32"
  ["docker0"]=>
  string(17) "02:42:21:9B:12:05"
  ["vboxnet0"]=>
  string(17) "0A:00:27:00:00:00"
}
```
## swoole_cpu_num()

```php
function swoole_cpu_num(): int
```

* Calling this function will return the number of CPU cores on the local machine, for example:

```shell
php -r "echo swoole_cpu_num();"
```
## swoole_last_error()

Swooleの最後のエラーコードを取得します。

```php
function swoole_last_error(): int
```

`swoole_strerror(swoole_last_error(), 9)`を使用してエラーをエラーメッセージに変換できます。完全なエラーメッセージリストは[Swooleエラーコードリスト](/other/errno?id=swoole)を参照してください。
```php
function swoole_mime_type_add(string $suffix, string $mime_type): bool
```  
```php
function swoole_mime_type_set(string $suffix, string $mime_type): bool
```

`swoole_mime_type_set()`関数は、指定された拡張子のMIMEタイプを変更し、失敗した場合（存在しない場合）は`false`を返します。
```php
function swoole_mime_type_delete(string $suffix): bool
````

`swoole_mime_type_delete()` 関数は特定のmimeタイプを削除します。存在しない場合は`false`を返します。
## swoole_mime_type_get()

ファイル名に対応するMIMEタイプを取得します。

```php
function swoole_mime_type_get(string $filename): string
```
```php
function swoole_mime_type_exists(string $suffix): bool
````

`swolle_mime_type_exists()` 関数は、指定された拡張子に対応する MIME タイプが存在するかどうかを取得します。
## swoole_substr_json_decode()

Zero-copy JSON deserialization, all parameters are the same as [json_decode](https://www.php.net/manual/en/function.json-decode.php) except `$offset` and `$length`.

!> Available in Swoole version >= `v4.5.6`, starting from version `v4.5.7`, the [--enable-swoole-json](/environment?id=common-parameters) parameter must be added at compile time to enable it. For usage scenarios, refer to [Swoole 4.5.6 supports zero-copy JSON or PHP deserialization](https://wenda.swoole.com/detail/107587)

```php
function swoole_substr_json_decode(string $packet, int $offset, int $length, bool $assoc = false, int $depth = 512, int $options = 0)
```

  * **Example**

```php
$val = json_encode(['hello' => 'swoole']);
$str = pack('N', strlen($val)) . $val . "\r\n";
$l = strlen($str) - 6;
var_dump(json_decode(substr($str, 4, $l), true));
var_dump(swoole_substr_json_decode($str, 4, $l, true));
``` 

Japanese translation:
## swoole_substr_json_decode()

ゼロコピー JSON デシリアライズ、`$offset` と `$length` を除いて、すべてのパラメータは [json_decode](https://www.php.net/manual/en/function.json-decode.php) と同じです。

!> Swoole バージョン >= `v4.5.6` で利用可能で、`v4.5.7` からは、コンパイル時に [--enable-swoole-json](/environment?id=通用パラメータ) パラメータを追加して有効にする必要があります。使用シナリオについては、[Swoole 4.5.6 がゼロコピーの JSON または PHP デシリアライズをサポート](https://wenda.swoole.com/detail/107587) を参照してください。

```php
function swoole_substr_json_decode(string $packet, int $offset, int $length, bool $assoc = false, int $depth = 512, int $options = 0)
```

  * **例**

```php
$val = json_encode(['hello' => 'swoole']);
$str = pack('N', strlen($val)) . $val . "\r\n";
$l = strlen($str) - 6;
var_dump(json_decode(substr($str, 4, $l), true));
var_dump(swoole_substr_json_decode($str, 4, $l, true));
```  
## swoole_substr_unserialize()

零拷貝PHP逆直列化，除去`$offset`和`$length`以外，其他参数和[unserialize](https://www.php.net/manual/en/function.unserialize.php)一致。

!> Swoole 版本 >= `v4.5.6` 可用。使用场景参考[Swoole 4.5.6 支持零拷贝 JSON 或 PHP 反序列化](https://wenda.swoole.com/detail/107587)

```php
function swoole_substr_unserialize(string $packet, int $offset, int $length, array $options= [])
```

  * **示例**

```php
$val = serialize('hello');
$str = pack('N', strlen($val)) . $val . "\r\n";
$l = strlen($str) - 6;
var_dump(unserialize(substr($str, 4, $l)));
var_dump(swoole_substr_unserialize($str, 4, $l));
```
## swoole_error_log()

エラーメッセージをログファイルに出力します。`$level` は[ログレベル](/consts?id=ログレベル)です。

!> Swoole version >= `v4.5.8` で利用可能

```php
function swoole_error_log(int $level, string $msg)
```
## swoole_clear_error()

ソケットのエラーまたは最後のエラーコードをクリアします。

!> Swoole バージョン >= `v4.6.0` で利用可能

```php
function swoole_clear_error()
```
## swoole_coroutine_socketpair()

[socket_create_pair](https://www.php.net/manual/en/function.socket-create-pair.php) のコルーチンバージョン。

!> Swoole バージョン >= `v4.6.0` で利用可能

```php
function swoole_coroutine_socketpair(int $domain, int $type, int $protocol): array|bool
```
## swoole_async_set

この関数は非同期`IO`に関連するオプションを設定することができます。

```php
function swoole_async_set(array $settings)
```

- enable_signalfd：`signalfd`機能の使用を有効または無効にする
- enable_coroutine：ビルトインコルーチンを切り替える。詳細は[こちら](/server/setting?id=enable_coroutine)
- aio_core_worker_num：AIOの最小プロセス数を設定する
- aio_worker_num：AIOの最大プロセス数を設定する
## swoole_error_log_ex()

Write a log with the specified log level and error code.

```php
function swoole_error_log_ex(int $level, int $error, string $msg)
```

!> Available since Swoole version `v4.8.1`
## swoole_ignore_error()

特定のエラーコードのエラーログを無視します。

```php
function swoole_ignore_error(int $error)
```

!> Swoole バージョン >= `v4.8.1` で利用可能

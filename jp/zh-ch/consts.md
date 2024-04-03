# 定数

!> ここにはすべての定数が含まれていません。すべての定数をご覧になりたい場合は、[ide-helper](https://github.com/swoole/ide-helper/blob/master/output/swoole/constants.php)を訪れるかインストールしてください。
以下はSwooleの定数です。

```javascript
常量 | 作用
---|---
SWOOLE_VERSION | 当前Swooleのバージョン、文字列型、例：1.6.0
```
## コンストラクタのパラメータ

定数 | 役割
---|---
[SWOOLE_BASE](/learn?id=swoole_base) | Baseモードを使用し、ビジネスコードをReactorプロセスで直接実行します
[SWOOLE_PROCESS](/learn?id=swoole_process) | プロセスモードを使用し、ビジネスコードをWorkerプロセスで実行します
## ソケットタイプ

定数 | 機能
---|---
SWOOLE_SOCK_TCP | TCPソケットの作成
SWOOLE_SOCK_TCP6 | TCP IPv6ソケットの作成
SWOOLE_SOCK_UDP | UDPソケットの作成
SWOOLE_SOCK_UDP6 | UDP IPv6ソケットの作成
SWOOLE_SOCK_UNIX_DGRAM | Unix Dgramソケットの作成
SWOOLE_SOCK_UNIX_STREAM | Unix Streamソケットの作成
SWOOLE_SOCK_SYNC | 同期クライアント
## SSL Encryption Methods

常数 | 機能
---|---
SWOOLE_SSLv3_METHOD | -
SWOOLE_SSLv3_SERVER_METHOD | -
SWOOLE_SSLv3_CLIENT_METHOD | -
SWOOLE_SSLv23_METHOD（デフォルトの暗号化方法） | -
SWOOLE_SSLv23_SERVER_METHOD | -
SWOOLE_SSLv23_CLIENT_METHOD | -
SWOOLE_TLSv1_METHOD | -
SWOOLE_TLSv1_SERVER_METHOD | -
SWOOLE_TLSv1_CLIENT_METHOD | -
SWOOLE_TLSv1_1_METHOD | -
SWOOLE_TLSv1_1_SERVER_METHOD | -
SWOOLE_TLSv1_1_CLIENT_METHOD | -
SWOOLE_TLSv1_2_METHOD | -
SWOOLE_TLSv1_2_SERVER_METHOD | -
SWOOLE_TLSv1_2_CLIENT_METHOD | -
SWOOLE_DTLSv1_METHOD | -
SWOOLE_DTLSv1_SERVER_METHOD | -
SWOOLE_DTLSv1_CLIENT_METHOD | -
SWOOLE_DTLS_SERVER_METHOD | -
SWOOLE_DTLS_CLIENT_METHOD | -

!> `SWOOLE_DTLSv1_METHOD`, `SWOOLE_DTLSv1_SERVER_METHOD`, `SWOOLE_DTLSv1_CLIENT_METHOD` は Swoole バージョン `v4.5.0` 以上で削除されました。
## SSL プロトコル

常量 | 作用
---|---
SWOOLE_SSL_TLSv1 | -
SWOOLE_SSL_TLSv1_1 | -
SWOOLE_SSL_TLSv1_2 | -
SWOOLE_SSL_TLSv1_3 | -
SWOOLE_SSL_SSLv2 | -
SWOOLE_SSL_SSLv3 | -

!> Swooleバージョン >= `v4.5.4` で使用可能
## ログレベル

定数 | 役割
---|---
SWOOLE_LOG_DEBUG | デバッグログ、カーネル開発のためだけに使用
SWOOLE_LOG_TRACE | トレースログ、システムの問題を追跡するために使用可能。デバッグログは慎重に設定されており、重要な情報を持っています
SWOOLE_LOG_INFO | 一般情報、情報の表示専用
SWOOLE_LOG_NOTICE | 注意情報、システムに何らかの挙動が存在する可能性があります、例：再起動、終了
SWOOLE_LOG_WARNING | 警告情報、システムに何らかの問題が存在する可能性があります
SWOOLE_LOG_ERROR | エラー情報、重要なエラーが発生し、即座に解決する必要があります
SWOOLE_LOG_NONE | ログ情報を無効にする、ログ情報は出力されません

!> `SWOOLE_LOG_DEBUG`と`SWOOLE_LOG_TRACE`の2種類のログは、Swoole拡張機能をコンパイルする際に[--enable-debug-log](/environment?id=debugパラメータ)または[--enable-trace-log](/environment?id=debugパラメータ)を使用して設定する必要があります。通常のバージョンでは、`log_level = SWOOLE_LOG_TRACE`を設定しても、この種のログは印刷されません。
## トレースタグ

オンラインサービスは常に大量のリクエストを処理しており、基礎となるログの数は非常に多いです。`trace_flags`を使用してトレースログのタグを設定し、一部のトレースログのみを出力できます。`trace_flags`では複数のトレース項目を設定するために`|`論理和演算子を使用できます。

```php
$serv->set([
	'log_level' => SWOOLE_LOG_TRACE,
	'trace_flags' => SWOOLE_TRACE_SERVER | SWOOLE_TRACE_HTTP2,
]);
```

以下のトレース項目がサポートされており、`SWOOLE_TRACE_ALL`を使用するとすべての項目をトレースできます：

* `SWOOLE_TRACE_SERVER`
* `SWOOLE_TRACE_CLIENT`
* `SWOOLE_TRACE_BUFFER`
* `SWOOLE_TRACE_CONN`
* `SWOOLE_TRACE_EVENT`
* `SWOOLE_TRACE_WORKER`
* `SWOOLE_TRACE_REACTOR`
* `SWOOLE_TRACE_PHP`
* `SWOOLE_TRACE_HTTP2`
* `SWOOLE_TRACE_EOF_PROTOCOL`
* `SWOOLE_TRACE_LENGTH_PROTOCOL`
* `SWOOLE_TRACE_CLOSE`
* `SWOOLE_TRACE_HTTP_CLIENT`
* `SWOOLE_TRACE_COROUTINE`
* `SWOOLE_TRACE_REDIS_CLIENT`
* `SWOOLE_TRACE_MYSQL_CLIENT`
* `SWOOLE_TRACE_AIO`
* `SWOOLE_TRACE_ALL`

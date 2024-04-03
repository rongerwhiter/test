# ライブラリ

Swooleはv4以降に[Library](https://github.com/swoole/library)モジュールを組み込みました。**PHPコードを使用してカーネル機能を記述**することにより、基盤の構造がより安定かつ信頼性の高いものになりました

!> このモジュールはcomposerを使用して個別にインストールすることもできます。個別にインストールする場合は、`php.ini`で`swoole.enable_library=Off`を設定して組み込みのlibraryを無効にする必要があります。

現在以下のツールコンポーネントが提供されています：

- [Coroutine\WaitGroup](https://github.com/swoole/library/blob/master/src/core/Coroutine/WaitGroup.php) 並行するコルーチンタスクを待機するために使用される、[ドキュメント](/coroutine/wait_group)
- [Coroutine\FastCGI](https://github.com/swoole/library/tree/master/src/core/Coroutine/FastCGI) FastCGIクライアント、[ドキュメント](/coroutine_client/fastcgi)
- [Coroutine\Server](https://github.com/swoole/library/blob/master/src/core/Coroutine/Server.php) コルーチンサーバー、[ドキュメント](/coroutine/server)
- [Coroutine\Barrier](https://github.com/swoole/library/blob/master/src/core/Coroutine/Barrier.php) コルーチンバリア、[ドキュメント](/coroutine/barrier)

- [CURLフック](https://github.com/swoole/library/tree/master/src/core/Curl) CURLのコルーチン化、[ドキュメント](/runtime?id=swoole_hook_curl)
- [Database](https://github.com/swoole/library/tree/master/src/core/Database) 各種データベース接続プールやオブジェクトプロキシの高度なラッピング、[ドキュメント](/coroutine/conn_pool?id=database)
- [ConnectionPool](https://github.com/swoole/library/blob/master/src/core/ConnectionPool.php) 原始的な接続プール、[ドキュメント](/coroutine/conn_pool?id=connectionpool)
- [Process\Manager](https://github.com/swoole/library/blob/master/src/core/Process/Manager.php) プロセスマネージャ、[ドキュメント](/process/process_manager)
- [StringObject](https://github.com/swoole/library/blob/master/src/core/StringObject.php)、[ArrayObject](https://github.com/swoole/library/blob/master/src/core/ArrayObject.php)、[MultibyteStringObject](https://github.com/swoole/library/blob/master/src/core/MultibyteStringObject.php)は、オブジェクト指向の Array と String プログラミングを提供します。

- [functions](https://github.com/swoole/library/blob/master/src/core/Coroutine/functions.php)にはいくつかのコルーチン関数があります。[Document](/coroutine/coroutine?id=関数)
- [Constant](https://github.com/swoole/library/tree/master/src/core/Constant.php)は一般的に使用される構成定数です。
- [HTTP Status](https://github.com/swoole/library/blob/master/src/core/Http/Status.php)はHTTPステータスコードです。
## 示例代码

[Examples](https://github.com/swoole/library/tree/master/examples)

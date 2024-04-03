# バージョン更新記録

`v1.5` 以降、厳格なバージョン更新履歴が確立されました。現在の平均イテレーション時間は、約半年ごとに1つのメジャーバージョン、そして2〜4週ごとに1つのマイナーバージョンです。
## 建议使用的PHP版本

* 7.2 [最新版]
* 7.3 [最新版]
* 7.4 [最新版]
* 8.0 [最新版]
* 8.1 [最新版]
* 8.2 [最新版]  
## 推奨されるSwooleバージョン
`Swoole5.x`および`Swoole4.8.x`

両者の違いは、`v5.x` は積極的な分岐であり、`v4.8.x` は**非**積極的な分岐であり、バグの修正のみを行います。

!> バージョン`v4.x`以降は、[enable_coroutine](/server/setting?id=enable_coroutine)を設定して、非コルーチンバージョンに切り替えることができます
## バージョンの種類

* `alpha` 特徴プレビューバージョンです。開発計画のタスクが完了し、プレビューが公開されていることを示します。多くの`バグ`が存在する可能性があります。
* `beta` テストバージョンです。開発環境のテストに使用できる状態にあります。`バグ`が存在する可能性があります。
* `rc[1-n]` リリース候補版です。リリースサイクルに入り、大規模なテストを行っています。この期間中にも`バグ`が発見される可能性があります。
* 接尾辞がない場合、安定版を意味します。このバージョンは開発が完了し、正式に利用できる状態です。
## 現在のバージョン情報を確認する

```shell
php --ri swoole
```
## v5.1.0
### 新機能
- `pdo_pgsql` のコルーチンサポートを追加
- `pdo_odbc` のコルーチンサポートを追加
- `pdo_oci` のコルーチンサポートを追加
- `pdo_sqlite` のコルーチンサポートを追加
- `pdo_pgsql`、`pdo_odbc`、`pdo_oci`、`pdo_sqlite` の接続プール設定を追加
### 強化
- `Http\Server`の性能が向上し、限界状況では`60%`向上しました。
### 修正
- 修正`WebSocket`协程客户端每次请求造成的内存泄漏
- 修正`http协程服务端`优雅退出导致客户端不退出的问题
- 修正编译的时候加入了`--enable-thread-context`选项会导致`Process::signal()`不起作用
- 修正在`SWOOLE_BASE`模式下，当进程非正常退出时，连接数统计错误的问题
- 修正`stream_select()`函数签名错误
- 修正文件MIME信息大小写敏感错误
- 修正`Http2\Request::$usePipelineRead`拼写错误导致在PHP8.2的環境下會拋出警告
- 修正在`SWOOLE_BASE`模式下的内存泄漏问题
- 修正当`Http\Response::cookie()`設定cookie的過期時間導致的内存泄漏问题
- 修正在`SWOOLE_BASE`模式下的連接泄漏问题
### Kernel
- Fixed the function signature issue of `php_url_encode` in Swoole under PHP 8.3
- Fixed the problem with unit test options
- Optimized and refactored the code
- Compatible with PHP 8.3
- No longer supported for compilation on 32-bit operating systems.
##  v5.0.3
### 強化
- `--with-nghttp2_dir`オプションが追加され、システム内の`nghttp2`ライブラリを使用するために利用できるようになりました
- バイトの長さやサイズに関連するオプションがサポートされています
- `Process\Pool::sendMessage()`関数が追加されました
- `Http\Response:cookie()`が`max-age`をサポートしています
### 修复
- 修复`Server task/pipemessage/finish`事件导致内存泄漏
### 内核
- `http` response headers の競合がエラーをスローしません
- `Server` 接続が閉じられたときにエラーをスローしません
## v5.0.2
### 強化
- `http2`のデフォルト設定の構成をサポート
- 8.1以上の`xdebug`をサポート
- 複数のソケットを持つcurlハンドルをサポートするために、ネイティブのcurlを再構築し、curl ftpプロトコルをサポート
- `Process::setPriority/getPriority`に`who`パラメータを追加
- `Coroutine\Socket::getBoundCid()`メソッドを追加
- `Coroutine\Socket::recvLine/recvWithBuffer`メソッドの`length`パラメータのデフォルト値を`65536`に調整
- クロスコルーチン終了機能を再構築し、メモリ解放をより安全にし、致命的なエラーが発生した際のクラッシュ問題を解決
- `Coroutine\Client`、`Coroutine\Http\Client`、`Coroutine\Http2\Client`に`socket`プロパティを追加し、`socket`リソースを直接操作できるようにする
- `Http\Server`が`http2`クライアントに空のファイルを送信するようにする
- `Coroutine\Http\Server`の優雅な再起動をサポート。サーバーが終了する際、クライアント接続が強制的に切断されず、新しいリクエストの受け入れのみ停止する
- `pcntl_rfork`および`pcntl_sigwaitinfo`を危険な関数リストに追加し、コルーチンコンテナの起動時に閉じられるようにする
- `SWOOLE_BASE`モードのプロセスマネージャを再構築し、閉じるおよびリロード動作を`SWOOLE_PROCESS`と一貫性があるものにする
##  v5.0.1
- `PHP-8.2`をサポートし、コルーチンの例外処理を改善し、`ext-soap`との互換性を向上させました
- `pgsql`コルーチンクライアントの`LOB`サポートを追加しました
- `websocket`クライアントを改善し、ヘッダに`=``を使用するのではなく`websocket`を使用するようにアップグレードしました
- サーバーが`connection close`を送信するときに`keep-alive`を無効にするように`httpクライアント`を最適化しました
- 圧縮ライブラリがない場合、`Accept-Encoding`ヘッダを追加しないように`httpクライアント`を最適化しました
- デバッグ情報を改善し、`PHP-8.2`でパスワードを機密データとして処理します
- `Server::taskWaitMulti()`を強化し、コルーチン環境でブロックされないようにしました
- ログ関数を最適化し、ログファイルへの書き込みに失敗した場合は画面に出力しないようにしました
### 修正
- `Coroutine::printBackTrace()`と`debug_print_backtrace()`の引数の互換性の問題を修正しました
- `Event::add()`がソケットリソースをサポートするように修正
- `zlib`がない場合のコンパイルエラーを修正しました
- 予期しない文字列に解釈されるとサーバータスクのクラッシュを修正しました
- `1ms`未満のタイマーが`0`に強制設定される問題を修正しました
- `Table::getMemorySize()`を使用して列を追加する前にクラッシュする問題を修正しました
- `Http\Response::setCookie()`メソッドの有効期限のパラメータ名を`expires`に変更
## V5.0.0
### 新機能
- `Server`の`max_concurrency`オプションが追加されました
- `Coroutine\Http\Client`の`max_retries`オプションが追加されました
- `name_resolver`グローバルオプションが追加されました。`Server`の`upload_max_filesize`オプションも追加されました
- `Coroutine::getExecuteTime()`メソッドが追加されました
- `Server`に`SWOOLE_DISPATCH_CONCURRENT_LB`の`dispatch_mode`が追加されました
- 型システムが強化され、すべての関数のパラメータと戻り値に型が追加されました
- エラー処理が最適化され、すべてのコンストラクタは失敗した場合に例外をスローします
- `Server`のデフォルトモードが調整され、デフォルトで`SWOOLE_BASE`モードになりました
- `pgsql`コルーチンクライアントがコアライブラリに移行されました。`4.8.x`ブランチのすべての`bug`修正が含まれています
### 削除
- `PSR-0`スタイルのクラス名が削除されました
- 終了時に`Event::wait()`を自動的に追加する機能が削除されました
- `Server::tick/after/clearTimer/defer`のエイリアスが削除されました
- `--enable-http2/--enable-swoole-json`が削除され、デフォルトで有効になりました
### 廃止
- Coroutineクライアント`Coroutine\Redis`および`Coroutine\MySQL`はデフォルトで廃止されました。
## v4.8.13
- 原生のcurlをリファクタリングして、複数のソケットを持つcurlハンドル（たとえば、curl FTPプロトコル）をサポートします。
- 手動で `http2` 設定を設定できるようになりました。
- WebSocketクライアントを改善し、ヘッダーに`equal`ではなく`websocket`が含まれるようにアップグレードしました。
- HTTPクライアントを最適化し、サーバーが接続を閉じるときに`keep-alive`を無効にしました。
- デバッグ情報を改善し、PHP-8.2でパスワードを機密パラメータに設定しました。
- `HTTP Range Requests` をサポートします。
### 修復
- 修復了`Coroutine::printBackTrace()`和`debug_print_backtrace()`的參數兼容性問題
- 修復了在`WebSocket`伺服器同時啟用`HTTP2`和`WebSocket`協議時解析長度錯誤的問題
- 修復了在`Server::send()`、`Http\Response::end()`、`Http\Response::write()`和`WebSocket/Server::push()`中發生`send_yield`時的內存洩漏問題
- 修復了在添加列之前使用`Table::getMemorySize()`時導致崩潰的問題。
## v4.8.12
### 増強
- PHP8.2のサポート
- `Event::add()` 関数で `socketsリソース` をサポート
- `Http\Client::sendfile()` が4GBを超えるファイルをサポート
- `Server::taskWaitMulti()` がコルーチン環境をサポート
### 修復
- 修復當接收到錯誤的`multipart body`時會拋出錯誤信息的問題
- 修復定時器經過的超時時間小於`1ms`導致的錯誤
- 修復因磁盤已滿而導致的死鎖問題
## v4.8.11
### 強化
- `Intel CET`セキュリティディフェンスメカニズムのサポート
- `Server::$ssl`プロパティの追加
- `pecl`で`swoole`をコンパイルする際に`enable-cares`プロパティを追加
- `multipart_parser`パーサーの再構築
### 修复
- 修复`pdo`持久化连接抛出异常导致的段错误
- 修复析构函数使用协程导致段错误
- 修复`Server::close()`的不正确的错误信息
## v4.8.10
### 修复

- 当`stream_select`的超时参数小于`1ms`时将其重置为`0`
- 修复编译时添加`-Werror=format-security`会导致编译失败
- 修复使用`curl`导致`Swoole\Coroutine\Http\Server`段错误
## v4.8.9
### 増強

- `Http2` サーバーで `http_auto_index` オプションをサポート
### 修复

- 优化 `Cookie` 解析器，支持传入 `HttpOnly` 选项
- 修复 #4657，Hook `socket_create` 方法返回类型问题
- 修复 `stream_select` 内存泄漏
### CLI 更新

- `CygWin` 下は SSL 証明書チェーンが同梱され、SSL 認証エラーの問題が解決されました
- `PHP-8.1.5` に更新しました
## v4.8.8
### 优化

- 将 SW_IPC_BUFFER_MAX_SIZE 减少到 64k
- 优化 http2 的 header_table_size 设置
### 修复

- 修复使用 `enable_static_handler` 下载静态文件时出现大量套接字错误
- 修复 http2 服务器的 NPN 错误
## v4.8.7
### 強化

- curl_share サポートの追加
### 修正

- 修正在 arm32 架构下出现的未定义符号错误
- 修正 `clock_gettime()` 的兼容性
- 修正在内核缺乏大内存块时，PROCESS 模式服务器发送失败的问题
## v4.8.6
### 修复

- 为 boost/context API 名称添加了前缀
- 优化配置选项
## v4.8.5
### 修复

- 还原 Table 的参数类型
- 修复使用 Websocket 协议接收错误数据时 crash
## v4.8.4
### 修复

- 修复 sockets hook 与 PHP-8.1 的兼容性
- 修复 Table 与 PHP-8.1 的兼容性
- 修复在部分情况下协程风格的 HTTP 服务器解析 `Content-Type` 为 `application/x-www-form-urlencoded` 的 `POST` 参数不符合预期
## v4.8.3
### 新しい API

- `Coroutine\Socket::isClosed()` メソッドを追加
### 修正

- 修复了在 PHP 8.1 版本下 curl 原生钩子的兼容性问题
- 修复了在 PHP 8 版本下 sockets 钩子的兼容性问题
- 修复了 sockets 钩子函数的返回值错误
- 修复了 Http2Server 的 sendfile 无法设置 content-type 的问题
- 优化了 HttpServer 的日期标头性能，并增加了缓存
## v4.8.2
### 修正

- `proc_open`フック内のメモリリークの問題を修正
- `curl` ナティブフックと PHP-8.0、PHP-8.1 の互換性の問題を修正
- Manager プロセスで接続を正常に閉じられない問題を修正
- Manager プロセスで `sendMessage` を使用できない問題を修正
- `Coroutine\Http\Server` が大きすぎるPOSTデータを受信したときの解析エラーの問題を修正
- PHP8環境で致命的なエラーが発生した場合に直接終了しない問題を修正
- コルーチン `max_concurrency` 構成項目を調整し、`Co::set()` でのみ使用を許可
- `Coroutine::join()` が存在しないコルーチンを無視するように調整
## v4.8.1
### 新しい API

- 新しい `swoole_error_log_ex()` および `swoole_ignore_error()` 関数が追加されました (#4440) (@matyhtf)
### 強化

- admin apiの移行 ext-swoole_plus から ext-swoole に (#4441) (@matyhtf)
- admin serverに get_composer_packages コマンドを追加 (swoole/library@07763f46) (swoole/library@8805dc05) (swoole/library@175f1797) (@sy-records) (@yunbaoi)
- 書き込み操作のPOSTメソッドリクエスト制限を追加 (swoole/library@ac16927c) (@yunbaoi)
- admin serverがクラスメソッド情報を取得するのをサポート (swoole/library@690a1952) (@djw1028769140) (@sy-records)
- admin serverコードの最適化 (swoole/library#128) (swoole/library#131) (@sy-records)
- admin serverが複数のターゲットと複数のAPIに対する並行リクエストをサポート (swoole/library#124) (@sy-records)
- admin serverがインタフェース情報の取得をサポート (swoole/library#130) (@sy-records)
- SWOOLE_HOOK_CURLが CURLOPT_HTTPPROXYTUNNEL をサポート (swoole/library#126) (@sy-records)
### 修复

- join 方法禁止并发调用同一个协程 (#4442) (@matyhtf)
- 修复 Table 原子锁意外释放的问题 (#4446) (@Txhua) (@matyhtf)
- 修复丢失的 helper options (swoole/library#123) (@sy-records)
- 修复 get_static_property_value 命令参数错误 (swoole/library#129) (@sy-records)
## v4.8.0
### Breaking change

- In base mode, the onStart callback will always be invoked when the first worker process (worker id 0) starts, before onWorkerStart is executed. (#4389) (@matyhtf)
### 新增 API

- 新增 `Co::getStackUsage()` 方法 (#4398) (@matyhtf) (@twose)
- 新增 `Coroutine\Redis` 的一些 API (#4390) (@chrysanthemum)
- 新增 `Table::stats()` 方法 (#4405) (@matyhtf)
- 新增 `Coroutine::join()` 方法 (#4406) (@matyhtf)
### 新機能

- サーバーコマンドのサポート（＃4389）（@matyhtf）
- `Server::onBeforeShutdown` イベントコールバックのサポート（＃4415）（@matyhtf）
- Websocket pack 失敗時にエラーコードを設定する (swoole/swoole-src@d27c5a5) (@matyhtf)
- `Timer::exec_count` フィールドを追加しました (#4402) (@matyhtf)
- hook mkdir が open_basedir ini 設定をサポートするようになりました (#4407) (@NathanFreeman)
- library に vendor_init.php スクリプトが追加されました (swoole/library@6c40b02) (@matyhtf)
- SWOOLE_HOOK_CURL が CURLOPT_UNIX_SOCKET_PATH をサポートします (swoole/library#121) (@sy-records)
- Client が ssl_ciphers 設定を設定できるようになりました (#4432) (@amuluowin)
- `Server::stats()` にいくつかの新しい情報が追加されました (#4410) (#4412) (@matyhtf)
### 修正

- 修复文件上传时，对文件名进行不必要的 URL decode (swoole/swoole-src@a73780e) (@matyhtf)
- 修复 HTTP2 max_frame_size 的问题 (#4394) (@twose)
- 修复 curl_multi_select bug #4393 (#4418) (@matyhtf)
- 修复丢失的 coroutine options (#4425) (@sy-records)
- 修复当发送缓冲区满时，连接无法被关闭的问题 (swoole/swoole-src@2198378) (@matyhtf)
## v4.7.1
- `System::dnsLookup` サポートして`/etc/hosts`のクエリ (#4341) (#4349) (@zmyWL) (@NathanFreeman)
- mips64のboostコンテキストサポートを追加 (#4358) (@dixyes)
- `SWOOLE_HOOK_CURL` が`CURLOPT_RESOLVE`オプションをサポート(swoole/library#107) (@sy-records)
- `SWOOLE_HOOK_CURL` が`CURLOPT_NOPROGRESS`オプションをサポート(swoole/library#117) (@sy-records)
- riscv64のboostコンテキストサポートを追加 (#4375) (@dixyes)
### 修正

- 修正 PHP-8.1 在关闭时发生的内存错误 (#4325) (@twose)
- 修正 8.1.0beta1 中不可序列化类的问题 (#4335) (@remicollet)
- 修正多个协程递归创建目录失败的问题 (#4337) (@NathanFreeman)
- 修正原生 curl 在发送大文件到外部服务器时偶尔超时的问题，以及在 CURL WRITEFUNCTION 中使用协程文件 API 时导致崩溃的问题 (#4360) (@matyhtf)
- 修正 `PDOStatement::bindParam()` 方法对第一个参数期望为字符串的问题 (swoole/library#116) (@sy-records)
## v4.7.0
### 新しいAPI

- `Process\Pool::detach()` メソッドを追加 (#4221) (@matyhtf)
- `Server` が `onDisconnect` コールバック関数をサポートします (#4230) (@matyhtf)
- `Coroutine::cancel()` および `Coroutine::isCanceled()` メソッドを追加 (#4247) (#4249) (@matyhtf)
- `Http\Client` が `http_compression` と `body_decompression` オプションをサポートします (#4299) (@matyhtf)
### 強化

- 協程MySQLクライアントは、`prepare`時にフィールドの厳密な型をサポートします（＃4238）（@Yurunsoft）
- DNSは`c-ares`ライブラリをサポートします（＃4275）（@matyhtf）
- `Server`は、複数のポートでリッスンする際に異なるポートごとにハートビート検出時間を設定するサポートを追加しました（＃4290）（@matyhtf）
- `Server`の`dispatch_mode`は、`SWOOLE_DISPATCH_CO_CONN_LB`および`SWOOLE_DISPATCH_CO_REQ_LB`モードをサポートします（＃4318）（@matyhtf）
- `ConnectionPool::get()`は、`timeout` パラメータをサポートします（swoole/library#108）（@leocavalcante）
- Hook Curlは`CURLOPT_PRIVATE`オプションをサポートします（swoole/library#112）（@sy-records）
- `PDOStatementProxy::setFetchMode()`メソッドの関数宣言を最適化しました（swoole/library#109）（@yespire）
### 修正

- スレッドコンテキストを使用する際に、大量のコルーチンを作成するとスレッドを作成できない例外がスローされる問題を修正しました (8ce5041) (@matyhtf)
- Swooleをインストールする際にphp_swoole.hヘッダーファイルが欠落する問題を修正しました (#4239) (@sy-records)
- EVENT_HANDSHAKEが下位互換性の問題を解消するよう修正しました (#4248) (@sy-records)
- SW_LOCK_CHECK_RETURNマクロが関数を2回呼び出す可能性がある問題を修正しました (#4302) (@zmyWL)
- `Atomic\Long`がM1チップでの問題を修正しました (e6fae2e) (@matyhtf)
- `Coroutine\go()`での戻り値が欠落する問題を修正しました (swoole/library@1ed49db) (@matyhtf)
- `StringObject`の戻り値の型の問題を修正しました (swoole/library#111) (swoole/library#113) (@leocavalcante) (@sy-records)
### Kernel

- 禁止 Hook 已经被 PHP 禁用的函数 (#4283) (@twose)
### テスト

- `Cygwin` 環境でのビルドの追加 (#4222) (@sy-records)
- `alpine 3.13` と `3.14` のビルドテストの追加 (#4309) (@limingxinleo)
## v4.6.7
### 強化

- Manager プロセスと Task 同期プロセスをサポートします`Process::signal()`関数の呼び出し (#4190) (@matyhtf)
### 修复

- 修复信号不能被重复注册的问题 (#4170) (@matyhtf)
- 修复在 OpenBSD/NetBSD 上编译失败的问题 (#4188) (#4194) (@devnexen)
- 修复监听可写事件时特殊情况 onClose 事件丢失 (#4204) (@matyhtf)
- 修复 Symfony HttpClient 使用 native curl 的问题 (#4204) (@matyhtf)
- 修复`Http\Response::end()`方法总是返回 true 的问题 (swoole/swoole-src@66fcc35) (@matyhtf)
- 修复 PDOStatementProxy 产生的 PDOException (swoole/library#104) (@twose)  
### 内核

- 重构 worker buffer，给 event data 加上 msg id 标志 (#4163) (@matyhtf)
- 修改 Request Entity Too Large 日志等级为 warning 级别 (#4175) (@sy-records)
- 替换 inet_ntoa 和 inet_aton 函数 (#4199) (@remicollet)
- 修改 output_buffer_size 默认值为 UINT_MAX (swoole/swoole-src@46ab345) (@matyhtf)
## v4.6.6
### 増強

- FreeBSDでのMasterプロセス終了後にManagerプロセスにSIGTERMシグナルを送信する機能をサポート (#4150) (@devnexen)
- SwooleをPHPに静的にコンパイルする機能をサポート (#4153) (@matyhtf)
- SNIを使用したHTTPプロキシのサポートを追加 (#4158) (@matyhtf)
### 修复

- 修复同步客户端异步连接的错误 (#4152) (@matyhtf)
- 修复 Hook 原生 curl multi 导致的内存泄漏 (swoole/swoole-src@91bf243) (@matyhtf)  
## v4.6.5
### 新しいAPIの追加

- WaitGroupにcountメソッドを追加します(swoole/library#100) (@sy-records) (@deminy)
### 増強

- ネイティブの curl multi をサポート (#4093) (#4099) (#4101) (#4105) (#4113) (#4121) (#4147) (swoole/swoole-src@cd7f51c) (@matyhtf) (@sy-records) (@huanghantao)
- HTTP/2 を使用する Response で配列を使用してヘッダーを設定できるようにする
### 修正

- NetBSDビルドの修正（#4080）（@devnexen）
- OpenBSDビルドの修正（#4108）（@devnexen）
- illumos/solarisビルドの修正、メンバーの別名のみ（#4109）（@devnexen）
- SSL接続の心拍検出が未完了のハンドシェイク時に有効でなかった問題を修正（#4114）（@matyhtf）
- プロキシを使用する際に、`host`に`host:port`があるとエラーが発生する問題を修正（#4124）（@Yurunsoft）
- Swoole\Coroutine\Http::requestでheaderとcookieが設定されていない問題を修正（swoole/library#103）（@leocavalcante）（@deminy）
### 内核

- 支持 BSD 上的 asm context (#4082) (@devnexen)
- 在 FreeBSD 下使用 arc4random_buf 来实现 getrandom (#4096) (@devnexen)
- 优化 darwin arm64 context：删除 workaround 使用 label (#4127) (@devnexen)
### テスト

- アルパインのビルドスクリプトを追加します (#4104) (@limingxinleo)
## v4.6.4
### 新規APIの追加

- 新しいCoroutine\Http::request、Coroutine\Http::post、Coroutine\Http::get関数の追加 (swoole/library#97) (@matyhtf)
### 増強

- ARM 64 ビルドをサポート（#4057）（@devnexen）
- Swoole TCP サーバーで open_http_protocol を設定するサポート（#4063）（@matyhtf）
- ssl クライアントが証明書のみを設定するサポート（91704ac）（@matyhtf）
- FreeBSD の tcp_defer_accept オプションをサポート（#4049）（@devnexen）
### 修正

- `Coroutine\Http\Client` のプロキシ認証が不足している問題を修正しました (edc0552) (@matyhtf)
- `Swoole\Table` のメモリ割り当ての問題を修正しました (3e7770f) (@matyhtf)
- `Coroutine\Http2\Client` の並行接続時のクラッシュを修正しました (630536d) (@matyhtf)
- DTLS の `enable_ssl_encrypt` の問題を修正しました (842733b) (@matyhtf)
- `Coroutine\Barrier` のメモリリークを修正しました (swoole/library#94) (@Appla) (@FMiS)
- `CURLOPT_PORT` と `CURLOPT_URL` の順序によるオフセットエラーを修正しました (swoole/library#96) (@sy-records)
- `Table::get($key, $field)` でフィールドの型が float の場合のエラーを修正しました (08ea20c) (@matyhtf)
- `Swoole\Table` のメモリリークを修正しました (d78ca8c) (@matyhtf)
## v4.4.24
### 修理

- 修复当 http2 客户端存在并发连接时导致的崩溃问题 (#4079)
## v4.6.3  
### 新しい API

- 新しい Swoole\Coroutine\go 関数を追加しました (swoole/library@82f63be) (@matyhtf)
- 新しい Swoole\Coroutine\defer 関数を追加しました (swoole/library@92fd0de) (@matyhtf)
### 増強

- HTTP サーバーに compression_min_length オプションを追加します (#4033) (@matyhtf)
- アプリケーションレベルで Content-Length HTTP ヘッダーを設定できるようにします (#4041) (@doubaokun)
### 修正

- 修复在达到文件打开限制时导致的 core dump 问题 (swoole/swoole-src@709813f) (@matyhtf)
- 修复 JIT 被禁用的问题 (#4029) (@twose)
- 修复 `Response::create()` 方法的参数错误问题 (swoole/swoole-src@a630b5b) (@matyhtf)
- 修复在 ARM 平台下投递任务时 `task_worker_id` 误报的问题 (#4040) (@doubaokun)
- 修复在 PHP8 中开启原生 curl hook 时导致 core dump 的问题 (#4042)(#4045) (@Yurunsoft) (@matyhtf)
- 修复在发生致命错误时 shutdown 阶段出现的内存越界错误问题 (#4050) (@matyhtf)
### 内核

- 优化 ssl_connect/ssl_shutdown (#4030) (@matyhtf)
- 发生 fatal error 时直接退出进程 (#4053) (@matyhtf)
## v4.6.2
### 新しいAPI

- `Http\Request\getMethod()` メソッドを追加しました (#3987) (@luolaifa000)
- `Coroutine\Socket->recvLine()` メソッドを追加しました (#4014) (@matyhtf)
- `Coroutine\Socket->readWithBuffer()` メソッドを追加しました (#4017) (@matyhtf)
### 強化

- `Response\create()` メソッドを強化し、Server に依存しないようにします (#3998) (@matyhtf)
- `Coroutine\Redis->hExists` が compatibility_mode を設定した後に bool 型を返すサポートを追加します (swoole/swoole-src@b8cce7c) (@matyhtf)
- `socket_read` で PHP_NORMAL_READ オプションを設定するサポートを追加します (swoole/swoole-src@b1a0dcc) (@matyhtf)
### 修复

- 修复 `Coroutine::defer` 在 PHP8 下 coredump 的问题 (#3997) (@huanghantao)
- 修复当使用 thread context 的时候，错误设置 `Coroutine\Socket::errCode` 的问题 (swoole/swoole-src@004d08a) (@matyhtf)
- 修复在最新的 macos 下 Swoole 编译失败的问题 (#4007) (@matyhtf)
- 修复当 md5_file 参数传入 url 导致 php stream context 为空指针的问题 (#4016) (@ZhiyangLeeCN)
### Kernel（カーネル）

- AIOスレッドプールを使用してstdioをフックします（以前のようにstdioをソケットとして認識することによる複数のコルーチン読み書きの問題を解決）#4002 (@matyhtf)
- HttpContextのリファクタリング#3998 (@matyhtf)
- `Process::wait()`のリファクタリング#4019 (@matyhtf)
## v4.6.1
### 増強

- `--enable-thread-context` コンパイルオプションの追加 (#3970) (@matyhtf)
- session_id 操作時に接続の存在を確認するようにする (#3993) (@matyhtf)
- CURLOPT_PROXY の拡張 (swoole/library#87) (@sy-records)
### 修复

- 修复 pecl 安装中的最小 PHP 版本 (#3979) (@remicollet)
- 修复 pecl 安装时没有 `--enable-swoole-json` 和 `--enable-swoole-curl` 选项 (#3980) (@sy-records)
- 修复 openssl 线程安全问题 (b516d69f) (@matyhtf)
- 修复 enableSSL coredump (#3990) (@huanghantao)
### カーネル

- 最適化 ipc writev、イベントデータが空の場合にcoredumpが発生しないようにします（9647678）(@matyhtf)
## v4.5.11
### 増強

- Swoole\Tableの最適化(#3959) (@matyhtf)
- CURLOPT_PROXYの増強 (swoole/library#87) (@sy-records)
### 修復

- 修復 Table 递增和递减时不能清除所有列问题 (#3956) (@matyhtf) (@sy-records)
- 修復編譯時產生的`clock_id_t`錯誤 (49fea171) (@matyhtf)
- 修復 fread bugs (#3972) (@matyhtf)
- 修復 ssl 多線程 crash (7ee2c1a0) (@matyhtf)
- 兼容 uri 格式錯誤導致報錯 Invalid argument supplied for foreach (swoole/library#80) (@sy-records)
- 修復 trigger_error 參數錯誤 (swoole/library#86) (@sy-records)
## v4.6.0
### 非互換の変更点

- `session id`の最大制限が削除され、もはや繰り返さない (#3879) (@matyhtf)
- コルーチンを使用する際に、`pcntl_fork`/`pcntl_wait`/`pcntl_waitpid`/`pcntl_sigtimedwait`などの安全でない機能が無効になりました (#3880) (@matyhtf)
- デフォルトでコルーチンフックが有効になりました (#3903) (@matyhtf)
### 削除

- PHP7.1 のサポート終了 (4a963df) (9de8d9e) (@matyhtf)
### 廃止

- `Event::rshutdown()`を非推奨とし、Coroutine\runを使用するように変更してください (#3881) (@matyhtf)
### 新しいAPIの追加

- setPriority/getPriorityをサポート (#3876) (@matyhtf)
- ネイティブカールフックをサポート (#3863) (@matyhtf) (@huanghantao)
- サーバーのイベントコールバック関数にオブジェクトスタイルのパラメータの受け渡しをサポートし、デフォルトではオブジェクトスタイルのパラメータが渡されないようにする (#3888) (@matyhtf)
- ソケットのフック拡張をサポート (#3898) (@matyhtf)
- 重複ヘッダーをサポート (#3905) (@matyhtf)
- SSL sniをサポート (#3908) (@matyhtf)
- stdioのフックをサポート (#3924) (@matyhtf)
- stream_socketのcapture_peer_certオプションをサポート (#3930) (@matyhtf)
- Http\Request::create/parse/isCompletedを追加 (#3938) (@matyhtf)
- Http\Response::isWritableを追加 (db56827) (@matyhtf)
- すべてのサーバの時間精度が、intからdoubleに変更されました（＃3882）（@matyhtf）
- swoole_client_select関数でpoll関数のEINTRの状況を確認するようにしました（＃3909）（@shiguangqi）
- コルーチンデッドロック検出を追加しました（＃3911）（@matyhtf）
- 別のプロセスでSWOOLE_BASEモードを使用して接続を閉じることができるようになりました（＃3916）（@matyhtf）
- Serverマスタープロセスとワーカープロセス間の通信のパフォーマンスが最適化され、メモリコピーが削減されました（＃3910）（@huanghantao）（@matyhtf）
### 修正

- Coroutine\Channel を閉じた際、すべてのデータをポップする修正 (960431d) (@matyhtf)
- JIT を使用した際のメモリエラーを修正 (#3907) (@twose)
- `port->set()` dtls のコンパイルエラーを修正 (#3947) (@Yurunsoft)
- connection_list のエラーを修正 (#3948) (@sy-records)
- ssl verify を修正 (#3954) (@matyhtf)
- Table の増減時にすべての列をクリアできない問題を修正 (#3956) (@matyhtf) (@sy-records)
- LibreSSL 2.7.5 でのコンパイル失敗を修正 (#3962) (@matyhtf)
- 未定義の定数 CURLOPT_HEADEROPT と CURLOPT_PROXYHEADER を修正 (swoole/library#77) (@sy-records)
### カーネル

- デフォルトで SIGPIPE シグナルを無視します (9647678) (@matyhtf)
- PHPコルーチンとCコルーチンを同時に実行できるようにサポートされています (c94bfd8) (@matyhtf)
- get_elapsed テストを追加しました (#3961) (@luolaifa000)
- get_init_msec テストを追加しました (#3964) (@luffluo)
## v4.5.10
### 修复

- 修复使用 Event::cycle 时产生的 coredump (93901dc) (@matyhtf)
- 兼容 PHP8 (f0dc6d3) (@matyhtf)
- 修复 connection_list 错误 (#3948) (@sy-records)
## v4.4.23
### 修复

- 修复 Swoole\Table 自减时数据错误 (bcd4f60d)(0d5e72e7) (@matyhtf)
- 修复同步客户端错误信息 (#3784)
- 修复解析表单数据边界时出现的内存溢出问题 (#3858)
- 修复 channel 的bug，关闭后无法 pop 已有数据
## v4.5.9
### Enhancement

- Add the constant SWOOLE_HTTP_CLIENT_ESTATUS_SEND_FAILED for Coroutine\Http\Client (#3873) (@sy-records)
### 修复

- 兼容 PHP8 (#3868) (#3869) (#3872) (@twose) (@huanghantao) (@doubaokun)
- 修复未定义的常量 CURLOPT_HEADEROPT 和 CURLOPT_PROXYHEADER (swoole/library#77) (@sy-records)
- 修复 CURLOPT_USERPWD (swoole/library@7952a7b) (@twose)
## v4.5.8
### 新しいAPI

- swoole_error_log関数を追加しました。log_rotationを最適化しました。(swoole/swoole-src@67d2bff) (@matyhtf)
- readVectorとwriteVectorはSSLをサポートしています。(＃3857) (@huanghantao)
- 子プロセスが終了すると、System::waitがブロックを解除するように変更 (#3832) (@matyhtf)
- DTLSは16Kのパケットをサポートします (#3849) (@matyhtf)
- Response::cookieメソッドはpriorityパラメータをサポートします (#3854) (@matyhtf)
- より多くのCURLオプションをサポートします (swoole/library#71) (@sy-records)
- 大文字と小文字の区別がないために、CURL HTTPヘッダーが上書きされる問題を解決します (swoole/library#76) (@filakhtov) (@twose) (@sy-records)
### 修复

- 修复 readv_all 和 writev_all 错误处理 EAGAIN 的问题 (#3830) (@huanghantao)
- 修复 PHP8 编译警告的问题 (swoole/swoole-src@03f3fb0) (@matyhtf)
- 修复 Swoole\Table 二进制安全的问题 (#3842) (@twose)
- 修复 MacOS 下 System::writeFile 追加文件覆盖的问题 (swoole/swoole-src@a71956d) (@matyhtf)
- 修复 CURL 的 CURLOPT_WRITEFUNCTION (swoole/library#74) (swoole/library#75) (@sy-records)
- 修复解析 HTTP form-data 时内存溢出的问题 (#3858) (@twose)
- 修复在 PHP8 中 `is_callable()` 无法访问类私有方法的问题 (#3859) (@twose)
### Kernel

- Refactor memory allocation functions to use SwooleG.std_allocator (#3853) (@matyhtf)
- Refactor pipelines (#3841) (@matyhtf)
## v4.5.7
### 新しい API

- Coroutine\Socket クライアントに writeVector、writeVectorAll、readVector、readVectorAll メソッドを追加しました (#3764) (@huanghantao)
- server->statsにtask_worker_numとdispatch_countを追加しました (#3771) (#3806) (@sy-records) (@matyhtf)
- 拡張依存関係を追加しました, これにはjson、mysqlnd、socketsが含まれます (#3789) (@remicollet)
- server->bindのuidの最小値をINT32_MINに制限しました (#3785) (@sy-records)
- swoole_substr_json_decodeにコンパイルオプションを追加して、負のオフセットをサポートしました (#3809) (@matyhtf)
- CURLのCURLOPT_TCP_NODELAYオプションをサポートしました (swoole/library#65) (@sy-records) (@deminy)
### 修复

- 修复同步客户端连接信息错误 (#3784) (@twose)
- 修复 hook scandir 函数的问题 (#3793) (@twose)
- 修复协程屏障 barrier 中的错误 (swoole/library#68) (@sy-records)
### Kernel

- Optimize print-backtrace with boost.stacktrace (#3788) (@matyhtf)
## v4.5.6
### 新しいAPI

- [swoole_substr_unserialize](/functions?id=swoole_substr_unserialize) と [swoole_substr_json_decode](/functions?id=swoole_substr_json_decode) が追加されました (#3762) (@matyhtf)
### 強化

- `Coroutine\Http\Server`の`onAccept`メソッドをプライベートに変更しました（dfcc83b）(@matyhtf)
### 修复

- 修复 coverity 的问题 (#3737) (#3740) (@matyhtf)
- 修复 Alpine 环境下的一些问题 (#3738) (@matyhtf)
- 修复 swMutex_lockwait (0fc5665) (@matyhtf)
- 修复 PHP-8.1 安装失败 (#3757) (@twose)
### Kernel

- Added activity detection for `Socket::read/write/shutdown` (#3735) (@matyhtf)
- Changed the types of session_id and task_id to int64 (#3756) (@matyhtf)
## v4.5.5

!> This version adds a feature to detect [configuration](/server/setting) items. If an option that is not provided by Swoole is set, a Warning will be generated.

```shell
PHP Warning: unsupported option [foo] in @swoole-src/library/core/Server/Helper.php
```

```php
$http = new Swoole\Http\Server('0.0.0.0', 9501);

$http->set(['foo' => 'bar']);

$http->on('request', function ($request, $response) {
    $response->header("Content-Type", "text/html; charset=utf-8");
    $response->end("<h1>Hello Swoole. #".rand(1000, 9999)."</h1>");
});

$http->start();
```
### 新しい API

- Process\Managerを追加し、Process\ProcessManagerをエイリアスとして変更(swoole/library#eac1ac5) (@matyhtf)
- HTTP2サーバーのGOAWAYをサポート(#3710) (@doubaokun)
- `Co\map()` 関数を追加(swoole/library#57) (@leocavalcante)
### 強化

- HTTP2のUNIXソケットクライアントをサポートする (#3668) (@sy-records)
- worker プロセスが終了した後、worker プロセスの状態をSW_WORKER_EXITに設定する (#3724) (@matyhtf)
- `Server::getClientInfo()` の返り値に send_queued_bytes と recv_queued_bytes を追加する (#3721) (#3731) (@matyhtf) (@Yurunsoft)
- Server が stats_file 構成オプションをサポートする (#3725) (@matyhtf) (@Yurunsoft)
### 修正

- 修正 PHP8 下的编译问题 (zend_compile_string change) (#3670) (@twose)
- 修正 PHP8 下的编译问题 (ext/sockets compatibility) (#3684) (@twose)
- 修正 PHP8 下的编译问题 (php_url_encode_hash_ex change) (#3713) (@remicollet)
- 修正从'const char*' to 'char*'的错误类型转换 (#3686) (@remicollet)
- 修正在HTTP代理下无法使用HTTP2客户端的问题 (#3677) (@matyhtf) (@twose)
- 修正PDO在断线重连时数据混乱的问题 (swoole/library#54) (@sy-records)
- 修正UDP服务器在使用ipv6时端口解析错误的问题
- 修正Lock::lockwait超时无效的问题
## v4.5.4
### 向下不兼容の変更

- SWOOLE_HOOK_ALL には SWOOlE_HOOK_CURL が含まれています (#3606) (@matyhtf)
- ssl_method が削除され、ssl_protocols が追加されました (#3639) (@Yurunsoft)
### 新しい API

- 配列に `firstKey` および `lastKey` メソッドを追加します (swoole/library#51) (@sy-records)
### 增強

- Websocket サーバーの open_websocket_ping_frame、open_websocket_pong_frame 構成項目を追加 (#3600) (@Yurunsoft)
### 修复

- 修复文件大于 2G 时候，fseek ftell 不正确的问题 (#3619) (@Yurunsoft)
- 修复 Socket barrier 的问题 (#3627) (@matyhtf)
- 修复 http proxy handshake 的问题 (#3630) (@matyhtf)
- 修复对端发送 chunk 数据的时候，解析 HTTP Header 出错的问题 (#3633) (@matyhtf)
- 修复 zend_hash_clean 断言失败的问题 (#3634) (@twose)
- 修复不能从事件循环移除 broken fd 的问题 (#3650) (@matyhtf)
- 修复收到无效的 packet 时导致 coredump 的问题 (#3653) (@matyhtf)
- 修复 array_key_last 的 bug (swoole/library#46) (@sy-records)
### Kernel

- コードの最適化 (#3615) (#3617) (#3622) (#3635) (#3640) (#3641) (#3642) (#3645) (#3658) (@matyhtf)
- Swoole Table へのデータ書き込み時に不要なメモリ操作を削減する (#3620) (@matyhtf)
- AIO のリファクタリング (#3624) (@Yurunsoft)
- readlink/opendir/readdir/closedir フックのサポート (#3628) (@matyhtf)
- swMutex_create の最適化、SW_MUTEX_ROBUST のサポート (#3646) (@matyhtf)
## v4.5.3
### 新しいAPI

- `Swoole\Process\ProcessManager`を追加しました (swoole/library#88f147b) (@huanghantao)
- ArrayObject::append、StringObject::equalsを追加しました (swoole/library#f28556f) (@matyhtf)
- [Coroutine::parallel](/coroutine/coroutine?id=parallel)を追加しました (swoole/library#6aa89a9) (@matyhtf)
- [Coroutine\Barrier](/coroutine/barrier)を追加しました (swoole/library#2988b2a) (@matyhtf)
- `usePipelineRead`の追加により、http2クライアントストリーミングをサポート(#3354) (@twose)
- ファイルをダウンロードする際、httpクライアントはデータを受信する前にファイルを作成しません(#3381) (@twose)
- httpクライアントは`bind_address`と`bind_port`構成をサポートします (#3390) (@huanghantao)
- httpクライアントは`lowercase_header`構成をサポートします (#3399) (@matyhtf)
- `Swoole\Server`は`tcp_user_timeout`構成をサポートします (#3404) (@huanghantao)
- `Coroutine\Socket`は、イベントバリアを追加して、コルーチンの切り替えを減らします (#3409) (@matyhtf)
- 特定のswStringに`memory allocator`を追加します (#3418) (@matyhtf)
- cURLが`__toString`をサポートします (swoole/library#38) (@twose)
- `WaitGroup`の構築子で直接`wait count`を設定できるようになりました (swoole/library#2fb228b8) (@matyhtf)
- `CURLOPT_REDIR_PROTOCOLS`を追加しました (swoole/library#46) (@sy-records)
- http1.1サーバーはトレイラーをサポートします (#3485) (@huanghantao)
- コルーチンスリープ時間が1ミリ秒未満の場合、現在のコルーチンをyieldします (#3487) (@Yurunsoft)
- http静的ハンドラーは、シンボリックリンクされたファイルをサポートします (#3569) (@LeiZhang-Hunter)
- `Server`がcloseメソッドを呼び出した後、WebSocket接続をすぐに閉じるようにします (#3570) (@matyhtf)
- `stream_set_blocking`をフックする機能をサポートします (#3585) (@Yurunsoft)
- 非同期HTTP2サーバーはフロー制御をサポートします (#3486) (@huanghantao) (@matyhtf)
- `onPackage`コールバック関数の実行が完了したら、ソケットのバッファを解放します (#3551) (@huanghantao) (@matyhtf)
### 修复

- 修复 WebSocket coredump, 处理协议错误的状态 (#3359) (@twose)
- 修复 swSignalfd_setup 函数以及 wait_signal 函数里的空指针错误 (#3360) (@twose)
- 修复在设置了 dispatch_func 时候，调用`Swoole\Server::close`会报错的问题 (#3365) (@twose)
- 修复`Swoole\Redis\Server::format`函数中 format_buffer 初始化问题 (#3369) (@matyhtf) (@twose)
- 修复 MacOS 上无法获取 mac 地址的问题 (#3372) (@twose)
- 修复 MySQL 测试用例 (#3374) (@qiqizjl)
- 修复多处 PHP8 兼容性问题 (#3384) (#3458) (#3578) (#3598) (@twose)
- 修复 hook 的 socket write 中丢失了 php_error_docref, timeout_event 和返回值问题 (#3383) (@twose)
- 修复异步 Server 无法在`WorkerStart`回调函数中关闭 Server 的问题 (#3382) (@huanghantao)
- 修复心跳线程在操作 conn->socket 的时候，可能会发生 coredump 的问题 (#3396) (@huanghantao)
- 修复 send_yield 的逻辑问题 (#3397) (@twose) (@matyhtf)
- 修复 Cygwin64 上的编译问题 (#3400) (@twose)
- 修复 WebSocket finish 属性无效的问题 (#3410) (@matyhtf)
- 修复遗漏的 MySQL transaction 错误状态 (#3429) (@twose)
- 修复 hook 后的`stream_select`与 hook 之前返回值行为不一致的问题 (#3440) (@Yurunsoft)
- 修复使用`Coroutine\System`来创建子进程时丢失`SIGCHLD`信号的问题 (#3446) (@huanghantao)
- 修复`sendwait`不支持 SSL 的问题 (#3459) (@huanghantao)
- 修复`ArrayObject`和`StringObject`的若干问题 (swoole/library#44) (@matyhtf)
- 修复 mysqli 异常信息错误 (swoole/library#45) (@sy-records)
- 修复当设置`open_eof_check`后，`Swoole\Client`无法获取正确的`errCode`的问题 (#3478) (@huanghantao)
- 修复 MacOS 上 `atomic->wait()`/`wakeup()`的若干问题 (#3476) (@Yurunsoft)
- 修复`Client::connect`连接拒绝的时候，返回成功状态的问题 (#3484) (@matyhtf)
- 修复 alpine 环境下 nullptr_t 没有被声明的问题 (#3488) (@limingxinleo)
- 修复 HTTP Client 下载文件的时候，double-free 的问题 (#3489) (@Yurunsoft)
- 修复`Server`被销毁时候，`Server\Port`没释放导致的内存泄漏问题 (#3507) (@twose)
- 修复 MQTT 协议解析问题 (318e33a) (84d8214) (80327b3) (efe6c63) (@GXhua) (@sy-records)
- 修复`Coroutine\Http\Client->getHeaderOut`方法导致的 coredump 问题 (#3534) (@matyhtf)
- 修复 SSL 验证失败后，丢失了错误信息的问题 (#3535) (@twose)
- 修复 README 中，`Swoole benchmark`链接错误的问题 (#3536) (@sy-records) (@santalex)
- 修复在`HTTP header/cookie`中使用`CRLF`后导致的`header`注入问题 (#3539) (#3541) (#3545) (@chromium1337) (@huanghantao)
- 修复 issue #3463 中提到的变量错误的问题 (#3547) (chromium1337) (@huanghantao)
- 修复 pr #3463 中提到的错别字问题 (#3547) (@deminy)
- 修复协程 WebSocket 服务器 frame->fd 为空的问题 (#3549) (@huanghantao)
- 修复心跳线程错误判断连接状态导致的连接泄漏问题 (#3534) (@matyhtf)
- 修复`Process\Pool`中阻塞了信号的问题 (#3582) (@huanghantao) (@matyhtf)
- 修复`SAPI`中使用 send headers 的问题 (#3571) (@twose) (@sshymko)
- 修复`CURL`执行失败的时候，未设置`errCode`和`errMsg`的问题 (swoole/library#1b6c65e) (@sy-records)
- 修复当调用了`setProtocol`方法后，`swoole_socket_coro`accept coredump 的问题 (#3591) (@matyhtf)
### カーネル

- C++スタイルを使用する (#3349) (#3351) (#3454) (#3479) (#3490) (@huanghantao) (@matyhtf)
- PHPオブジェクトのプロパティ読み取りパフォーマンスを向上させるために `Swoole known strings` を追加 (#3363) (@huanghantao)
- 多くの部分でコードを最適化する (#3350) (#3356) (#3357) (#3423) (#3426) (#3461) (#3463) (#3472) (#3557) (#3583) (@huanghantao) (@twose) (@matyhtf)
- 多くのテストコードを最適化する (#3416) (#3481) (#3558) (@matyhtf)
- `Swoole\Table` の `int` 型を簡略化する (#3407) (@matyhtf)
- `sw_memset_zero` を追加し、`bzero` 関数を置き換える (#3419) (@CismonX)
- ログモジュールの最適化 (#3432) (@matyhtf)
- 多くの libswoole リファクタリング (#3448) (#3473) (#3475) (#3492) (#3494) (#3497) (#3498) (#3526) (@matyhtf)
- 多くのヘッダーファイルのリファクタリング (#3457) (@matyhtf) (@huanghantao)
- `Channel::count()` および `Channel::get_bytes()` を追加 (f001581) (@matyhtf)
- `scope guard` を追加する (#3504) (@huanghantao)
- libswoole カバレッジテストを追加する (#3431) (@huanghantao)
- lib-swoole/ext-swoole MacOS 環境のテストを追加する (#3521) (@huanghantao)
- lib-swoole/ext-swoole Alpine 環境のテストを追加する (#3537) (@limingxinleo)
## v4.5.2

[v4.5.2](https://github.com/swoole/swoole-src/releases/tag/v4.5.2) は、バグ修正のバージョンであり、下位互換性のない変更はありません。
### 強化

- サーバーが毎日ログファイルを生成するために`Server->set(['log_rotation' => SWOOLE_LOG_ROTATION_DAILY])` をサポート (#3311) (@matyhtf)
- 信号リスナーが存在する場合、reactor が終了しないように`swoole_async_set(['wait_signal' => true])` をサポート (#3314) (@matyhtf)
- 空ファイルを送信するために`Server->sendfile` をサポート (#3318) (@twose)
- ワーカーのビジー／アイドル状態の警告情報を最適化 (#3328) (@huanghantao)
- HTTPSプロキシの場合、ホストヘッダーに関する設定を最適化（ssl_host_nameを使用して構成） (#3343) (@twose)
- SSLはデフォルトでecdh autoモードを使用 (#3316) (@matyhtf)
- SSLクライアントは接続が切断された場合にサイレントな終了を使用 (#3342) (@huanghantao)
### 修正

- 修正 `Server->taskWait` 在 OSX 平台上的问题 (#3330) (@matyhtf)
- 修正 MQTT 协议解析错误的 bug (8dbf506b) (@guoxinhua) (2ae8eb32) (@twose)
- 修正 Content-Length int 类型溢出的问题 (#3346) (@twose)
- 修正 PRI 包长度检查缺失的问题 (#3348) (@twose)
- 修正 CURLOPT_POSTFIELDS 无法置空的问题 (swoole/library@ed192f64) (@twose)
- 修正 最新的连接对象在接收到下一个连接之前无法被释放的问题 (swoole/library@1ef79339) (@twose)
### 内核

- ソケットに書き込む際のゼロコピー機能 (#3327) (@twose)
- グローバル変数の読み書きを置き換えるために、swoole_get_last_error/swoole_set_last_error の利用 (#3315) (@huanghantao) (@matyhtf) (#3315) (@huanghantao)
## v4.5.1

[v4.5.1](https://github.com/swoole/swoole-src/releases/tag/v4.5.1) は、BU​Gが修正されたバージョンであり、`v4.5.0`で導入された System ファイル関数の廃止マークを補完しました。
- ソケットコンテキストのbindto構成をサポートします（＃3275）（＃3278）（@codinghuang）
- クライアント::sendtoが自動的にDNS解決をサポートします（＃3292）（@codinghuang）
- Process->exit(0)により、プロセスが直ちに終了されます。shutdown_functionsを実行してから終了するには、PHPの提供するexitを使用してください（a732fe56）（@matyhtf）
- `log_date_format`を変更して日付形式を変更できるようになりました。ログにマイクロ秒のタイムスタンプを表示するには、`log_date_with_microseconds`を使用します（baf895bc）（@matyhtf）
- CURLOPT_CAINFOとCURLOPT_CAPATHをサポートします（swoole/library＃32）（@sy-records）
- CURLOPT_FORBID_REUSEをサポートします（swoole/library＃33）（@sy-records）
### 修正

- 修正在 32 位系统下的构建失败 (#3276) (#3277) (@remicollet) (@twose)
- 修正协程客户端在重复连接时未显示 EISCONN 错误信息的问题 (#3280) (@codinghuang)
- 修正 Table 模块中潜在的 bug (d7b87b65) (@matyhtf)
- 修正服务器中由于未定义行为导致的空指针(防御性编程) (#3304) (#3305) (@twose)
- 修正启用心跳配置后导致的空指针错误问题 (#3307) (@twose)
- 修正 mysqli 配置未生效的问题 (swoole/library#35)
- 修正响应中存在不规范的 header(缺少空格)时的解析问题 (swoole/library#27) (@Yurunsoft)
### 廃止

- Coroutine\System::(fread/fgets/fwrite)などのメソッドを廃止にマークしました（代わりにフック機能を使用してください。PHPのファイル関数を直接使用してください）（c7c9bb40）(@twose)
### 内核

- 使用 zend_object_alloc 为自定义对象分配内存 (cf1afb25) (@twose)
- 一些优化, 为日志模块添加更多配置项 (#3296) (@matyhtf)
- 大量代码优化工作和增加单测 (swoole/library) (@deminy)
## v4.5.0

[v4.5.0](https://github.com/swoole/swoole-src/releases/tag/v4.5.0)、これは大規模な更新であり、v4.4.xで非推奨とされている一部のモジュールが削除されました。
### 新しいAPI

- DTLS サポートが追加され、これによりWebRTCアプリケーションを構築できるようになりました (#3188) (@matyhtf)
- 組み込みの `FastCGI` クライアントが追加され、1行のコードでリクエストをFPMにプロキシしたり、FPMアプリケーションを呼び出すことができます (swoole/library#17) (@twose)
- `Co::wait`, `Co::waitPid` (子プロセスの回収に使用), `Co::waitSignal` (シグナルの待機に使用) が追加されました (#3158) (@twose)
- `Co::waitEvent` (指定したイベントがソケットで発生するのを待機するために使用) が追加されました (#3197) (@twose)
- `Co::set(['exit_condition' => $callable])` (プログラムの終了条件をカスタマイズするために使用) が追加されました (#2918) (#3012) (@twose)
- `Co::getElapsed` (コルーチンの実行時間を取得し、統計やゾンビコルーチンを見つけるのに役立ちます) が追加されました (#3162) (@doubaokun)
- `Socket::checkLiveness` (システムコールを使用して接続の活性を確認), `Socket::peek` (リードバッファを覗く) が追加されました (#3057) (@twose)
- `Socket->setProtocol(['open_fastcgi_protocol' => $bool])` (組み込みの FastCGI デコードサポート) が追加されました (#3103) (@twose)
- `Server::get(Master|Manager|Worker)Pid`, `Server::getWorkerId` (非同期Serverのインスタンスと情報を取得) が追加されました (#2793) (#3019) (@matyhtf)
- `Server::getWorkerStatus` (workerプロセスの状態を取得し、SWOOLE_WORKER_BUSY、SWOOLE_WORKER_IDLEの定数で忙しさを表します) が追加されました (#3225) (@matyhtf)
- `Server->on('beforeReload', $callable)` と `Server->on('afterReload', $callable)` (managerプロセスで発生する再起動イベント) が追加されました (#3130) (@hantaohuang)
- `Http\Server` の静的ファイルハンドラに、`http_index_files` と `http_autoindex` の設定が追加されました (#3171) (@hantaohuang)
- `Http2\Client->read(float $timeout = -1)` メソッドが、ストリーム式のレスポンスを読み取るサポートが追加されました (#3011) (#3117) (@twose)
- `Http\Request->getContent` (rawContent メソッドのエイリアス) が追加されました (#3128) (@hantaohuang)
- `swoole_mime_type_(add|set|delete|get|exists)()` (mime 関連のAPI、組み込みのmimeタイプを追加、削除、取得、存在チェックすることができます) が追加されました (#3134) (@twose)
### 強化

- マスターとワーカー間のメモリコピーを最適化（極限状況で4倍のパフォーマンス向上）（#3075）（#3087）（@hantaohuang）
- WebSocket ディスパッチロジックの最適化（#3076）（@matyhtf）
- WebSocket フレーム構築時の1回のメモリコピーを最適化（#3097）（@matyhtf）
- SSL検証モジュールの最適化（#3226）（@matyhtf）
- SSL acceptとSSL handshakeの分離、低速SSLクライアントが協力サーバーをブロックする問題を解決（#3214）（@twose）
- MIPSアーキテクチャのサポート（#3196）（@ekongyun）
- UDPクライアントが入力されたドメインを自動解析できるようになりました（#3236）（#3239）（@huanghantao）
- Coroutine\Http\Serverが一部の一般的なオプションをサポートするようになりました（#3257）（@twose）
- WebSocketハンドシェイク時にクッキーを設定するサポートを追加しました（#3270）（#3272）（@twose）
- CURLOPT_FAILONERRORのサポート（swoole/library#20）（@sy-records）
- CURLOPT_SSLCERTTYPE、CURLOPT_SSLCERT、CURLOPT_SSLKEYTYPE、CURLOPT_SSLKEYのサポート（swoole/library#22）（@sy-records）
- CURLOPT_HTTPGETのサポート（swoole/library@d730bd08）（@shiguangqi）
### 削除

- `Runtime::enableStrictMode`メソッドを削除する (b45838e3) (@twose)
- `Buffer`クラスを削除する (559a49a8) (@twose)
- 新しいC++のAPI：coroutine::async関数にlambdaを渡すことで非同期スレッドタスクを開始できます (#3127) (@matyhtf)
- ローレベルイベントAPI中の整数型fdをswSocketオブジェクトに再構築しました (#3030) (@matyhtf)
- すべてのコアCファイルがC++ファイルに変換されました (#3030) (71f987f3) (@matyhtf)
- 一連のコードの最適化 (#3063) (#3067) (#3115) (#3135) (#3138) (#3139) (#3151) (#3168) (@hantaohuang)
- ヘッダーファイルの規範化の最適化 (#3051) (@matyhtf)
- 'enable_reuse_port'構成項目をより規範的に再構築しました (#3192) (@matyhtf)
- より規範的なSocket関連APIの再構築 (#3193) (@matyhtf)
- バッファ予測による不要なシステムコールの削減 (3b5aa85d) (@matyhtf)
- ローレベルのタイマーリフレッシュswServerGS::nowを削除し、時間関数を直接使用します (#3152) (@hantaohuang)
- プロトコル構成子の最適化 (#3108) (@twose)
- より良い互換性のあるC構造の初期化方法 (#3069) (@twose)
- ビットフィールドをuchar型に統一しました (#3071) (@twose)
- 並行テストのサポート、より高速な処理です (#3215) (@twose)
### 修复

- 修复 enable_delay_receive 开启后 onConnect 无法触发的问题 (#3221) (#3224) (@matyhtf)
- 所有其它的 bug 修复都已合并到 v4.4.x 分支并在更新日志中体现, 在此不再赘述
## v4.4.22
### 修复

- 修复 HTTP2 client 在 HTTP proxy 下无法工作的问题 (#3677) (@matyhtf) (@twose)
- 修复 PDO 断线重连时数据混乱的问题 (swoole/library#54) (@sy-records)
- 修复 swMutex_lockwait (0fc5665) (@matyhtf)
- 修复 UDP Server 使用ipv6时端口解析错误
- 修复 systemd fds 的问题
## v4.4.20

[v4.4.20](https://github.com/swoole/swoole-src/releases/tag/v4.4.20), これはバグ修正バージョンで、下位互換性のない変更はありません。
### 修復

- 修復在設置了 dispatch_func 時候，調用`Swoole\Server::close`會報錯的問題 (#3365) (@twose)
- 修復`Swoole\Redis\Server::format`函數中 format_buffer 初始化問題 (#3369) (@matyhtf) (@twose)
- 修復 MacOS 上無法獲取 mac 地址的問題 (#3372) (@twose)
- 修復 MySQL 測試用例 (#3374) (@qiqizjl)
- 修復異步 Server 無法在`WorkerStart`回調函數中關閉 Server 的問題 (#3382) (@huanghantao)
- 修復遺漏的 MySQL transaction 錯誤狀態 (#3429) (@twose)
- 修復 HTTP Client 下載文件的時候，double-free 的問題 (#3489) (@Yurunsoft)
- 修復`Coroutine\Http\Client->getHeaderOut`方法導致的 coredump 問題 (#3534) (@matyhtf)
- 修復在`HTTP header/cookie`中使用`CRLF`後導致的`header`注入問題 (#3539) (#3541) (#3545) (@chromium1337) (@huanghantao)
- 修復協程 WebSocket 伺服器 frame->fd 為空的問題 (#3549) (@huanghantao)
- 修復 hook phpredis 產生的`read error on connection`問題 (#3579) (@twose)
- 修復 MQTT 協議解析問題 (#3573) (#3517) (9ad2b455) (@GXhua) (@sy-records)
## v4.4.19

[v4.4.19](https://github.com/swoole/swoole-src/releases/tag/v4.4.19) はバグ修正バージョンであり、下位互換性のない変更はありません。

!> 注意: v4.4.x はもはやメインテナンスされているバージョンではなく、必要に応じてバグが修正されています。
### 修复

- 从 v4.5.2 合并了所有 bug 修复补丁

翻訳: 
### 修正

- v4.5.2 からすべてのバグ修正パッチをマージしました
## v4.4.18

[v4.4.18](https://github.com/swoole/swoole-src/releases/tag/v4.4.18) は、バグ修正バージョンであり、下位互換性のない変更はありません。
- UDPクライアントは、今後ドメイン名を自動的に解析することができます (#3236) (#3239) (@huanghantao)
- CLIモードでは、標準出力と標準エラー出力が閉じられなくなりました（シャットダウン後に生成されるエラーログが表示されます） (#3249) (@twose)
- Coroutine\Http\Serverに一般的なオプションがサポートされました (#3257) (@twose)
- WebSocketのハンドシェイク時にクッキーを設定する機能が追加されました (#3270) (#3272) (@twose)
- CURLOPT_FAILONERRORをサポートするようになりました (swoole/library#20) (@sy-records)
- CURLOPT_SSLCERTTYPE、CURLOPT_SSLCERT、CURLOPT_SSLKEYTYPE、CURLOPT_SSLKEYをサポートするようになりました (swoole/library#22) (@sy-records)
- CURLOPT_HTTPGETをサポートするようになりました (swoole/library@d730bd08) (@shiguangqi)
- すべてのPHP-Redis拡張機能のバージョンとの互換性を向上させました（異なるバージョンの構築関数に異なる引数を渡す） (swoole/library#24) (@twose)
- 接続オブジェクトのクローンを禁止しました (swoole/library#23) (@deminy)
### 修復

- 修復 SSL 握手失敗的問題 (dc5ac29a) (@twose)
- 修復生成錯誤信息時產生的內存錯誤 (#3229) (@twose)
- 修復空白的代理認證信息 (#3243) (@twose)
- 修復 Channel 的內存泄漏問題 (並非真正的內存泄漏) (#3260) (@twose)
- 修復 Co\Http\Server 在循環引用時產生的一次性內存洩漏 (#3271) (@twose)
- 修復`ConnectionPool->fill`中的書寫錯誤 (swoole/library#18) (@NHZEX)
- 修復 curl 客戶端遭遇重定向時沒有更新連接的問題 (swoole/library#21) (@doubaokun)
- 修復產生 ioException 時空指針的問題 (swoole/library@4d15a4c3) (@twose)
- 修復 ConnectionPool@put 傳入 null 時沒有歸還新連接導致的死鎖問題 (swoole/library#25) (@Sinute)
- 修復 mysqli 代理實現導致的 write_property 錯誤 (swoole/library#26) (@twose)
## v4.4.17

[v4.4.17](https://github.com/swoole/swoole-src/releases/tag/v4.4.17)、これはバグ修正バージョンで、下位互換性の変更はありません。
### 増強

- SSLサーバーの性能向上 (#3077) (85a9a595) (@matyhtf)
- HTTPヘッダーサイズ制限の削除 (#3187) limitation (@twose)
- MIPSのサポート (#3196) (@ekongyun)
- CURLOPT_HTTPAUTHのサポート (swoole/library@570318be) (@twose)
- 修正 package_length_func 的行为和潜在的一次性内存泄漏 (#3111) (@twose)
- 修正 HTTP 状态码 304 下的错误行为 (#3118) (#3120) (@twose)
- 修正 Trace ログの誤ったマクロ展開によるメモリエラー (#3142) (@twose)
- 修正 OpenSSL 関数の署名 (#3154) (#3155) (@twose)
- 修正 SSL エラーメッセージ (#3172) (@matyhtf) (@twose)
- PHP-7.4 での互換性を修正 (@twose) (@matyhtf)
- HTTP-chunk の長さ解析エラーを修正 (19a1c712) (@twose)
- chunked モード時のマルチパートリクエストパーサーの動作を修正 (3692d9de) (@twose)
- PHP-Debug モード下の ZEND_ASSUME アサーションの失敗を修正 (fc0982be) (@twose)
- ソケットのエラーアドレスを修正 (d72c5e3a) (@twose)
- ソケットの getname を修正 (#3177) (#3179) (@matyhtf)
- 空ファイルに対する静的ファイルハンドラのエラー処理を修正 (#3182) (@twose)
- Coroutine\Http\Server におけるファイルのアップロードの問題を修正 (#3189) (#3191) (@twose)
- シャットダウン中に発生する可能性のあるメモリエラーを修正 (44aef60a) (@matyhtf)
- Server->heartbeat を修正 (#3203) (@matyhtf)
- CPU スケジューラが無限ループをスケジュールできない可能性がある問題を修正 (#3207) (@twose)
- 不変配列上の無効な書き込み操作を修正 (#3212) (@twose)
- WaitGroup での複数回の待機問題を修正 (swoole/library@537a82e1) (@twose)
- 空のヘッダー処理を修正 (cURL と一致) (swoole/library@7c92ed5a) (@twose)
- IO メソッドが false を返す場合に例外をスローする問題を修正 (swoole/library@f6997394) (@twose)
- cURL-hook でのプロキシポート番号がヘッダーに複数回追加される問題を修正 (swoole/library@5e94e5da) (@twose)
## v4.4.16

[v4.4.16](https://github.com/swoole/swoole-src/releases/tag/v4.4.16), これはバグ修正バージョンであり、下位互換性のない変更はありません。
### 強化

- 現在、[Swoole のバージョンサポート情報](https://github.com/swoole/swoole-src/blob/master/SUPPORTED.md) を取得できます
- より使いやすいエラーメッセージ (0412f442) (09a48835) (@twose)
- 特定の特殊システムでのシステムコールの無限ループを防止 (069a0092) (@matyhtf)
- PDOConfig でドライバーオプションを追加 (swoole/library#8) (@jcheron)
### 修复

- 修复 http2_session.default_ctx 内存错误 (bddbb9b1) (@twose)
- 修复未初始化的 http_context (ce77c641) (@twose)
- 修复 Table 模块中的书写错误 (可能会造成内存错误) (db4eec17) (@twose)
- 修复 Server 中 task-reload 的潜在问题 (e4378278) (@GXhua)
- 修复不完整协程 HTTP 服务器请求原文 (#3079) (#3085) (@hantaohuang)
- 修复 static handler (当文件为空时, 不应返回 404 响应) (#3084) (@Yurunsoft)
- 修复 http_compression_level 配置无法正常工作 (16f9274e) (@twose)
- 修复 Coroutine HTTP2 Server 由于没有注册 handle 而产生空指针错误 (ed680989) (@twose)
- 修复配置 socket_dontwait 不工作的问题 (27589376) (@matyhtf)
- 修复 zend::eval 可能会被执行多次的问题 (#3099) (@GXhua)
- 修复 HTTP2 服务器由于在连接关闭后响应而产生的空指针错误 (#3110) (@twose)
- 修复 PDOStatementProxy::setFetchMode 适配不当的问题 (swoole/library#13) (@jcheron)

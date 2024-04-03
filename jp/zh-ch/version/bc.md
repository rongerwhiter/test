# Non-Backward-Compatible Change
## v5.0.0
* Adjusted the default running mode of `Server` to `SWOOLE_BASE`
* Increased the minimum required `PHP` version to `8.0`
* Added type declarations to all class methods and functions, enforcing strict typing
* Removed the underscored `PSR-0` class aliases, only namespace-style class names are now allowed, for example, `swoole_server` must be modified to `Swoole\Server`
* Deprecated `Swoole\Coroutine\Redis` and `Swoole\Coroutine\MySQL`, recommended usage is `Runtime Hook` with native `Redis`/`MySQL` client.
## v4.8.0

- `BASE` モードの場合、`onStart` コールバックは常に最初のワーカープロセス（`workerId` が `0`）が起動するときに呼び出され、`onWorkerStart` の実行よりも先に呼び出されます。`onStart` 関数内では常にコルーチン `API` を使用できます。`Worker-0` で致命的なエラーが発生して再起動されると、再度 `onStart` がコールバックされます。
以前のバージョンでは、1つのワーカープロセスしかない場合、`onStart` は `Worker-0` でコールバックされました。複数のワーカープロセスがある場合は、`Manager` プロセスで実行されました。
## v4.7.0

- Removed `Table\Row`, `Table` no longer supported reading and writing as arrays
## v4.6.0

- `session id`の最大制限が削除され、重複しなくなりました
- コルーチンを使用する際には、`pcntl_fork`/`pcntl_wait`/`pcntl_waitpid`/`pcntl_sigtimedwait`などの安全でない機能が無効になります
- デフォルトでコルーチンフックが有効になりました
- PHP7.1はサポートされなくなりました
- `Event::rshutdown()`は非推奨とマークされ、代わりにCoroutine\runを使用してください
## v4.5.4

- `SWOOLE_HOOK_ALL` は `SWOOLE_HOOK_CURL` を包括します
- `ssl_method` が削除され、`ssl_protocols` がサポートされました
## v4.4.12

- このバージョンでは、WebSocketフレームの圧縮がサポートされ、pushメソッドの3番目の引数がflagsに変更されました。strict_typesが設定されていない場合、コードの互換性に影響はありませんが、そうでない場合はboolがintに暗黙的に変換されないという型エラーが発生します。この問題はv4.4.13で修正される予定です
## v4.4.1

- Registered signals are no longer considered as a condition to maintain the event loop. **If the program only registers signals without performing any other work, it will be considered idle and will exit immediately** (you can prevent the process from exiting by registering a timer at this point)
## v4.4.0

- 和`PHP`官方保持一致, 不再支持`PHP7.0` (@matyhtf)
- 移除`Serialize`模块, 在单独的 [ext-serialize](https://github.com/swoole/ext-serialize) 扩展中维护
- 移除`PostgreSQL`模块，在单独的 [ext-postgresql](https://github.com/swoole/ext-postgresql) 扩展中维护
- `Runtime::enableCoroutine`不再会自动兼容协程内外环境, 一旦开启, 则一切阻塞操作必须在协程内调用 (@matyhtf)
- 由于引入了全新的协程`MySQL`客户端驱动, 底层设计更加规范, 但有一些小的向下不兼容的变化 (詳細は [4.4.0更新日誌](https://wiki.swoole.com/wiki/page/p-4.4.0.html))
## v4.3.0

- 移除了所有异步模块, 详见 [独立异步扩展](https://wiki.swoole.com/wiki/page/p-async_ext.html) 或 [4.3.0更新日志](https://wiki.swoole.com/wiki/page/p-4.3.0.html)
## v4.2.13

> Due to historical API design issues leading to inevitable incompatible changes

* Changes in coroutine Redis client subscription mode operations, see more details [here](https://wiki.swoole.com/#/coroutine_client/redis?id=%e8%ae%a2%e9%98%85%e6%a8%a1%e5%bc%8f)
## v4.2.12

> Experimental feature + Unavoidable incompatible changes due to historical API design issues

- Removed the `task_async` configuration option, replacing it with [task_enable_coroutine](https://wiki.swoole.com/#/server/setting?id=task_enable_coroutine)
## v4.2.5

- Removed support for `UDP` clients in `onReceive` and `Server::getClientInfo`.
## v4.2.0

- Removed the asynchronous `swoole_http2_client` completely, please use the coroutine HTTP2 client instead.
## v4.0.4

From this version onward, the asynchronous `Http2\Client` will trigger `E_DEPRECATED` warnings and will be removed in the next version. Please use `Coroutine\Http2\Client` instead.

The `body` property of `Http2\Response` has been renamed to `data`. This change is made to ensure consistency between `request` and `response`, and to better align with the frame type names in the HTTP2 protocol.

Starting from this version, `Coroutine\Http2\Client` has relatively complete support for the HTTP2 protocol, meeting the demands of enterprise-level production applications such as `grpc`, `etcd`, etc. Therefore, the series of changes related to HTTP2 are necessary.
## v4.0.3

`swoole_http2_response` と `swoole_http2_request` を一貫性のあるものにしました。全ての属性名が複数形式に変更されました。以下の属性が関連しています。

- `headers`
- `cookies`
## v4.0.2

> Due to the complexity of the underlying implementation, which is difficult to maintain and often causes misunderstandings when used by users, the following API has been temporarily removed:

- `Coroutine\Channel::select`

However, the second parameter `timeout` has been added to the `Coroutine\Channel->pop` method to meet the development needs.
## v4.0

> Coroutineのカーネルがアップグレードされたため、任意の関数の任意の場所でCoroutineを呼び出すことができるようになり、特別な処理をする必要がなくなったため、以下のAPIが削除されました。

- `Coroutine::call_user_func`
- `Coroutine::call_user_func_array`

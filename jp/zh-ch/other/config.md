# ini配置

配置 | 默认值 | 作用
---|---|---
swoole.enable_coroutine | On | `On`, `Off` 開閉内部協程，[詳見](/server/setting?id=enable_coroutine)。
swoole.display_errors | On | 開啟/關閉`Swoole`錯誤信息。
swoole.unixsock_buffer_size | 8M | 設置進程間通信的`Socket`緩存區尺寸，等價於[socket_buffer_size](/server/setting?id=socket_buffer_size)。
swoole.use_shortname | On | 是否啟用短別名，[詳見](/other/alias?id=協程短名稱)。
swoole.enable_preemptive_scheduler | Off | 可防止某些協程死循環佔用CPU時間過長(10ms的CPU時間)導致其它協程得不到[調度](/coroutine?id=協程調度)，[示例](https://github.com/swoole/swoole-src/tree/master/tests/swoole_coroutine_scheduler/preemptive)。
swoole.enable_library | On | 開啟/關閉擴展內建的library

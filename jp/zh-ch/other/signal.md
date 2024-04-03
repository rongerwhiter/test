# Linux Signal List

こちらはLinuxの信号一覧です。
## 完整対照表

| 信号      | 取值     | 默认动作 | 含义（发出信号的原因）                  |
| --------- | -------- | -------- | --------------------------------------- |
| SIGHUP    | 1        | Term     | 终端的挂断或进程死亡                    |
| SIGINT    | 2        | Term     | 来自键盘的中断信号                      |
| SIGQUIT   | 3        | Core     | 来自键盘的离开信号                      |
| SIGILL    | 4        | Core     | 非法指令                                |
| SIGABRT   | 6        | Core     | 来自 abort 的异常信号                   |
| SIGFPE    | 8        | Core     | 浮点例外                                |
| SIGKILL   | 9        | Term     | 杀死                                    |
| SIGSEGV   | 11       | Core     | 段非法错误(内存引用无效)                |
| SIGPIPE   | 13       | Term     | 管道损坏：向一个没有读进程的管道写数据  |
| SIGALRM   | 14       | Term     | 来自 alarm 的计时器到时信号             |
| SIGTERM   | 15       | Term     | 终止                                    |
| SIGUSR1   | 30,10,16 | Term     | 用户自定义信号 1                        |
| SIGUSR2   | 31,12,17 | Term     | 用户自定义信号 2                        |
| SIGCHLD   | 20,17,18 | Ign      | 子进程停止或终止                        |
| SIGCONT   | 19,18,25 | Cont     | 如果停止，继续执行                      |
| SIGSTOP   | 17,19,23 | Stop     | 非来自终端的停止信号                    |
| SIGTSTP   | 18,20,24 | Stop     | 来自终端的停止信号                      |
| SIGTTIN   | 21,21,26 | Stop     | 后台进程读终端                          |
| SIGTTOU   | 22,22,27 | Stop     | 后台进程写终端                          |
|           |          |          |                                         |
| SIGBUS    | 10,7,10  | Core     | 总线错误（内存访问错误）                |
| SIGPOLL   |          | Term     | Pollable 事件发生(Sys V)，与 SIGIO 同义 |
| SIGPROF   | 27,27,29 | Term     | 统计分布图用计时器到时                  |
| SIGSYS    | 12,-,12  | Core     | 非法系统调用(SVr4)                      |
| SIGTRAP   | 5        | Core     | 跟踪/断点自陷                           |
| SIGURG    | 16,23,21 | Ign      | socket 紧急信号(4.2BSD)                 |
| SIGVTALRM | 26,26,28 | Term     | 虚拟计时器到时(4.2BSD)                  |
| SIGXCPU   | 24,24,30 | Core     | 超过 CPU 时限(4.2BSD)                   |
| SIGXFSZ   | 25,25,31 | Core     | 超过文件长度限制(4.2BSD)                |
|           |          |          |                                         |
| SIGIOT    | 6        | Core     | IOT 自陷，与 SIGABRT 同义               |
| SIGEMT    | 7,-,7    |          | Term                                    |
| SIGSTKFLT | -,16,-   | Term     | 协处理器堆栈错误(不使用)                |
| SIGIO     | 23,29,22 | Term     | 描述符上可以进行 I/O 操作               |
| SIGCLD    | -,-,18   | Ign      | 与 SIGCHLD 同义                         |
| SIGPWR    | 29,30,19 | Term     | 电力故障(System V)                      |
| SIGINFO   | 29,-,-   |          | 与 SIGPWR 同义                          |
| SIGLOST   | -,-,-    | Term     | 文件锁丢失                              |
| SIGWINCH  | 28,28,20 | Ign      | 窗口大小改变(4.3BSD, Sun)               |
| SIGUNUSED | -,31,-   | Term     | 未使用信号(will be SIGSYS)              |
## 非可靠信号

| 名称      | 说明                        |
| --------- | --------------------------- |
| SIGHUP    | 接続が切断されました          |
| SIGINT    | 端末中断シグナル             |
| SIGQUIT   | 端末終了シグナル             |
| SIGILL    | 不正なハードウェア命令       |
| SIGTRAP   | ハードウェアの問題           |
| SIGABRT   | 異常終了（abort）            |
| SIGBUS    | ハードウェアの問題           |
| SIGFPE    | 算術例外                    |
| SIGKILL   | 終了シグナル                  |
| SIGUSR1   | ユーザー定義シグナル          |
| SIGUSR2   | ユーザー定義シグナル          |
| SIGSEGV   | 無効なメモリ参照             |
| SIGPIPE   | 書き込みが読み取り可能でないプロセスに送信されました |
| SIGALRM   | タイマーが時間切れしました    |
| SIGTERM   | 終了シグナル                  |
| SIGCHLD   | 子プロセスの状態が変更されました |
| SIGCONT   | 一時停止中のプロセスを再開します |
| SIGSTOP   | 停止シグナル                  |
| SIGTSTP   | 端末停止シグナル              |
| SIGTTIN   | バックグラウンドで読み取り制御tty |
| SIGTTOU   | バックグラウンドで書き込み制御tty |
| SIGURG    | 緊急状況（ソケット）         |
| SIGXCPU   | CPU制限を超えました（setrlimit）|
| SIGXFSZ   | ファイルサイズ制限を超えました（setrlimit）|
| SIGVTALRM | 仮想時間アラーム（setitimer）  |
| SIGPROF   | プロファイリングタイムアウト（setitimer） |
| SIGWINCH  | 端末ウィンドウサイズが変更されました |
| SIGIO     | 非同期I/O                   |
| SIGPWR    | 電源の故障/再起動           |
| SIGSYS    | 無効なシステムコール         |
## 可信信号

| 名称        | 用户自定义 |
| ----------- | ---------- |
| SIGRTMIN    |            |
| SIGRTMIN+1  |            |
| SIGRTMIN+2  |            |
| SIGRTMIN+3  |            |
| SIGRTMIN+4  |            |
| SIGRTMIN+5  |            |
| SIGRTMIN+6  |            |
| SIGRTMIN+7  |            |
| SIGRTMIN+8  |            |
| SIGRTMIN+9  |            |
| SIGRTMIN+10 |            |
| SIGRTMIN+11 |            |
| SIGRTMIN+12 |            |
| SIGRTMIN+13 |            |
| SIGRTMIN+14 |            |
| SIGRTMIN+15 |            |
| SIGRTMAX-14 |            |
| SIGRTMAX-13 |            |
| SIGRTMAX-12 |            |
| SIGRTMAX-11 |            |
| SIGRTMAX-10 |            |
| SIGRTMAX-9  |            |
| SIGRTMAX-8  |            |
| SIGRTMAX-7  |            |
| SIGRTMAX-6  |            |
| SIGRTMAX-5  |            |
| SIGRTMAX-4  |            |
| SIGRTMAX-3  |            |
| SIGRTMAX-2  |            |
| SIGRTMAX-1  |            |
| SIGRTMAX    |            |

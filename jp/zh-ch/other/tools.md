# 工具使用

こちらは翻訳せず、元のテキストをそのまま使います。
## yasd

[yasd](https://github.com/swoole/yasd)

单步调试工具，可用于`Swoole`协程环境，支持`IDE`以及命令行的调试模式。  
## tcpdump

ネットワーク通信プログラムをデバッグする際には、tcpdumpが必須のツールです。tcpdumpは非常に強力であり、ネットワーク通信の細部を見ることができます。TCPの場合、3-way handshake、PUSH / ACKデータの送信、4-way handshakeのクローズなど、すべての詳細を確認できます。また、各ネットワークパケットのバイト数や時間なども含まれます。
### 使用方法

最もシンプルな使用例：

```shell
sudo tcpdump -i any tcp port 9501
```
* -i フラグはネットワークインターフェースを指定します。anyはすべてのネットワークインターフェースを表します。
* tcp はTCPプロトコルのみを監視することを指定します。
* port は監視するポートを指定します。

!> tcpdumpを実行するには管理者権限が必要です。通信内容を見たい場合は、`-Xnlps0` フラグを追加することができます。他の詳細なオプションについては、オンラインの記事を参照してください。
### 実行結果

```
13:29:07.788802 IP localhost.42333 > localhost.9501: Flags [S], seq 828582357, win 43690, options [mss 65495,sackOK,TS val 2207513 ecr 0,nop,wscale 7], length 0
13:29:07.788815 IP localhost.9501 > localhost.42333: Flags [S.], seq 1242884615, ack 828582358, win 43690, options [mss 65495,sackOK,TS val 2207513 ecr 2207513,nop,wscale 7], length 0
13:29:07.788830 IP localhost.42333 > localhost.9501: Flags [.], ack 1, win 342, options [nop,nop,TS val 2207513 ecr 2207513], length 0
13:29:10.298686 IP localhost.42333 > localhost.9501: Flags [P.], seq 1:5, ack 1, win 342, options [nop,nop,TS val 2208141 ecr 2207513], length 4
13:29:10.298708 IP localhost.9501 > localhost.42333: Flags [.], ack 5, win 342, options [nop,nop,TS val 2208141 ecr 2208141], length 0
13:29:10.298795 IP localhost.9501 > localhost.42333: Flags [P.], seq 1:13, ack 5, win 342, options [nop,nop,TS val 2208141 ecr 2208141], length 12
13:29:10.298803 IP localhost.42333 > localhost.9501: Flags [.], ack 13, win 342, options [nop,nop,TS val 2208141 ecr 2208141], length 0
13:29:11.563361 IP localhost.42333 > localhost.9501: Flags [F.], seq 5, ack 13, win 342, options [nop,nop,TS val 2208457 ecr 2208141], length 0
13:29:11.563450 IP localhost.9501 > localhost.42333: Flags [F.], seq 13, ack 6, win 342, options [nop,nop,TS val 2208457 ecr 2208457], length 0
13:29:11.563473 IP localhost.42333 > localhost.9501: Flags [.], ack 14, win 342, options [nop,nop,TS val 2208457 ecr 2208457], length 0
```
* `13:29:11.563473` マイクロ秒単位の時刻
*  localhost.42333 > localhost.9501 通信の方向を示しており、42333がクライアント、9501がサーバーである
* [S] SYN要求を表す
* [.] ACK確認パケットを表し、(client)SYN->(server)SYN->(client)ACK は3-way handshakeのプロセスである
* [P] データプッシュを表し、サーバーからクライアントへのプッシュや、クライアントからサーバーへのプッシュができる
* [F] FINパケットであり、接続を閉じる操作であり、client/server 両方が発信可能である
* [R] RSTパケットであり、Fパケットと同じ機能だが、RSTは接続が閉じられたとき、まだ処理されていないデータがあることを示す。接続を強制的に切断すると理解できる
* win 342 はスライディングウィンドウのサイズを示す
* length 12 はデータパケットのサイズを示す
## strace

straceはシステムコールの実行状況をトレースでき、プログラムに問題が発生した場合にstraceを使用して問題を分析およびトレースすることができます。

!> FreeBSD/MacOSではtrussを使用できます。
### 使用方法

```shell
strace -o /tmp/strace.log -f -p $PID
```

* -f 表示追跡多重スレッドおよび複数プロセスです。-f パラメータを付けないと、子プロセスやサブスレッドの実行状況を取得できません。
* -o 結果をファイルに出力します。
* -p $PID、追跡するプロセスIDを指定します。ps aux で確認できます。
* -tt システムコールの発生時間を印字します。マイクロ秒単位まで表示します。
* -s 文字列の印字長さを制限します。例えば、recvfrom システムコールで受信したデータのみを32 バイトだけ表示します。
* -c 各システムコールの実行時間のリアルタイム統計を取得します。
* -T 各システムコールの実行時間を表示します。
## gdb

GDBはGNUがリリースした強力なUNIX用プログラムデバッグツールで、C/C++で開発されたプログラムのデバッグに使用できます。PHPとSwooleはC言語で開発されているため、PHP+Swooleプログラムをデバッグする際にGDBを使用できます。

gdbデバッグはコマンドラインインタラクティブであり、一般的な命令を理解する必要があります。
### 使用法

```shell
gdb -p processID
gdb php
gdb php core
```

gdbには3つの使用法があります：

* 実行中のPHPプログラムをトレースするには、gdb -p processIDを使用します。
* PHPプログラムを実行してデバッグするには、gdb php -> run server.phpでデバッグを行います。
* PHPプログラムがコアダンプを作成した後には、gdbを使用してcoreメモリイメージをロードしてデバッグします。 gdb php core

!> PATH環境変数にphpが含まれていない場合、gdbを使用する際には絶対パスを指定する必要があります。例：gdb /usr/local/bin/php
### 常用指令

* `p`：print，変数の値を表示する
* `c`：continue，中断されたプログラムの実行を続ける
* `b`：breakpoint，ブレイクポイントを設定する。関数名で設定する場合は`b zif_php_function`のように設定でき、ソースコードの行数で指定する場合は`b src/networker/Server.c:1000`のようにできる
* `t`：thread，スレッドを切り替える。プロセスに複数のスレッドがある場合、異なるスレッドに切り替えるために`t`コマンドを使用できる
* `ctrl + c`：現在実行中のプログラムを中断する。`c`命令と組み合わせて使用する
* `n`：next，次の行を実行する。ステップ実行
* `info threads`：実行中のすべてのスレッドを表示する
* `l`：list，ソースコードを表示する。`l 関数名`または`l 行番号`を使用できる
* `bt`：backtrace，実行時の関数呼び出しスタックを表示する
* `finish`：現在の関数を終了する
* `f`：frame，`bt`と組み合わせて使用し、関数呼び出しスタックの特定のレベルに切り替えることができる
* `r`：run，プログラムを実行する
### zbacktrace

zbacktraceはPHPソースパッケージに含まれるgdbのカスタムコマンドで、btコマンドと似た機能を持ちますが、異なる点はzbacktraceが見るスタックトレースがPHP関数呼び出しのスタックであり、C関数ではありません。

php-srcをダウンロードし、展開した後、ルートディレクトリから`.gdbinit`ファイルを見つけ、gdbシェルで次のコマンドを入力します。

```shell
source .gdbinit
zbacktrace
```

`.gdbinit`には他にも多くのコマンドが用意されており、詳細情報はソースコードを確認してください。
#### gdb+zbacktraceを使用してデッドロックの問題を追跡する

```shell
gdb -p <プロセスID>
```

* デッドロックを引き起こすWorkerプロセスのIDを見つけるために`ps aux`ツールを使用します
* `gdb -p`コマンドで指定したプロセスを追跡します
* 繰り返し`ctrl + c` 、`zbacktrace`、`c` を呼び出して、どのPHPコードがループしているかを確認します
* 該当するPHPコードを見つけ出して問題を解決します
## lsof

Linux platform provides the `lsof` tool that can be used to view the file descriptors opened by a process. It can be used to track all the opened sockets, files, and resources by swoole's worker processes.
### 使用法

```shell
lsof -p [プロセスID]
```
```shell
lsof -p 26821
lsof: WARNING: can't stat() tracefs file system /sys/kernel/debug/tracing
      Output information may be incomplete.
COMMAND   PID USER   FD      TYPE             DEVICE SIZE/OFF    NODE NAME
php     26821  htf  cwd       DIR                8,4     4096 5375979 /home/htf/workspace/swoole/examples
php     26821  htf  rtd       DIR                8,4     4096       2 /
php     26821  htf  txt       REG                8,4 24192400 6160666 /opt/php/php-5.6/bin/php
php     26821  htf  DEL       REG                0,5          7204965 /dev/zero
php     26821  htf  DEL       REG                0,5          7204960 /dev/zero
php     26821  htf  DEL       REG                0,5          7204958 /dev/zero
php     26821  htf  DEL       REG                0,5          7204957 /dev/zero
php     26821  htf  DEL       REG                0,5          7204945 /dev/zero
php     26821  htf  mem       REG                8,4   761912 6160770 /opt/php/php-5.6/lib/php/extensions/debug-zts-20131226/gd.so
php     26821  htf  mem       REG                8,4  2769230 2757968 /usr/local/lib/libcrypto.so.1.1
php     26821  htf  mem       REG                8,4   162632 6322346 /lib/x86_64-linux-gnu/ld-2.23.so
php     26821  htf  DEL       REG                0,5          7204959 /dev/zero
php     26821  htf    0u      CHR             136,20      0t0      23 /dev/pts/20
php     26821  htf    1u      CHR             136,20      0t0      23 /dev/pts/20
php     26821  htf    2u      CHR             136,20      0t0      23 /dev/pts/20
php     26821  htf    3r      CHR                1,9      0t0      11 /dev/urandom
php     26821  htf    4u     IPv4            7204948      0t0     TCP *:9501 (LISTEN)
php     26821  htf    5u     IPv4            7204949      0t0     UDP *:9502 
php     26821  htf    6u     IPv6            7204950      0t0     TCP *:9503 (LISTEN)
php     26821  htf    7u     IPv6            7204951      0t0     UDP *:9504 
php     26821  htf    8u     IPv4            7204952      0t0     TCP localhost:8000 (LISTEN)
php     26821  htf    9u     unix 0x0000000000000000      0t0 7204953 type=DGRAM
php     26821  htf   10u     unix 0x0000000000000000      0t0 7204954 type=DGRAM
php     26821  htf   11u     unix 0x0000000000000000      0t0 7204955 type=DGRAM
php     26821  htf   12u     unix 0x0000000000000000      0t0 7204956 type=DGRAM
php     26821  htf   13u  a_inode               0,11        0    9043 [eventfd]
php     26821  htf   14u     unix 0x0000000000000000      0t0 7204961 type=DGRAM
php     26821  htf   15u     unix 0x0000000000000000      0t0 7204962 type=DGRAM
php     26821  htf   16u     unix 0x0000000000000000      0t0 7204963 type=DGRAM
php     26821  htf   17u     unix 0x0000000000000000      0t0 7204964 type=DGRAM
php     26821  htf   18u  a_inode               0,11        0    9043 [eventpoll]
php     26821  htf   19u  a_inode               0,11        0    9043 [signalfd]
php     26821  htf   20u  a_inode               0,11        0    9043 [eventpoll]
php     26821  htf   22u     IPv4            7452776      0t0     TCP localhost:9501->localhost:59056 (ESTABLISHED)
```

* soファイルはプロセスがロードしているダイナミックリンクライブラリです
* IPv4/IPv6 TCP (LISTEN) はサーバーが待ち受けているポートです
* UDP はサーバーが待ち受けているUDPポートです
* unix type=DGRAM の場合、それはプロセスが作成した[Unixソケット](/learn?id=什么是IPC)です
* IPv4 (ESTABLISHED) はサーバーに接続しているTCPクライアントを示し、クライアントのIPとポート、および状態(ESTABLISHED)が含まれています
* 9u / 10u はファイルハンドルのfd値(ファイルディスクリプタ)を表します
* 他の詳細情報についてはlsofのマニュアルを参照してください
## perf

`perf`ツールはLinuxカーネルが提供する非常に強力なダイナミックトレースツールであり、`perf top`コマンドは実行中のプログラムのパフォーマンス問題をリアルタイムで分析するために使用できます。`callgrind`、`xdebug`、`xhprof`などとは異なり、`perf`はコードの変更なしにプロファイル結果ファイルをエクスポートすることができます。
### 使用法

```shell
perf top -p [プロセスID]
```
### 出力結果

![perf topの出力結果](../_images/other/perf.png)

perfの結果には、現在のプロセスの実行中に各C関数の実行時間が明確に表示されており、どのC関数がCPUリソースを多く使用しているかがわかります。

Zend VMに精通している場合、特定のZend関数が多く呼び出されている場合、プログラム内で特定の関数が大量に使用されていることを示し、CPU使用率が高すぎるため、対処方法として最適化を行うことができます。

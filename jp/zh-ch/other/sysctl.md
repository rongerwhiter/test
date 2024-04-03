# カーネルパラメータの調整

## ulimitの設定

`ulimit -n`を100000またはそれ以上に調整する必要があります。 コマンドラインで`ulimit -n 100000`を実行して変更できます。変更できない場合は`/etc/security/limits.conf`に以下の設定を追加する必要があります。

```
* soft nofile 262140
* hard nofile 262140
root soft nofile 262140
root hard nofile 262140
* soft core unlimited
* hard core unlimited
root soft core unlimited
root hard core unlimited
```

注意：`limits.conf`ファイルを変更した後には、システムを再起動して有効にする必要があります。

## カーネルの設定

`Linux`オペレーティングシステムには、カーネルパラメータを変更する方法が3つあります：

- `/etc/sysctl.conf`ファイルを変更し、`key = value`形式の設定オプションを追加してから、`sysctl -p`を呼び出して新しい設定をロードします。
- `sysctl`コマンドを使用して一時的に変更します。例： `sysctl -w net.ipv4.tcp_mem="379008 505344 758016"`
- `/proc/sys/`ディレクトリ内のファイルを直接変更します。例： `echo "379008 505344 758016" > /proc/sys/net/ipv4/tcp_mem`

> 最初の方法はオペレーティングシステムを再起動した後に自動的に有効になります。2番目と3番目の方法は再起動後に無効になります。

### net.unix.max_dgram_qlen = 100

swooleはプロセス間通信にunixソケットdgramを使用しているため、リクエスト量が多い場合はこのパラメータを調整する必要があります。システムのデフォルトは10ですが、100またはそれ以上に設定できます。または、workerプロセスの数を増やして、個々のworkerプロセスに割り当てられるリクエスト量を減らすこともできます。

### net.core.wmem_max

このパラメータを変更して、ソケットのバッファサイズを増やします

```
net.ipv4.tcp_mem  =   379008       505344  758016
net.ipv4.tcp_wmem = 4096        16384   4194304
net.ipv4.tcp_rmem = 4096          87380   4194304
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
```

### net.ipv4.tcp_tw_reuse

ソケット再利用の有無を示すパラメータです。この関数の目的は、サーバーが再起動するときに、ポートをすばやく再利用できるようにすることです。このパラメータを設定しないと、サーバーが再起動した際にポートが適切に解放されないため、起動に失敗する可能性があります。

### net.ipv4.tcp_tw_recycle

ソケットの速やかな再収集を行う必要がある場合、ショートコネクションサーバーはこのパラメータを有効にする必要があります。このパラメータは、TIME-WAITソケットの速やかな収集を有効にするもので、Linuxシステムではデフォルトで0となっています。このパラメータを開くと、NATユーザー接続の安定性が損なわれる可能性があるため、慎重にテストを行った後に有効にしてください。

## メッセージキューの設定

プロセス間通信手段としてメッセージキューを使用する場合は、このカーネルパラメータを調整する必要があります。

- kernel.msgmnb = 4203520、メッセージキューの最大バイト数
- kernel.msgmni = 64、作成可能な最大メッセージキュー数
- kernel.msgmax = 8192、メッセージキューの単一データの最大長さ

## FreeBSD/MacOS

- sysctl -w net.local.dgram.maxdgram=8192
- sysctl -w net.local.dgram.recvspace=200000
  Unixソケットのバッファエリアサイズを変更

## CoreDumpを有効にする

カーネルパラメータを設定します

```
kernel.core_pattern = /data/core_files/core-%e-%p-%t
```

現在のコアダンプファイルの制限を確認するには、`ulimit -c`コマンドを使用します。

```shell
ulimit -c
```

これが0の場合、`/etc/security/limits.conf`を変更して限界を設定する必要があります。

> Core-Dumpを有効にすると、プロセスに異常が発生した場合、プロセスをファイルにエクスポートします。プログラムの問題を調査するのに非常に役立ちます。

## その他の重要な設定

- net.ipv4.tcp_syncookies=1
- net.ipv4.tcp_max_syn_backlog=81920
- net.ipv4.tcp_synack_retries=3
- net.ipv4.tcp_syn_retries=3
- net.ipv4.tcp_fin_timeout = 30
- net.ipv4.tcp_keepalive_time = 300
- net.ipv4.tcp_tw_reuse = 1
- net.ipv4.tcp_tw_recycle = 1
- net.ipv4.ip_local_port_range = 20000 65000
- net.ipv4.tcp_max_tw_buckets = 200000
- net.ipv4.route.max_size = 5242880

## 設定が有効かどうかを確認する

たとえば、`net.unix.max_dgram_qlen = 100`を変更した後、次のコマンドを使用してそれを確認できます。

```shell
cat /proc/sys/net/unix/max_dgram_qlen
```

変更が成功した場合、ここに新しい設定値が表示されます。

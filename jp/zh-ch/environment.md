# Swooleのインストール

`Swoole`拡張機能は、`PHP`の標準拡張機能として構築されています。`phpize`を使用してコンパイル検証スクリプトを生成し、`./configure`でコンパイル構成の検証、`make`でコンパイル、`make install`でインストールします。

* 特別な要件がない場合、必ず`Swoole`の最新バージョンを[こちら](https://github.com/swoole/swoole-src/releases/)からコンパイルしてインストールしてください。
* 現在のユーザーが`root`でない場合、`PHP`のインストールディレクトリに書き込み権限がない可能性がありますので、インストール時には`sudo`または`su`が必要です。
* `git`ブランチで直接`git pull`してコードを更新する場合、再コンパイルする前に必ず`make clean`を実行してください。
* `Linux`（2.3.32以上のカーネル）、`FreeBSD`、`MacOS`の３つのオペレーティングシステムのみをサポートしています。
* 低バージョンのLinuxシステム（例：`CentOS 6`）では、`RedHat`が提供する`devtools`を使用してコンパイルできます。[参考文献](https://blog.csdn.net/ppdouble/article/details/52894271)。
* `Windows`プラットフォームでは、`WSL（Windows Subsystem for Linux）`または`CygWin`を使用できます。
* 一部の拡張機能と`Swoole`拡張機能が互換性がない場合があります。[拡張機能の衝突](/getting_started/extension)を参照してください。
## インストールの準備

インストールする前に、システムに以下のソフトウェアがインストールされていることを確認してください。

- バージョン `4.8` では `php-7.2` 以上が必要です
- バージョン `5.0` では `php-8.0` 以上が必要です
- `gcc-4.8` 以上
- `make`
- `autoconf`
## 迅速なインストール

> 1. Swooleのソースコードをダウンロード

* [https://github.com/swoole/swoole-src/releases](https://github.com/swoole/swoole-src/releases)
* [https://pecl.php.net/package/swoole](https://pecl.php.net/package/swoole)
* [https://gitee.com/swoole/swoole/tags](https://gitee.com/swoole/swoole/tags)

> 2. ソースコードからのコンパイルとインストール

ソースコードパッケージをダウンロードしたら、ターミナルでソースコードディレクトリに移動し、以下のコマンドを実行してコンパイルとインストールを行います。

!> Ubuntuに`phpize`がない場合は、`sudo apt-get install php-dev`コマンドを実行して`phpize`をインストールしてください。

```shell
cd swoole-src && \
phpize && \
./configure && \
sudo make && sudo make install
```

> 3. 拡張機能を有効化

システムに正常にコンパイルされインストールされた後、`php.ini`に`extension=swoole.so`の行を追加してSwoole拡張機能を有効にします。
## 進んだ完全なコンパイルの例

!> Swooleに初めて触れる開発者は、まず上記の簡単なコンパイルを試してください。さらなる必要がある場合は、具体的な要件やバージョンに基づいて、以下の例のコンパイルパラメータを調整できます。[コンパイルパラメータの参考](/environment?id=コンパイルオプション)

以下のスクリプトは`master`ブランチのソースコードをダウンロードしてコンパイルします。すべての依存関係がインストールされていることを確認してください。そうでないと、さまざまな依存関係のエラーが発生します。

```shell
mkdir -p ~/build && \
cd ~/build && \
rm -rf ./swoole-src && \
curl -o ./tmp/swoole.tar.gz https://github.com/swoole/swoole-src/archive/master.tar.gz -L && \
tar zxvf ./tmp/swoole.tar.gz && \
mv swoole-src* swoole-src && \
cd swoole-src && \
phpize && \
./configure \
--enable-openssl --enable-sockets --enable-mysqlnd --enable-swoole-curl --enable-cares --enable-swoole-pgsql && \
sudo make && sudo make install
```
## PECL

> 注意：PECLのリリースはGitHubのリリースよりも遅れています

SwooleプロジェクトはPHP公式拡張ライブラリに収録されており、手動でダウンロードしてコンパイルするだけでなく、PHP公式の`pecl`コマンドを使用して一括でダウンロードしてインストールすることもできます

```shell
pecl install swoole
```

PECLを使用してSwooleをインストールする際に、インストールプロセス中に特定の機能を有効にするかどうかを尋ねられることがあります。これは、例えば、インストールを実行する前に指定することもできます：

```shell
pecl install -D 'enable-sockets="no" enable-openssl="yes" enable-http2="yes" enable-mysqlnd="yes" enable-swoole-json="no" enable-swoole-curl="yes" enable-cares="yes"' swoole

#または
pecl install --configureoptions 'enable-sockets="no" enable-openssl="yes" enable-http2="yes" enable-mysqlnd="yes" enable-swoole-json="no" enable-swoole-curl="yes" enable-cares="yes"' swoole
```
## Swooleをphp.iniに追加する

最後に、コンパイルとインストールが成功したら、`php.ini`に以下を追加してください。

```ini
extension=swoole.so
```

`php -m`コマンドを使って`swoole.so`が正常に読み込まれたかどうかを確認できます。もし読み込まれていない場合は、`php.ini`のパスが間違っている可能性があります。  
`Loaded Configuration File`の項目に表示される`php.ini`ファイルのパスを特定するために`php --ini`コマンドを使用できます。もし値が`none`と表示された場合は、どの`php.ini`ファイルも読み込まれていないことを意味し、自分で作成する必要があります。

!> `PHP`のバージョンサポートと`PHP`公式のメンテナンスバージョンを一致させてください。[PHPサポートバージョンのスケジュール](http://php.net/supported-versions.php)を参照してください。
## 他のプラットフォームでのコンパイル

ARMプラットフォーム（Raspberry PI）

* `GCC`をクロスコンパイルに使用します。
* `Swoole`をコンパイルする際には、`Makefile`を手動で変更して`-O2`のコンパイルオプションを削除する必要があります。

MIPSプラットフォーム（OpenWrtルーター）

* GCCをクロスコンパイルに使用します。

Windows WSL

`Windows 10` システムでは `Linux` サブシステムサポートが追加され、`BashOnWindows` 環境でも `Swoole` を使用できます。インストールコマンド

```shell
apt-get install php7.0 php7.0-curl php7.0-gd php7.0-gmp php7.0-json php7.0-mysql php7.0-opcache php7.0-readline php7.0-sqlite3 php7.0-tidy php7.0-xml  php7.0-bcmath php7.0-bz2 php7.0-intl php7.0-mbstring  php7.0-mcrypt php7.0-soap php7.0-xsl  php7.0-zip
pecl install swoole
echo 'extension=swoole.so' >> /etc/php/7.0/mods-available/swoole.ini
cd /etc/php/7.0/cli/conf.d/ && ln -s ../../mods-available/swoole.ini 20-swoole.ini
cd /etc/php/7.0/fpm/conf.d/ && ln -s ../../mods-available/swoole.ini 20-swoole.ini
```

!> `WSL` 環境では `daemonize` オプションを無効にする必要があります。  
`17101`未満の`WSL`の場合、ソースコードのインストール後に `configure` を変更して `config.h` で `HAVE_SIGNALFD` を無効にする必要があります。
## Docker公式イメージ

- GitHub: [https://github.com/swoole/docker-swoole](https://github.com/swoole/docker-swoole)  
- dockerhub: [https://hub.docker.com/r/phpswoole/swoole](https://hub.docker.com/r/phpswoole/swoole)
## コンパイルオプション

ここには`./configure`のコンパイル設定に追加する追加パラメータがあります。これにより特定の機能が有効になります。
### General Parameters
#### --enable-openssl

`SSL`サポートを有効にする

> 使用操作系统提供的`libssl.so`动态连接库
#### --with-openssl-dir

`SSL`サポートを有効にし、`openssl`ライブラリのパスを指定します。パスを指定する必要があります。例：`--with-openssl-dir=/opt/openssl/`
#### --enable-http2

開始對`HTTP2`的支持

> 依賴`nghttp2`庫。在`V4.3.0`版本後不再需要安裝依賴，改為內置，但仍需要增加該編譯參數來開啟`http2`支持，`Swoole5`默認啟用該參數。
#### --enable-swoole-json

[swoole_substr_json_decode](/functions?id=swoole_substr_json_decode)のサポートを有効にします。`Swoole5`以降、このパラメーターはデフォルトで有効になっています

>`json`拡張機能に依存しており、`v4.5.7`のバージョンで利用可能です。
#### --enable-swoole-curl

启用[SWOOLE_HOOK_NATIVE_CURL](/runtime?id=swoole_hook_native_curl)的支持

> `v4.6.0`版本可用。如果编译报错`curl/curl.h: No such file or directory`，请查看[安装问题](/question/install?id=libcurl)
#### --enable-cares

`c-ares` のサポートを有効にする

> このオプションは `c-ares` ライブラリに依存しており、`v4.7.0` のバージョンが必要です。もし `ares.h: No such file or directory` というエラーが発生した場合は、[インストールの問題](/question/install?id=libcares) を参照してください。
#### --with-jemalloc-dir

`jemalloc` サポートを有効にする
#### --enable-brotli

`libbrotli` 圧縮サポートを有効にする
#### --with-brotli-dir

`libbrotli`の圧縮サポートを有効にし、`libbrotli`ライブラリのパスを指定します。パスを引数として指定します。例: `--with-brotli-dir=/opt/brotli/`
#### --enable-swoole-pgsql

`PostgreSQL`データベースのコルーチン化を有効にします。

> `Swoole5.0`以前は、`PostgreSQL`をコルーチン化するためにコルーチンクライアントを使用していましたが、`Swoole5.1`以降、コルーチンクライアントを使用して`PostgreSQL`をコルーチン化するだけでなく、ネイティブの`pdo_pgsql`を使用して`PostgreSQL`をコルーチン化することもできます。
#### --with-swoole-odbc

`pdo_odbc`をコルーチン化するために有効化します。このパラメータを有効にすると、`odbc`インターフェースをサポートするすべてのデータベースがコルーチン化されます。

>使用可能：`v5.1.0`以降
#### --with-swoole-oracle

`pdo_oci`のコルーチン化を有効にし、このパラメータを有効にすると、`oracle`データベースの追加、削除、更新、検索すべてがコルーチン操作をトリガーします。

>`v5.1.0`のバージョン以降利用可能
#### --enable-swoole-sqlite

`pdo_sqlite`のコルーチン対応を有効にします。このパラメータを有効にすると、`sqlite`データベースの挿入、削除、更新、検索はすべてコルーチン操作をトリガーします。

>`v5.1.0`リリース以降で利用可能です。
### 特殊参数

!> **歴史的な理由がない限り、有効化しないでください**
#### --enable-mysqlnd

`mysqlnd` サポートを有効にし、`Coroutine\MySQL::escapse` メソッドを有効にします。このオプションを有効にすると、`PHP` は `mysqlnd` モジュールを持つ必要があります。そうでない場合、`Swoole` は動作しなくなります。

> `mysqlnd` 拡張に依存します。
#### --enable-sockets

PHPの`sockets`リソースのサポートを追加します。このパラメータを有効にすると、[Swoole\Event::add](/event?id=add)を使用して`sockets`拡張で作成された接続を`Swoole`の[イベントループ](/learn?id=什么是eventloop)に追加できます。  
`Server`および`Client`の[getSocket()](/server/methods?id=getsocket)メソッドも、このコンパイルパラメータに依存します。

> 依赖`sockets`拓展, `v4.3.2`版本后该参数的作用被削弱了, 因为Swoole内置的[Coroutine\Socket](/coroutine_client/socket)可以完成大部分事情
### デバッグパラメータ

!> **本番環境での有効化は許可されていません**
#### --enable-debug

デバッグモードを有効にします。`gdb`を使用して`Swoole`をビルドする際にこのパラメータを追加する必要があります。
#### --enable-debug-log

```
// Open debug log for the kernel. (Swoole version >= 4.2.0)
```
#### --enable-trace-log

トレースログを有効にすると、さまざまなデバッグログの詳細が出力されます。カーネル開発時にのみ使用してください。
#### --enable-swoole-coro-time

協働実行時間の計算を有効にすると、Swoole\Coroutine::getExecuteTime()を使用して協働実行時間を計算できます。I/O待機時間は含まれません。
### PHP compile parameters

### PHPコンパイルパラメータ
#### --enable-swoole

PHPにSwoole拡張機能を静的にコンパイルし、下記の手順に従えば`--enable-swoole`オプションを利用できます。

```shell
cp -r /home/swoole-src /home/php-src/ext
cd /home/php-src
./buildconf --force
./configure --help | grep swoole
```

!> このオプションはSwooleをコンパイルするのではなく、PHPをコンパイルする際に使用されます。
## よくある質問

* [Swooleのインストールに関するよくある質問](/question/install)

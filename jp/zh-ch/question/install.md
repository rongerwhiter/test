このページでは、次の手順に従って問題を解決する方法について説明します。

1. **ソフトウェアを再起動する。**
   この手順が問題を解決することがあります。

2. **ソフトウェアをアンインストールして再インストールする。**
   インストール時に何か問題があった可能性がありますので、再インストールしてみてください。

3. **最新バージョンにアップデートする。**
   インストールされているソフトウェアが最新バージョンかどうかチェックしてください。

もし問題が解決しない場合や他の質問がある場合は、お知らせください。お手伝いさせていただきます。
## Swooleバージョンのアップグレード

peclを使用してインストールおよびアップグレードすることができます。

```shell
pecl upgrade swoole
```

また、github/gitee/peclから新しいバージョンをダウンロードして、再インストールしてコンパイルすることもできます。

* Swooleのバージョンを更新する際、古いバージョンをアンインストールまたは削除する必要はありません。インストールプロセスで古いバージョンが上書きされます。
* Swooleをコンパイルしてインストールすると、追加のファイルはありません。1 つのswoole.so ファイルのみがあります。他のマシンでコンパイルされたバイナリバージョンの場合は、swoole.so を直接上書きすればバージョンの切り替えができます。
* git clone でコードを取得した場合、git pull でコードを更新した後、必ず `phpize`、`./configure`、`make clean`、`make install` を再実行してください。
* 対応するDockerを使用して、Swooleのバージョンをアップグレードすることもできます。
```php
## PHP Information 结果

 - 在 php -m 中有但是在 php -i 或 phpinfo 中却没有扩展信息

这表明在 CLI 模式下可能有，可以通过运行以下命令来检查：`php --ri swoole`

如果输出了 Swoole 的扩展信息，那么你的安装是成功的！

**99.999% 的人在这一步就已经能成功使用 Swoole 了**

不需要担心 `php -m` 或者 `phpinfo` 网页上是否有 Swoole 扩展信息

因为 Swoole 是在 CLI 模式下运行的，在传统的 fpm 模式下功能受限

在 fpm 模式下，任何异步/协程等核心功能都**无法使用**，99.999% 的人无法在 fpm 模式下获得他们想要的部分，却为何在 fpm 模式下没有扩展信息而纠结

**先确保你真正了解 Swoole 的运行模式，然后再继续探究安装信息问题！**
```
### 原因

Swooleをコンパイルしてインストールした後、`phpinfo`ページ上では`php-fpm/apache`に表示されますが、コマンドラインの`php -m`には表示されない場合、原因はおそらく`cli/php-fpm/apache`が異なるphp.ini設定を使用しているためです。
### 解決策

1. PHP.ini ファイルの場所を確認します

`cli` コマンドラインで `php -i | grep php.ini` または `php --ini` を実行し、php.ini の絶対パスを見つけます

`php-fpm/apache` では、`phpinfo` ページから php.ini の絶対パスを見つけます

2. 対応する php.ini ファイルに `extension=swoole.so` があるか確認します

```shell
cat /path/to/php.ini | grep swoole.so
```
## pcre.h: No such file or directory

Swooleの拡張機能をコンパイルするときに

```bash
fatal error: pcre.h: No such file or directory
```

というエラーが表示された場合、pcreが不足しているため、libpcreをインストールする必要があります。
### ubuntu/debian

```shell
sudo apt-get install libpcre3 libpcre3-dev
```
### centos/redhat

```shell
sudo yum install pcre-devel
```
### その他のLinux

[PCREの公式ウェブサイト](http://www.pcre.org/)からソースコードをダウンロードし、`pcre`ライブラリをコンパイルしてインストールします。

`PCRE`ライブラリがインストールされたら、`swoole`を再コンパイルしてインストールする必要があります。その後、`php --ri swoole`を使用して`swoole`拡張機能に関する情報を表示し、`pcre => enabled`が表示されているか確認します。
## '__builtin_saddl_overflow' was not declared in this scope

 ```
error: '__builtin_saddl_overflow' was not declared in this scope
  if (UNEXPECTED(__builtin_saddl_overflow(Z_LVAL_P(op1), 1, &lresult))) {

note: in definition of macro 'UNEXPECTED'
 # define UNEXPECTED(condition) __builtin_expect(!!(condition), 0)
```

This is a known issue. The problem is that the default gcc on CentOS lacks the necessary definition, even after upgrading gcc, the old compiler is still found by PECL.

To install the driver, you need to first upgrade gcc by installing the devtoolset collection as follows:

```shell
sudo yum install centos-release-scl
sudo yum install devtoolset-7
scl enable devtoolset-7 bash
```
## fatal error: 'openssl/ssl.h' file not found

コンパイル時に、指定された openssl ライブラリのパスを示す [--with-openssl-dir](/environment?id=通用参数) フラグを追加してください。

!> [pecl](/environment?id=pecl)を使用してSwooleをインストールする場合、opensslを有効にするには `enable openssl support? [no] : yes --with-openssl-dir=/opt/openssl/` のように [--with-openssl-dir](/environment?id=通用参数) フラグを追加することもできます。
## make or make install cannot be executed or compile error

注意：PHP消息：PHP警告：PHP启动：swoole：无法初始化模块  
使用模块API=20090626编译的模块  
PHP使用的模块API=20121212编译  
这些选项需要匹配  
位置未知第0行  

PHP版本和编译时使用的`phpize`和`php-config`不对应，需要使用绝对路径来进行编译，以及使用绝对路径来执行PHP。

```shell
/usr/local/php-5.4.17/bin/phpize
./configure --with-php-config=/usr/local/php-5.4.17/bin/php-config

/usr/local/php-5.4.17/bin/php server.php
```  
## xdebugのインストール

```shell
git clone git@github.com:swoole/sdebug.git -b sdebug_2_9 --depth=1

cd sdebug

phpize
./configure
make clean
make
make install

# もしphpize、php-configなどの設定ファイルがデフォルトの場合、次のコマンドを実行できます
./rebuild.sh
```

php.iniファイルを変更して、以下の情報を追加します

```ini
zend_extension=xdebug.so

xdebug.remote_enable=1
xdebug.remote_autostart=1
xdebug.remote_host=localhost
xdebug.remote_port=8000
xdebug.idekey="xdebug"
```

正常にロードされたか確認します

```shell
php --ri sdebug
```
## configure: error: C preprocessor "/lib/cpp" fails sanity check

インストール中に以下のエラーが発生する場合

```shell
configure: error: C preprocessor "/lib/cpp" fails sanity check
```

必要な依存ライブラリが不足していることを示しています。以下のコマンドを使用してインストールできます

```shell
yum install glibc-headers
yum install gcc-c++
```
MacOSでPHP7.4.11+を使用して新しいバージョンのSwooleをコンパイルすると、次のようなエラーが発生することがあります。

```shell
/usr/local/Cellar/php/7.4.12/include/php/Zend/zend_operators.h:523:10: error: 'asm goto' constructs are not supported yet
        __asm__ goto(
                ^
/usr/local/Cellar/php/7.4.12/include/php/Zend/zend_operators.h:586:10: error: 'asm goto' constructs are not supported yet
        __asm__ goto(
                ^
/usr/local/Cellar/php/7.4.12/include/php/Zend/zend_operators.h:656:10: error: 'asm goto' constructs are not supported yet
        __asm__ goto(
                ^
/usr/local/Cellar/php/7.4.12/include/php/Zend/zend_operators.h:766:10: error: 'asm goto' constructs are not supported yet
        __asm__ goto(
                ^
4 errors generated.
make: *** [ext-src/php_swoole.lo] Error 1
ERROR: `make' failed
```

対処方法：`/usr/local/Cellar/php/7.4.12/include/php/Zend/zend_operators.h`のソースコードを編集し、対応するヘッダーファイルのパスを変更します。次のコードで`ZEND_USE_ASM_ARITHMETIC`を`0`に変更します。

```c
#if defined(HAVE_ASM_GOTO) && !__has_feature(memory_sanitizer)
# define ZEND_USE_ASM_ARITHMETIC 1
#else
# ifdef ZEND_USE_ASM_ARITHMETIC 0
#endif
`--enable-swoole-curl`オプションを有効にした後、Swoole拡張をコンパイルしようとすると、次のエラーが発生します。

```bash
fatal error: curl/curl.h: No such file or directory
```

これは、curlの依存関係が不足しているためであり、libcurlをインストールする必要があります。
### ubuntu/debian

```shell
sudo apt-get install libcurl4-openssl-dev
```
### CentOS/Red Hat

```shell
sudo yum install libcurl-devel
````

### CentOS/Red Hat

```shell
sudo yum install libcurl-devel
````

### CentOS/Red Hat

```shell
sudo yum install libcurl-devel
````

### CentOS/Red Hat

```shell
sudo yum install libcurl-devel
````

### アルパイン

```shell
apk add curl-dev
`````


## fatal error: ares.h: No such file or directory :id=libcares

`--enable-cares` オプションを有効にした後、Swoole拡張機能をコンパイルしようとすると、以下のエラーが発生します。

```bash
fatal error: ares.h: No such file or directory
```

これは c-ares 依存関係が不足しているためで、libcares をインストールする必要があります。
### ubuntu/debian

```shell
sudo apt-get install libc-ares-dev
```
### centos/redhat

```shell
sudo yum install c-ares-devel
```````

### centos/redhat

```shell
sudo yum install c-ares-devel
```  
### アルパイン

```shell
apk add c-ares-dev
```
### MacOs

```shell
brew install c-ares
```  

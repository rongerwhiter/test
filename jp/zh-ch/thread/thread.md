# Swoole\Thread

`6.0` バージョンからは、マルチスレッドサポートが提供されており、スレッドAPIを使用してマルチプロセスを代替できます。複数のプロセスに比べて、`Thread` はより豊富な並列データコンテナを提供し、ゲームサーバーや通信サーバーの開発において便利です。
## コンパイル
- `PHP`は`ZTS`モードで必要であり、`PHP`をコンパイルする際には`--enable-zts`を追加する必要があります。
- `Swoole`をコンパイルする際には、`--enable-swoole-thread`コンパイルオプションを追加する必要があります。
## 情報を確認

```shell
php -v
PHP 8.1.23 (cli) (built: Mar 20 2024 19:48:19) (ZTS)
Copyright (c) The PHP Group
Zend Engine v4.1.23, Copyright (c) Zend Technologies
```

`(ZTS)` はスレッドセーフが有効であることを示しています

```shell
php --ri swoole
php --ri swoole

swoole

Swoole => enabled
...
thread => enabled
...
```

`thread => enabled` はマルチスレッドサポートが有効になっていることを示しています
## スレッドを作成する

```php
Thread::exec(string $script_file, array ...$argv);
```

親スレッドから継承されるリソースはないため、作成した子スレッドで以下のものがクリアされます。それらは再度作成または設定する必要があります：
- ロードされていた `PHP` ファイル、再度 `include/require` が必要
- `autoload`
- クラス、関数、定数、これらはクリアされ再度 `PHP` ファイルをロードして作成する必要があります
- グローバル変数、例えば `$GLOBALS`、`$_GET/$_POST` など、これらはクリアされます
- クラスの静的プロパティ、関数の静的変数、これらは初期値にリセットされます
- `php.ini` オプション、例えば `error_reporting()` は、子スレッド内で再設定する必要があります

データを子スレッドに渡すためにはスレッドパラメータを使用する必要があります。子スレッド内でも新しいスレッドを作成することができます。
### パラメータ
- `$script_file`: スレッドが開始された後に実行するスクリプト
- `...$argv`: スレッドに渡すパラメータ。シリアライズ可能な変数でなければならず、`resource` リソースハンドルを渡すことはできません。サブスレッド内で `Thread::getArguments()` を使用して取得できます。
### 戻り値
`Thread`オブジェクトが返され、親スレッドで`join()`などの操作を子スレッドに対して行うことができます。

スレッドオブジェクトが破棄されると、自動的に `join()`が実行されて子スレッドの終了を待ちます。これによりブロックが発生する可能性がありますが、`$thread->detach()`メソッドを使用して子スレッドを親スレッドから分離し、独立して実行させることも可能です。
```php
use Swoole\Thread;

$args = Thread::getArguments();
$c = 4;

if (empty($args)) {
    # 親スレッド
    for ($i = 0; $i < $c; $i++) {
        $threads[] = Thread::exec(__FILE__, $i);
    }
    for ($i = 0; $i < $c; $i++) {
        $threads[$i]->join();
    }
} else {
    # 子スレッド
    echo "Thread #" . $args[0] . "\n";
    while (1) {
        sleep(1);
        file_get_contents('https://www.baidu.com/');
    }
}
```
## 定数
- `Thread::HARDWARE_CONCURRENCY` はハードウェアシステムでサポートされる並行スレッド数を取得します。つまり、 `CPU` のコア数です。

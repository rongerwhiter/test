# 拡張機能の衝突

いくつかのトレースデバッグを行っている`PHP`拡張機能が大量のグローバル変数を使用しているため、`Swoole`のコルーチンがクラッシュする可能性があります。次の関連する拡張機能を無効にしてください：

* phptrace
* aop
* molten
* xhprof
* phalcon（`Swoole`のコルーチンは`phalcon`フレームワークで動作しません）
## Xdebug Support
`Swoole` プログラムをデバッグするために、 `5.1` バージョン以降で `xdebug` 拡張機能を直接使用することができます。これを有効にするには、コマンドライン引数を使用するか、`php.ini` を変更します。

```ini
swoole.enable_fiber_mock=On
```

または

```shell
php -d swoole.enable_fiber_mock=On your_file.php
```

# コルーチンのデバッグ

`Swoole`コルーチンを使用する際に、次の方法でデバッグすることができます。

## GDBデバッグ

### GDBに入る

```shell
gdb php test.php
```

### gdbinit

```shell
(gdb) source /path/to/swoole-src/gdbinit
```

### ブレークポイントを設定する

例えば `co::sleep` 関数

```shell
(gdb) b zim_swoole_coroutine_util_sleep
```

### 現在のプロセスのすべてのコルーチンと状態をプリントする

```shell
(gdb) co_list 
coroutine 1 SW_CORO_YIELD
coroutine 2 SW_CORO_RUNNING
```

### 現在実行中のコルーチンのコールスタックをプリントする

```shell
(gdb) co_bt 
coroutine cid:[2]
[0x7ffff148a100] Swoole\Coroutine->sleep(0.500000) [internal function]
[0x7ffff148a0a0] {closure}() /home/shiguangqi/php/swoole-src/examples/coroutine/exception/test.php:7 
[0x7ffff141e0c0] go(object[0x7ffff141e110]) [internal function]
[0x7ffff141e030] (main) /home/shiguangqi/php/swoole-src/examples/coroutine/exception/test.php:10
```

### 特定のコルーチンIDのコールスタックをプリントする

``` shell
(gdb) co_bt 1
[0x7ffff1487100] Swoole\Coroutine->sleep(0.500000) [internal function]
[0x7ffff14870a0] {closure}() /home/shiguangqi/php/swoole-src/examples/coroutine/exception/test.php:3 
[0x7ffff141e0c0] go(object[0x7ffff141e110]) [internal function]
[0x7ffff141e030] (main) /home/shiguangqi/php/swoole-src/examples/coroutine/exception/test.php:10 
```

### グローバルコルーチンの状態をプリントする

```shell
(gdb) co_status 
	 stack_size: 2097152
	 call_stack_size: 1
	 active: 1
	 coro_num: 2
	 max_coro_num: 3000
	 peak_coro_num: 2
```

## PHPコードデバッグ

現在のプロセス内のすべてのコルーチンをイテレートして、コールスタックをプリントします。

```php
Swoole\Coroutine::listCoroutines(): Swoole\Coroitine\Iterator
```

！> `4.1.0`またはそれ以上のバージョンが必要です

* イテレーターが返され、`foreach`を使用して繰り返したり、`iterator_to_array`を使用して配列に変換したりできます。

```php
use Swoole\Coroutine;
$coros = Coroutine::listCoroutines();
foreach($coros as $cid)
{
	var_dump(Coroutine::getBackTrace($cid));
}
```

[Swoole微課程のビデオチュートリアル](https://course.swoole-cloud.com/course-video/66) を参照してください。

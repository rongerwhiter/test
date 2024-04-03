# Thread management

スレッド管理
## Thread::join()

子スレッドが終了するのを待ちます。子スレッドがまだ実行中の場合、`join()` はブロックされます。

```php
$thread = Thread::exec(__FILE__, $i);
$thread->join();
```
## Thread::joinable()

サブスレッドが終了しているかどうかを確認します。
### 戻り値
- `true` は、サブスレッドが終了していることを示し、`join()` を呼び出してもブロックされません
- `false` は、終了していないことを示します

```php
$thread = Thread::exec(__FILE__, $i);
var_dump($thread->joinable());
```
## Thread::detach()

親スレッドから子スレッドを分離し、`join()` が必要なく、スレッドの終了を待つ必要がなくなります。リソースを回収します。

```php
$thread = Thread::exec(__FILE__, $i);
$thread->detach();
unset($thread);
```
```php
var_dump(Thread::getId());
```
## Thread::getId()

静的メソッドで、現在のスレッドの `ID` を取得します。子スレッドで呼び出されます。

```php
var_dump(Thread::getId());
```
## Thread::getArguments()

静的メソッドは、現在のスレッドの引数を取得します。子スレッド内で呼び出され、親スレッドは `Thread::exec()` を実行する際に引数として渡されます。

```php
var_dump(Thread::getArguments());
```
## Thread::$id

このオブジェクトのプロパティを使用して、サブスレッドの `ID` を取得します

```php
$thread = Thread::exec(__FILE__, $i);
var_dump($thread->id);
```

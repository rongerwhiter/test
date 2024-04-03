# 協調プログラミングの要点

Swooleの[協調処理](/coroutine)機能を使用する際には、この章のプログラミング要点を注意深く読んでください。
## プログラミングパラダイム

* コルーチン内ではグローバル変数の使用は禁止されています
* コルーチンは`use`キーワードを使用して外部変数を現在のスコープに取り込むことができますが、参照は使用禁止です
* コルーチン間の通信には[Channel](/coroutine/channel)を使用する必要があります

!> つまり、コルーチン間の通信には、グローバル変数を使用したり、外部変数を現在のスコープに取り込んだりせず、`Channel`を使用する必要があります

* プロジェクトで`zend_execute_ex`または`zend_execute_internal`を`hook`する場合、Cスタックに特に注意する必要があります。[Co::set](/coroutine/coroutine?id=set)を使用してCスタックのサイズを再設定できます

!> これらの入口関数を`hook`すると、通常、フラットなPHP命令呼び出しが`C`関数呼び出しに変わり、Cスタックの消費量が増加する場合があります。
## コルーチンの終了

Swooleの古いバージョンでは、コルーチン内で`exit`を使用してスクリプトを強制終了すると、予期しない結果や`coredump`が発生し、Swooleサービス内で`exit`を使用するとサービスプロセス全体が終了し、内部のすべてのコルーチンが異常終了する重大な問題が発生します。Swooleは長い間、開発者が`exit`を使用することを禁止していますが、代わりに、例外をスローしてトップレベルの`catch`で同じ終了ロジックを実装するなど、非常に異なる方法で終了することができます。

!> v4.2.2以降のバージョンでは、スクリプト(`http_server`が作成されていない)は現在のコルーチンのみで`exit`を使って終了することができます。

Swooleの**v4.1.0**以降では、`コルーチン`や`サーバーのイベントループ`でPHPの`exit`を直接サポートしています。この場合、Swooleは自動的にキャッチ可能な`Swoole\ExitException`をスローし、開発者は必要な位置でキャッチして、元のPHPと同様の終了ロジックを実装することができます。
### Swoole\ExitException

`Swoole\ExitException`は`Exception`を継承し、`getStatus`と`getFlags`という2つのメソッドが追加されています。

```php
namespace Swoole;

class ExitException extends \Exception
{
	public function getStatus(): mixed
	public function getFlags(): int
}
```
#### getStatus()

`exit($status)` で終了したときに渡される `status` パラメータを取得します。任意の変数タイプをサポートします。

```php
public function getStatus(): mixed
```
#### getFlags()

exit時の環境情報のマスクを取得します。

```php
public function getFlags(): int
```

以下のマスクが現在使用可能です：

| 定数 | 説明 |
| -- | -- |
| SWOOLE_EXIT_IN_COROUTINE | コルーチン内での終了 |
| SWOOLE_EXIT_IN_SERVER | Server内での終了 |
### Instructions
```php
use Swoole\Coroutine;
use function Swoole\Coroutine\run;

function route()
{
    controller();
}

function controller()
{
    your_code();
}

function your_code()
{
    Coroutine::sleep(.001);
    exit(1);
}

run(function () {
    try {
        route();
    } catch (\Swoole\ExitException $e) {
        var_dump($e->getMessage());
        var_dump($e->getStatus() === 1);
        var_dump($e->getFlags() === SWOOLE_EXIT_IN_COROUTINE);
    }
});
```
```php
use function Swoole\Coroutine\run;

$exit_status = 0;
run(function () {
    try {
        exit(123);
    } catch (\Swoole\ExitException $e) {
        global $exit_status;
        $exit_status = $e->getStatus();
    }
});
var_dump($exit_status);
```
## 例外処理

コルーチンプログラミングでは、`try/catch`を使用して例外を処理できます。**ただし、例外はコルーチン内でのみキャッチする必要があり、コルーチン間での例外キャッチは許可されていません**。

!> アプリケーション層で`throw`される`Exception`だけでなく、低レベルで発生するいくつかのエラーもキャッチすることができます。たとえば、`function`、`class`、`method`が存在しない場合でもキャッチできます。
### エラー例

以下のコードでは、`try/catch`と`throw`が異なるコルーチン内で実行されているため、コルーチン内でこの例外をキャッチすることができません。コルーチンが終了する際に未キャッチの例外が発生すると致命的なエラーが発生します。

```bash
PHP Fatal error:  Uncaught RuntimeException
```

```php
try {
	Swoole\Coroutine::create(function () {
		throw new \RuntimeException(__FILE__, __LINE__);
	});
}
catch (\Throwable $e) {
	echo $e;
}
```
### 正しい例

例外をコルーチン内でキャッチする。

```php
function test() {
	throw new \RuntimeException(__FILE__, __LINE__);
}

Swoole\Coroutine::create(function () {
	try {
		test();
	}
	catch (\Throwable $e) {
		echo $e;
	}
});
```
## __get / __set マジックメソッド内でのコルーチン切り替えの禁止

理由：[PHP7内核剖析を参照](https://github.com/pangudashu/php7-internal/blob/40645cfe087b373c80738881911ae3b178818f11/3/zend_object.md)

> **Note:** クラスに__get()メソッドが存在する場合、インスタンス化されたオブジェクトに属性のメモリ(すなわち:properties_table)が割り当てられる際に追加のzvalが割り当てられます。このzvalはHashTable型であり、__get($var)が呼び出される度に、入力された$var名前がこのハッシュテーブルに格納されます。これは再帰呼び出しを防ぐための措置です。以下に例を示します：
> 
> ***public function __get($var) { return $this->$var; }***
>
> この場合、__get()の呼び出しで存在しない属性にアクセスすることになるため、再帰的に__get()を呼び出すと、一直に再帰呼び出しを行います。そのため、__get()を呼び出す前に現在の$varが既に__get()内にあるかどうかを判断し、もしあれば__get()を再度呼び出さず、そうでなければ$varをキーとしてそのHashTableに挿入し、ハッシュ値を次のように設定します：*guard |= IN_ISSET。__get()の呼び出しが完了した後、ハッシュ値を以下のように設定します：*guard &= ~IN_ISSET。
>
> このHashTableは__get()だけでなく、他のマジックメソッドも使用します。そのため、そのハッシュ値の型はzend_longであり、異なるマジックメソッドが異なるビット位置を占有します。さらに、すべてのオブジェクトがこのHashTableを追加で割り当てるわけではなく、オブジェクトの作成時に ***zend_class_entry.ce_flags*** が ***ZEND_ACC_USE_GUARDS*** を含むかどうかで確認し、クラスコンパイル時に__get()、__set()、__unset()、__isset()メソッドが定義されている場合はce_flagsにこのマスクを適用します。

コルーチンが切り替わった後、次回の呼び出しは再帰呼び出しとして判断されます。この問題はPHPの**機能**に起因するものであり、PHP開発チームとの交渉の結果、現在は解決策が見つかっていません。

注意：マジックメソッドにはコルーチン切り替えを引き起こす可能性はありませんが、コルーチンプリエンプティブスケジューリングが有効になっている場合、マジックメソッドが強制的に別のコルーチンに切り替わる可能性があります。

おすすめ：`get`/`set`メソッドを明示的に呼び出すように自分で実装してください

元の問題のリンク：[#2625](https://github.com/swoole/swoole-src/issues/2625)
```python
def divide_numbers(x, y):
    result = x / y
    return result
```

この関数に渡すことによって、0での除算が発生し、重大なエラーが発生する可能性があります。
### 複数のコルーチンで1つの接続を共有する

同期ブロッキングプログラムとは異なり、コルーチンはリクエストを並行して処理するため、同時に多くのリクエストが処理されることがあります。一度クライアント接続を共有すると、異なるコルーチン間でデータが混在する可能性があります。参考：[複数のコルーチンでTCP接続を共有する方法](/question/use?id=client-has-already-been-bound-to-another-coroutine)
### 使用クラスの静的変数/グローバル変数でコンテキストを保存する

複数のコルーチンは並行して実行されるため、コルーチンのコンテキスト内容を保存するためにクラスの静的変数/グローバル変数を使用することはできません。ローカル変数を使用することは安全です。なぜなら、ローカル変数の値は自動的にコルーチンのスタックに保存され、他のコルーチンからコルーチンのローカル変数にアクセスすることはできないからです。
### エラーの例

```php
$server = new Swoole\Http\Server('127.0.0.1', 9501);

$_array = [];
$server->on('request', function ($request, $response) {
    global $_array;
    // リクエスト /a（コルーチン 1）
    if ($request->server['request_uri'] == '/a') {
        $_array['name'] = 'a';
        co::sleep(1.0);
        echo $_array['name'];
        $response->end($_array['name']);
    }
    // リクエスト /b（コルーチン 2）
    else {
        $_array['name'] = 'b';
        $response->end();
    }
});
$server->start();
```

並列リクエストを`2`つ発行します。

```shell
curl http://127.0.0.1:9501/a
curl http://127.0.0.1:9501/b
```

- コルーチン`1`でグローバル変数`$_array['name']`の値を`a`に設定しています
- コルーチン`1`は`co::sleep`を呼び出して一時停止します
- コルーチン`2`が実行され、`$_array['name']`の値は`b`に設定され、コルーチン2が終了します
- この時にタイマーが戻り、低レベルでコルーチン`1`の実行を再開します。しかし、コルーチン`1`のロジックにはコンテキストの依存関係があります。`$_array['name']`の値を再度出力しようとすると、プログラムの期待値は`a`ですが、この値は既にコルーチン`2`によって変更されているため、実際の結果は`b`になります。これによって不整合が発生します
- 同様に、クラスの静的変数`Class::$array`やグローバルオブジェクトプロパティ`$object->array`、他のスーパーグローバル変数`$GLOBALS`などを使用してコンテキストを保存することは非常に危険です。意図しない動作が発生する可能性があります。

![](../_images/coroutine/notice-1.png)
#### 正しい例：Contextを使用してコンテキストを管理する

`Context`クラスを使用してコルーチンのコンテキストを管理できます。`Context`クラスでは、`Coroutine::getuid`を使用してコルーチンの`ID`を取得し、異なるコルーチン間でグローバル変数を隔離し、コルーチンが終了する際にコンテキストデータをクリアしています。

```php
use Swoole\Coroutine;

class Context
{
    protected static $pool = [];

    static function get($key)
    {
        $cid = Coroutine::getuid();
        if ($cid < 0)
        {
            return null;
        }
        if(isset(self::$pool[$cid][$key])){
            return self::$pool[$cid][$key];
        }
        return null;
    }

    static function put($key, $item)
    {
        $cid = Coroutine::getuid();
        if ($cid > 0)
        {
            self::$pool[$cid][$key] = $item;
        }

    }

    static function delete($key = null)
    {
        $cid = Coroutine::getuid();
        if ($cid > 0)
        {
            if($key){
                unset(self::$pool[$cid][$key]);
            }else{
                unset(self::$pool[$cid]);
            }
        }
    }
}
```

使用方法：

```php
use Swoole\Coroutine\Context;

$server = new Swoole\Http\Server('127.0.0.1', 9501);

$server->on('request', function ($request, $response) {
    if ($request->server['request_uri'] == '/a') {
        Context::put('name', 'a');
        co::sleep(1.0);
        echo Context::get('name');
        $response->end(Context::get('name'));
        //コルーチンを終了する際にクリアする
        Context::delete('name');
    } else {
        Context::put('name', 'b');
        $response->end();
        //コルーチンを終了する際にクリアする
        Context::delete();
    }
});
$server->start();
```

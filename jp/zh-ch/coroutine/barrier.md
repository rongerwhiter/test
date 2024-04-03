# Coroutine\Barrier

[Swooleライブラリ](https://github.com/swoole/library)内で、より便利なコルーチン並行管理ツールである`Coroutine\Barrier`コルーチンバリア、またはコルーチンバリアが提供されています。これは、`PHP`の参照カウントと`Coroutine API`に基づいて実装されています。

[Coroutine\WaitGroup](/coroutine/wait_group)に比べて、`Coroutine\Barrier`はよりシンプルに使用でき、パラメーターの渡し方や`use`構文を使ってサブコルーチン関数を導入するだけで済みます。

!> Swoole バージョン >= v4.5.5 のときに使用可能です。

## 使用例

```php
use Swoole\Coroutine\Barrier;
use Swoole\Coroutine\System;
use function Swoole\Coroutine\run;
use Swoole\Coroutine;

run(function () {
    $barrier = Barrier::make();

    $count = 0;
    $N = 4;

    foreach (range(1, $N) as $i) {
        Coroutine::create(function () use ($barrier, &$count) {
            System::sleep(0.5);
            $count++;
        });
    }

    Barrier::wait($barrier);
    
    assert($count == $N);
});
```

## 実行フロー

* `Barrier::make()` を使用して新しいコルーチンバリアを作成します。
* サブコルーチンで`use`構文を使用してバリアを渡し、参照カウントを増やします。
* 待つ必要がある位置で`Barrier::wait($barrier)`を追加すると、現在のコルーチンが自動的に一時停止し、そのコルーチンバリアを参照するサブコルーチンが終了するのを待ちます。
* サブコルーチンが終了すると、`$barrier`オブジェクトの参照カウントが減少し、最終的に`0`になります。
* すべてのサブコルーチンがタスクを完了して終了すると、`$barrier`オブジェクトの参照カウントが`0`になり、`Barrier::wait($barrier)`から戻り、バックグラウンドでそのコルーチンを自動的に再開します。

`Coroutine\Barrier` は [WaitGroup](/coroutine/wait_group) や [Channel](/coroutine/channel) よりも使いやすい並行制御ツールであり、`PHP`の並行プログラミングのユーザーエクスペリエンスを大幅に向上させます。

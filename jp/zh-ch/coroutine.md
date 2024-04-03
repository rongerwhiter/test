# コルーチン

このセクションでは、コルーチンの基本的な概念や一般的な問題について説明し、[Swooleビデオチュートリアル](https://course.swoole-cloud.com/course-video/6) で確認することもできます。

バージョン4.0から、`Swoole`は完全な`コルーチン（Coroutine）` + `チャネル（Channel）`機能を提供し、新しい`CSP`プログラミングモデルをもたらします。

1. 開発者は同期的なコーディング方式で[非同期IO](/learn?id=同期io非同期io)の効果とパフォーマンスを実現することができ、伝統的な非同期コールバックによる分散したコードロジックや深いコールバックの多層化に陥ることを回避できます。
2. 同様に、低レベルでコルーチンがカプセル化されているため、伝統的な`PHP`層のコルーチンフレームワークと比較して、開発者は`yield`キーワードを使用してコルーチン`IO`操作を示す必要がありません。そのため、`yield`の意味を深く理解する必要や各レベルの呼び出しをすべて`yield`に変更する必要がなくなり、開発効率が大幅に向上します。
3. 多様な種類の[コルーチンクライアント](/coroutine_client/init)が提供され、ほとんどの開発者の要求を満たすことができます。
```python
def greet():
    yield "Hello"
    yield "こんにちは"
    yield "Bonjour"
    
g = greet()
print(next(g))  # Output: Hello
print(next(g))  # Output: こんにちは
print(next(g))  # Output: Bonjour
```

上記のコードは、ジェネレータ関数と呼ばれる単純なコルーチンの例です。コードブロック外のテキストを翻訳します。
## Channelとは

`Channel`は、メッセージキューと考えることができますが、これはコルーチン間のメッセージキューです。複数のコルーチンは、メッセージの生産や消費を行うために`push`および`pop`操作を使用してキューを操作し、データの送受信を行うためにコルーチン間で通信を行います。`Channel`はプロセス間を横断することはできず、1つの`Swoole`プロセス内のコルーチン間での通信に使用されます。最も典型的な用途は[コネクションプール](/coroutine/conn_pool)や[並行呼び出し](/coroutine/multi_call)です。
## 协程コンテナとは何ですか

`Coroutine::create`または`go()`メソッドを使用してコルーチンを作成し、作成されたコルーチンでのみコルーチン`API`を使用できます。 そのため、コルーチンはコルーチンコンテナ内で作成する必要があります。 詳細は[コルーチンコンテナ](/coroutine/scheduler)を参照してください。
## コルーチンスケジューリング

ここでは、コルーチンスケジューリングとは何かについてできるだけわかりやすく説明します。まず、各コルーチンを1つのスレッドとして簡単に理解できます。皆さんはマルチスレッドがプログラムの並行性を向上させるためにあることを知っていますが、同様にマルチコルーチンも並行性を向上させるために存在しています。

ユーザーの各リクエストは1つのコルーチンを作成し、リクエストが終了するとコルーチンも終了します。したがって、同時に何千もの並行リクエストがある場合、ある時点でプロセス内には何千ものコルーチンが存在する可能性があります。しかしCPUリソースは限られていますので、具体的にどのコルーチンのコードを実行するかを決定する必要があります。

CPUがどのコルーチンのコードを実行するかを決定するプロセスが「コルーチンスケジューリング」です。では、`Swoole`のスケジューリング戦略はどのようなものでしょうか？

- 最初に、あるコルーチンのコードが実行されているときに、そのコードが`Co::sleep()`に遭遇したり、ネットワーク`IO`が発生した場合（例：`MySQL->query()`）、これは明らかに時間がかかるプロセスです。そのため、`Swoole`はそのMySQL接続のFdを[EventLoop](/learn?id=什么是eventloop)に追加します。

    - そして、そのコルーチンのCPUを他のコルーチンに譲る：**つまり`yield`(一時中断)**
    - MySQLデータが戻ってきた後、そのコルーチンを再開する：**つまり`resume`(再開)**

- 次に、コルーチンのコードにCPU集中型のコードがある場合、[enable_preemptive_scheduler](/other/config)を有効にすることができます。SwooleはそのコルーチンにCPUを強制的に譲ります。
## 親子コルーチンの優先順位

子コルーチン（つまり `go()` 関数内のロジック）が優先的に実行され、コルーチン `yield`（Co::sleep() の部分）が発生するまで続行され、その後外側のコルーチンに切り替わります。 

```php
use Swoole\Coroutine;
use function Swoole\Coroutine\run;

echo "main start\n";
run(function () {
    echo "coro " . Coroutine::getcid() . " start\n";
    Coroutine::create(function () {
        echo "coro " . Coroutine::getcid() . " start\n";
        Coroutine::sleep(.2);
        echo "coro " . Coroutine::getcid() . " end\n";
    });
    echo "coro " . Coroutine::getcid() . " do not wait children coroutine\n";
    Coroutine::sleep(.1);
    echo "coro " . Coroutine::getcid() . " end\n";
});
echo "end\n";

/*
main start
coro 1 start
coro 2 start
coro 1 do not wait children coroutine
coro 1 end
coro 2 end
end
*/
```
## 注意事項

Swoole プログラミングを始める前に注意すべき点：
### 全域変数

コルーチンにより、従来の非同期ロジックが同期化されますが、コルーチン間の切り替えは暗黙的に行われるため、コルーチンの切り替え前後でグローバル変数や`static`変数の一貫性を保証することはできません。

`PHP-FPM`では、コンテキスト内のリクエストパラメータやサーバーパラメータなどをグローバル変数を介して取得できますが、`Swoole`内では`$_GET/$_POST/$_REQUEST/$_SESSION/$_COOKIE/$_SERVER`などの`$_`で始まる変数を使用して何らかの属性パラメータを取得することは**できません**。

[context](/coroutine/coroutine?id=getcontext)を使用して、コルーチンIDごとに隔離されたコンテキストを作成し、グローバル変数の隔離を実現できます。
```
### Multiple coroutines sharing a TCP connection

[Reference](/question/use?id=client-has-already-been-bound-to-another-coroutine)
```

### 複数のコルーチンがTCP接続を共有する

[参照](/question/use?id=client-has-already-been-bound-to-another-coroutine)

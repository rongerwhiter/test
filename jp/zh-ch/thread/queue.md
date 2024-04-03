# 並行キュー

並行の `Queue` 構造を作成し、スレッドのパラメータとして子スレッドに渡すことができるようにします。他のスレッドから読み書き時には可視です。
詳細な特性は [並行マップ](thread/map.md) を参照してください。
## 使用方法
`Thread\Queue` は先入先出構造体であり、2つの主要な操作があります：
- `Queue::push($value)` データをキューに書き込む
- `Queue::pop()` キューの先頭からデータを取り出す
## 注意事項
- `ArrayList` は要素の追加のみが可能であり、要素のランダム削除や代入はできません。
```php
use Swoole\Thread;
use Swoole\Thread\Queue;

$args = Thread::getArguments();
$c = 4;
$n = 128;

if (empty($args)) {
    $threads = [];
    $queue = new Queue;
    for ($i = 0; $i < $c; $i++) {
        $threads[] = Thread::exec(__FILE__, $i, $queue);
    }
    while ($n--) {
        $queue->push(base64_encode(random_bytes(16)), Queue::NOTIFY_ONE);
        usleep(random_int(10000, 100000));
    }
    $n = 4;
    while ($n--) {
        $queue->push('', Queue::NOTIFY_ONE);
    }
    for ($i = 0; $i < $c; $i++) {
        $threads[$i]->join();
    }
    var_dump($queue->count());
} else {
    $queue = $args[1];
    while (1) {
        $job = $queue->pop(-1);
        if (!$job) {
            break;
        }
        var_dump($job);
    }
}
```  
## Method List
### Queue::push()

向キューにデータを書き込みます

```php
function Queue::push(mixed $value, int $notify = 0);
```

- `$value`：書き込むデータの内容
- `$notify`：データを読み取り待ちのスレッドに通知するかどうか、`Queue::NOTIFY_ONE` は1つのスレッドを起こし、`Queue::NOTIFY_ALL` はすべてのスレッドを起こします
### Queue::pop()

キューの先頭からデータを取り出します

```php
function Queue::pop(double $timeout = 0);
```

- `$timeout=0`：デフォルト値で、キューが空の場合にはすぐに `NULL` を返します
- `$timeout!=0`：キューが空の場合には、生産者がデータを `push()` するのを待ちます。 `$timeout` が負の場合はタイムアウトしません
### Queue::count()
キューの要素数を取得します。
### Queue::clean()
すべての要素をクリアします。

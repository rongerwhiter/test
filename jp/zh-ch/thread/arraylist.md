# 並行リスト

サブスレッドにスレッドパラメータとして渡すことができる、並行して利用可能な `List` 構造を作成します。他のスレッドから読み書きが可能であり、可視性が維持されます。
詳細な特性は[Concurrent Map](thread/map.md)を参照してください。
## 使用方法
`Thread\AraryList`は`ArrayAccess`インターフェースを実装しており、配列として直接操作することができます。
## 注意事項
- `ArrayList` には要素の追加のみができ、要素のランダムな削除や代入はできません。
```php
use Swoole\Thread;
use Swoole\Thread\AraryList;

$args = Thread::getArguments();
if (empty($args)) {
    $list = new AraryList;
    $thread = Thread::exec(__FILE__, $i, $list);
    sleep(1);
    $list[] = unique();
    $thread->join();
} else {
    $list = $args[1];
    sleep(2);
    var_dump($list[0]);
}
```

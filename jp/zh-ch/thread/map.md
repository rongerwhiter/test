# 並行マップ

並行の `Map` 構造を作成し、スレッドに渡して子スレッドで使用できるようにします。他のスレッドから読み書きするときにも見えるようにします。
## 特性
- `Map`、`ArrayList`、`Queue` は自動的にメモリが割り当てられ、`Table` のように固定的に割り当てる必要がありません
- バックエンドでは自動的にロックがかけられ、スレッドセーフです
- `null/bool/int/float/string` 型のみサポートされており、他の型は書き込み時に自動的にシリアライズされ、読み込み時に逆シリアライズされます
- イテレータはサポートされておらず、イテレータ内で要素を削除するとメモリエラーが発生します
- スレッドを作成する前に、`Map`、`ArrayList`、`Queue` オブジェクトをスレッドのパラメータとして子スレッドに渡す必要があります
## 使用方法
`Thread\Map` は `ArrayAccess` インターフェースを実装しており、配列として直接操作することができます。
```php
use Swoole\Thread;
use Swoole\Thread\Map;

$args = Thread::getArguments();
if (empty($args)) {
    $map = new Map;
    $thread = Thread::exec(__FILE__, $i, $map);
    sleep(1);
    $map['test'] = unique();
    $thread->join();
} else {
    $map = $args[1];
    sleep(2);
    var_dump($map['test']);
}
```
この文書は特定の手順に関する情報を提供します。
```cpp
int count = myMap.count();
```

### Map::count() 
要素の数を取得します。
### Map::keys()
すべての `key` を返します。
### Map::clean()
すべての要素をクリアします。

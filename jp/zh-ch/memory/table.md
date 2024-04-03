# ハイパフォーマンス共有メモリーテーブル

`PHP`言語はマルチスレッドをサポートしていないため、`Swoole`はマルチプロセスモードを使用し、プロセス間メモリの分離が存在します。作業プロセス内で`global`グローバル変数やスーパーグローバル変数を変更すると、他のプロセスでは無効です。

`worker_num=1`に設定された場合、プロセスの分離が存在せず、グローバル変数を使用してデータを保存できます。

```php
$fds = array();
$server->on('connect', function ($server, $fd){
    echo "connection open: {$fd}\n";
    global $fds;
    $fds[] = $fd;
    var_dump($fds);
});
```

`$fds`はグローバル変数ですが、現在のプロセス内でのみ有効です。`Swoole`サーバーの下には複数の`Worker`プロセスが作成され、`var_dump($fds)`に表示される値は一部の接続の`fd`だけです。

対応策は外部ストレージサービスを使用することです：

* データベース、たとえば: `MySQL`、`MongoDB`
* キャッシュサーバー、たとえば: `Redis`、`Memcache`
* ディスクファイル、複数のプロセスが同時に読み書きする場合はロックが必要です。

通常のデータベースやディスクファイル操作には多くの`IO`待ち時間があります。したがって、次のものを推奨します：

* `Redis`インメモリデータベース、読み書き速度が非常に速いが、TCP接続などの問題があるため、パフォーマンスが最高ではありません。
* `/dev/shm`インメモリファイルシステム、すべての読み書き操作がメモリ内で完了するため、`IO`の消費はなく、非常に高いパフォーマンスですが、データはフォーマットされていないため、データ同期の問題があります。

?> これまで述べたストレージ以外に、データを保存するために共有メモリを使用することをお勧めします。`Swoole\Table`は、共有メモリとロックを使用して実装された超高性能並行データ構造です。マルチプロセス/マルチスレッドデータ共有と同期のロックの問題を解決するために使用されます。`Table`のメモリ容量は`PHP`の`memory_limit`に影響を受けません。

!> `Table`を読み書きする際に配列方式を使用しないでください。必ずドキュメントで提供されているAPIを使用して操作してください。  
配列方式で取り出される`Table\Row`オブジェクトは使い捨てのオブジェクトであり、その後の操作に依存しないでください。
`v4.7.0`以降、配列方式での`Table`の読み書きはサポートされておらず、`Table\Row`オブジェクトが削除されました。

* **利点**

  * 驚異的なパフォーマンス、単一スレッドは1秒に200万回読み書き可能；
  * アプリケーションコードにはロックを追加する必要がなく、`Table`内に行ロックの自旋ロックが組み込まれており、すべての操作はマルチスレッド/マルチプロセスセーフです。ユーザーレベルではデータ同期の問題を考慮する必要はありません；
  * 複数プロセスをサポートし、`Table`は複数のプロセス間でデータを共有できます；
  * 行ロックを使用し、グローバルロックではなく、2つのプロセスが同じ`CPU`時間で同じデータを並行して読み取る場合にのみロックが発生します。

* **反復子**

!> 反復中に削除操作を行わないでください（すべての`key`を取得してから削除してください）

`Table`クラスはイテレーターおよび`Countable`インターフェースを実装しており、`foreach`を使用して反復処理し、`count`で現在の行数を計算できます。

```php
foreach($table as $row)
{
  var_dump($row);
}
echo count($table);
```
```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
```

このコードは、`Person`クラスを定義しています。`Person`クラスには`name`と`age`という属性があります。
```php
Swoole\Table->size;
```

`size`プロパティは、テーブルの最大行数を取得します。
### memorySize

メモリの実際の使用サイズをバイト単位で取得します。

```php
Swoole\Table->memorySize;
```
```python
def greet():
    print("Hello, World!")
```

### __construct()

メモリーテーブルを作成します。

```php
Swoole\Table::__construct(int $size, float $conflict_proportion = 0.2);
```

  * **パラメータ** 

    * **`int $size`**
      * **機能**：テーブルの最大行数を指定します
      * **デフォルト値**：なし
      * **その他の値**：なし

      !> `Table` は共有メモリー上に構築されるため、動的に拡張することはできません。そのため、`$size` は事前に計算して設定する必要があります。`Table` が格納できる最大行数は `$size` と関連していますが、完全に一致するわけではありません。たとえば、`$size` が `1024` の場合、実際に格納できる行数は `1024` よりも**少ない**です。もし `$size` が大きすぎると、マシンのメモリーが不足している場合は、`Table` の作成が失敗します。

    * **`float $conflict_proportion`**
      * **機能**：ハッシュの衝突の最大比率
      * **デフォルト値**：`0.2`（つまり`20%`）
      * **その他の値**：最小は`0.2`、最大は`1`

  * **容量の計算**

      * もし `$size` が `1024`、`8192`、`65536` などの `2` の `N` 乗ではない場合、内部的に最も近い数値に自動調整されます。`1024` より小さい場合はデフォルトで `1024` になります。`v4.4.6` のバージョンから最小値は `64` になります。
      * `Table` が占めるメモリ総量は (`HashTable構造体の長さ` + `KEYの長さ64バイト` + `$sizeの値`) * (`1 + $conflict_proportionをハッシュ衝突として`) * (`列サイズ`) です。
      * もしデータの `Key` とハッシュ衝突率が `20%` を超える場合、予約された衝突メモリブロックの容量が不足していると、新しいデータを `set` しようとすると `Unable to allocate memory` エラーが発生し、`false` が返されて保存に失敗します。この場合は `$size` の値を大きくしてサービスを再起動する必要があります。
      * 十分なメモリがある場合は、この値をできるだけ大きく設定してください。
### column()

内存テーブルに列を追加します。

```php
Swoole\Table->column(string $name, int $type, int $size = 0);
```

  * **パラメータ** 

    * **`string $name`**
      * **役割**：フィールド名を指定します
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $type`**
      * **役割**：フィールドのタイプを指定します
      * **デフォルト値**：なし
      * **その他の値**：`Table::TYPE_INT`, `Table::TYPE_FLOAT`, `Table::TYPE_STRING`

    * **`int $size`**
      * **役割**：最大文字列長を指定します【文字列タイプのフィールドは`$size`を指定する必要があります】
      * **単位**：バイト
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **`$type` の説明**

タイプ | 説明
---|---
Table::TYPE_INT | デフォルトは8バイトです
Table::TYPE_STRING | 設定後、指定された最大長`$size`を超える文字列は設定できません
Table::TYPE_FLOAT | 8バイトのメモリを占有します
### create()

メモリテーブルを作成します。テーブルの構造を定義した後、`create`を実行してオペレーティングシステムにメモリを要求し、テーブルを作成します。

```php
Swoole\Table->create(): bool
```

`create`メソッドを使用してテーブルを作成した後、実際に使用されるメモリのサイズを取得するには[memorySize](/memory/table?id=memorysize)属性を使用できます。

  * **ヒント**

     * `set`、`get`などのデータ操作メソッドを使用する前に`create`を呼び出すことはできません
     * `create`を呼び出した後は、新しいフィールドを追加するための`column`メソッドを使用できません
     * システムメモリが不足している場合、要求が失敗し、`create`は`false`を返します
     * メモリの確保に成功すると、`create`は`true`を返します

    !> `Table`はデータを保存するために共有メモリを使用しており、子プロセスを作成する前に`Table->create()`を必ず実行してください；  
    `Server`内で`Table`を使用する場合、`Table->create()`は`Server->start()`の前に実行する必要があります。

  * **使用例**

```php
$table = new Swoole\Table(1024);
$table->column('id', Swoole\Table::TYPE_INT);
$table->column('name', Swoole\Table::TYPE_STRING, 64);
$table->column('num', Swoole\Table::TYPE_FLOAT);
$table->create();

$worker = new Swoole\Process(function () {}, false, false);
$worker->start();

//$serv = new Swoole\Server('127.0.0.1', 9501);
//$serv->start();
```
### set()

データ行を設定します。`Table`は`key-value`の形式でデータにアクセスします。

```php
Swoole\Table->set(string $key, array $value): bool
```

  * **パラメータ** 

    * **`string $key`**
      * **機能**：データの`key`
      * **デフォルト値**：なし
      * **他の値**：なし

      !> 同じ`$key`は同じ行のデータに対応し、`set`を使って同じ`key`を設定すると、直前のデータが上書きされます。`key`の最大長は63バイトを超えてはいけません。

    * **`array $value`**
      * **機能**：データの`value`
      * **デフォルト値**：なし
      * **他の値**：なし

      !> 配列でなければならず、フィールドの定義された`$name`と完全に一致している必要があります。

  * **戻り値**

    * 成功した場合は`true`を返します。
    * 失敗した場合は`false`を返します。ハッシュの衝突が多すぎて動的空間にメモリが割り当てられない可能性があり、構築方法の2番目のパラメータを大きくすることができます。

!> -`Table->set()` はすべてのフィールドの値を設定することもできますし、一部のフィールドのみを変更することもできます。  
   -`Table->set()` を設定していない場合、その行のデータはすべてのフィールドが空です。  
   -`set`/`get`/`del` には行ロックが内蔵されているため、`lock`メソッドを呼び出す必要はありません。  
   -**キーはバイナリーセーフではなく、文字列型でなければなりません。バイナリーデータは渡せません。**

  * **使用例**

```php
$table->set('1', ['id' => 1, 'name' => 'test1', 'age' => 20]);
$table->set('2', ['id' => 2, 'name' => 'test2', 'age' => 21]);
$table->set('3', ['id' => 3, 'name' => 'test3', 'age' => 19]);
```

  * **最大長を超える文字列を設定**

    列の定義時に設定された最大サイズを超える文字列が渡された場合、内部では自動的に切り捨てられます。

    ```php
    $table->column('str_value', Swoole\Table::TYPE_STRING, 5);
    $table->set('hello', array('str_value' => 'world 123456789'));
    var_dump($table->get('hello'));
    ```

    * `str_value`列の最大サイズは5バイトですが、`set`で5バイトを超える文字列が設定されています
    * 内部では5バイトのデータが自動的に切り取られ、最終的に`str_value`の値は`world`になります。

!> `v4.3`以降、内部でメモリ長さに対してアライメント処理が行われます。文字列の長さは8の倍数である必要があり、長さが5の場合は自動的に8バイトにアライメントされるため、`str_value`の値が`world 12`になります。
### incr()

原子インクリメント操作。

```php
Swoole\Table->incr(string $key, string $column, mixed $incrby = 1): int
```

  * **パラメータ** 

    * **`string $key`**
      * **機能**：データの`key`【`$key`に対応する行が存在しない場合、デフォルトで列の値は`0`になります】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $column`**
      * **機能**：列名を指定【浮動小数点数と整数のフィールドのみサポートされます】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $incrby`**
      * **機能**：増分 【列が`int`の場合、`$incrby`は`int`型である必要があります。列が`float`型の場合、`$incrby`は`float`型である必要があります】
      * **デフォルト値**：`1`
      * **その他の値**：なし

  * **戻り値**

    最終的な結果の数値
### decr()

原子デクリメント操作。

```php
Swoole\Table->decr(string $key, string $column, mixed $decrby = 1): int
```

  * **パラメータ** 

    * **`string $key`**
      * **機能**：データの`key`【`$key`に対応する行が存在しない場合、デフォルトで列の値は`0`となります】
      * **デフォルト値**：無し
      * **その他の値**：無し

    * **`string $column`**
      * **機能**：指定された列名【浮動小数点数型と整数型のフィールドのみサポートされます】
      * **デフォルト値**：無し
      * **その他の値**：無し

    * **`string $decrby`**
      * **機能**：増分【列が`int`の場合、`$decrby`は`int`型でなければならず、列が`float`の場合、`$decrby`は`float`型でなければなりません】
      * **デフォルト値**：`1`
      * **その他の値**：無し

  * **戻り値**

    途中経過の数値が返されます

    !> 数値が`0`の場合、デクリメントすると負の数になります
### get()

1行のデータを取得します。

```php
Swoole\Table->get(string $key, string $field = null): array|false
```

  * **Parameters** 

    * **`string $key`**
      * **Description**：データの`key`【文字列である必要があります】
      * **Default value**：なし
      * **Other values**：なし

    * **`string $field`**
      * **Description**：`$field`が指定されている場合、そのフィールドの値を返し、レコード全体ではない
      * **Default value**：なし
      * **Other values**：なし
      
  * **Return value**

    * `$key`が存在しない場合、`false`を返します
    * 成功した場合は結果の配列を返します
    * `$field`が指定されている場合は、そのフィールドの値を返し、レコード全体ではない
### exist()

あるキーがテーブル内に存在するかどうかをチェックします。

```php
Swoole\Table->exist(string $key): bool
```

  * **パラメータ** 

    * **`string $key`**
      * **機能**：データのキー【文字列型である必要があります】
      * **デフォルト値**：なし
      * **その他の値**：なし
### count()

```php
Swoole\Table->count(): int
```
### del()

データを削除します。

!> `Key`はバイナリセーフではなく、文字列型である必要があり、バイナリデータを渡すことはできません。**イテレーション中に削除しないでください**。

```php
Swoole\Table->del(string $key): bool
```

  * **返り値**

    * `$key`に対応するデータがない場合、`false`が返ります
    * 成功した場合は`true`を返します
### stats()

`Swoole\Table` の状態を取得します。

```php
Swoole\Table->stats(): array
```

!> Swoole version >= `v4.8.0` で利用可能
## ヘルパー関数: swoole_table

`Swoole\Table` を簡単に作成するための関数です。

```php
function swoole_table(int $size, string $fields): Swoole\Table
```

!> Swoole バージョン >= `v4.6.0` で利用可能です。`$fields` のフォーマットは `foo:i/foo:s:num/foo:f` です。

| 短い名前 | 長い名前 | タイプ               |
| ---- | ------ | ------------------ |
| i    | int    | Table::TYPE_INT    |
| s    | string | Table::TYPE_STRING |
| f    | float  | Table::TYPE_FLOAT  |

例：

```php
$table = swoole_table(1024, 'fd:int, reactor_id:i, data:s:64');
var_dump($table);

$table = new Swoole\Table(1024, 0.25);
$table->column('fd', Swoole\Table::TYPE_INT);
$table->column('reactor_id', Swoole\Table::TYPE_INT);
$table->column('data', Swoole\Table::TYPE_STRING, 64);
$table->create();
var_dump($table);
```
```php
<?php
$table = new Swoole\Table(1024);
$table->column('fd', Swoole\Table::TYPE_INT);
$table->column('reactor_id', Swoole\Table::TYPE_INT);
$table->column('data', Swoole\Table::TYPE_STRING, 64);
$table->create();

$serv = new Swoole\Server('127.0.0.1', 9501);
$serv->set(['dispatch_mode' => 1]);
$serv->table = $table;

$serv->on('receive', function ($serv, $fd, $reactor_id, $data) {

	$cmd = explode(" ", trim($data));

	//get
	if ($cmd[0] == 'get')
	{
		//get self
		if (count($cmd) < 2)
		{
			$cmd[1] = $fd;
		}
		$get_fd = intval($cmd[1]);
		$info = $serv->table->get($get_fd);
		$serv->send($fd, var_export($info, true)."\n");
	}
	//set
	elseif ($cmd[0] == 'set')
	{
		$ret = $serv->table->set($fd, array('reactor_id' => $data, 'fd' => $fd, 'data' => $cmd[1]));
		if ($ret === false)
		{
			$serv->send($fd, "ERROR\n");
		}
		else
		{
			$serv->send($fd, "OK\n");
		}
	}
	else
	{
		$serv->send($fd, "command error.\n");
	}
});

$serv->start();
```  

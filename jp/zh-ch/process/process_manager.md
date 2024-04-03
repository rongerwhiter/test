# Process\Manager

プロセスマネージャーは、[Process\Pool](/process/process_pool)に基づいて実装されています。複数のプロセスを管理できます。`Process\Pool`と比較して、さまざまなタスクを実行する多くのプロセスを非常に簡単に作成し、また各プロセスがコルーチン環境にある必要があるかどうかを制御できます。
## バージョンサポート状況

| バージョン | クラス名                         | 更新内容                                 |
| ------ | ----------------------------- | ---------------------------------------- |
| v4.5.3 | Swoole\Process\ProcessManager | -                                        |
| v4.5.5 | Swoole\Process\Manager        | 名前変更、ProcessManager は Manager のエイリアス |

!> `v4.5.3`以上のバージョンで使用可能。
## 使用例

```php
use Swoole\Process\Manager;
use Swoole\Process\Pool;

$pm = new Manager();

for ($i = 0; $i < 2; $i++) {
    $pm->add(function (Pool $pool, int $workerId) {
    });
}

$pm->start();
```
このセクションでは、さまざまな方法を説明します。
### __construct()

コンストラクタ。

```php
Swoole\Process\Manager::__construct(int $ipcType = SWOOLE_IPC_NONE, int $msgQueueKey = 0);
```

* **パラメーター**

  * **`int $ipcType`**
    * **機能**：プロセス間通信のモード、`Process\Pool`の`$ipc_type`と同じです【デフォルトは`0`で、特定のプロセス間通信機能がないことを示します】
    * **デフォルト値**：`0`
    * **その他の値**：なし

  * **`int $msgQueueKey`**
    * **機能**：メッセージキューの `key`、`Process\Pool`の`$msgqueue_key`と同じです
    * **デフォルト値**：なし
    * **その他の値**：なし
### setIPCType()

設定工作進程之間的通訊方式。

```php
Swoole\Process\Manager->setIPCType(int $ipcType): self;
```

* **參數**

  * **`int $ipcType`**
    * **功能**：進程間通訊的模式
    * **默認值**：無
    * **其他值**：無
```php
Swoole\Process\Manager->getIPCType(): int;
```
### setMsgQueueKey()

メッセージキューの「key」を設定します。

```php
Swoole\Process\Manager->setMsgQueueKey(int $msgQueueKey): self;
```

* **パラメーター**

  * **`int $msgQueueKey`**
    * **機能**：メッセージキューの「key」
    * **デフォルト値**：なし
    * **その他の値**：なし
### getMsgQueueKey()

メッセージキューの `key` を取得します。

```php
Swoole\Process\Manager->getMsgQueueKey(): int;
```
### add()

ワーカープロセスを追加します。

```php
Swoole\Process\Manager->add(callable $func, bool $enableCoroutine = false): self;
```

* **パラメータ**

  * **`callable $func`**
    * **機能**：現在のプロセスで実行されるコールバック関数
    * **デフォルト値**：なし
    * **他の値**：なし

  * **`bool $enableCoroutine`**
    * **機能**：このプロセスのためにコルーチンを作成してコールバック関数を実行するかどうか
    * **デフォルト値**：false
    * **他の値**：なし
### addBatch()

バッチ処理でワーカープロセスを追加します。

```php
Swoole\Process\Manager->addBatch(int $workerNum, callable $func, bool $enableCoroutine = false): self
```

* **Parameters**

  * **`int $workerNum`**
    * **Description**: 追加するプロセスの数
    * **Default**: なし
    * **Other values**: なし

  * **`callable $func`**
    * **Description**: これらのプロセスが実行するコールバック関数
    * **Default**: なし
    * **Other values**: なし

  * **`bool $enableCoroutine`**
    * **Description**: これらのプロセスのコールバック関数を実行するためにコルーチンを作成するかどうか
    * **Default**: なし
    * **Other values**: なし
### start()

ワーカープロセスを起動します。

```php
Swoole\Process\Manager->start(): void
```

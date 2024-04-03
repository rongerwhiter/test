# Coroutine\System

システム関連の`API`のコルーチンラッパー。このモジュールは`v4.4.6`の公式バージョン以降で使用可能です。ほとんどの`API`は`AIO`スレッドプールをベースにしています。

!> バージョン`v4.4.6`より前の場合は、`Co`または`Swoole\Coroutine`を使用してください。例: `Co::sleep`または`Swoole\Coroutine::sleep`  
`v4.4.6`以降のバージョンでは公式に**推奨**されるのは`Co\System::sleep`または`Swoole\Coroutine\System::sleep`です。  
この変更は名前空間を標準化することを目的としていますが、同時に下位互換性も確保しています（つまり、`v4.4.6`以前のバージョンの書き方も問題ありません）。
## 方法

以下の手順を実行してください。

```python
def greet():
    print("Hello, World!")
```

以上のコードを使用して、挨拶の関数を作成します。
```php
Swoole\Coroutine\System::statvfs(string $path): array|false
```

  * **パラメータ** 

    * **`string $path`**
      * **機能**：ファイルシステムのマウントされたディレクトリ【例`/`、dfまたは`mount -l`コマンドで取得できます】
      * **デフォルト値**：無し
      * **その他の値**：無し

  * **使用例**

    ```php
    Swoole\Coroutine\run(function () {
        var_dump(Swoole\Coroutine\System::statvfs('/'));
    });
    
    // array(11) {
    //   ["bsize"]=>
    //   int(4096)
    //   ["frsize"]=>
    //   int(4096)
    //   ["blocks"]=>
    //   int(61068098)
    //   ["bfree"]=>
    //   int(45753580)
    //   ["bavail"]=>
    //   int(42645728)
    //   ["files"]=>
    //   int(15523840)
    //   ["ffree"]=>
    //   int(14909927)
    //   ["favail"]=>
    //   int(14909927)
    //   ["fsid"]=>
    //   int(1002377915335522995)
    //   ["flag"]=>
    //   int(4096)
    //   ["namemax"]=>
    //   int(255)
    // }
    ```
### fread()

ファイルを読み込むためのコルーチン方式。

```php
Swoole\Coroutine\System::fread(resource $handle, int $length = 0): string|false
```

!> `v4.0.4`未満では`fread`メソッドが`STDIN`や`Socket`などの非ファイルタイプの`stream`をサポートしていません。このようなリソースで`fread`を使用しないでください。  
`v4.0.4`以上では、`fread`メソッドが非ファイルタイプの`stream`リソースをサポートし、内部的に`stream`タイプに応じて`AIO`スレッドプールまたは[EventLoop](/learn?id=什么是eventloop)で実装が選択されます。

  * **Parameters** 

    * **`resource $handle`**
      * **Description**: ファイルハンドル【`fopen`で開かれたファイルタイプの`stream`リソースでなければなりません】
      * **Default**: なし
      * **Other values**: なし

    * **`int $length`**
      * **Description**: 読み込む長さ【デフォルトは`0`で、ファイル全体を読み込むことを意味します】
      * **Default**: `0`
      * **Other values**: なし

  * **Return Value** 

    * 読み取りに成功した場合、文字列コンテンツを返し、失敗した場合は`false`を返します。

  * **Example**  

    ```php
    $fp = fopen(__FILE__, "r");
    Swoole\Coroutine\run(function () use ($fp)
    {
        $r = Swoole\Coroutine\System::fread($fp);
        var_dump($r);
    });
    ```
### fwrite()

ファイルにデータを書き込むためのコルーチン方式。

```php
Swoole\Coroutine\System::fwrite(resource $handle, string $data, int $length = 0): int|false
```

!> バージョン`v4.0.4`未満の`fwrite`メソッドは、`STDIN`や`Socket`などのファイル以外の`stream`をサポートしていませんので、このようなリソースで`fwrite`を使用しないでください。  
バージョン`v4.0.4`以降、`fwrite`メソッドはファイル以外の`stream`リソースをサポートします。内部で`stream`の種類に応じて`AIO`スレッドプールまたは[EventLoop](/learn?id=什么是eventloop)を自動的に選択します。

  * **パラメータ** 

    * **`resource $handle`**
      * **機能**：ファイルハンドル【`fopen`で開かれたファイルタイプの`stream`リソースである必要があります】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $data`**
      * **機能**：書き込むデータ内容【テキストまたはバイナリーデータである可能性があります】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $length`**
      * **機能**：読み取る長さ【デフォルトは`0`で、`$data`の全内容を書き込みます。`$length`は`$data`の長さよりも小さくする必要があります】
      * **デフォルト値**：`0`
      * **その他の値**：なし

  * **戻り値** 

    * 書き込みに成功するとデータの長さを返し、失敗すると`false`を返します。

  * **使用例**  

    ```php
    $fp = fopen(__DIR__ . "/test.data", "a+");
    Swoole\Coroutine\run(function () use ($fp)
    {
        $r = Swoole\Coroutine\System::fwrite($fp, "hello world\n", 5);
        var_dump($r);
    });
    ```  
### fgets()

ファイルの内容を行ごとに読み込むためのコルーチン方式です。

下層では`php_stream`バッファを使用しており、デフォルトサイズは`8192`バイトですが、`stream_set_chunk_size`を使用してバッファサイズを設定することができます。

```php
Swoole\Coroutine\System::fgets(resource $handle): string|false
```

!> `fgets`関数はファイルタイプの`stream`リソースのみに使用でき、Swooleバージョン >= `v4.4.4` で利用可能です

  * **パラメータ** 

    * **`resource $handle`**
      * **機能**：ファイルハンドル【`fopen`で開かれたファイルタイプの`stream`リソースである必要があります】
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **戻り値** 

    * `EOL`（`\r`または`\n`）が読み取られると、行データが返され、`EOL`も含まれます
    * `EOL`が読み取られないが、コンテンツの長さが`php_stream`バッファの`8192`バイトを超えると、`EOL`を含まない`8192`バイトのデータが返されます
    * ファイルの末尾`EOF`に到達した場合、空の文字列が返され、ファイルが読み込まれたかどうかは`feof`で判断できます
    * 読み取りに失敗した場合は`false`が返され、エラーコードは[swoole_last_error](/functions?id=swoole_last_error)関数を使用して取得できます

  * **使用例**  

    ```php
    $fp = fopen(__DIR__ . "/defer_client.php", "r");
    Swoole\Coroutine\run(function () use ($fp)
    {
        $r = Swoole\Coroutine\System::fgets($fp);
        var_dump($r);
    });
    ```  
### readFile()

ファイルをコルーチンで読み取ります。

```php
Swoole\Coroutine\System::readFile(string $filename): string|false
```

  * **Parameters** 

    * **`string $filename`**
      * **機能**：ファイル名
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **Return Value** 

    * 読み取りに成功した場合は文字列内容が返され、失敗した場合は`false`が返されます。エラー情報は[swoole_last_error](/functions?id=swoole_last_error)を使用して取得できます。
    * `readFile`メソッドにはサイズ制限がありません。読み取られた内容はメモリに格納されるため、非常に大きなファイルを読み取るとメモリを大量に消費する可能性があります。

  * **Example**  

    ```php
    $filename = __DIR__ . "/defer_client.php";
    Swoole\Coroutine\run(function () use ($filename)
    {
        $r = Swoole\Coroutine\System::readFile($filename);
        var_dump($r);
    });
    ```  
### `writeFile()`

ファイルに書き込むためのコルーチン方式です。

```php
Swoole\Coroutine\System::writeFile(string $filename, string $fileContent, int $flags): bool
```

  * **パラメータ** 

    * **`string $filename`**
      * **機能**：ファイル名【書き込み権限が必要で、ファイルが存在しない場合は自動的に作成されます。ファイルを開くのに失敗するとすぐに`false`が返ります】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`string $fileContent`**
      * **機能**：ファイルに書き込む内容【最大`4M`まで書き込み可能です】
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $flags`**
      * **機能**：書き込みオプション【通常、現在のファイル内容を削除しますが、`FILE_APPEND`を使用してファイルの末尾に追記することもできます】
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **戻り値** 

    * 書き込みに成功すると`true`が返ります
    * 書き込みに失敗すると`false`が返ります

  * **使用例**  

    ```php
    $filename = __DIR__ . "/defer_client.php";
    Swoole\Coroutine\run(function () use ($filename)
    {
        $w = Swoole\Coroutine\System::writeFile($filename, "hello swoole!");
        var_dump($w);
    });
    ```  
### sleep()

待機状態に入ります。

これは`PHP`の`sleep`関数に相当し、異なる点は`Coroutine::sleep`が[コルーチンスケジューラ](/coroutine?id=コルーチンスケジューラ)によって実装されていることです。内部では現在のコルーチンを`yield`してタイムスライスを譲り、非同期タイマーを追加します。タイムアウトが発生すると、再び現在のコルーチンを`resume`して実行を再開します。

`sleep`インターフェースを使用すると、タイムアウト待ちなどの機能を簡単に実装できます。

```php
Swoole\Coroutine\System::sleep(float $seconds): void
```

* **パラメーター** 

    * **`float $seconds`**
        * **機能**：待機する時間【0より大きく、最大で1日（86400秒）を超えない必要があります】
        * **単位**：秒、最小精度はミリ秒（0.001秒）
        * **デフォルト値**：なし
        * **その他の値**：なし


* **使用例**  

    ```php
    $server = new Swoole\Http\Server("127.0.0.1", 9502);

    $server->on('Request', function($request, $response) {
        // 200ms 待機してからレスポンスをブラウザに送信
        Swoole\Coroutine\System::sleep(0.2);
        $response->end("<h1>Hello Swoole!</h1>");
    });

    $server->start();
    ```
### exec()

一つのシェルコマンドを実行します。内部で[協力スケジューリング](/coroutine?id=協力スケジューリング)が自動的に行われます。

```php
Swoole\Coroutine\System::exec(string $cmd): array
```

  * **パラメータ** 

    * **`string $cmd`**
      * **機能**：実行する`shell`コマンド
      * **デフォルト値**：なし
      * **その他の値**：なし

  * **戻り値**

    * 実行に失敗すると`false`が返り、成功するとプロセスの終了ステータス、シグナル、出力内容が含まれた配列が返ります。

    ```php
    array(
        'code'   => 0,  // プロセスの終了ステータス
        'signal' => 0,  // シグナル
        'output' => '', // 出力内容
    );
    ```

  * **使用例**  

    ```php
    Swoole\Coroutine\run(function() {
        $ret = Swoole\Coroutine\System::exec("md5sum ".__FILE__);
    });
    ```

  * **注意**

  !>スクリプトコマンドの実行時間が長すぎると、タイムアウトして終了する可能性があります。この場合は、[socket_read_timeout](/coroutine_client/init?id=タイムアウト規則)を増やすことでこの問題を解決できます。
### gethostbyname()

IPアドレスに対応するドメイン名を取得します。同期スレッドプールを模倣して実装されており、自動的に[コルーチンスケジューリング](/coroutine?id=コルーチンスケジューリング)が行われます。

```php
Swoole\Coroutine\System::gethostbyname(string $domain, int $family = AF_INET, float $timeout = -1): string|false
```

  * **Parameters** 

    * **`string $domain`**
      * **Description**: ドメイン名
      * **Default**: なし
      * **Other values**: なし

    * **`int $family`**
      * **Description**: ドメインファミリ【`AF_INET`は`IPv4`アドレスを返すことを示し、`AF_INET6`を使用すると`IPv6`アドレスを返します】
      * **Default**: `AF_INET`
      * **Other values**: `AF_INET6`

    * **`float $timeout`**
      * **Description**: タイムアウト時間
      * **Unit**: 秒、最小精度はミリ秒（`0.001`秒）
      * **Default**: `-1`
      * **Other values**: なし

  * **Return Value**

    * 成功時はドメインに対応する`IP`アドレスを返し、失敗時は`false`を返します。エラー情報は[swoole_last_error](/functions?id=swoole_last_error)を使用して取得できます

    ```php
    array(
        'code'   => 0,  // プロセスの終了ステータスコード
        'signal' => 0,  // シグナル
        'output' => '', // 出力内容
    );
    ```

  * **Extensions**

    * **Timeout Control**

      `$timeout`パラメータはコルーチンの待機タイムアウトを制御できます。指定された時間内に結果が返らない場合、コルーチンは即座に`false`を返して続行します。内部実装では、この非同期タスクを`cancel`としてマークし、`gethostbyname`は引き続き`AIO`スレッドプールで実行されます。

      `/etc/resolv.conf`を変更して`gethostbyname`や`getaddrinfo`の基本となる`C`関数のタイムアウト時間を設定することもできます。詳細は[DNS解決のタイムアウトとリトライの設定](/learn_other?id=設定dns解決のタイムアウトとリトライ)をご覧ください。

  * **Usage Example**  

    ```php
    Swoole\Coroutine\run(function () {
        $ip = Swoole\Coroutine\System::gethostbyname("www.baidu.com", AF_INET, 0.5);
        echo $ip;
    });
    ```
### getaddrinfo()

DNS解決を行い、ドメイン名に対応する`IP`アドレスをクエリします。

`gethostbyname`とは異なり、`getaddrinfo`はより多くのパラメータ設定をサポートし、複数の`IP`結果を返します。

```php
Swoole\Coroutine\System::getaddrinfo(string $domain, int $family = AF_INET, int $socktype = SOCK_STREAM, int $protocol = STREAM_IPPROTO_TCP, string $service = null, float $timeout = -1): array|false
```

  * **Parameters** 

    * **`string $domain`**
      * **Function**: ドメイン名
      * **Default Value**: None
      * **Other Values**: None

    * **`int $family`**
      * **Function**: ファミリ【`AF_INET`は`IPv4`アドレスを返し、`AF_INET6`を使用すると`IPv6`アドレスが返されます】
      * **Default Value**: None
      * **Other Values**: None
      
      !> その他のパラメータ設定については`man getaddrinfo`ドキュメントを参照してください

    * **`int $socktype`**
      * **Function**: プロトコルタイプ
      * **Default Value**: `SOCK_STREAM`
      * **Other Values**: `SOCK_DGRAM`、`SOCK_RAW`

    * **`int $protocol`**
      * **Function**: プロトコル
      * **Default Value**: `STREAM_IPPROTO_TCP`
      * **Other Values**: `STREAM_IPPROTO_UDP`、`STREAM_IPPROTO_STCP`、`STREAM_IPPROTO_TIPC`、`0`

    * **`string $service`**
      * **Function**:
      * **Default Value**: None
      * **Other Values**: None

    * **`float $timeout`**
      * **Function**: タイムアウト時間
      * **Value Unit**: seconds, minimum precision is milliseconds (`0.001` seconds)
      * **Default Value**: `-1`
      * **Other Values**: None

  * **Return Value**

    * 成功：複数の`IP`アドレスから成る配列、失敗：`false`

  * **Example**  

    ```php
    Swoole\Coroutine\run(function () {
        $ips = Swoole\Coroutine\System::getaddrinfo("www.baidu.com");
        var_dump($ips);
    });
    ```
### dnsLookup()

ドメイン名のアドレスを検索します。

`Coroutine\System::gethostbyname`とは異なり、`Coroutine\System::dnsLookup`は、`UDP`クライアントネットワーク通信に直接基づいており、`libc`の`gethostbyname`関数を使用していません。

!> Swooleのバージョンが`v4.4.3`以上である必要があり、内部で`/etc/resolve.conf`を読み取って`DNS`サーバーのアドレスを取得します。現時点では`AF_INET(IPv4)`のドメイン解決のみサポートされています。Swooleのバージョンが`v4.7`以上である場合は、`AF_INET6(IPv6)`をサポートするために第三引数を使用できます。

```php
Swoole\Coroutine\System::dnsLookup(string $domain, float $timeout = 5, int $type = AF_INET): string|false
```

  * **パラメータ** 

    * **`string $domain`**
      * **機能**：ドメイン名
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`float $timeout`**
      * **機能**：タイムアウト時間
      * **値の単位**：秒、最小精度はミリ秒（`0.001`秒）
      * **デフォルト値**：`5`
      * **その他の値**：なし

    * **`int $type`**
        * **値の単位**：秒、最小精度はミリ秒（`0.001`秒）
        * **デフォルト値**：`AF_INET`
        * **その他の値**：`AF_INET6`

    !> Swooleのバージョンが`v4.7`以上の場合、`$type`パラメータを使用できます。

  * **戻り値**

    * 解析に成功した場合は対応するIPアドレスが返されます
    * 失敗した場合は`false`が返され、[swoole_last_error](/functions?id=swoole_last_error)を使用してエラー情報を取得できます

  * **一般的なエラー**

    * `SWOOLE_ERROR_DNSLOOKUP_RESOLVE_FAILED`：このドメイン名を解決できませんでした。クエリが失敗しました
    * `SWOOLE_ERROR_DNSLOOKUP_RESOLVE_TIMEOUT`：解決がタイムアウトしました。DNSサーバーに障害がある可能性があり、指定された時間内に結果を返せませんでした

  * **使用例**  

    ```php
    Swoole\Coroutine\run(function () {
        $ip = Swoole\Coroutine\System::dnsLookup("www.baidu.com");
        echo $ip;
    });
    ```
### wait()

[Process::wait](/process/process?id=wait)の対応版ですが、このAPIはコルーチンバージョンであり、コルーチンを一時停止させるため、`Swoole\Process::wait`や`pcntl_wait`関数を置き換えることができます。

!> Swooleバージョン >= `v4.5.0` で利用可能です。

```php
Swoole\Coroutine\System::wait(float $timeout = -1): array|false
```

* **パラメータ** 

    * **`float $timeout`**
      * **機能**：タイムアウト時間、負数はタイムアウトなしを意味します
      * **値の単位**：秒、最小精度はミリ秒（`0.001`秒）
      * **デフォルト値**：`-1`
      * **その他の値**：なし

* **戻り値**

  * 成功すると、子プロセスの`PID`、終了ステータスコード、どのシグナルによって終了したかを含む配列が返されます
  * 失敗すると`false`が返されます

!> 各子プロセスは起動後、親プロセスが`wait()`（または`waitPid()`）をコールして回収しなければ、子プロセスはゾンビプロセスとなり、オペレーティングシステムのプロセスリソースを浪費します。  
コルーチンを使用する場合、まずプロセスを作成し、そのプロセス内でコルーチンを開始する必要があります。逆は適用されません。そうしないと、コルーチンをforkしたときに非常に複雑な状況が発生し、下位レイヤーが処理しにくくなります。

* **例**

```php
use Swoole\Coroutine;
use Swoole\Coroutine\System;
use Swoole\Process;

$process = new Process(function () {
    echo 'Hello Swoole';
});
$process->start();

Coroutine\run(function () use ($process) {
    $status = System::wait();
    assert($status['pid'] === $process->pid);
    var_dump($status);
});
```
### waitPid()

和上述wait方法基本一致, 不同的是此API可以指定等待特定的进程

!> Swoole版本 >= `v4.5.0` 可用

```php
Swoole\Coroutine\System::waitPid(int $pid, float $timeout = -1): array|false
```

* **参数** 

    * **`int $pid`**
      * **功能**：进程id
      * **默认值**：`-1` (表示任意进程, 此时等价于wait方法)
      * **其它值**：任意自然数

    * **`float $timeout`**
      * **功能**：超时时间，负数表示永不超时
      * **值单位**：秒，最小精度为毫秒（`0.001`秒）
      * **默认值**：`-1`
      * **其它值**：无

* **返回值**

  * 操作成功会返回一个数组包含子进程的`PID`、退出状态码、被哪种信号`KILL`
  * 失败返回`false`

!> 每个子进程启动后，父进程必须都要派遣一个协程调用`wait()`(或`waitPid()`)进行回收，否则子进程会变成僵尸进程，会浪费操作系统的进程资源。

* **示例**

```php
use Swoole\Coroutine;
use Swoole\Coroutine\System;
use Swoole\Process;

$process = new Process(function () {
    echo 'Hello Swoole';
});
$process->start();

Coroutine\run(function () use ($process) {
    $status = System::waitPid($process->pid);
    var_dump($status);
});
```
### waitSignal()

シグナルリスナーのコルーチンバージョンで、シグナルがトリガーされるまで現在のコルーチンをブロックします。 `Swoole\Process::signal` および `pcntl_signal` 関数と置き換えることができます。

!> Swooleバージョン >= `v4.5.0` で使用可能

```php
Swoole\Coroutine\System::waitSignal(int $signo, float $timeout = -1): bool
```

  * **Parameters** 

    * **`int $signo`**
      * **Description**: シグナルの種類
      * **Default**: なし
      * **Other values**: `SIGTERM`, `SIGKILL`などのSIGシリーズの定数

    * **`float $timeout`**
      * **Description**: タイムアウト時間、負数はタイムアウトしないことを示す
      * **Unit**: 秒、最小精度はミリ秒（`0.001`秒）
      * **Default**: `-1`
      * **Other values**: なし

  * **Return Value**

    * シグナルを受信した場合は`true`
    * タイムアウトしてシグナルを受信しなかった場合は`false`

  * **Example**

```php
use Swoole\Coroutine;
use Swoole\Coroutine\System;
use Swoole\Process;

$process = new Process(function () {
    Coroutine\run(function () {
        $bool = System::waitSignal(SIGUSR1);
        var_dump($bool);
    });
});
$process->start();
sleep(1);
$process::kill($process->pid, SIGUSR1);
```
### waitEvent()

`waitEvent()`は、イベントがトリガーされるまで現在のコルーチンをブロックするコルーチンバージョンのシグナルリスナーです。IOイベントの待機に使用し、`swoole_event`関連の関数と置き換えることができます。

!> Swoole version >= `v4.5`で利用可能

```php
Swoole\Coroutine\System::waitEvent(mixed $socket, int $events = SWOOLE_EVENT_READ, float $timeout = -1): int | false
```

* **パラメータ**

    * **`mixed $socket`**
      * **機能**：ファイル記述子（ソケットオブジェクト、リソースなどの変換可能な任意の型）
      * **デフォルト値**：なし
      * **その他の値**：なし

    * **`int $events`**
      * **機能**：イベントのタイプ
      * **デフォルト値**：`SWOOLE_EVENT_READ`
      * **その他の値**：`SWOOLE_EVENT_WRITE`または`SWOOLE_EVENT_READ | SWOOLE_EVENT_WRITE`

    * **`float $timeout`**
      * **機能**：タイムアウト時間、負数は無制限を表す
      * **単位**：秒、最小精度はミリ秒（`0.001`秒）
      * **デフォルト値**：`-1`
      * **その他の値**：なし

* **戻り値**

  * トリガーされたイベントタイプの論理和（複数のビットで表現される可能性あり）、引数`$events`に依存
  * 失敗した場合は`false`が返ります。エラー情報は[swoole_last_error](/functions?id=swoole_last_error)を使用して取得できます

* **例**

> 同期ブロッキングコードをこのAPIを使って非同期ブロックすることができます

```php
use Swoole\Coroutine;

Coroutine\run(function () {
    $client = stream_socket_client('tcp://www.qq.com:80', $errno, $errstr, 30);
    $events = Coroutine::waitEvent($client, SWOOLE_EVENT_READ | SWOOLE_EVENT_WRITE);
    assert($events === SWOOLE_EVENT_WRITE);
    fwrite($client, "GET / HTTP/1.1\r\nHost: www.qq.com\r\n\r\n");
    $events = Coroutine::waitEvent($client, SWOOLE_EVENT_READ);
    assert($events === SWOOLE_EVENT_READ);
    $response = fread($client, 8192);
    echo $response;
});
```

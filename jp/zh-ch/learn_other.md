# その他の知識

## DNS解決のタイムアウトとリトライの設定

ネットワークプログラミングでは、ドメイン名の解決に`gethostbyname`や`getaddrinfo`などの`C`関数をよく使用しますが、これらの関数にはタイムアウトパラメータが提供されていません。実際には、`/etc/resolv.conf`を編集してタイムアウトとリトライロジックを設定することができます。

!> `man resolv.conf`ドキュメントを参照してください

### 複数のNameServer <!-- {docsify-ignore} -->

```
nameserver 192.168.1.3
nameserver 192.168.1.5
option rotate
```

複数の`nameserver`を設定することができます。最初の`nameserver`でのクエリが失敗した場合、自動的に次の`nameserver`に切り替えて再試行します。

`option rotate`は`nameserver`への負荷分散を実現し、ラウンドロビンモードを使用します。

### タイムアウト制御 <!-- {docsify-ignore} -->

```
option timeout:1 attempts:2
```

* `timeout`：`UDP`受信のタイムアウト時間を制御します。単位は秒で、デフォルトは`5`秒です。
* `attempts`：試行回数を制御します。`2`に設定すると、最大試行回数は`2`回で、デフォルトは`5`回です。

例えば、`2`つの`nameserver`があり、`attempts`が`2`で、タイムアウトが`1`である場合、すべてのDNSサーバーが応答しない場合、最長待機時間は`4`秒になります（`2x2x1`）。

### コールトレース <!-- {docsify-ignore} -->

[strace](/other/tools?id=strace)を使用してトレース結果を確認できます。

名前解決のために存在しない2つのIPを`nameserver`に設定し、`PHP`コードで`var_dump(gethostbyname('www.baidu.com'));`を使用してドメイン名を解決します。

```
ここでは、合計で4回リトライが行われており、`poll`呼び出しのタイムアウトは`1000ms`（`1秒`）に設定されています。

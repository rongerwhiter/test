# エラーレポートの提出

## 注意事項

Swooleのコアバグを見つけた場合は、レポートを提出してください。Swooleのコア開発者たちは、問題の存在を知らないかもしれません。あなたが主動的にレポートを提出しない限り、バグは発見されにくく修正されない可能性があります。GitHubのissueエリアでエラーレポートを提出できます([こちらをクリックしてください](https://github.com/swoole/swoole-src/issues))。ここで提出されたエラーレポートは優先して解決されます。

メーリングリストや個人メッセージでエラーレポートを送信しないでください。GitHubのissueエリアでもSwooleに関する要望や提案を出すことができます。

エラーレポートを提出する前に、以下の**エラーレポートの提出方法**をよくお読みください。

## 新規問題の提出

まず、issueを作成する際には、以下のテンプレートが表示されますので、この情報をしっかりと記入してください。情報が不足しているとissueが無視される可能性があります。

```markdown

Please answer these questions before submitting your issue. Thanks!
> Submit the following questions before submitting the Issue:

1. What did you do? If possible，provide a simple script for reproducing the error.
> 問題が発生する手順を詳しく説明してください。関連コードを貼り付け、できればエラーを再現するための単純なスクリプトも提供してください。

2. What did you expect to see?
> どのような結果を期待していましたか？

3. What did you see instead?
> 代わりに何が表示されましたか？

4. What version of Swoole are you using (`php --ri swoole`)?
> 使用しているSwooleのバージョンは何ですか？ (`php --ri swoole`コマンドの出力を貼り付けてください)

5. What is your machine environment used (including the version of kernel & php & gcc)?
> 使用しているマシンの環境は何ですか（カーネルのバージョン、PHP、gccのバージョンなど）？
> `uname -a`、`php -v`、`gcc -v`コマンドを使用して情報を表示してください

```

特に重要なのは**再現可能な簡単なスクリプトコード**を提供することです。そうでない場合、開発者がエラーの原因を判断するためにできるだけ多くの情報を提供する必要があります。

## メモリ解析（強く推奨）

より多くの場合、Valgrindはgdbよりもメモリの問題を発見できます。以下のコマンドを使用してプログラムを実行し、バグが発生するまで実行します。

```shell
USE_ZEND_ALLOC=0 valgrind --log-file=/tmp/valgrind.log php your_file.php
```

* プログラムがエラーを出力した場合は、`ctrl+c`を入力して終了し、`/tmp/valgrind.log`ファイルをアップロードして開発チームがバグを特定できるようにします。

## セグメンテーション違反について（コアダンプ）

さらに、特定の場合にはデバッグツールを使用して問題を特定できます。

```shell
WARNING	swManager_check_exit_status: worker#1 abnormal exit, status=0, signal=11
```

上記のようなSwooleログに(signal11)が表示される場合、プログラムは`セグメンテーション違反`が発生したことを示します。問題の発生位置を特定するためにデバッグツールを使用する必要があります。

> `gdb`を使用して`swoole`をトレースする前に、`--enable-debug`フラグをコンパイル時に追加して詳細情報を保持する必要があります。

コアダンプファイルを有効にします。
```shell
ulimit -c unlimited
```

バグを引き起こし、コアダンプファイルがプログラムディレクトリまたはシステムルートまたは`/cores`ディレクトリに生成されます（システム構成により異なります）。

以下のコマンドを入力してgdbでプログラムをデバッグモードにします。

```
gdb php core
gdb php /tmp/core.1234
```

その後、`bt`を入力してエラーの呼び出しスタックを確認できます。
```
(gdb) bt
```

特定の呼び出しスタックフレームを表示するには、`f 数字`を入力してください。
```
(gdb) f 1
(gdb) f 0
```

これらの情報をすべてissueに貼り付けてください。

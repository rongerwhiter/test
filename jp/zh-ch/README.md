# Swoole

?> `Swoole` は、`C++` 言語で書かれた非同期イベント駆動およびコルーチンベースの並列ネットワーク通信エンジンであり、`PHP` に[コルーチン](/coroutine)や[高性能](/question/use?id=swoole性能如何)なネットワークプログラミングサポートを提供します。さまざまな通信プロトコルのネットワークサーバーおよびクライアントモジュールが提供されており、`TCP/UDPサービス`、`高性能Web`、`WebSocketサービス`、`IoT`、`リアルタイムコミュニケーション`、`ゲーム`、`マイクロサービス`などを簡単かつ迅速に実装でき、`PHP` を従来のWeb領域に固定しないでおくことができます。
## Swooleクラス図

!>対応するドキュメントページに直接リンクすることができます

[//]: # (https://naotu.baidu.com/file/bd9d2ba7dfae326e6976f0c53f88b18c)

<embed src="_images/swoole_class.svg" type="image/svg+xml" alt="Swooleアーキテクチャ図" />
## 公式ウェブサイト

* [Swoole Website](//www.swoole.com)
* [商用製品およびサポート](//business.swoole.com)
* [Swoole Q&A](//wenda.swoole.com)
## プロジェクトのリンク

* [GitHub](//github.com/swoole/swoole-src) **（サポートのためスターを押してください）**
* [码云](//gitee.com/swoole/swoole)
* [Pecl](//pecl.php.net/package/swoole)
## 開発ツール

* [IDE Helper](https://github.com/swoole/ide-helper)
* [Yasd](https://github.com/swoole/yasd)
* [debugger](https://github.com/swoole/debugger)
* [sdebug](https://github.com/swoole/sdebug)
## 著作権情報

この文書の初版は、以前の [古いSwooleドキュメント](https://wiki.swoole.com/wiki/index/prid-1) から引用されており、長年にわたって文書に対する皆さんからの不満を解決することを目的としています。現代的な文書構造を採用し、`Swoole4`の内容のみを含み、古い文書の間違いを多く修正し、文書の詳細を最適化し、サンプルコードや教育コンテンツを追加して、`Swoole`の初心者により親しみやすいものにしました。

本文書のすべての内容、テキスト、画像、音声/動画資料を含む著作権は **上海识沃网络科技有限公司** に帰属します。メディア、Webサイト、個人は外部リンク形式で引用することができますが、契約による許可なしに、いかなる形でも複製、公開/発表することはできません。
## 文档発起者

* [杨才](https://github.com/TTSimple)
* [郭新华](https://www.weibo.com/u/2661945152)
* [鲁飞](https://github.com/sy-records) [Weibo](https://weibo.com/5384435686)
## 問題フィードバック

このドキュメントの内容に関する問題（例：誤字、例の誤り、内容の欠落など）や要望は、すべて [swoole-inc/report](https://github.com/swoole-inc/report) プロジェクトに統一して`issue`を提出してください。また、右上の [フィードバック](/?id=main) をクリックすることで`issue`ページに直接アクセスできます。

採用されると、投稿者の情報が [貢献者リスト](/CONTRIBUTING) に追加されて感謝の意を示します。
## ドキュメント原則

わかりやすい言葉を使用し、できるだけ`Swoole`の内部技術の詳細やいくつかの下位概念を少なくすることを心がけます。下位の内容は後で特定の「hack」セクションを管理することができます。

回避できないいくつかの概念がある場合、その概念を集中的に紹介する場所が**必ず**必要であり、他の場所へのリンクも含める必要があります。例：[イベントループ](/learn?id=什么是eventloop)；

ドキュメントを書く際には、他人が理解できるかどうかを見る立場から考え方を変える必要があります。

将来の機能変更が発生した場合、関連するすべての箇所を確実に修正する必要があります。一部のみを修正してはいけません。

各機能モジュールには、完全な例が**必ず**必要です。

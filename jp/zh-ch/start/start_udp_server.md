# UDP サーバー

## プログラムコード

以下のコードをudpServer.phpに記述してください。

```php
$server = new Swoole\Server('127.0.0.1', 9502, SWOOLE_PROCESS, SWOOLE_SOCK_UDP);

//データ受信イベントをリッスンする。
$server->on('Packet', function ($server, $data, $clientInfo) {
    var_dump($clientInfo);
    $server->sendto($clientInfo['address'], $clientInfo['port'], "Server：{$data}");
});

//サーバーを起動する
$server->start();
```

UDPサーバーはTCPサーバーとは異なり、UDPには接続の概念がありません。サーバーを起動した後、クライアントはConnectする必要がなく、直接サーバーがポート9502でリッスンしているところにデータパケットを送信できます。対応するイベント名はonPacketです。

* `$clientInfo`はクライアントの関連情報で、クライアントのIPやポートなどが含まれる配列です。
* `$server->sendto`メソッドを使用してクライアントにデータを送信します。
!> DockerはデフォルトでTCPプロトコルを使用して通信します。UDPプロトコルを使用する場合は、Dockerネットワークを設定する必要があります。
```shell
docker run -p 9502:9502/udp <image-name>
```

## サービスの起動

```shell
php udpServer.php
```

UDPサーバーは`netcat -u`を使用して接続テストを行うことができます。

```shell
netcat -u 127.0.0.1 9502
hello
Server: hello
```

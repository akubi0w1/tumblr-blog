date: 2020/08/12

# RabbitMQとは

### overview
Advanced Message Queuing Protocol (AMQP)を使用したメッセージ指向ミドルウェア。
種機能として、メッセージの仲介をしてくれる。つまり、メッセージを受け取って、送りつけてくれる、郵便局みたいなイメージ。

扱うデータは、データのbinary blobs = messageって呼ぶ。

非同期のメッセージングが可能。
メッセージキューイングや複数のプロトコルをサポートしてたりする。

### 用語について

- Producing
	- messageを送ること
	- messaegを送るプログラムをproducerとする
- queue
	- 郵便でいうポストみたいなもの
	- messageはRabbitMQ Server -> アプリケーションと流れていく
	- ホストのメモリ、ディスクの制限内であればキューとして溜まっていく
	- たくさんのproducersがqueueにメッセージを送ってくる。
	- で、たくさんのconsumersがqueueからデータを受け取ることを試みる
- consuming
	- messageを受け取ること
	- consumerがmessageの受け取りを待つ

note: producer, consumer, brokerは基本的に同じホストに配置しない。というかされない。

### チュートリアルを覗く

[RabbitMQ tutorial-one-go](https://www.rabbitmq.com/tutorials/tutorial-one-go.html)をやってみた。

#### 前準備

- dockerでRabbitMQのサーバを準備

```
$ docker run -it --rm --name rabbitmq -p 5672:5672 rabbitmq:3-management
```

- 必要なパッケージを落っことす

```
$ go get -v github.com/streadway/amqp
```

#### goでproducer側の実装

`producer -> queue`の部分

```go
// send.go
package main

import (
	"log"

	"github.com/streadway/amqp"
)

func main() {
	// connect rabbitmq server
	// プロトコルのバージョンとかネゴシエーションとか、認証とかの面倒を見てくれる
	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	failOnError(err, "failed to connect rabbitmq")
	defer conn.Close()

	// create channel
	// メッセージを送ったり、受け取ったり、色々するためのAPIのほとんどがここにいる
	ch, err := conn.Channel()
	failOnError(err, "failed to open a channel")
	defer ch.Close()

	// send
	// 1. キューを宣言する必要がある(declare)
	// queueが存在しなかった場合だけ、作成される(キューの宣言は冪等)
	// 2. メッセージをキューに送る(publish)
	q, err := ch.QueueDeclare(
		"hello", // 名前 name
		false,   // 永続化するか？ durable
		false,   // 使われなかったとき削除するか delete when unused
		false,   // 排他的か？？？？ exclusive
		false,   // またない？？？ no-wait
		nil,     // 引数 argments
	)
	failOnError(err, "failed to declare a queue")

	body := "hello world!"
	err = ch.Publish(
		"",     // exchange
		q.Name, // routing key
		false,  // mandatory 強制的な...?
		false,  //immediate 即時...?
		amqp.Publishing{
			ContentType: "text/plain",
			Body:        []byte(body),
		},
	)
	failOnError(err, "failed to publish a message")

	log.Println("success to send message!")
}

func failOnError(err error, msg string) {
	if err != nil {
		log.Fatalf("%s: %s", msg, err)
	}
}
```

#### consumer側の実装

`queue -> consumer`の部分

```go
// receive.go
package main

import (
	"log"

	"github.com/streadway/amqp"
)

func main() {
	conn, err := amqp.Dial("amqp://guest:guest@localhost:5672/")
	failOnError(err, "failed to connect to rabbitmq")
	defer conn.Close()

	ch, err := conn.Channel()
	failOnError(err, "failed to open a channel")
	defer ch.Close()

	q, err := ch.QueueDeclare(
		"hello",
		false,
		false,
		false,
		false,
		nil,
	)
	failOnError(err, "failed to declare a queue")

	// メッセージをqueueから非同期で受け取る
	// goroutine内で、チャネル(amqp.Consumeによって返される)からメッセージ読む
	msgs, err := ch.Consume(
		q.Name, // queue
		"",     // consumer
		true,   // auto-ack
		false,  // exclusive
		false,  // no-local
		false,  // no-wait
		nil,    // args
	)
	failOnError(err, "failed to register a consumer")

	forever := make(chan bool)

	go func() {
		for d := range msgs {
			log.Printf("received a message: %s", d.Body)
		}
	}()

	log.Printf("[*] waiting for messages. to exit press CTRL+C")
	<-forever // mainが走り切らないためにブロックする
}

func failOnError(err error, msg string) {
	if err != nil {
		log.Fatalf("%s: %s", msg, err)
	}
}

```

#### execute

```
// send message
$ go run send.go

// receive message
$ go run receive.go
```

### 参考

- [https://www.rabbitmq.com/](https://www.rabbitmq.com/)
- [https://www.rabbitmq.com/download.html](https://www.rabbitmq.com/download.html)
- [https://godoc.org/github.com/streadway/amqp](https://godoc.org/github.com/streadway/amqp)
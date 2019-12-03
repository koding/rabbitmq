### Golang AMPQ client to rabbitmq

This repository was forked from [koding/rabbitmq](https://github.com/koding/rabbitmq)

To publish:

```.go
package main

import (
	"fmt"

	"github.com/koding/logging"
	"github.com/involvestecnologia/rabbitmq"
	"github.com/streadway/amqp"
)

func main() {
	rmq := rabbitmq.New(
		&rabbitmq.Config{
			Host:     "localhost",
			Port:     5672,
			Username: "guest",
			Password: "guest",
			Vhost:    "/",
		},
		logging.NewLogger("producer"),
	)

	exchange := rabbitmq.Exchange{
		Name: "EXCHANGE_NAME",
	}

	queue := rabbitmq.Queue{}
	publishingOptions := rabbitmq.PublishingOptions{
		Tag:        "ProducerTagHede",
		RoutingKey: "naber",
	}

	publisher, err := rmq.NewProducer(exchange, queue, publishingOptions)
	if err != nil {
		panic(err)
	}
	defer publisher.Shutdown()
	publisher.RegisterSignalHandler()

	// may be we should autoconvert to byte array?
	msg := amqp.Publishing{
		Body: []byte("2"),
	}

	publisher.NotifyReturn(func(message amqp.Return) {
		fmt.Println(message)
	})

	for i := 0; i < 10; i++ {
		err = publisher.Publish(msg)
		if err != nil {
			fmt.Println(err, i)
		}
	}
}

```

To consume:

```.go
package main

import (
	"fmt"

	"github.com/koding/logging"
	"github.com/involvestecnologia/rabbitmq"
	"github.com/streadway/amqp"
)

func main() {
	rmq := rabbitmq.New(
		&rabbitmq.Config{
			Host:     "localhost",
			Port:     5672,
			Username: "guest",
			Password: "guest",
			Vhost:    "/",
		},
		logging.NewLogger("producer"),
	)

	exchange := rabbitmq.Exchange{
		Name:    "EXCHANGE_NAME",
		Type:    "fanout",
		Durable: true,
	}

	queue := rabbitmq.Queue{
		Name:    "WORKER_QUEUE_NAME",
		Durable: true,
	}
	binding := rabbitmq.BindingOptions{
		RoutingKey: "hede",
	}

	consumerOptions := rabbitmq.ConsumerOptions{
		Tag: "ElasticSearchFeeder",
	}

	consumer, err := rmq.NewConsumer(exchange, queue, binding, consumerOptions)
	if err != nil {
		fmt.Print(err)
		return
	}
	defer consumer.Shutdown()
	err = consumer.QOS(3)
	if err != nil {
		panic(err)
	}
	fmt.Println("Elasticsearch Feeder worker started")
	consumer.RegisterSignalHandler()
	consumer.Consume(handler)
}

var handler = func(delivery amqp.Delivery) {
	message := string(delivery.Body)
	fmt.Println(message)
	delivery.Ack(false)
}


```
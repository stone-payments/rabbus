## Rabbus 🚌 ✨

* A tiny wrapper over [amqp](https://github.com/streadway/amqp) exchanges and queues.
* Automatic retries and exponential backoff for sending messages.
* Makes use of [gobreaker](https://github.com/sony/gobreaker).
* Automatic reconnect to RabbitMQ broker.
* Golang channel API.

## Installation
```bash
go get -u github.com/rafaeljesus/rabbus
```

## Usage
The rabbus package exposes an interface for emitting and listening RabbitMQ messages.

### Emit
```go
import (
  "github.com/rafaeljesus/rabbus"
)

func main() {
  r, err := rabbus.NewRabbus(rabbus.Config{
    Dsn           : "amqp://guest:guest@localhost:5672",
    Durable       : true,
    Retry         : rabbus.Retry {
      Attempts    : 5,
      Sleep       : time.Second * 2,
    },
    CircuitBreaker: rabbus.CircuitBreaker {
      Threshold: 3,
      OnStateChange: func(name, from, to string) {
        // do something when state is changed
      }
    },
  })

  select {
    case r.EmitAsync() <- Message{
      Exchange  : "test_ex",
      Kind      : "topic",
      Key       : "test_key",
      Payload   : []byte(`foo`),
    }
    case r.EmitOk():
     // message was sent
    case r.EmitErr():
     // failed to send message
  }
}
```

### Listen
```go
import (
  "encoding/json"

  "github.com/rafaeljesus/rabbus"
)

func main() {
  r, err := rabbus.NewRabbus(rabbus.Config{
    Dsn           : "amqp://guest:guest@localhost:5672",
    Durable       : true,
    Retry         : rabbus.Retry {
      Attempts    : 3,
      Sleep       : time.Second * 2,
    },
    CircuitBreaker: rabbus.CircuitBreaker {
      Threshold: 3,
      OnStateChange: func(name, from, to string) {
        // do something when state is changed
      }
    },
  })

  messages, err := r.Listen(rabbus.ListenConfig{
    Exchange: "events_ex",
    Kind:     "topic",
    Key:      "events_key",
    Queue:    "events_q",
  })
  if err != nil {
    // handle errors during adding listener
  }

  go func() {
    for m := range messages {
      m.Ack(false)
    }
  }()
}
```

## Contributing
- Fork it
- Create your feature branch (`git checkout -b my-new-feature`)
- Commit your changes (`git commit -am 'Add some feature'`)
- Push to the branch (`git push origin my-new-feature`)
- Create new Pull Request

## Badges

[![Build Status](https://circleci.com/gh/rafaeljesus/rabbus.svg?style=svg)](https://circleci.com/gh/rafaeljesus/rabbus)
[![Go Report Card](https://goreportcard.com/badge/github.com/rafaeljesus/rabbus)](https://goreportcard.com/report/github.com/rafaeljesus/rabbus)
[![Go Doc](https://godoc.org/github.com/rafaeljesus/rabbus?status.svg)](https://godoc.org/github.com/rafaeljesus/rabbus)

---

> GitHub [@rafaeljesus](https://github.com/rafaeljesus) &nbsp;&middot;&nbsp;
> Medium [@_jesus_rafael](https://medium.com/@_jesus_rafael) &nbsp;&middot;&nbsp;
> Twitter [@_jesus_rafael](https://twitter.com/_jesus_rafael)

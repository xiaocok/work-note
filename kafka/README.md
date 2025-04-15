

## github.com/segmentio/kafka-go

### 生产者

#### 同步生产：批量生产

> 默认为批量发送：通过批量发送消息，增加吞吐量。超过1s发送或者超过100条发送或者数据大小超过1M后发送。

优点：

- 发送效率高。

缺点：

- 一次性发送100条(BatchSize)，错误时，100条都收到相同错误信息，无法确定具体是那一条错误。不确定服务器端是全部成功，全部失败。还是存在部分成功，部分失败。
- 存在还没有超过100条，或者BatchBytes，或者BatchTimeout，重启，那么可能存在消息丢失。需要维护一个消息发送成功的具体位置。如果维护这个位置存在部分少量的消息重复的情况。

```go
w := &kafka.Writer{
    Addr:                   kafka.TCP("192.168.195.134:9092"),
    Topic:                  "test",
    Balancer:               &kafka.LeastBytes{},
    RequiredAcks:           kafka.RequireAll,
    BatchSize:              100,			// 批量发送消息：默认100，当缓存中存在100条时，触发发送
    BatchBytes:             1048576,		// 批量发送消息：默认1048576（1M），当消息达到1M时，触发发送
    BatchTimeout:           time.Second,    // 批量发送消息：默认1s，当消息累计1s后，触发发送
    WriteTimeout:           time.Second * 10, 
    MaxAttempts:            10,             // 最大尝试次数：发送消息的次数， 默认10次
    Async:                  false,          // 默认false，同步发送，true异步发送
}

messages := []kafka.Message{
    {Value: []byte("1")},
    {Value: []byte("2")},
    {Value: []byte("3")},
}

ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
defer cancel()

// 批量发送消息：当消息没有出发100条和1M时，超过1s发送，这里批量发送，总共触发一次BatchTimeout，总共耗时1s
// used time: 1.04s
now := time.Now()
// 消息的条数 大于 BatchSize会报错，消息的总大小 大于 BatchBytes，会报错。
err := w.WriteMessages(ctx, messages...)
fmt.Printf("used time: %s", time.Now().Sub(now).Truncate(time.Millisecond))
if err != nil {
    fmt.Println(err.Error())
}

// 循环发送消息：每条消息发送，都会触发一次BatchTimeout超时，公共耗时3s
// used time: 3.022s
now = time.Now()
for i := range messages {
    err := w.WriteMessages(ctx, messages[i])
    if err != nil {
        fmt.Println(err.Error())
    }
}
fmt.Printf("used time: %s", time.Now().Sub(now).Truncate(time.Millisecond))

_ = w.Close()
```

场景一：

​	参数：BatchSize = 100，BatchTimeout = time.Second

​	数据条数：220条

​	发送流程：

​		1、取1~100条，满足100条，统计前100条后，立即发送。

​		2、取101~200条，满足100条，立即发送。

​		3、取201~220条，不满足100条，等超时时间BatchTimeout (1秒)后，触发超时发送。

场景二：

​	参数：BatchSize = 100，BatchTimeout = time.Second

​	数据条数：20条

​	发送流程：

​		1、取1~20条，不满足100条，等超时时间BatchTimeout (1秒)后，触发超时发送。

总结：

​	1、当每次发送的数据条数较小，小于BatchSize，则每次都会触发BatchTimeout 后发送，发送较慢。

​	2、发送数据条数较多性能较好。



#### 同步生产：单条生产

缺点：性能慢

优点：单条记录，失败或重启，重复生产的只有1条

```go
w := &kafka.Writer{
    Addr:                   kafka.TCP("192.168.195.134:9092"),
    Topic:                  "test",
    Balancer:               &kafka.LeastBytes{},
    RequiredAcks:           kafka.RequireAll,
    BatchSize:              1,			    // 单条记录，立即发送
    WriteTimeout:           time.Second * 10, 
    MaxAttempts:            10,             // 最大尝试次数：发送消息的次数， 默认10次
    Async:                  false,          // 默认false，同步发送，true异步发送
}

messages := []kafka.Message{
    {Value: []byte("1")},
}

// 触发BatchSize，立即发送，不延迟
err := w.WriteMessages(ctx, messages[i])
if err != nil {
    fmt.Println(err.Error())
}
_ = w.Close()
```



#### 异步生产

```go
w := &kafka.Writer{
    Addr:                   kafka.TCP("192.168.195.134:9092"),
    Topic:                  "test",
    Balancer:               &kafka.LeastBytes{},
    RequiredAcks:           kafka.RequireAll,
    BatchSize:              1,			    // 单条记录，立即发送
    WriteTimeout:           time.Second * 10, 
    MaxAttempts:            10,             // 最大尝试次数：发送消息的次数， 默认10次
    Async:                  true,           // 默认false，同步发送，true异步发送
    // 异步生成，WriteMessages立即返回。实际发送消息跟同步生成一致，由BatchSize、BatchBytes、BatchTimeout3个参数决定。
	// 由于立即返回，因此无法得到错误信息，通过这里完成后的回调函数实现处理处理。
    Completion: func(messages []kafka.Message, err error) {

    },
}

messages := []kafka.Message{
    {Value: []byte("1")},
}

// 异步发送，立即返回
err := w.WriteMessages(ctx, messages[i])
if err != nil {
    fmt.Println(err.Error())
}
_ = w.Close()
```



### 消费者组

#### 缓冲队列、手动异步提交、StartOffset

```go
kafka.ReaderConfig{
    Brokers: brokers,
    GroupID: groupID,
    Topic:   topic,
    // 队列容量，每次从服务器端获取消息时，最多缓存多少条消息。
	// 初始化时，创建QueueCapacity这个长度的msg队列，主Reader进程会循环从服务器端读取数据，往msg队列中push数据。
	// c.reader.FetchMessage(ctx)则从msg队列中pull获取数据。
    QueueCapacity: 100,
    // 自动提交的时间间隔：默认为0，关闭自动提交，采用手动提交模式
    // 消费者在每次调用 FetchMessage 获取消息后，会将消息的 Offset 缓存到内存中。这些 Offset 会被记录为“已消费但未提交”的状态。
    // 自动提交模式会按照这个时间间隔，将已内存中标记为“已消费但未提交”的 Offset，并提交给 Kafka 服务器。
    CommitInterval: 0,
    // 只有当消费者真正消费了一条消息并提交 Offset 后，Kafka 服务器才会记录该 Offset。如果未提交 Offset，Kafka 无法记住上次消费的位置，而是每次都根据 StartOffset 决定起始位置。
    // 在 Kafka 消费组模式下，只有当消费者成功提交 Offset 后，Kafka 服务器才会记录该 Offset。如果未提交 Offset，消费者重新启动时会根据 StartOffset 决定起始位置。
    // 因此，当启动时，如果消费者没有提交过 Offset，则 Kafka 会根据 StartOffset 决定起始位置。
    // 而当已经提交过一次Offset，服务的记录了Offset，即使指定了StartOffset参数，也会按服务器端记录的Offset来维护，而不在采用StartOffset参数。
    StartOffset: kafka.LastOffset,
    //Logger:,
    //ErrorLogger:,
}
```



#### 消息消费说明

```go
for {
    msg, err := c.reader.FetchMessage(ctx)
    // 业务异常，或者体提交异常，这里只要continue继续FetchMessage，主要服务端没有异常，能够继续FetchMessage不报错
    // 这里FetchMessage的Offset会自动递增，而不是持续刚才异常的那一条消息继续消费。当然重启后，会从已提交的消息的下一条开始消费。
    // 如果要保证每条消息都能正确消费，要么这里异常后，退出从新启动消费，从已提交的下一条从新消费。要么使用死信队列，将异常消费发送至死信队列。
    if errors.Is(err, io.EOF) {
        // close
        log.Printf("failed to fetch kafka message by close reader: %s\n", err.Error())
        return
    } else if err != nil {
        log.Printf("failed to fetch kafka message: %s\n", err.Error())
        continue
    }

    if err = handle(ctx, msg); err != nil {
        log.Printf("failed to handle kafka message: %s\n", err.Error())
        continue
    }

    // 底层会自动尝试3次提交
    err = c.reader.CommitMessages(ctx, msg)
    if errors.Is(err, io.ErrClosedPipe) {
        // close
        log.Printf("failed to commit kafka message by close reader: %s\n", err.Error())
        return
    } else if err != nil {
        log.Printf("failed to commit kafka messages: %s\n", err.Error())
    }
}
```



#### 消费时，获取LastOffset、Offset、Lag

> 启动后，成功消费一条消息后。通过消息message本身的数据，也可以得到Offset和HighWaterMark(LastOffset)，通过计算得到Lag。
>
> 消费者模式：只能通过查询reader的状态reader.Stats获取到Offset和Lag。不能通过reader.Offset()和reader.Lag()查询，消费者模式不支持这样查询，返回-1。

```go
msg, err := reader.FetchMessage(context.Background())
if err != nil {
    return
}

// 消息中获取：Offset、HighWaterMark(EndOffset)、计算得到Lag
value, _ := json.Marshal(&msg)
fmt.Println(string(value))

// 通过reader获取
fmt.Println(reader.Offset()) // -1
fmt.Println(reader.Lag())    // -1
stats := reader.Stats()
value, _ = json.Marshal(&stats)
```

消息内容

> `Offset`：消息在分区中的偏移量。
>
> `HighWaterMark`：消息所在分区的高水位标记。

可以通过HighWaterMark-Offset，间接得到消费延迟Lag的值。

可以通过`当前时间`-Time，得到消息消费延迟的时间。

```json
{
	"Topic": "test",
	"Partition": 0,
    // 消息在分区中的偏移量
	"Offset": 20739,
    // 消息所在分区的高水位标记
	"HighWaterMark": 20746,
	"Key": null,
	"Value": "",
	"Headers": null,
	"Time": "2025-04-04T11:22:37.115Z"
}
```

reader.Stats内容

```json
{
	"Dials": 1,
	"Fetches": 1,
	"Messages": 3,
	"Bytes": 570,
	"Rebalances": 1,
	"Timeouts": 0,
	"Errors": 0,
	"DialTime": {
		"Avg": 430278600,
		"Min": 430278600,
		"Max": 430278600
	},
	"ReadTime": {
		"Avg": 0,
		"Min": 0,
		"Max": 0
	},
	"WaitTime": {
		"Avg": 37223500,
		"Min": 37223500,
		"Max": 37223500
	},
	"FetchSize": {
		"Avg": 0,
		"Min": 0,
		"Max": 0
	},
	"FetchBytes": {
		"Avg": 0,
		"Min": 0,
		"Max": 0
	},
	"Offset": 20741, // 当前Offset
	"Lag": 5,        // 消息延迟的Lag条数
	"MinBytes": 1,
	"MaxBytes": 10000000,
	"MaxWait": 10000000000,
	"QueueLength": 1,
	"QueueCapacity": 1,
	"ClientID": "",
	"Topic": "test",
	"Partition": "-1",
	"DeprecatedFetchesWithTypo": 1
}
```



#### 自动提交

```go
for {
    // 消费者组模式，读取消息后，则立即直接提交。然后这里返回处理相关逻辑
    m, err := reader.ReadMessage(ctx)
    if err != nil {
        break
    }
    // 处理消息...
    
    // 自动提交：无需再调用reader.CommitMessages(ctx, m)提交
}
```



#### 手动提交：同步提交

```go
reader := kafka.NewReader(kafka.ReaderConfig{
    Brokers: []string{"192.168.195.134:9092"},
    Topic:   "test",
    GroupID: "consumer-group-id",
    StartOffset: kafka.FirstOffset, // kafka.FirstOffset最开始的位置，kafka.LastOffset最后的位置
    MaxBytes:       10e6, // 10MB
    GroupBalancers: []kafka.GroupBalancer{kafka.RangeGroupBalancer{}},
    // 自动提交/手动提交：CommitInterval，自动提交Offset的时间间隔：默认为0，表示不自动提交offset
    // CommitInterval = 0 手动提交Offset
    // CommitInterval = 1s 表示每1s提交一次offset，无需手动调用reader.CommitMessages提交确认消息
    CommitInterval: 0,
    Logger:         kafka.LoggerFunc(Printf),
    ErrorLogger:    kafka.LoggerFunc(Printf),
})

for {
    m, err := reader.FetchMessage(ctx)
    if err != nil {
        break
    }
    // 处理消息...
    
    // 手动调用/同步提交：延迟等待提交服务器成功
    if err := reader.CommitMessages(ctx, m); err != nil {
        log.Printf("提交失败: %v", err)
    }
}
```



#### 手动提交：异步提交

```go
reader := kafka.NewReader(kafka.ReaderConfig{
    Brokers: []string{"192.168.195.134:9092"},
    Topic:   "test",
    GroupID: "consumer-group-id",
    StartOffset: kafka.FirstOffset, // kafka.FirstOffset最开始的位置，kafka.LastOffset最后的位置
    MaxBytes:       10e6, // 10MB
    GroupBalancers: []kafka.GroupBalancer{kafka.RangeGroupBalancer{}},
    // 自动提交/手动提交：CommitInterval，自动提交Offset的时间间隔：默认为0，表示不自动提交offset
    // CommitInterval = 0 手动提交Offset
    // CommitInterval = 1s 表示每1s提交一次offset
    // 消费者在每次调用 FetchMessage 获取消息后，会将消息的 Offset 缓存到内存中。这些 Offset 会被记录为“已消费但未提交”的状态。
	// 自动提交模式会按照这个时间间隔，将已内存中标记为“已消费但未提交”的 Offset，并提交给 Kafka 服务器。
	// 手动提交性能慢，自动提交性能快
    CommitInterval: time.Second,
    Logger:         kafka.LoggerFunc(Printf),
    ErrorLogger:    kafka.LoggerFunc(Printf),
})

for {
    m, err := reader.FetchMessage(ctx)
    if err != nil {
        break
    }
    // 处理消息...
    
    // 如果这里不手动触发reader.CommitMessages(ctx, m)提交，那么底层不会触发提交请求至服务器端，消息则一直未提交。
	// 同步提交：阻塞，等待提交响应结果。将这里的Offset存入缓存，并发起提交请求，底层会自动尝试3此提交，并等待结果返回。
	// 异步提交：立即返回，不阻塞。这里的提交是将Offset信息保存在内存中，等待时间到了后再提交，底层会自动尝试3此提交。
    if err := reader.CommitMessages(ctx, m); err != nil {
        log.Fatal("failed to commit messages:", err)
    }
}
```



### 普通消费模式

> 非消费者模式

#### Seek

```go
dialer := kafka.Dialer{
    Timeout:   10 * time.Second,
    DualStack: true,
    SASLMechanism: plain.Mechanism{
        Username: "",
        Password: "",
    },
}
conn, err := dialer.DialLeader(context.Background(), "tcp", brokers[0], topic, 0)
if err != nil {
    return err
}
defer conn.Close()

// 只对手动维护的Offset 有效，对消费者组模式的Offset无效
// SeekStart, 从开始位置往后偏移Offset：Offset=50,表示从开始位置往后偏移50条消息；Offset小于0报错
// SeekAbsolute，Offset=50，表示从第50条消息开始读取消息
// SeekEnd，从末尾位置往前偏移Offset：Offset=50,表示从末尾位置往前偏移50条消息；Offset小于0报错
// SeekCurrent，当前位置偏移Offset：Offset=50,表示当前位置往后偏移50条消息；Offset=-50，表示当前位置往前偏移50条消息
n, err := conn.Seek(50, kafka.SeekAbsolute)
if err != nil {
    fmt.Print(err.Error())
}
```



#### 消费时，获取FirstOffset，LastOffset

> 非消费者组模式：也可以查询reader的状态reader.Stats获取到Offset和Lag，还能从reader.Offset()和reader.Lag()查询(非消费者模式可以返回)。

##### 使用reader的非消费者模式

```go
msg, err := reader.FetchMessage(context.Background())
if err != nil {
    return
}

// 消息中获取：Offset、HighWaterMark(EndOffset)、计算得到Lag
value, _ := json.Marshal(&msg)
fmt.Println(string(value))

// 通过reader获取
fmt.Println(reader.Offset()) // 20741
fmt.Println(reader.Lag())    // 5
stats := reader.Stats()
value, _ = json.Marshal(&stats)
```

reader.Stats内容

```json
{
	"Dials": 1,
	"Fetches": 1,
	"Messages": 3,
	"Bytes": 570,
	"Rebalances": 1,
	"Timeouts": 0,
	"Errors": 0,
	"DialTime": {
		"Avg": 430278600,
		"Min": 430278600,
		"Max": 430278600
	},
	"ReadTime": {
		"Avg": 0,
		"Min": 0,
		"Max": 0
	},
	"WaitTime": {
		"Avg": 37223500,
		"Min": 37223500,
		"Max": 37223500
	},
	"FetchSize": {
		"Avg": 0,
		"Min": 0,
		"Max": 0
	},
	"FetchBytes": {
		"Avg": 0,
		"Min": 0,
		"Max": 0
	},
	"Offset": 20741, // 当前Offset
	"Lag": 5,        // 消息延迟的Lag条数
	"MinBytes": 1,
	"MaxBytes": 10000000,
	"MaxWait": 10000000000,
	"QueueLength": 1,
	"QueueCapacity": 1,
	"ClientID": "",
	"Topic": "test",
	"Partition": "-1",
	"DeprecatedFetchesWithTypo": 1
}
```

##### 使用常规连接方式

```go
dialer := kafka.Dialer{
    Timeout:   10 * time.Second,
    DualStack: false,
    SASLMechanism: plain.Mechanism{
        Username: config.UserName,
        Password: config.Password,
    },
}

// 获取一个分区
conn, err := dialer.DialLeader(context.Background(), "tcp", config.Brokers[0], config.Topic, 0)
if err != nil {
    fmt.Println(err.Error())
    return
}

// 获取全部分区
partitions, err := conn.ReadPartitions()
if err != nil {
    return err
}

// 根据分区ID重新对每个分区建立连接

// 使用kafka.Conn对象获取
fmt.Println(conn.ReadFirstOffset()) // 20739 <nil>
fmt.Println(conn.ReadLastOffset())  // 20746 <nil>
fmt.Println(conn.ReadOffsets())     // 20739 20746 <nil>  
```








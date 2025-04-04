

### 消费者组

#### 消费者组参数说明

```go
kafka.ReaderConfig{
    Brokers: brokers,
    GroupID: groupID,
    Topic:   topic,
    // 队列容量，每次从服务器端获取消息时，最多缓存多少条消息。
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

    // 底层会自动尝试3此提交
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


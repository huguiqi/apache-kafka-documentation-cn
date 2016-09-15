## [2. API](#)

Apache Kafka包含了新的Java客户端（在org.apache.kafka.clients package包）。它的目的是取代原来的Scala客户端，但是为了兼容它们将并存一段时间。老的Scala客户端还打包在服务器中，这些客户端在不同的jar保证并包含着最小的依赖。

### [2.1 生产者 API Producer API](#producerapi)<a id="producerapi"></a>

我们鼓励所有新的开发都使用新的Java生产者。这个客户端经过了生产环境测试并且通常情况它比原来Scals客户端更加快速、功能更加齐全。你可以通过添加以下示例的Maven坐标到客户端依赖中来使用这个新的客户端（你可以修改版本号来使用新的发布版本）：

```
	<dependency>
	    <groupId>org.apache.kafka</groupId>
	    <artifactId>kafka-clients</artifactId>
	    <version>0.10.0.0</version>
	</dependency>

```

生产者的使用演示可以在这里找到[**javadocs**](http://kafka.apache.org/0100/javadoc/index.html?org/apache/kafka/clients/producer/KafkaProducer.html "Kafka 0.10.0 Javadoc")。

对老的Scala生产者API感兴趣的人，可以在[这里](http://kafka.apache.org/081/documentation.html#producerapi)找到相关信息。

### [2.2 消费者API](#consumerapi)<a id="consumerapi"></a>

As of the 0.9.0 release we have added a new Java consumer to replace our existing high-level ZooKeeper-based consumer and low-level consumer APIs. This client is considered beta quality. To ensure a smooth upgrade path for users, we still maintain the old 0.8 consumer clients that continue to work on an 0.9 Kafka cluster. In the following sections we introduce both the old 0.8 consumer APIs \(both high-level ConsumerConnector and low-level SimpleConsumer\) and the new Java consumer API respectively.

在0.9.0发布时我们添加了一个新的Java消费者来取代原来的上层的（high-level）基于ZooKeeper的消费者和底层的（low-level）消费者API。这个客户端被认为是测试质量(beta quality)。

#### [2.2.1 Old High Level Consumer API](#highlevelconsumerapi)<a id="highlevelconsumerapi"></a>

```java
class Consumer {
  /**
   *  Create a ConsumerConnector
   *
   *  @param config  at the minimum, need to specify the groupid of the consumer and the zookeeper
   *                 connection string zookeeper.connect.
   */
  public static kafka.javaapi.consumer.ConsumerConnector createJavaConsumerConnector(ConsumerConfig config);
}

/**
 *  V: type of the message
 *  K: type of the optional key associated with the message
 */
public interface kafka.javaapi.consumer.ConsumerConnector {
  /**
   *  Create a list of message streams of type T for each topic.
   *
   *  @param topicCountMap  a map of (topic, #streams) pair
   *  @param decoder a decoder that converts from Message to T
   *  @return a map of (topic, list of  KafkaStream) pairs.
   *          The number of items in the list is #streams. Each stream supports
   *          an iterator over message/metadata pairs.
   */
  public <K,V> Map<String, List<KafkaStream<K,V>>>
    createMessageStreams(Map<String, Integer> topicCountMap, Decoder<K> keyDecoder, Decoder<V> valueDecoder);

  /**
   *  Create a list of message streams of type T for each topic, using the default decoder.
   */
  public Map<String, List<KafkaStream<byte[], byte[]>>> createMessageStreams(Map<String, Integer> topicCountMap);

  /**
   *  Create a list of message streams for topics matching a wildcard.
   *
   *  @param topicFilter a TopicFilter that specifies which topics to
   *                    subscribe to (encapsulates a whitelist or a blacklist).
   *  @param numStreams the number of message streams to return.
   *  @param keyDecoder a decoder that decodes the message key
   *  @param valueDecoder a decoder that decodes the message itself
   *  @return a list of KafkaStream. Each stream supports an
   *          iterator over its MessageAndMetadata elements.
   */
  public <K,V> List<KafkaStream<K,V>>
    createMessageStreamsByFilter(TopicFilter topicFilter, int numStreams, Decoder<K> keyDecoder, Decoder<V> valueDecoder);

  /**
   *  Create a list of message streams for topics matching a wildcard, using the default decoder.
   */
  public List<KafkaStream<byte[], byte[]>> createMessageStreamsByFilter(TopicFilter topicFilter, int numStreams);

  /**
   *  Create a list of message streams for topics matching a wildcard, using the default decoder, with one stream.
   */
  public List<KafkaStream<byte[], byte[]>> createMessageStreamsByFilter(TopicFilter topicFilter);

  /**
   *  Commit the offsets of all topic/partitions connected by this connector.
   */
  public void commitOffsets();

  /**
   *  Shut down the connector
   */
  public void shutdown();
}


```

You can follow [**this example**](https://cwiki.apache.org/confluence/display/KAFKA/Consumer+Group+Example "Kafka 0.8 consumer example") to learn how to use the high level consumer api.

#### [2.2.2 Old Simple Consumer API](#simpleconsumerapi)<a id="simpleconsumerapi"></a>

```
class kafka.javaapi.consumer.SimpleConsumer {
  /**
   *  Fetch a set of messages from a topic.
   *
   *  @param request specifies the topic name, topic partition, starting byte offset, maximum bytes to be fetched.
   *  @return a set of fetched messages
   */
  public FetchResponse fetch(kafka.javaapi.FetchRequest request);

  /**
   *  Fetch metadata for a sequence of topics.
   *
   *  @param request specifies the versionId, clientId, sequence of topics.
   *  @return metadata for each topic in the request.
   */
  public kafka.javaapi.TopicMetadataResponse send(kafka.javaapi.TopicMetadataRequest request);

  /**
   *  Get a list of valid offsets (up to maxSize) before the given time.
   *
   *  @param request a [[kafka.javaapi.OffsetRequest]] object.
   *  @return a [[kafka.javaapi.OffsetResponse]] object.
   */
  public kafka.javaapi.OffsetResponse getOffsetsBefore(OffsetRequest request);

  /**
   * Close the SimpleConsumer.
   */
  public void close();
}

```

For most applications, the high level consumer Api is good enough. Some applications want features not exposed to the high level consumer yet \(e.g., set initial offset when restarting the consumer\). They can instead use our low level SimpleConsumer Api. The logic will be a bit more complicated and you can follow the example in [**here**](https://cwiki.apache.org/confluence/display/KAFKA/0.8.0+SimpleConsumer+Example "Kafka 0.8 SimpleConsumer example").

#### [2.2.3 New Consumer API](#newconsumerapi)<a id="newconsumerapi"></a>

This new unified consumer API removes the distinction between the 0.8 high-level and low-level consumer APIs. You can use this client by adding a dependency on the client jar using the following example maven co-ordinates \(you can change the version numbers with new releases\):

```
	<dependency>
	    <groupId>org.apache.kafka</groupId>
	    <artifactId>kafka-clients</artifactId>
	    <version>0.10.0.0</version>
	</dependency>

```

Examples showing how to use the consumer are given in the [**javadocs**](http://kafka.apache.org/0100/javadoc/index.html?org/apache/kafka/clients/consumer/KafkaConsumer.html "Kafka 0.9.0 Javadoc").

### [2.3 Streams API](#streamsapi)<a id="streamsapi"></a>

As of the 0.10.0 release we have added a new client library named **Kafka Streams** to let users implement their stream processing applications with data stored in Kafka topics. Kafka Streams is considered alpha quality and its public APIs are likely to change in future releases. You can use Kafka Streams by adding a dependency on the streams jar using the following example maven co-ordinates \(you can change the version numbers with new releases\):

```
	<dependency>
	    <groupId>org.apache.kafka</groupId>
	    <artifactId>kafka-streams</artifactId>
	    <version>0.10.0.0</version>
	</dependency>

```

Examples showing how to use this library are given in the [**javadocs**](http://kafka.apache.org/0100/javadoc/index.html?org/apache/kafka/streams/KafkaStreams.html "Kafka 0.10.0 Javadoc") \(note those classes annotated with**@InterfaceStability.Unstable**, indicating their public APIs may change without backward-compatibility in future releases\).

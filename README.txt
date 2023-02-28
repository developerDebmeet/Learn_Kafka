Kafka Topic : a particular stream of data within a kafka cluster
    like a table in a database(without all constraints)
    as many topics as needed
    topic is identified by name
    any format
    sequence of messages is called a data stream[hence kafka is called a data streaming platform].
    we cannot query topics, instead, we use Kafka Producers to send data and Kafka Consumers to read the data


Partitions and offsets
    topics can be split into partitions
        messages within each partition are ordered
        each message within a partition gets an incremental id, called offset
    kafka topics are immutable, once data is written to a partition, it cannot be changed[no delete/update]
    data is kept in kafka for a limited time (it is configurable, but the default is 1 week)
    offset only have a meaning for a specific partition
        eg offset 3 in partition 0 does not have same data as offset 3 in partition 1
        offsets are not re-used even if previous messages have been deleted
    Order is guaranteed only within a partition (not across partitions)
    Data is assigned randomly to a partition unless a key is provided
    we can have as many partitions per topic as we want


Producers
    producers write data to topics(topics are made of partitions)
    the load is balanced to many brokers thanks to the number of partitions
    producers know to which partition to write to (and which kafka broker[server] has it)
    in case of kafka broker failure, producers will automatically recover
    Message keys
        producers can choose to send a key with the message( string, number, binary etc)
        if key==null, data is sent in round robin manner
        if key!=null, then all messages for that key will always go to the same partition(hashing)
    a kafka partitioner is a code logic that takes a record and determines to which partition to send it to
        key hashing is the process of determining the mapping of a key to a partition
        in the default kafka partitioner, the keys are hashed using the murmur2 algorithm
            targetPartition = Math.abs(Utils.murmur2(keyBytes)) % (numPartitions - 1)


Kafka Message anatomy
    key-binary - can be null
    value-binary - can be null
    compression type : none/gzip/snappy/lz4/zstd
    headers - optional
    partition + offset
    timestamp -  set by system or user


Kafka Message Serializer
    kafka only accepts bytes as a n input from producers and sends bytes out as an output to consumers
    message serialization means transforming objects/data into bytes
    they are used on the value and the key
    common serializers
        string
        json
        int/float
        avro
        protobuf


Consumers
    consumers read data from a topic(identified by name) - pull model
    consumers know which broker to read from
    in case of broker failures consumers can recover
    data is read in order from low to high offset within each partitions


Consumer deserializer
    deserialize indicates how to transform bytes into objects/data
    they are used on the value and the key of the message
    the serialization/deserialization type must not change during a topic lifecycle(create a new topic instead if needed)


Consumer Groups
    all the consumers in an application read data as a consumer group
    each consumer within a group reads from exclusive partitions
    if we have more consumers than partitions, some consumers will be inactive


Multiple consumers on one topic
    in apache kafka it is acceptable to have multiple consumer groups on the same topic
    to create distinct consumer groups, use the consumer property : group.id


Consumer Offsets
    Kafka stores the offsets at which a consumer group has been reading
    the offsets committed are in Kfka topic named __consumer_offsets
    When a consumer in a grp has processed data received from Kafka, it should periodically commit the offsets
        the kafka broker will write to __consumer_offsets, not the group itself
    if a consumer dies, it will be able to read back from where it left off thanks to the committed consumer offsets


Delivery semantics for consumers
    by default, java consumers will automatically commit offsets(at least once)
    there are 3 delivery semantics if we choose to commit manually
    at least once is usually preferred
        offsets are committed after the message is processed
        if the processing goes wrong the message will be read again
        this can result in duplicate processing of messages. Make sure your processing is idempotent(ie processing again the messages will not impact the system)
    at most once
        offsets are committed as soon as messages are received
        if the processing goes wrong, some messages will be lost(they wont be read again)
    exactly once
        for kafka to kafka workflows : use the transactional API - easy with kafka streams API
        for kafka to external system workflows - use an idempotent consumer


Kafka Brokers
    a kafka cluster is composed of multiple brokers(servers)
    each broker is identified with its ID (integer)
    each broker contains certain topic partitions
    after connecting to any broker( called a bootstrap broker) we will be connected to the entire cluster ( kafka clients have smart mechanics for that)




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






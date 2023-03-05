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
    after connecting to any broker( called a bootstrap broker) we will be connected to the entire cluster
        ( kafka clients have smart mechanics for that)
    a good number to get started is 3 brokers, but some big clusters have over 100 brokers


Brokers and topics
    example : a topic A with 3 partitions and a topic B with 2 partitions
        Broker 101 : topic A, Partition 0 + topic B, Partition 1
        Broker 102 : topic A, Partition 2 + topic B, Partition 0
        Broker 103 : topic A, Partition 1
        hence the data is distributed and broker 103 does not have any topic B data

    Kafka broker discovery
        every kafka broker is called a bootstrap server
        that means we only need to connect to one broker and the kafka clients will know how to be connected to the entire cluster(smart clients)
        each broker knows about all brokers, topics and partitions(metadata)


Topic replication factor
    topics should have a replication factor > 1 (usually between 2 and 3)
    this way if a broker is down, another broker can serve the data
    Example topic A with 2 partitions
        Broker 101 :       A,0
        Broker 102 : A,1 + A,0
        Broker 103 : A,1
    This basically introduces some redundancy, and it helps make system resilient in case lets say Broker 102 is down, we still have all required data


Concept of leader for a partition
    at any time only one broker can be a leader for a given partition
    producers can only send data to the broker that is the leader of a partition
    Example topic A with 2 partitions
        Broker 101 :       A,0 (leader)
        Broker 102 : A,1 (leader) + A,0
        Broker 103 : A,1
    the other brokers will replicate the data
    therefore each partition has one leader and multiple ISR ( in sync replica)
    kafka consumers by default will read from the leader broker for a partition
    Kafka consumers replica fetching (kafka v2.4+)
        it is possible to configure consumers to read from the closest replica
        this may help improve latency, and also decrease network costs if using the cloud


Producer Acknowledgements (acks)
    producers can choose to receive ack of data writes:
        acks=0 : producer wont wait for ack (possible data loss)
        acks=1 : producer will wait for leader ack (limited data loss)
        acks=all : leader + replicas acknowledgement (no data loss)


Kafka topic durability
    for a topic replication factor of 3, topic data durability can withdtand 2 brokers loss
    as a rule, for a replication factor of N, we can lose up to N-1 brokers and still recover full data.


Zookeeper
    zookeeper manages brokers ( keeps a list of them )
    zookeeper helps in performing leader election for partitions
    zookeeper sends notifications to kafka in case of changes
        eg new topic/broker dies/broker comes up/ delete topics etc
    kafka 2.x cannot work without Zookeeper
    kafka 3.x can work without Zookeeper (KIP-500) - using Kafka Raft instead
    kafka 4.x will not have zookeeper
    zookeeper by design operates with an odd number of servers ( 1,3,5,7 )
    zookeeper has a leader(writes) the rest of the servers are followers(reads)
    Zookeeper does NOT store consumer offsets with kafka > v0.10
    Zookeeper shows scaling issues when kafka clusters have > 100,000 partitions
    by removing Zookeeper, Apache Kafka can
        scale to millions of partitions, and becomes easier to maintain and set-up
        improve stability, makes it easier to monitor, support and administer
        single security model for the whole system
        single process to start with kafka
        faster controller shutdown and recovery time
        kafka 3.x now implements the Raft protocol (KRaft) in order to replace Zookeeper


Starting Kafka :
    java 11 + latest kafka
    add kafka to path
    zookeeper-server-start.sh ~/kafka_2.13-3.4.0/config/zookeeper.properties
    kafka-server-start.sh ~/kafka_2.13-3.4.0/config/server.properties


Kafka CLI
    kafka-topics.sh
    try to use --bootstrap-server option, not --zookeeper
    kafka-topics.sh --bootstrap-server localhost:9092 --topic first_topic --create
    kafka-topics.sh --bootstrap-server localhost:9092 --topic second_topic --create --partitions 5
    kafka-topics.sh --bootstrap-server localhost:9092 --topic third_topic --create --replication-factor 3
    kafka-topics.sh --bootstrap-server localhost:9092 --list
    kafka-topics.sh --bootstrap-server localhost:9092 --topic first_topic --describe
    kafka-topics.sh --bootstrap-server localhost:9092 --topic second_topic --delete





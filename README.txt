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


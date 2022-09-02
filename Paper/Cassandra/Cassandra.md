### Cassandra - A Decentralized Structured Storage System

#### Data model 
Row key in a table is a string with no size restrictions, altho typically 16-30  bytes long.

#### API
Column families

- insert(table, key, row Mutation)
- get(table, key, row Mutation)
- delete(table, key, row Mutation)

#### System Architecture
Eventual consistency

write: quorum of replicas

read: routes the requests to the closest replica or routes them to all replicas and waits for a quorum of responses.

#### Partitioning
Cassandra partitions data across the cluster using consistent hashing. (coordinator)

Advantages:
Departure or arrival of a node only affects its immediate neighbors and other nodes remain unaffected.

Non-uniform data and load distribution solution
- nodes to get assigned to multiple positions in the circle
- analyze load info on the ring and have slightly loaded nodes move on the ring to alleviate heavily loaded nodes 

#### Replication

High availability && durability

replication factor

Cassandra system elects a leader among its nodes using Zookeeper

#### Membership
Cluster membership is based on a very efficient, anti-entropy Gossip based machanism.

#### Bootstrapping
Random token, persisted to disk locally and also in Zookeeper.

#### Local Persistence
Typical write operation involves a write into a commit log for durability and recoverability and an update into an in-memory data structure.

A merge process runs in the background to collect the differerent files into one file.

Bloom filter to prevent lookups into files that do not contain the key.

#### Implementation Details
Each of these modules rely on an event driven substrate where the message processing pipeline and the task pipeline 
are split into multiple stages along the line of the SEDA.

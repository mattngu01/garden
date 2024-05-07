
## Chapter 3 - storage & retrieval (from the database side)

- log structured storage engines 
	- "append-only sequence of records"
	- essentially like appending a key-value (kv) to a txt file
	- keeps history of all values
	- use indexes to make reading faster
		- hash indexes
		- compaction
			- logs into segments, removing duplicates & keeping recent values for keys
			- can merge several segments together when performing compaction
			- delete old segment files after
	- Sorted String Table (SSTable) -> kv sorted by key
		- don't need to keep index of all keys in memory
			- Log Structured Merge-Tree (LSM)
	- B Tree
- WAL (write ahead logs) -> "changes to data files (where tables and indexes reside) must be written only after those changes have been logged, that is, after log records describing the changes have been flushed to permanent storage"
- page oriented storage engines
- data warehouse  -> essentially a separate db for specific data analytic queries
- column oriented storage  -> store all values in column together


## Chapter 4 - Encoding
- binary encoding 
	- Thrift & protobuf essentially pack fields together
		- needs schema to efficiently pack
	- Avro
- Data flow
	- databases
	- through services: REST & RPC
		- RPC  -> remote procedure calls, think of it like remote function calls 
	- Message Passing Workflow
		- request = message
		- messages go to message broker (which is a queue essentially)
		- asynchronous


## Chapter 5 - Replication
- reasons to replicate data
	- have data closer geographically, less latency
	- allow system to continue working even if parts failed (increase availability)
	- increase read throughput by scaling up # of machines that can serve read queries
- leader-based replication (master/slave, active/passive)
	- clients write to leader
	- leader sends data change to all followers in replication log / change stream
		- follower applies data changes in same order that was processed by leader
	- can read either follower or leader
	- relational db -> postgreSQL, MySQL
	- non-relational db -> MongoDB, rethinkDB, Espresso
	- distributed msg brokers -> Kafka, RabbitMQ
- Synchornous & Asynchronous Replicatoin
	- sync - leader waits for follower to confirm write was received
		- guaranteed to have same copy
		- can be slow if follower is recovering, at max load, etc
	- async - don't wait for confirmation
	- usually just one follower is synchronous & the others are async
- catching a follower up
	- get a snapshot of leader db
	- copy snapshopt to follower
	- follower connect to leader and request data changes that happened since snapshot
		- snapshot has a position in the replication log
			- Postgres -> log sequence number
			- mySQL -> binlog coordinates
	- follow process data changes from position to current
- failover -> promoting new leader
	- detect leader has failed
	- choose new leader - best candidate usually w/ hte most up to date data
		- usually all nodes have to agree on a leader
	- reconfigure sys to use new leader (reroute requests to new leader)
	- problems
		- in async, new leader will lose data / receive conflicting writes, usually old leader's unreplicated writes are discarded
		- ex: mysql follower promoted, has autoincrement primary key & got out of sync, github users could see other user's private info
		- split brain -> two followers think they are the leader

### Replication Logs
-
	- statement based replication -> log every write request
	- WAL shipping -> copy of same data structures found on leader
		- PostgreSQL & Oracle
		- if db changes storage format, can't do this
		- leader and follower have to be on the same version
		- upgrades will require downtime since they have to be on the same version, instead of upgrading & and promoting it to leader
	- logical (row-based) replication
		- diff log formats for replication & for the storage engine
		- for relational db, usually sequence of records
			- inserted row -> new vlaues of all columns
			- deleted row -> have enough info to identify row (pk, other info if no pk)
			- updated row -> new values & identifying info
		- modifying several rows? log records + transaction completed log
		- MySQL binlog follows this
	- trigger based replication -> application layer, custom application code executed when data change occurs
		- can log into separate table, read by external process
		- greater overhead, but flexible since it is external to db

### Replication Lag Examples & Solutions
- async is the only option when having a lot of followers
	- inconsistencies in db as async write propagates
	- eventual consistency - the followers will eventually catch up
- reading your own writes 
	- user viewing data shortly after a write, data didn't catch up & looks like user's submitted data was lost
	- solution: read-after-write consitency (read your writes) -> user reloads page, they will always see any updates they submitted
		- assures user their write went through, may have delay seeing other users' writes though
	- implementations: tldr; read from leader 
		- read user writes from leader, otherwise from follower
		- various criteria to determine when to read from leader
		- client remember timestamp of recent write, read from follower if exists otherwise leader
- monotonic reads
	- reading from multiple followers that have varying levels of lag, may see write happen then disappear
	- monotonic reads -> guarantee this does not happen (one usere making several reads in sequence will not read older data than previously read newer data)
	- user reads from same replica (replica can be chosen on hash from UID)
- consistent prefix reads
	- someone reading writes may see the writes out of order, and those writes may depend on each other 
		- write A -> write B, a 3rd person may see write B -> write A
		- consistent prefix reads -> reading writes will see the same order as the order it was written
	- usually see this in paritioned / sharded db's
	- one solution: make sure writes that are causally (i.e. B caused by A) related to one another are in same partition

### Multi-leader Replication
- each node processes writes, and forwards it to all other nodes
- useful in multidata center operations (leader within each datacenter)
	- leader replicates changes to other leaders in other DC's
	- con: write conflicts must be handled well
- example: calendar on phone, acts as a local leader & forwards writes when it connects to internet & syncs
- example: google doc, make changes on local version & push changes
	- applicatoin obtain lock before user can edit it (can be on granular scale such as single keystroke)

#### Write Conflicts
- if you want synchronous conflict detection (wait for write to replicate to all db's), then might as well use single leader as it defeats purpose of accepting writes from multiple leaders if it has to wait for ack
- avoid conflict by having writes for a particular record go through a specified leader
	- i.e. user always writes to their geographically close DC
- eventual convergence
	- last write wins
	- writes that originated from a "higher-ranked" replica take precence over lower rank (no apparent order in ranks)
	- merge values together
	- record conflict & have applicatoin resolve later (prompt the user?)

#### Multi-leader topology
- all-to-all
	- subject to network conditions
- star topology -> one designate root node forwards writes to all other nodes
	- like circular, subject to single point of failure & reconfiguration usually done manually
- circular topology
- write is taged w/ identifiers of all nodes it went through, to prevent infinite loops

### Leaderless Replication
- made popular again by Amazon _Dynamo_ system
- Riak, Cassandra, Voldemort
- node down?
	- in leader, nee to perform failover
	- leaderless -> send write to multiple nodes and just get 2/3 acks back, deemed ok (quorum write)
	- read requests sent in parallel to multiple nodes & takes most up to date value
- read repair -> client receives stale value, rewrites newer value to oudated replica 
- anti-entropy process -> bg process look for difference in data
- `w + r > n`, write + read > replicas
	- typically `w = r = > n/2`, not necessarily majority but r/w nodes overlap on at least one node
	 - `w + r <= n`, likely to read stale values but lower latency & higher availability

## Chapter 6: Partitioning

### Secondary Indexes
- bread & butter for elasticsearch (a search server)
- definition: 
- not necessarily identify record uniquely but search for occurrence of a particular value, ex: all red cars

```
Document indexing
Primary Index
191 -> [color: red", make: ...]
214 -> [color: "black", make: ...]

Secondary Index
color:red -> [191, ...]
```

## Chapter 7: Transactions
- transaction -> grouping several reads & writes into one logical unit
- ACID (atomiciity, consistency, isolation, durability)
	- most db's have a different definition of isolation, etc
	- atomicity -> several writes in one transaction, all writes get rolled back if transaction fails
	- consistency -> statements abt data that must be true (invariants), if transaction is valid, then invariants will always hold true on db 
	- isolation -> concurrently executing transactins are isolated from one another
		- result is if the transactions were run serially
		- most db's don't use serializable isolation bc of performance deficiencies
	- durability -> once a transaction has finished, it cannot be forgotten
- single object writes are ez, for implementing atomicity (but aren't really transactions)
- multi-object transactions needed for secondary indexes, foreign keys, etc
- Django doesn't retry aborted transactions, just throw exception (the whole point of aborts is to allow safe retries)
- true isolation is hard to maintain, and unreasonable in high traffic environments so they do weak isolation
	- read committed isolation ->only read data that has been committed (no dirty reads), and only overwrite data that has been committed (no dirty writes)
		- prevent dirty writes by having row-level locks
		- prevent dirty reads by remembering both old committed value & new value by transaction w/ writing lock, while writing any transactions are given old value
	- snapshot isolation -> transaction reads from db all data committed in db at start of transaction, throughout the whole transaction even if db changes
		- implemented by maintaining multipe versions of an obj (MVCC, multi-version concurrency control)
			- transaction is given `transaction ID`, increasing, data a transaction wrties is tagged w/ TID of writer
			- `created_by` & `deleted_by` populated by TID, marked for deletion by filling in `deleted_by` field & later time garbage collection
		- observing a consistent snapshot
			1. db makes list of all other transactions in progress, any writes in that list are ignored
			2. writes made by aborted transactions ignored
			3. writes w/ later TID are ignored whether or not it is committed
			4. other writes are visible to application's queries
- preventing lost updates, another type of concurrent writing issue
	- happens in read modify write situations, 
		- incrementing counter
		- two users editing a wiki page at the same time, user saves changes by sending entire page
	- atomic write operations 
		- usually implemented w/ a lock
		- explicit locking -> application has lock on client side
			- error prone, can forget to get a lock
	- detecting lost updates (alternative to atomic writes)
		- abort transaction that is lost and force to retry
	- compare & set -> allow an update to happen only if the value has not changed since you last read
	- conflict resolution in replicated envs
		- locks & compare and set ops assume single up to date copy, not guaranteed in leaderless / multi-leader replication
		- allow concurrent writes to create several conflicting versions of value, then let app code or other means to resolve
	- write skew -> two transactions read the same objs, then proceed w/ their update (dirty write & lost update refer to writing to the same obj, write skew can be other objs being written to that are not necessarily related)
		- ex: double booking same meeting room @ same time, claiming a username
		- causes:
			- application uses `SELECT` query checks for some requirement in rows (good way to prevent this is to use `FOR UPDATE` which locks the rows)
		- phantom -> write in one transaction changes result of search query in another transaction
		- materializing conflicts: create locks artificially for certain "rows" to allow SELECT query to attach locks (otherwise for absent rows can't lock)
- Serializable Isolation -> transactions may execute in parallel, but end result is as if they executed serially
	- serial execution -> throughput linked to single core speed (VoltDB, Redis, Datomic)
		- stored procedure -> code executed on DB that encompasses multiple transactions
			- most time between queries have long idle time (user deciding, etc), so w/ single thread it would be stuck waiting for client response
			- w/ stored procedure, send data & db can execute code all together & in memory instead of waiting for client response
		- to take advantage of multiple cores can give core their own data partition
			- cross partition requires coordination between multiple cores
		- one large transaction can lock db
		- active dataset needs to fit in memory
		- single core must be able to handle write throughput
	- two phase locking (2PL), historical solution
		- several concurrent reads allowed if nobody is writing to particular obj
		- can't write to obj unless no one is reading obj
		- can't read obj if someone is currently writing to obj
		- shared lock -> multiple transactions can hold this, typically readers
		- exclusive lock -> no other transaction can interact w/ obj
		- transaction holds lock until end of transaction
			- 1st phase -> acquire lock
			- 2nd phase -> release lock after commit / abort
		- performance: high overhead acquiring lock, can lead to queue of transactions, unstable latencies, slow at high percentile load
		- predicate lock -> lock belongs to all objs that match some search condition, instead of being tied to a particular obj
			- index range lock -> instead of locking search condition, lock on index / superset of search condition 
				- lower overhead than predicate lock
	- optimistic concurrency control
	- serializable snapshot isolation -> optimistic concurrency control (pessimistic assumes transaction will go wrong) that rolls back if anything goes wrong
		- performs badly if there is high contention for the same obj
		- works best w/ spare capacity
		- calculates causal relationship between query & subsequent transactions affecting result from initial query
			- detecting reads of stale MVCC (multiversion concurrency control) obj vers
				- db tracks when transaction ignore another transaction's writes due to MVCC visibility, db checks if any other ignored writes have been committed before committing transaction
			- detecting writes that affect prior reads

OLTP (online transactional processing)

#technical #programming

Here is a summary fo how the ACID properties are interpreted by a relational DBMS:

Atomicity requires that each transaction is executed in its entirety, or fail without any change being applied.
Consistency requires that the database only passes from a valid state to the next one, without intermediate points.
Isolation requires that if transactions are executed concurrently, the result is equivalent to their serial execution. A transaction cannot see the partial result of the application of another one.
Durability means that the the result of a committed transaction is permanent, even if the database crashes immediately or in the event of a power loss.
Atomicity
MongoDB provides only a document-wide transaction: writes are never partially applied to an inserted or updated document. The operation is atomic in the sense that it either fails or succeeds, for the document in its entirety.

Thus at least Mongo it's not as low-level as using a bunch of files, since the equivalent would be a set of files each with its own lock.

There is no possibility of atomic changes that span multiple documents or collections: either you model the state changes of your application as additional documents, or you can't use Mongo where these database transactions are required. A classic example is to model the operations of a bank account with movement documents, instead of with a single account one: the insertion of a movement either succeed or fails.

If you would have to implement 2 phase commit by yourself, just stick to a relational database for that persistence component of your application (I'm not saying anything on the rest of the data.)

Consistency
Even in replica set configurations, the primary Mongo server is targeted with all the writes; single server consistency is easy to guarantee.

The secondary nodes may be out of date with respect to the primary, as eventual consistency only guarantees that if after a long enough period with no writes, they will get up to date with respect to the primary. However by default the secondary servers cannot answer reads, so you are able to distribute your traffic with the penalty of inconsistency only if you want to and configure them to do so. Consistency and availability are incompatible due to the CAP theorem - you have to choose.

Isolation
Up to a few months ago, MongoDB had a server-wide write lock! I guess you can say it's a perfect isolation mechanism. Read locks instead can be taken by multiple connections at the same time, as long as no one is writing.

From the 2.2 version, Mongo started to use database-specific write locks, and many operation started to yield locks upon encountering slow events such as page faults. Mongo will move in the future at least to collection-specific locks.

However, keep in mind that the Mongo model is similar to transaction auto commits for relational databases: you can't really talk about isolation since every operation is immediately visibile to any other connection.

Durability
Durability of writes is the biggest issue with Mongo. After the 2.0 version, the situation of a single server is:

database files are committed every 60 seconds.
The journal of operations (a write-ahead log) is committed every 100 milliseconds.
These parameters are configurable with the syncdelay and journalCommitInterval configuration options. Committing a file means issuing the OS sync command over it; the journal is, as for all databases, synced very frequently so that in the event of a crash or a forced shutdown the database can be rebuilt from it.

---
title: Positional vs GTID
---

Positional is the old style of tracking replication positions. It is deprecated in all our supported databases major releases.

### GTID

| Support Status  | Test Case |  
| ----------------|-----------|
| Production      | 170 |       

**replication-manager** does not support MariaDB versions lower than 10.0, or MySQL versions lower than 5.6.

For MariaDB Server 10.0 and upper **replication-manager** always uses GTID positions for failover or switchover regardless of using GTID in the replication stream definition. In MariaDB, GTID are always present in the binary logs of MariaDB and can be used to point a slave to his master.

> For MariaDB Server 10.0 and upper **replication-manager** can force positional replication back after a failover or a switchover but this will disable crash safe slave and usage of concurrency in replication streams.    

##### `force-slave-no-gtid-mode` (1.1)

| Item          | Value |
| ----          | ----- |
| Description   | Automatically disable gtid mode on slave. |
| Type          | Boolean |
| Default Value | false |

### Positional Replication

| Support Status  | Test Case |  
| ----------------|-----------|
| Experimental    | 1 |       


Positional replication will be used for pointing replication on MySQL or Percona Servers < 5.7 and >= 5.7 when no GTID replication is set.

In this scenario **replication-manager** is working in best effort mode as it was designed to work agentless. **replication-manager** does not parse binary logs to try to match correct position based on the content of replication EVENTS. GTID are here for fixing this requirement and we state that using them is the correct way to go.   

**replication-manager** would not introduce extra complexity to correct a database lack of unique EVENT identifier. If you feel that this has some importance we could introduce PSEUDO GTID on demand like it is done in Orchestrator to enable finding position by reversing binary log via SQL command.

**replication-manager** design choices have implications:

On MariaDB 10.0 and later, positional replication will be lost after first switchover in favor of using GTID POS.

For other non-GTID databases extra slaves are staying under the old master. They are later moved on elected master by fixing invalid relay architectures. That means that the old master needs to rejoin the cluster first and we can't give insurance that this will succeed, if does not, you may have to reseed all slaves.

One can be inventive like having tree topology with slaves not monitored by the same **replication-manager** cluster so that the new elected master already gets available extra slaves.
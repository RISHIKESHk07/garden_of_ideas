-[Link](https://kellabyte.substack.com/p/building-a-distributed-sql-database)


- A bit about AI 's place in fast prototyping and various model the author used .
- The main obj of the blog , build a redis compt kv db using Accord style consensus (use in cassandra)
- Below being the whole features of *Holostore*

```
Stronger crash/restart behavior

Linearizability checks with Porcupine

Dynamic cluster membership and control-plane tooling with holoctl

Dynamic shard split and movement workflows

Recovery hardening around checkpoints and WAL behavior 
```

- A bit about how the author built the entire thing using Claude & and added sutiable test cases and documentation continuously to help the model move in a proper path ...
- Mention of two log-structured embeddable kv storage engines fjall  & raft engine (TiKV) impl of both are in rust  , WAL by raft engine & on disk storage durability by Fjall
- Impl the dynamic sharding similar too CockroachDB was added .
- Added a holoctl topology for operating the the system ....
- Finally *HoloFusion* a distributed sql engine built on top of Holostore using ballista query engine & DataFusion 

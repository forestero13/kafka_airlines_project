# airline-commercial-stream

Learning project: simulated airline commercial event stream.

Cheat sheet for Session 1

Worth saving into your repo — I'd put this directly in README.md since that's exactly what it's for.

Concepts

Concept	What it means
Partition	Append-only, ordered log shard. Reads only move forward.
Offset	Per-partition message index. Never repeats, never reused.
Key → partition	hash(key) % num_partitions. Same key, same partition, always.
Ordering guarantee	Per-partition only. Never assume order across a whole topic.
Consumer group	Set of consumer instances sharing work on a topic. One partition → one consumer instance, at a time.
Rebalance	Partition reassignment when group membership changes. Preserves offset-order; does not preserve any in-memory state a consumer was holding.
Lag	LOG-END-OFFSET − CURRENT-OFFSET. The health metric. Zero is healthy; climbing means consumers can't keep up.
Partition count	Fixed for practical purposes once messages exist for a key. Hard ceiling on consumer group parallelism.

Commands

bash
# One-time per new terminal (or add to ~/.bashrc)
export MSYS_NO_PATHCONV=1

# Container lifecycle
docker compose up -d
docker compose ps
docker compose down -v          # -v wipes all data too

# Topic admin
docker exec -it broker /opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic <name> --partitions <n> --replication-factor 1
docker exec -it broker /opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic <name>
docker exec -it broker /opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list

# Producer / consumer
docker exec -it broker /opt/kafka/bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic <name> --property parse.key=true --property key.separator=:
docker exec -it broker /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic <name> --from-beginning --property print.key=true --property print.partition=true --property print.offset=true

# Consumer group inspection
docker exec -it broker /opt/kafka/bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group <group-name>

Design decisions made, and why

Topic name commercial.booking.events.v1 — <domain>.<entity>.<type>.<version>, version because partition-count and schema changes both need a migration path, not an in-place edit.
One topic for all booking event types, not one per type — preserves lifecycle ordering.
Key = PNR, not passenger ID or flight number — the entity whose ordering matters.
3 partitions, auto.create.topics.enable=false, KRaft (no ZooKeeper) — production hygiene, not local-only shortcuts.
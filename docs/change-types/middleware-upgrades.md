# Middleware Upgrades

Middleware upgrade change types handle version migrations for message brokers and search engines. Each type runs a rolling, node-by-node upgrade with a filesystem snapshot taken before any changes apply, ensuring rollback restores the previous state exactly.

---

## RabbitMQ Upgrade

**Change type:** `rabbitmq_upgrade`

Upgrades RabbitMQ from version 3.x to 4.x on a cluster. Quorum queues are migrated to the new metadata format as part of the upgrade. The upgrade proceeds node by node — each node is drained, upgraded, and verified before the next node begins.

**Phases:**

1. Preflight — verify all nodes are reachable, check current RabbitMQ and Erlang versions, confirm quorum queue membership is healthy
2. Snapshot — take a filesystem snapshot of each node's data directory and queue state
3. Upgrade nodes — drain, upgrade, and restart each node in sequence; verify the node rejoins the cluster before proceeding to the next
4. Migrate queues — apply quorum queue policy updates for version 4 compatibility
5. Verify — confirm all nodes report the target version and all queues are healthy
6. Report — emit the per-node upgrade result and queue migration summary

**Rollback:** Restore each node from its filesystem snapshot and revert queue policies to the pre-upgrade configuration.

**Connector:** Nexplane Agent

---

## Kafka ZooKeeper to KRaft Bridge

**Change type:** `kafka_zk_to_kraft_bridge`

Transitions a Kafka cluster from ZooKeeper-coordinated mode to KRaft metadata mode using the Apache Kafka bridge migration path. Bridge mode runs both ZooKeeper and KRaft metadata stores simultaneously, allowing a controlled migration before full ZooKeeper removal.

**Phases:**

1. Preflight — verify ZooKeeper version is 2.8 or later, confirm all brokers are reachable, check Kafka version supports KRaft bridge mode
2. Snapshot — take a filesystem snapshot of each broker's data directory
3. Generate KRaft cluster ID — generate a new KRaft cluster UUID and store it in the execution context
4. Enable bridge mode — add KRaft controller configuration to each broker, perform a rolling restart to activate bridge mode
5. Migrate metadata — run `kafka-metadata-migration` to copy ZooKeeper metadata into the KRaft log
6. Verify — confirm all brokers report KRaft bridge mode active and metadata is consistent
7. Report — emit broker-by-broker migration status

**Rollback:** Restore broker configurations and data directories from filesystem snapshots, then perform a rolling restart to return all brokers to ZooKeeper-only mode.

**Connector:** Nexplane Agent

---

## Kafka KRaft Cutover

**Change type:** `kafka_kraft_cutover`

Removes the ZooKeeper dependency from a Kafka cluster that has completed bridge mode migration. This is the second and final step after `kafka_zk_to_kraft_bridge`. After cutover, ZooKeeper nodes are decommissioned.

!!! warning "Irreversible after ZooKeeper decommission"
    Once the decommission phase fires and ZooKeeper nodes are terminated, rollback is no longer possible. Rollback is available during all earlier phases by restoring bridge configuration from snapshot.

**Phases:**

1. Preflight — verify bridge migration is complete by checking all brokers report KRaft bridge mode and metadata logs are in sync
2. Snapshot — take a filesystem snapshot of each broker's data directory
3. Disable ZooKeeper — update broker configs to remove ZooKeeper references, perform a rolling restart to activate KRaft-only mode
4. Verify — confirm all brokers report KRaft-only operation with no ZooKeeper connection attempts
5. Decommission ZooKeeper nodes — terminate ZooKeeper instances

**Rollback:** Before the decommission phase: restore broker configurations from snapshot and perform a rolling restart to re-enable bridge mode. After ZooKeeper decommission: not available.

**Connector:** Nexplane Agent

---

## Elasticsearch Upgrade

**Change type:** `elasticsearch_upgrade`

Upgrades an Elasticsearch cluster from version 7.x to 8.x using a rolling node-by-node strategy. Shard allocation is disabled before each node is upgraded to prevent data movement during the restart window, then re-enabled after the node rejoins the cluster.

**Phases:**

1. Preflight — verify cluster health is green, confirm all nodes are reachable, check current Elasticsearch version
2. Snapshot — trigger an Elasticsearch snapshot to the configured repository; record the snapshot ID in the execution context
3. Upgrade nodes — for each node: disable shard allocation, stop Elasticsearch, upgrade the package, restart, wait for the node to rejoin, re-enable shard allocation, verify cluster health before proceeding
4. Verify — confirm all nodes report the target version and cluster health is green
5. Report — emit per-node upgrade result and final cluster health status

**Rollback:** Restore each node from the filesystem snapshot taken in phase 2.

**Connector:** Nexplane Agent

---

## OpenSearch Upgrade

**Change type:** `opensearch_upgrade`

Upgrades an OpenSearch cluster from version 1.3 to 2.x using the same rolling node-by-node pattern as the Elasticsearch upgrade. Plugin compatibility is checked in preflight before any changes apply.

**Phases:**

1. Preflight — verify cluster health is green, confirm all nodes are reachable, check current OpenSearch version, enumerate installed plugins and verify each is compatible with the target version
2. Snapshot — trigger an OpenSearch snapshot to the configured repository
3. Upgrade nodes — for each node: disable shard allocation, stop OpenSearch, upgrade the package and any plugins, restart, wait for the node to rejoin, re-enable shard allocation, verify cluster health before proceeding
4. Verify — confirm all nodes report the target version, all plugins are loaded, and cluster health is green
5. Report — emit per-node upgrade result, plugin compatibility report, and final cluster health

**Rollback:** Restore each node from the filesystem snapshot taken in phase 2.

**Connector:** Nexplane Agent

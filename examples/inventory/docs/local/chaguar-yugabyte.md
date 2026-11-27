# local/chaguar-yugabyte.yaml

[`chaguar-yugabyte.yaml`](../../local/chaguar-yugabyte.yaml) runs the default local container network with the committer storage layer switched from PostgreSQL to YugabyteDB.

Use it when you want to test the committer against YugabyteDB while keeping Fabric CA enrollment, TLS, mTLS, one load generator, and local monitoring.

## Table of Contents <!-- omit in toc -->

- [Network Diagram](#network-diagram)
- [Inventory Details](#inventory-details)

## Network Diagram

The diagram below summarizes this inventory's Chaguar services and how they fit together.

![local Chaguar YugabyteDB inventory](../../../images/chaguar-yugabyte.drawio.png)

## Inventory Details

All long-running services run as local containers. The Fabric CA databases still use PostgreSQL containers, while the committer database is a compact YugabyteDB deployment.

This inventory deploys these logical services on the local machine:

- 5 Fabric CA servers and 5 PostgreSQL databases for Fabric CA state.
- 4 orderer groups. Each group has 1 router, 1 consenter, 1 assembler, and 1 batcher.
- 1 committer with validator, verifier, coordinator, sidecar, and query service.
- 1 YugabyteDB master and 1 YugabyteDB tablet in cluster `1`.
- 1 Block Explorer server and UI with its own PostgreSQL storage, streaming blocks from the committer sidecar.
- 1 load generator.
- Monitoring with node exporter, cAdvisor, Prometheus, Grafana, Loki, and Alloy.

> [!NOTE]
> You can scale YugabyteDB for stronger performance by adding more master and tablet hosts. See the [distributed Chaguar inventory](../distributed/chaguar.md) for a larger topology with replicated YugabyteDB masters and tablets.

```mermaid
flowchart TD
  all --> network
  all --> fabric_cas
  network --> chaguar
  all --> load_generators
  all --> monitoring
  monitoring --> prometheus
  monitoring --> grafana
  monitoring --> loki
  monitoring --> alloy
  monitoring --> node_exporter
  monitoring --> cadvisor
  grafana --> prometheus
  grafana --> loki
  alloy --> loki
  prometheus --> node_exporter
  prometheus --> cadvisor
  fabric_cas --> fabric_ca_servers
  fabric_cas --> fabric_ca_dbs
  chaguar --> chaguar_orderers
  chaguar --> chaguar_committers
  chaguar --> chaguar_block_explorer
  chaguar_committers --> chaguar_committer
  chaguar_orderers --> chaguar_orderer_1
  chaguar_orderers --> chaguar_orderer_2
  chaguar_orderers --> chaguar_orderer_3
  chaguar_orderers --> chaguar_orderer_4
  chaguar_committer --> committer_dbs
```

The validator and query service both reference `yugabyte_cluster_ref_id: 1`, which points them at the YugabyteDB hosts under `committer_dbs`.

PostgreSQL is still present for Fabric CA state, but it is not the committer storage backend. Monitoring omits the PostgreSQL exporter used by PostgreSQL-backed local inventories.

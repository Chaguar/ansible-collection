# k8s/chaguar-yugabyte.yaml

[`chaguar-yugabyte.yaml`](../../k8s/chaguar-yugabyte.yaml) deploys the Kubernetes sample with the committer storage layer switched from PostgreSQL to YugabyteDB.

Use it when you need to validate Chaguar plus YugabyteDB on Kubernetes: workloads, services, NodePort exposure, TLS material, and committer-to-YugabyteDB references.

## Table of Contents <!-- omit in toc -->

- [Network Diagram](#network-diagram)
- [Inventory Details](#inventory-details)

## Network Diagram

The diagram below summarizes this inventory's Chaguar services and how they fit together.

![Kubernetes Chaguar YugabyteDB inventory](../../../images/chaguar-k8s-yugabyte.drawio.png)

## Inventory Details

Fabric CA, CA databases, orderer, committer, YugabyteDB, load generator, node exporter, Prometheus, Grafana, Loki, and Alloy use Kubernetes task paths. YugabyteDB master and tablet webserver ports are exposed through fixed NodePorts for inspection.

This inventory deploys these logical services as Kubernetes workloads and services:

- 5 Fabric CA servers and 5 PostgreSQL databases for Fabric CA state.
- 4 orderer groups. Each group has 1 router, 1 consenter, 1 assembler, and 1 batcher.
- 1 committer with validator, verifier, coordinator, sidecar, and query service.
- 1 YugabyteDB master and 1 YugabyteDB tablet in cluster `1`.
- 1 Block Explorer server and UI with its own PostgreSQL storage, streaming blocks from the committer sidecar and exposed through NodePort.
- 1 load generator.
- Monitoring with node exporter, Prometheus, Grafana, Loki, and Alloy.

> [!NOTE]
> You can scale YugabyteDB for stronger performance by adding more master and tablet hosts. See the [distributed Chaguar inventory](../distributed/chaguar.md) for a larger topology with replicated YugabyteDB masters and tablets.

```mermaid
flowchart TD
  all --> network
  network --> fabric_cas
  network --> chaguar
  all --> load_generators
  all --> monitoring
  monitoring --> prometheus
  monitoring --> grafana
  monitoring --> loki
  monitoring --> alloy
  monitoring --> node_exporter
  grafana --> prometheus
  grafana --> loki
  alloy --> loki
  prometheus --> node_exporter
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

The validator and query service both use `yugabyte_cluster_ref_id: 1`, which points them at the YugabyteDB hosts under `committer_dbs`.

PostgreSQL is still present for Fabric CA state, but the committer database is YugabyteDB. Monitoring omits the PostgreSQL exporter used by PostgreSQL-backed Kubernetes inventories.

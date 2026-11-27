# local/chaguar-no-tls.yaml

[`chaguar-no-tls.yaml`](../../local/chaguar-no-tls.yaml) is the local container sample with TLS and mTLS disabled.

Use it only when plaintext endpoints are deliberate, such as local protocol, connectivity, or compatibility debugging.

> [!WARNING]
> This inventory is meant for debugging only. It disables both TLS encryption and mTLS client authentication.

## Table of Contents <!-- omit in toc -->

- [Network Diagram](#network-diagram)
- [Inventory Details](#inventory-details)

## Network Diagram

The diagram below summarizes this inventory's Chaguar services and how they fit together.

![local Chaguar no TLS inventory](../../../images/chaguar.drawio.png)

## Inventory Details

All long-running services run as local containers. Ansible connects locally and uses the same container runtime paths as [`chaguar.yaml`](./chaguar.md).

This inventory deploys the same service layout as the default local sample:

- 5 Fabric CA servers and 5 PostgreSQL databases for Fabric CA state.
- 4 orderer groups. Each group has 1 router, 1 consenter, 1 assembler, and 1 batcher.
- 1 committer with validator, verifier, coordinator, sidecar, query service, and PostgreSQL storage.
- 1 Block Explorer server and UI with PostgreSQL storage, streaming blocks from the committer sidecar.
- 1 load generator.
- Monitoring with node exporter, PostgreSQL exporter, cAdvisor, Prometheus, Grafana, Loki, and Alloy.

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
  monitoring --> postgres_exporter
  monitoring --> cadvisor
  grafana --> prometheus
  grafana --> loki
  alloy --> loki
  prometheus --> node_exporter
  prometheus --> postgres_exporter
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
```

TLS-related variables are intentionally omitted for Chaguar, Fabric CA, PostgreSQL, load generator, and monitoring services. Because TLS is disabled, mTLS is disabled as well.

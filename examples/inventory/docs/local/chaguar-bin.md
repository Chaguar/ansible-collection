# local/chaguar-bin.yaml

[`chaguar-bin.yaml`](../../local/chaguar-bin.yaml) exercises binary deployment mode for Chaguar and Fabric CA components on a single machine.

Use it when you want to test install, start, stop, and log handling for local binaries instead of containers.

> [!NOTE]
> This inventory runs these components as binaries managed through `tmux`:
>
> - Fabric CA servers
> - Orderer components
> - Committer components
> - Load generator
> - `cryptogen` CLI
> - Fabric CA client CLI
>
> PostgreSQL databases still run as containers.

## Table of Contents <!-- omit in toc -->

- [Network Diagram](#network-diagram)
- [Inventory Details](#inventory-details)

## Network Diagram

The diagram below summarizes this inventory's Chaguar services and how they fit together.

![local Chaguar binary inventory](../../../images/chaguar-bin.drawio.png)

## Inventory Details

Chaguar, Fabric CA, and the load generator use binary task paths and are managed through `tmux`. PostgreSQL databases still run through container task paths.

This inventory deploys these logical services on the local machine:

- 5 Fabric CA servers as binaries and 5 PostgreSQL databases for Fabric CA state as containers.
- 4 orderer groups. Each group has 1 router, 1 consenter, 1 assembler, and 1 batcher as binaries.
- 1 committer with validator, verifier, coordinator, sidecar, and query service as binaries.
- 1 PostgreSQL committer database as a container.
- 1 Block Explorer server and UI as containers, with PostgreSQL storage, streaming blocks from the committer sidecar.
- 1 load generator as a binary.
- Monitoring with node exporter, PostgreSQL exporter, cAdvisor, Prometheus, Grafana, Loki, and Alloy.

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

The variables `orderer_use_bin`, `committer_use_bin`, `fabric_ca_server_use_bin`, and `loadgen_use_bin` select binary task paths. `cryptogen_use_bin` and `fabric_ca_client_use_bin` exercise the local helper binaries too.

The security posture matches the default local inventory: Fabric CA, TLS, and mTLS are enabled where supported.

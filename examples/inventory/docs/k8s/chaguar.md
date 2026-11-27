# k8s/chaguar.yaml

[`chaguar.yaml`](../../k8s/chaguar.yaml) is the default Kubernetes sample. It deploys a complete Chaguar network with Fabric CA, PostgreSQL, TLS, mTLS, and selected NodePort exposure.

Use it as the baseline when validating Kubernetes workloads, services, storage, and externally reachable endpoints.

## Table of Contents <!-- omit in toc -->

- [Network Diagram](#network-diagram)
- [Inventory Details](#inventory-details)

## Network Diagram

The diagram below summarizes this inventory's Chaguar services and how they fit together.

![Kubernetes Chaguar inventory](../../../images/chaguar-k8s.drawio.png)

## Inventory Details

Fabric CA, CA databases, orderer, committer, PostgreSQL, load generator, node exporter, Prometheus, Grafana, Loki, and Alloy use Kubernetes task paths. Ansible still runs from the control node, but inventory hosts represent Kubernetes resources rather than SSH machines.

This inventory deploys these logical services as Kubernetes workloads and services:

- 5 Fabric CA servers and 5 PostgreSQL databases for Fabric CA state.
- 4 orderer groups. Each group has 1 router, 1 consenter, 1 assembler, and 1 batcher.
- 1 committer with validator, verifier, coordinator, sidecar, query service, and PostgreSQL storage.
- 1 Block Explorer server and UI with PostgreSQL storage, streaming blocks from the committer sidecar and exposed through NodePort.
- 1 load generator.
- Monitoring with node exporter, PostgreSQL exporter, Prometheus, Grafana, Loki, and Alloy.

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
  grafana --> prometheus
  grafana --> loki
  alloy --> loki
  prometheus --> node_exporter
  prometheus --> postgres_exporter
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

This is the Kubernetes equivalent of the default local inventory. External-facing services use fixed NodePort values, while internal services stay behind ClusterIP services.

`actual_host` defaults to `K8S_NODE_IP` or `localhost` for services that must be reachable through NodePort.

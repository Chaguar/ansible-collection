# LFDT Chaguar Deployment Samples

This directory contains sample inventories and wrapper playbooks for running LFDT Chaguar networks with the `lfdt.chaguar` collection.

> [!NOTE]
> The samples are intentionally small enough to inspect and adapt. They are not a production blueprint. Use them to learn the inventory contract, validate changes locally, and create your own topology.

## Table of Contents <!-- omit in toc -->

- [Common Network Shape](#common-network-shape)
  - [Orderer Components](#orderer-components)
  - [Committer Components](#committer-components)
  - [Namespaces](#namespaces)
- [Inventory Families](#inventory-families)
  - [Local Inventories](#local-inventories)
  - [Kubernetes Inventories](#kubernetes-inventories)
  - [Distributed Inventories](#distributed-inventories)
- [Selecting an Inventory](#selecting-an-inventory)
- [Playbooks](#playbooks)

## Common Network Shape

If you are new to Chaguar, read an inventory as a map of services:

| Service family | What to look for in inventory                                       |
| -------------- | ------------------------------------------------------------------- |
| Fabric CA      | Certificate authorities and CA databases used to enroll identities. |
| Orderer        | Router, batcher, consenter, and assembler services.                 |
| Committer      | Validator, verifier, coordinator, sidecar, query service, and a DB. |
| Load generator | A client process that submits test traffic.                         |
| Monitoring     | Exporters, Prometheus, Grafana, Loki, and Alloy.                    |

Chaguar keeps the Fabric governance and identity model, but decomposes the ordering and peer-side work into independently scalable services. Chaguar currently supports a single channel, and state is partitioned across namespaces within that channel.

Inventories define both topology and behavior. A variable such as `orderer_use_k8s: true` changes the runtime mode, while `orderer_use_mtls: true` changes the security posture. Most samples use the same logical Chaguar shape:

```mermaid
flowchart LR
  LG[Load generator]

  subgraph ORDS[Chaguar ordering service - 4 instances]
    direction LR
    R[Routers x4] --> B[Batchers x4]
    B --> C[Consenters x4]
    C --> A[Assemblers x4]
    A -. batch feedback .-> B
  end

  subgraph COM[Chaguar Committer]
    direction LR
    SC[Sidecar]
    CO[Coordinator]
    VAL[Validators]
    VER[Verifiers]
    DB[(PostgreSQL or YugabyteDB)]
    SC -->|relays blocks| CO
    CO -->|distributes work| VAL
    CO -->|distributes work| VER
    VAL -->|stores ledger transactions| DB
  end

  LG -->|submits transactions| R
  LG -->|fetches blocks| A
  LG -->|fetches blocks| SC
  A -->|ordered blocks| SC
```

The monitoring services observe the same three parts of the deployment without participating in the transaction path:

```mermaid
flowchart LR
  subgraph TARGETS[Chaguar deployment]
    direction TB
    LG[Load generator]
    ORD[Ordering service]
    COM[Committer]
  end

  subgraph OBS[Observability]
    direction TB
    PROM[Prometheus] -->|metrics| GRAF[Grafana]
    ALLOY[Alloy] -->|logs| LOKI[Loki]
    LOKI -->|logs| GRAF
  end

  LG -. metrics .-> PROM
  ORD -. metrics .-> PROM
  COM -. metrics .-> PROM
  LG -. logs .-> ALLOY
  ORD -. logs .-> ALLOY
  COM -. logs .-> ALLOY
```

Inventories differ mostly in runtime mode, security settings, crypto source, and database backend.

### Orderer Components

The Chaguar orderer is based on Arma, a BFT ordering service that separates transaction dissemination from consensus. Consensus orders compact metadata for transaction batches rather than full transaction payloads.

| Component | Inventory value                     | Role in the pipeline                                                                                                                                                  |
| --------- | ----------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Router    | `orderer_component_type: router`    | Accepts transaction submissions and dispatches business traffic to batchers. Routers are the client-facing orderer entry point.                                       |
| Batcher   | `orderer_component_type: batcher`   | Groups transactions into batches inside a shard and sends batch attestations to consenters. Adding batcher shards is the main orderer scaling lever in these samples. |
| Consenter | `orderer_component_type: consensus` | Runs the BFT consensus protocol and orders batch attestations.                                                                                                        |
| Assembler | `orderer_component_type: assembler` | Pulls ordered attestations and batches, then assembles ordered blocks for the committer.                                                                              |

Clients submit through routers. Committer sidecars consume blocks from assemblers.

### Committer Components

The Chaguar committer handles post-ordering validation, commit, query, and notification work.

| Component           | Inventory value                           | Role in the pipeline                                                                                                                                                                                                                  | Deployment notes                                                                                                                              |
| ------------------- | ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Sidecar             | `committer_component_type: sidecar`       | Receives ordered blocks from the orderer, persists block flow locally, relays work to the coordinator, delivers committed blocks to client applications, and exposes a notification service for per-transaction status subscriptions. | Stateful; single instance per committer in the samples.                                                                                       |
| Coordinator         | `committer_component_type: coordinator`   | Orchestrates validation and commit by splitting work across verifier and validator-committer services.                                                                                                                                | Stateless; keep one instance in the sample topologies unless intentionally changing the coordination model.                                   |
| Verifier            | `committer_component_type: verifier`      | Verifies transaction signatures against namespace endorsement policies.                                                                                                                                                               | Stateless and horizontally scalable.                                                                                                          |
| Validator-committer | `committer_component_type: validator`     | Performs MVCC validation and commits valid writes to the state database.                                                                                                                                                              | Stateless and horizontally scalable; validators must reference the selected committer database backend.                                       |
| Query service       | `committer_component_type: query-service` | Serves read-only state queries from the committed database state.                                                                                                                                                                     | Stateless.                                                                                                                                    |
| Database            | PostgreSQL or YugabyteDB hosts            | Stores world state, transaction status, and namespace policy data.                                                                                                                                                                    | Stateful; PostgreSQL is compact for local samples, while YugabyteDB is the horizontally scalable DB to choose for high performance use-cases. |

Validators and verifiers can be scaled by adding hosts with the matching `committer_component_type` and unique ports. Validators also need to reference the same database backend used by the rest of the committer deployment.

### Namespaces

A namespace is the unit of state isolation in Chaguar, similar to a chaincode in Hyperledger Fabric. Each namespace carries an endorsement policy that specifies which organizations must endorse a transaction before it can be committed to that namespace's state.

Because Chaguar uses a single channel, all namespaces share the same ordered block stream and the same committer pipeline. Isolation is enforced at the endorsement-policy and state-key level, not at the channel level.

Namespaces are created after the network is started, using the `fxconfig` role through the [init process](../playbooks/fxconfig/README.md#create_namespacesyaml). The `fxconfig` playbooks submit namespace-creation transactions to the running network. No namespace needs to exist for the network itself to start.

## Inventory Families

The inventories live under [`inventory/`](./inventory/) and are grouped by deployment environment. Choose the smallest inventory that exercises the behavior you care about.

### Local Inventories

Local inventories use `ansible_connection: local` to run a complete network on the controller machine as containers or binaries. They are useful for development, functional testing, and documentation examples.

Generated configuration defaults to `LOCAL_ANSIBLE_HOST` or `localhost`. When running containers on macOS, set:

```shell
export LOCAL_ANSIBLE_HOST=host.docker.internal
```

| Inventory                                                                     | Runtime    | TLS | mTLS | Best for                                                                   |
| ----------------------------------------------------------------------------- | ---------- | --- | ---- | -------------------------------------------------------------------------- |
| [`local/chaguar.yaml`](./inventory/docs/local/chaguar.md)                     | Containers | Yes | Yes  | Full local Chaguar network.                                                |
| [`local/chaguar-yugabyte.yaml`](./inventory/docs/local/chaguar-yugabyte.md)   | Containers | Yes | Yes  | Full local Chaguar network with YugabyteDB as committer persistence layer. |
| [`local/chaguar-bin.yaml`](./inventory/docs/local/chaguar-bin.md)             | Binaries   | Yes | Yes  | Full local Chaguar network deployed using binaries.                        |
| [`local/chaguar-cryptogen.yaml`](./inventory/docs/local/chaguar-cryptogen.md) | Containers | Yes | Yes  | Local Chaguar deployment without Fabric CA services for quick debug.       |
| [`local/chaguar-no-mtls.yaml`](./inventory/docs/local/chaguar-no-mtls.md)     | Containers | Yes | No   | Full local Chaguar network without mTLS for debug only.                    |
| [`local/chaguar-no-tls.yaml`](./inventory/docs/local/chaguar-no-tls.md)       | Containers | No  | No   | Full local Chaguar network without TLS and mTLS for debug only.            |

### Kubernetes Inventories

Kubernetes inventories use Ansible host entries to describe Kubernetes workloads and services rather than SSH machines. They require a reachable cluster, `kubectl`, and collection dependencies.

The samples expose externally useful services with NodePort settings. For remote clusters, set:

```shell
export K8S_NODE_IP=<node-ip>
```

| Inventory                                                                 | Runtime    | TLS | mTLS | Best for                                                                      |
| ------------------------------------------------------------------------- | ---------- | --- | ---- | ----------------------------------------------------------------------------- |
| [`k8s/chaguar.yaml`](./inventory/docs/k8s/chaguar.md)                     | Kubernetes | Yes | Yes  | Full Chaguar network deploy on `k8s`.                                         |
| [`k8s/chaguar-yugabyte.yaml`](./inventory/docs/k8s/chaguar-yugabyte.md)   | Kubernetes | Yes | Yes  | Full Chaguar network on `k8s` with YugabyteDB as committer persistence layer. |
| [`k8s/chaguar-cryptogen.yaml`](./inventory/docs/k8s/chaguar-cryptogen.md) | Kubernetes | Yes | Yes  | Chaguar deployment on `k8s` without Fabric CA services for quick debug.       |
| [`k8s/chaguar-no-mtls.yaml`](./inventory/docs/k8s/chaguar-no-mtls.md)     | Kubernetes | Yes | No   | Full Chaguar network on `k8s` without mTLS for debug only.                    |
| [`k8s/chaguar-no-tls.yaml`](./inventory/docs/k8s/chaguar-no-tls.md)       | Kubernetes | No  | No   | Full Chaguar network on `k8s` without TLS and mTLS for debug only.            |

### Distributed Inventories

Distributed inventories use SSH-managed machines for performance-oriented topologies with containers, `cryptogen`, mTLS, and YugabyteDB. They require SSH access, remote Python, a container engine, and real hostnames.

> [!WARNING]
> The distributed sample uses `host_machine_*` placeholders. Replace all of them before running it.

| Inventory                                                             | Runtime    | TLS | mTLS | Best for                                                    |
| --------------------------------------------------------------------- | ---------- | --- | ---- | ----------------------------------------------------------- |
| [`distributed/chaguar.yaml`](./inventory/docs/distributed/chaguar.md) | Containers | Yes | Yes  | Full distributed Chaguar deploy for performance evaluation. |

## Selecting an Inventory

The default examples configuration points to `inventory/local/chaguar.yaml`:

```ini
inventory = ./inventory/local/chaguar.yaml
```

You can change the inventory in [`ansible.cfg`](./ansible.cfg), or override it at runtime:

```shell
export ANSIBLE_INVENTORY=examples/inventory/k8s/chaguar.yaml
```

## Playbooks

The lifecycle playbooks are part of the collection, in the top level of [`playbooks/`](../playbooks/README.md#network-lifecycle-playbooks). The numbered playbooks in [`examples/playbooks/`](./playbooks/) are sample helpers (metrics, logs, vault, hard wipe, run command). They work with the group names used by the sample inventories.

Run the usual lifecycle through the repository `Makefile`:

```shell
make setup
make start
make init
make teardown
```

Without a clone of this repository, run the same lifecycle from the installed collection. `lfdt.chaguar.deploy` imports `binaries` through `init` in one run, which is equivalent to `make setup start init`:

```shell
ansible-playbook lfdt.chaguar.deploy -i inventory.yaml
ansible-playbook lfdt.chaguar.teardown -i inventory.yaml
```

`binaries`, `generate-crypto`, `configs`, `start`, `stop`, `teardown`, and `wipe` likewise drive Semaphore UI (see the [`semaphore_ui` playbooks](../playbooks/semaphore_ui/README.md)) when the loaded inventory defines a `semaphore_ui` group — no network inventory does, so this only takes effect against its own dedicated inventory ([`examples/inventory/local/semaphore-ui.yaml`](./inventory/local/semaphore-ui.yaml)).

For the complete playbook workflow and group contract, see the [playbooks documentation](../playbooks/README.md).

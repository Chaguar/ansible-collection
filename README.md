# LFDT Chaguar Ansible Collection

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE) [![Docs](https://img.shields.io/badge/docs-online-brightgreen.svg)](https://chaguar.github.io/ansible-collection/) ![Tests](https://github.com/Chaguar/ansible-collection/actions/workflows/test.yaml/badge.svg) ![Lint](https://github.com/Chaguar/ansible-collection/actions/workflows/lint.yaml/badge.svg) ![Publish](https://github.com/Chaguar/ansible-collection/actions/workflows/publish.yaml/badge.svg) [![Galaxy Downloads](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fgalaxy.ansible.com%2Fapi%2Fv3%2Fplugin%2Fansible%2Fcontent%2Fpublished%2Fcollections%2Findex%2Fhyperledger%2Fchaguar%2F&query=%24.download_count&label=Galaxy%20Downloads&logo=ansible&color=blue)](https://galaxy.ansible.com/ui/repo/published/hyperledger/chaguar/)

<p align="center">
  <img src="./examples/images/chaguar-ansible-collection-logo.png" alt="LFDT Chaguar Ansible Collection" width="240">
</p>

LFDT Chaguar is an open source permissioned ledger built for performance at scale. It builds on the core principles of Hyperledger Fabric (_sovereign_, _horizontally scalable smart contract execution_ and a _modular_, _agile_ architecture) and is tailored for digital asset use cases, making it well-suited to meet the governance and compliance needs of regulated digital assets.

This repository contains the `hyperledger.chaguar` Ansible collection, which can be used to deploy an LFDT Chaguar network locally, on Kubernetes, or across multiple nodes.

> [!TIP]
> New to Chaguar? Start with the [**Tutorial**](./docs/tutorial/index.md) — a twelve-lesson learning path that takes you from the core concepts to a running network, and on to writing your own inventory. The rest of this README is reference material.

## Table of Contents <!-- omit in toc -->

- [Installation](#installation)
  - [Option 1: Install from Ansible Galaxy](#option-1-install-from-ansible-galaxy)
  - [Option 2: Clone under `ANSIBLE_COLLECTIONS_PATHS` (for development)](#option-2-clone-under-ansible_collections_paths-for-development)
  - [Option 3: Install from source](#option-3-install-from-source)
- [Usage](#usage)
- [Prerequisites](#prerequisites)
  - [Setup the control node](#setup-the-control-node)
  - [Setup the remote nodes](#setup-the-remote-nodes)
- [Run a sample Chaguar network](#run-a-sample-chaguar-network)
  - [Choose another sample inventory](#choose-another-sample-inventory)
  - [Run with Podman/Docker on macOS](#run-with-podmandocker-on-macos)
  - [1. Setup the network](#1-setup-the-network)
  - [2. Start the network](#2-start-the-network)
  - [3. Initialize the network](#3-initialize-the-network)
  - [4. Observe the network](#4-observe-the-network)
  - [5. Teardown the network](#5-teardown-the-network)
- [Supported commands](#supported-commands)
  - [Restrict commands to a group of hosts](#restrict-commands-to-a-group-of-hosts)
- [Contributing](#contributing)
  - [Updating role documentation](#updating-role-documentation)
- [License](#license)

## Installation

### Option 1: Install from Ansible Galaxy

To install the latest published version of the `hyperledger.chaguar` collection from [Ansible Galaxy](https://galaxy.ansible.com/ui/repo/published/hyperledger/chaguar/), run:

```shell
ansible-galaxy collection install hyperledger.chaguar
```

Then install the collection's dependencies:

```shell
ansible-galaxy collection install -r ~/.ansible/collections/ansible_collections/hyperledger/chaguar/requirements.yml
```

### Option 2: Clone under `ANSIBLE_COLLECTIONS_PATHS` (for development)

To install the `hyperledger.chaguar` collection on your control node, run:

> [!NOTE]
> This is the recommended way if you plan to develop and change the scripts, since it allows to test directly the modified scripts avoiding to reinstall the collection at every change.

```shell
git clone https://github.com/Chaguar/ansible-collection.git ~/.ansible/collections/ansible_collections/hyperledger/chaguar
cd ~/.ansible/collections/ansible_collections/hyperledger/chaguar
make install-deps
```

The sample inventories default `project_dir` to `~/.ansible/collections/ansible_collections/hyperledger/chaguar`. If your checkout lives elsewhere (for example, an older clone under `ansible_collections/hyperledger/fabricx`), move it to that path or export `PROJECT_DIR` with the path of your checkout.

> [!WARNING]
> Do not run `make install` with this setup — the collection is already live from the cloned directory. Running it would overwrite your checkout with a built artifact. Use `make install-deps` to install dependencies only. The Makefile will also guard against this and abort if it detects the risk.

### Option 3: Install from source

If you don't know where the `COLLECTIONS_PATHS` is located, or you don't plan to develop/change the Ansible scripts within the collection, you can install from source using the commands:

```shell
git clone https://github.com/Chaguar/ansible-collection.git
cd ansible-collection
make install
```

## Usage

The collection provides a set of Ansible roles that can be used to deploy a LFDT Chaguar network in a distributed manner. Each role is devoted to a specific component.

Each role comes with tasks and each task performs a specific operation, such as starting, stopping, configuring, or wiping a component. The collection also includes predefined playbooks that compose these roles into common lifecycle operations. If your inventory follows the expected group names, you can use those playbooks directly instead of writing your own orchestration layer.

The collection is organized around three concepts:

| Concept     | Description                                                                                                                     |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Roles       | Low-level component automation. Use these when you need custom orchestration.                                                   |
| Playbooks   | Reusable lifecycle workflows that call the roles in the expected order.                                                         |
| Inventories | The deployment model: which components exist, where they run, which ports they use, and which security/runtime mode is enabled. |

For example, the playbook [playbooks/orderer/start.yaml](./playbooks/orderer/start.yaml) shows how to use the `hyperledger.chaguar.orderer` role to start a Chaguar Orderer node:

```yaml
- name: Start Chaguar Orderer components
  ansible.builtin.include_role:
    name: hyperledger.chaguar.orderer
    tasks_from: start
```

For more information, see the [roles](./roles/README.md), [playbooks](./playbooks/README.md), and [example inventories](./examples/README.md) documentation.

## Prerequisites

### Setup the control node

To run such Ansible collection, you need to have the following prerequisites installed on your control node:

- `python` >= 3.11;
- `docker` or `podman`;
- `go` >= 2.16 (only needed if you run binary-based inventories);
- `jq`;
- `kubectl` (needed to run [k8s inventories](./examples/inventory/README.md#kubernetes)).

After having cloned this repository, run:

```shell
make install-deps
```

### Setup the remote nodes

The collection comes with a playbook that can be used to automatically setup all the remote nodes at once. From your control node, run:

```shell
make install-remote-node-deps
```

> [!WARNING]
> The playbook installs the needed packages and requires `sudo` permission. Make sure to use a passwordless `sudo` user so the playbook can complete.

## Run a sample Chaguar network

This repository comes with Ansible inventories and example playbooks that can start sample Chaguar networks on your local machine, in Kubernetes, or across multiple machines. See the [examples README](./examples/README.md) for the available inventory families.

By default, the [`local/chaguar.yaml`](./examples/inventory/docs/local/chaguar.md) inventory is used:

![chaguar-inventory](./examples/images/chaguar.drawio.png)

This diagram shows the default local sample topology: Fabric CA services issue identities, four orderer groups produce ordered blocks, a PostgreSQL-backed committer validates and stores state, and monitoring collects metrics.

To run it on your local machine, follow these steps.

The sample lifecycle is:

```mermaid
flowchart LR
  setup[make setup] --> start[make start]
  start --> init[make init]
```

### Choose another sample inventory

The current sample inventory families are:

| Family                                                      | Documentation                                                                    |
| ----------------------------------------------------------- | -------------------------------------------------------------------------------- |
| [Local](./examples/README.md#local-inventories)             | Single-machine deployment, useful for trying and testing on your local machine   |
| [Distributed](./examples/README.md#distributed-inventories) | Multi-machine deployment to scale the network and test high-throughput use-cases |
| [Kubernetes](./examples/README.md#kubernetes-inventories)   | Kubernetes deployments                                                           |

Set `ANSIBLE_INVENTORY` to use another inventory without editing [`examples/ansible.cfg`](./examples/ansible.cfg):

```shell
export ANSIBLE_INVENTORY=examples/inventory/k8s/chaguar.yaml
```

### Run with Podman/Docker on macOS

If you run a local container inventory on macOS, you can run into connectivity issues between host processes and containers. Docker and Podman run Linux containers in a VM, so `localhost` inside a container is not the same endpoint as `localhost` on macOS.

Set `LOCAL_ANSIBLE_HOST` so generated configuration points at an address containers can resolve:

```yaml
# add this to .bashrc or any other file sourced by your shell
export LOCAL_ANSIBLE_HOST="host.docker.internal"
```

If binaries also need to resolve the same name from the macOS host, add it to `/etc/hosts`:

```ini
echo "127.0.0.1 host.docker.internal" | sudo tee -a /etc/hosts
```

### 1. Setup the network

The first step consists in generating the artifacts needed by the nodes to run:

```shell
make setup
```

### 2. Start the network

Once the artifacts have been correctly generated and distributed, you can start the nodes with the command:

```shell
make start
```

### 3. Initialize the network

After the components are up, run:

```shell
make init
```

This command runs post-start initialization, such as creating the namespaces indicated in the inventory using `fxconfig`.

### 4. Observe the network

You can access [Grafana dashboards](http://localhost:3000/dashboards) (user=_admin_, password=_adminPWD_) to see how the Chaguar network is handling the transactions processing.

> [!NOTE]
> These Grafana credentials are sample defaults. Change them before using an adapted inventory in a shared environment.

### 5. Teardown the network

To shut the network down, run:

```shell
make teardown
```

The command proceeds by stopping all the running instances and also cleaning any artifact that has been generated on disk by such instances.

## Supported commands

All the high-level commands are defined within the [Makefile](./Makefile). To get the list of all the possible commands, run:

```shell
make help
```

The most frequently used commands are:

| Command                    | Usage                                                                        |
| -------------------------- | ---------------------------------------------------------------------------- |
| `install`                  | Build and install the `hyperledger.chaguar` collection locally.              |
| `install-deps`             | Wrapper for `install-venv` + `install-python-deps` + `install-ansible-deps`. |
| `install-venv`             | Install a `venv` environment.                                                |
| `install-python-deps`      | Install Python dependencies on the control node.                             |
| `install-ansible-deps`     | Install the Ansible collections required by this repository.                 |
| `install-remote-node-deps` | Install the needed dependencies on the remote hosts.                         |
| `lint`                     | Run `ansible-lint` checks.                                                   |
| `check-license-header`     | Verify license headers on all files.                                         |
| `check-trailing-spaces`    | Check for trailing spaces in `.j2` files.                                    |
| `login-cr`                 | Log a container engine within a container registry.                          |
| `setup`                    | Wrapper for `binaries` + `artifacts` + `configs`.                            |
| `artifacts`                | Wrapper for `generate-crypto` + `genesis-block`.                             |
| `generate-crypto`          | Generate the crypto material on the controller node.                         |
| `genesis-block`            | Build the genesis block for the network.                                     |
| `binaries`                 | Build/install binaries on controller or remote nodes for the targeted hosts. |
| `clean`                    | Clean all the artifacts and binaries built on the controller node.           |
| `clean-cache`              | Clean the Ansible cache.                                                     |
| `configs`                  | Create/Ship the configs to the remote nodes.                                 |
| `start`                    | Start the targeted hosts.                                                    |
| `stop`                     | Stop the targeted hosts without deleting the data.                           |
| `teardown`                 | Teardown the targeted hosts (stop and delete data).                          |
| `update`                   | Update the targeted hosts (stop + binaries + start).                         |
| `restart`                  | Restart the targeted hosts (stop + start).                                   |
| `hard-restart`             | Hard restart the targeted hosts (teardown + start).                          |
| `wipe`                     | Wipe out the config artifacts and the binaries from the remote hosts.        |
| `hard-wipe`                | Wipe the deploy folder from the remote hosts.                                |
| `targets`                  | Generate Makefile targets for all inventory hosts.                           |
| `run-command`              | Run a generic command on the targeted hosts.                                 |
| `ping`                     | Check that the component ports are open.                                     |
| `get-metrics`              | Get the metrics from the targeted components.                                |
| `fetch-crypto`             | Fetch the crypto material from the targeted hosts.                           |
| `fetch-logs`               | Fetch the logs from the targeted hosts.                                      |
| `limit-rate`               | Set the TPS rate on the load generators.                                     |

### Restrict commands to a group of hosts

By default all the `Makefile` commands target all the hosts which are defined within the reference inventory. However, the playbooks have been tailored in such a way that the actions can be restricted to a sub-group (or even a single host) through the `target_hosts` Ansible variable, which is reflected as the `TARGET_HOSTS` variable in the [Makefile](./Makefile).

The `Makefile` comes with a set of [predefined host groups](./target_groups.mk) that can be used to easily restrict commands:

| Group                | Target                                          |
| -------------------- | ----------------------------------------------- |
| `fabric_cas`         | The Fabric CA servers                           |
| `chaguar`            | The Chaguar network nodes (orderers+committer). |
| `chaguar_orderers`   | All the Chaguar orderers.                       |
| `chaguar_committers` | The Chaguar committer components.               |
| `chaguar_evm`        | The Chaguar EVM gateway.                        |
| `load_generators`    | All the load_generators.                        |
| `monitoring`         | All the monitoring instances.                   |

For example, running:

```shell
make chaguar_orderers start
```

restricts the command to the host group `chaguar_orderers` defined within the inventory.

Group/host targets can also be chained on the same command line to union their hosts, for example:

```shell
make chaguar_orderers chaguar_committers start
```

restricts the command to the union of the `chaguar_orderers` and `chaguar_committers` host groups.

All these groups are reflected in the [sample inventories](./examples/README.md). If you plan to use the playbooks provided with the collection, keep the group names identical so the high-level lifecycle commands keep working.

## Contributing

Contributions to this project are welcomed and encouraged.

If you'd like to help improve this project, please follow these steps:

1. **Fork** the repository;

2. **Create a new branch** for your feature or bugfix:

```bash
git checkout -b your-feature-name
```

### Updating role documentation

Role READMEs and role defaults are automatically generated from `meta/argument_specs.yaml`. When you add a new role task entry point or introduce variables for an existing entry point, update that role's `argument_specs.yaml` and then run:

```shell
make generate-roles-docs
```

## License

This project is licensed under the Apache License 2.0.  
You are free to use, modify, and distribute this software in accordance with the terms of the license.

For more details, please refer to the [LICENSE](./LICENSE) file included in this repository.

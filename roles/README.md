# LFDT Chaguar Ansible Collection Roles

This document lists all the roles that come together within the collection and that can be used to manage the LFDT Chaguar components.

## Table of Contents <!-- omit in toc -->

- [How to use](#how-to-use)
- [Roles list](#roles-list)

## How to use

The roles within this repository can be used to handle the different components that are needed in order to boostrap a Chaguar network.

They are structured in multiple tasks, each one devoted to a specific action. To perform an action, you can use the `ansible.builtin.include_role` functionality:

```yaml
- name: Generate the crypto material with cryptogen
  ansible.builtin.include_role:
    name: hyperledger.chaguar.cryptogen
    tasks_from: start
```

The roles are interconnected each other, meaning that some roles play as base for other role functions (e.g. the [bin](./bin) or [container](./container) roles).

## Roles list

The roles that come within such collection are listed hereafter. Click on them to access their documentation:

- [`hyperledger.chaguar.armageddon`](./armageddon);
- [`hyperledger.chaguar.bin`](./bin);
- [`hyperledger.chaguar.block_explorer`](./block_explorer);
- [`hyperledger.chaguar.cadvisor`](./cadvisor);
- [`hyperledger.chaguar.committer`](./committer);
- [`hyperledger.chaguar.configtxgen`](./configtxgen);
- [`hyperledger.chaguar.configtxlator`](./configtxlator);
- [`hyperledger.chaguar.container`](./container);
- [`hyperledger.chaguar.cryptogen`](./cryptogen);
- [`hyperledger.chaguar.evm`](./evm);
- [`hyperledger.chaguar.fabric_ca`](./fabric_ca);
- [`hyperledger.chaguar.fxadmin`](./fxadmin);
- [`hyperledger.chaguar.fxconfig`](./fxconfig);
- [`hyperledger.chaguar.git`](./git);
- [`hyperledger.chaguar.go`](./go);
- [`hyperledger.chaguar.grafana`](./grafana);
- [`hyperledger.chaguar.jaeger`](./jaeger);
- [`hyperledger.chaguar.k8s`](./k8s);
- [`hyperledger.chaguar.loadgen`](./loadgen);
- [`hyperledger.chaguar.node_exporter`](./node_exporter);
- [`hyperledger.chaguar.openshift`](./openshift);
- [`hyperledger.chaguar.openssl`](./openssl);
- [`hyperledger.chaguar.orderer`](./orderer);
- [`hyperledger.chaguar.package`](./package);
- [`hyperledger.chaguar.postgres`](./postgres);
- [`hyperledger.chaguar.postgres_exporter`](./postgres_exporter);
- [`hyperledger.chaguar.prometheus`](./prometheus);
- [`hyperledger.chaguar.semaphore_ui`](./semaphore_ui);
- [`hyperledger.chaguar.tmux`](./tmux);
- [`hyperledger.chaguar.utils`](./utils);
- [`hyperledger.chaguar.yugabyte`](./yugabyte).

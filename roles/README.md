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
    name: lfdt.chaguar.cryptogen
    tasks_from: start
```

The roles are interconnected each other, meaning that some roles play as base for other role functions (e.g. the [bin](./bin) or [container](./container) roles).

## Roles list

The roles that come within such collection are listed hereafter. Click on them to access their documentation:

- [`lfdt.chaguar.armageddon`](./armageddon);
- [`lfdt.chaguar.bin`](./bin);
- [`lfdt.chaguar.block_explorer`](./block_explorer);
- [`lfdt.chaguar.cadvisor`](./cadvisor);
- [`lfdt.chaguar.committer`](./committer);
- [`lfdt.chaguar.configtxgen`](./configtxgen);
- [`lfdt.chaguar.configtxlator`](./configtxlator);
- [`lfdt.chaguar.container`](./container);
- [`lfdt.chaguar.cryptogen`](./cryptogen);
- [`lfdt.chaguar.evm`](./evm);
- [`lfdt.chaguar.fabric_ca`](./fabric_ca);
- [`lfdt.chaguar.fxadmin`](./fxadmin);
- [`lfdt.chaguar.fxconfig`](./fxconfig);
- [`lfdt.chaguar.git`](./git);
- [`lfdt.chaguar.go`](./go);
- [`lfdt.chaguar.grafana`](./grafana);
- [`lfdt.chaguar.jaeger`](./jaeger);
- [`lfdt.chaguar.k8s`](./k8s);
- [`lfdt.chaguar.loadgen`](./loadgen);
- [`lfdt.chaguar.node_exporter`](./node_exporter);
- [`lfdt.chaguar.openshift`](./openshift);
- [`lfdt.chaguar.openssl`](./openssl);
- [`lfdt.chaguar.orderer`](./orderer);
- [`lfdt.chaguar.package`](./package);
- [`lfdt.chaguar.postgres`](./postgres);
- [`lfdt.chaguar.postgres_exporter`](./postgres_exporter);
- [`lfdt.chaguar.prometheus`](./prometheus);
- [`lfdt.chaguar.semaphore_ui`](./semaphore_ui);
- [`lfdt.chaguar.tmux`](./tmux);
- [`lfdt.chaguar.utils`](./utils);
- [`lfdt.chaguar.yugabyte`](./yugabyte).

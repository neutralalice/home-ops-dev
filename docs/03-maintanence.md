## 🛠️ Talos and Kubernetes Maintenance

### ⚙️ Updating Talos node configuration

> [!TIP]
> Ensure you have updated `talconfig.yaml` and any patches with your updated
> configuration. In some cases you **not only need to apply the configuration
> but also upgrade talos** to apply new configuration.

```sh
# (Re)generate the Talos config
just talos generate-config
# Apply the config to the node
just talos apply-node <ip>
# e.g. just talos apply-node 10.10.10.10
```

### ⬆️ Updating Talos and Kubernetes versions

> [!TIP]
> Ensure the `talosVersion` and `kubernetesVersion` in `talenv.yaml` are
> up-to-date with the version you wish to upgrade to.

```sh
# (Re)generate the Talos config
just talos generate-config
# Apply the config to the node
just talos apply-node <ip>
# e.g. just talos apply-node 10.10.10.10
just talos upgrade-node <ip>
# e.g. just talos upgrade-node 10.10.10.10
```

```sh
# Upgrade cluster to a newer Kubernetes version
just talos upgrade-k8s
```

### ➕ Adding a node to your cluster

At some point you might want to expand your cluster to run more workloads and/or
improve the reliability of your cluster. Keep in mind it is recommended to have
an **odd number** of control plane nodes for quorum reasons.

You don't need to re-bootstrap the cluster to add new nodes. Follow these steps:

1. **Prepare the new node**: Review the
   [Stage 2: Machine Preparation](#stage-2-machine-preparation) section and boot
   your new node into maintenance mode.

2. **Get the node information**: While the node is in maintenance mode, retrieve
   the disk and MAC address information needed for configuration:

    ```sh
    talosctl get disks -n <ip> --insecure
    talosctl get links -n <ip> --insecure
    ```

3. **Update the configuration**: Read the documentation for
   [talhelper](https://budimanjojo.github.io/talhelper/latest/) and extend the
   `talconfig.yaml` file manually with the new node information (including the
   disk and MAC address from step 2).

4. **Generate and apply the configuration**:

    ```sh
    # Render your talosconfig based on the talconfig.yaml file
    just talos generate-config

    # Apply the configuration to the node
    just talos apply-node <ip>
    # e.g. just talos apply-node 10.10.10.10
    ```

The node should join the cluster automatically and workloads will be scheduled
once they report as ready.

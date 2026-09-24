# Talos and Kubernetes Maintenance

## Updating components

Parts to this consist of

- Talos configuration: The underlying talos resources
- Talos versioning: The actual changes talos operating system
- Kubernetes versioning: The core apiserver,kubelet,scheduler versions

### Updating Talos node configuration

```sh
# Rendering the config just lets it be inspected
just talos::render-config
# Apply the config to a node
just talos::apply-node -n $ip
# Apply the config to the whole cluster
just talos::apply-cluster -n $ip
```

### Updating Talos and Kubernetes versions

```sh
# Upgrade a talos node version
just talos::upgrade-node -n $ip
# Upgrade kubernetes across the cluster
just talos::upgrade-k8s
```

## Adding nodes

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

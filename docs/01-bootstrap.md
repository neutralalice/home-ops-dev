# Bootstrap

## 01: Node prep

1. Head to the [Talos Linux Image Factory](https://factory.talos.dev) and follow
   the instructions. Choose just `i915` and `intel-ucode` as system extensions.

2. Download the ISO.

3. Flash the Talos ISO on USB drive and boot from it.

4. Verify with `nmap` that the nodes are available on the network.

    ```sh
    nmap -Pn -n -p 50000 10.10.85.11/24 -vv | grep 'Discovered'
    ```

## 02: Workstation prep

1. Clone this repository

2. **Install** the
   [Mise CLI](https://mise.jdx.dev/getting-started.html#installing-mise-cli)

3. **Activate** Mise via
   [activation guide](https://mise.jdx.dev/getting-started.html#activate-mise).

4. Use `mise` to install the **required** CLI tools:

    ```sh
    mise trust
    mise install
    ```

5. Logout of GHCR

    ```sh
    podman logout ghcr.io
    helm registry logout ghcr.io
    ```

## 03: Cloudflare prep

1. Create a Cloudflare API token for use with cloudflared and external-dns by
   reviewing the official
   [documentation](https://developers.cloudflare.com/fundamentals/api/get-started/create-token/)
   and following the instructions below.

    - Click the blue `Use template` button for the `Edit zone DNS` template.
    - Name your token `kubernetes`
    - Under `Permissions`, click `+ Add More` and add permissions
      `Zone - DNS - Edit` and `Account - Cloudflare Tunnel - Read`
    - Limit the permissions to a specific account and/or zone resources and then
      click `Continue to Summary` and then `Create Token`.
    - **Save this token**

2. Create the Cloudflare Tunnel:

    ```sh
    cloudflared tunnel login
    cloudflared tunnel create --credentials-file cloudflare-tunnel.json kubernetes
    ```

### 04: Bootstrap Talos, Kubernetes, and Flux

1. Install Talos:

    ```sh
    just bootstrap::talos
    ```

2. Install cilium, coredns, spegel, flux:

    ```sh
    just bootstrap::apps
    ```

3. Watch the rollout:

    ```sh
    kubectl get pods --all-namespaces --watch
    ```

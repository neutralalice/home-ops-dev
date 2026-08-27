### Stage 2: Machine Preparation

1. Head over to the [Talos Linux Image Factory](https://factory.talos.dev) and
   follow the instructions. Be sure to only choose the **bare-minimum system
   extensions** as some might require additional configuration and prevent Talos
   from booting without it. Depending on your CPU start with the Intel/AMD
   system extensions (`i915`, `intel-ucode` & `mei` **or** `amdgpu` &
   `amd-ucode`), you can always add system extensions after Talos is installed
   and working.

2. This will eventually lead you to download a Talos Linux ISO (or for SBCs a
   RAW) image. Make sure to note the **schematic ID** you will need this later
   on.

3. Flash the Talos ISO or RAW image to a USB drive and boot from it on your
   nodes.

4. Verify with `nmap` that your nodes are available on the network. (Replace
   `192.168.1.0/24` with the network your nodes are on.)

    ```sh
    nmap -Pn -n -p 50000 192.168.1.0/24 -vv | grep 'Discovered'
    ```

### Stage 3: Local Workstation

> [!TIP]
> It is recommended to set the visibility of your repository to `Public` so you
> can easily request help if you get stuck.

1. Create a new repository by clicking the green `Use this template` button at
   the top of this page, then clone the new repo you just created and `cd` into
   it. Alternatively you can use the [GitHub CLI](https://cli.github.com/) ...

    ```sh
    export REPONAME="home-ops"
    gh repo create $REPONAME --template onedr0p/cluster-template --public --clone
    cd $REPONAME
    ```

2. **Install** the
   [Mise CLI](https://mise.jdx.dev/getting-started.html#installing-mise-cli) on
   your local workstation.

3. **Activate** Mise in your shell by following the
   [activation guide](https://mise.jdx.dev/getting-started.html#activate-mise).

4. Use `mise` to install the **required** CLI tools:

    ```sh
    mise trust
    pip install pipx
    mise install
    ```

    📍 _**Having trouble installing the tools?** Try unsetting the `GITHUB_TOKEN`
    env var and then run these commands again_

    📍 _**Having trouble compiling Python?** Try running
    `mise settings python.compile=0` and then run these commands again_

5. Logout of the GitHub Container Registry as this may cause authorization
   problems in future steps when using the public registry:

    ```sh
    docker logout ghcr.io
    helm registry logout ghcr.io
    ```

### Stage 4: Cloudflare configuration

> [!WARNING]
> If any of the commands fail with `command not found` or `unknown command` it
> means `mise` is either not installed, activated or it could be configured
> incorrectly.

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
    - **Save this token somewhere safe**, you will need it later on.

2. Create the Cloudflare Tunnel:

    ```sh
    cloudflared tunnel login
    cloudflared tunnel create --credentials-file cloudflare-tunnel.json kubernetes
    ```

### Stage 5: Cluster configuration

1. Generate the config files from the sample files:

    ```sh
    just init
    ```

2. Fill out the `cluster.toml` configuration file using the comments in it as a
   guide.

3. Template out the kubernetes and talos configuration files, if any issues come
   up be sure to read the error and adjust your config files accordingly.

    ```sh
    just configure
    ```

4. Push your changes to git:

    📍 _**Verify** all the `./kubernetes/**/*.sops.*` files are **encrypted**
    with SOPS_

    ```sh
    git add -A
    git commit -m "chore: initial commit :rocket:"
    git push
    ```

> [!TIP]
> Using a **private repository**? Make sure to paste the public key from
> `github-deploy.key.pub` into the deploy keys section of your GitHub repository
> settings. This will make sure Flux has read/write access to your repository.

### Stage 6: Bootstrap Talos, Kubernetes, and Flux

> [!WARNING]
> It might take a while for the cluster to be setup (10+ minutes is normal).
> During which time you will see a variety of error messages like: "couldn't get
> current server API group list," "error: no matching resources found", etc.
> 'Ready' will remain "False" as no CNI is deployed yet. **This is normal.** If
> this step gets interrupted, e.g. by pressing <kbd>Ctrl</kbd> + <kbd>C</kbd>,
> you likely will need to [reset the cluster](#-reset) before trying again

1. Install Talos:

    ```sh
    just bootstrap talos
    ```

2. Push your changes to git:

    ```sh
    git add -A
    git commit -m "chore: add talhelper encrypted secret :lock:"
    git push
    ```

3. Install cilium, coredns, spegel, flux and sync the cluster to the repository
   state:

    ```sh
    just bootstrap apps
    ```

4. Watch the rollout of your cluster happen:

    ```sh
    kubectl get pods --all-namespaces --watch
    ```

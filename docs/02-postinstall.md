## 📣 Post installation

### ✅ Verifications

1. Check the status of Cilium:

    ```sh
    kubectl -n kube-system exec ds/cilium --container cilium-agent -- cilium status
    ```

2. Check the status of Flux and if the Flux resources are up-to-date and in a
   ready state:

    📍 _Run `just kube reconcile` to force Flux to sync your Git repository
    state_

    ```sh
    flux check
    flux get sources git flux-system
    flux get ks -A
    flux get hr -A
    ```

3. Check TCP connectivity to both the internal and external gateways:

    📍 _The variables are only placeholders, replace them with your actual
    values_

    ```sh
    nmap -Pn -n -p 443 ${gateways_internal} ${gateways_external} -vv
    ```

4. Check you can resolve DNS for `echo`, this should resolve to
   `${gateways_external}`:

    📍 _The variables are only placeholders, replace them with your actual
    values_

    ```sh
    dig @${gateways_dns} echo.${cloudflare_domain}
    ```

5. Check the status of your wildcard `Certificate`:

    ```sh
    kubectl -n network describe certificates
    ```

### 🌐 Public DNS

> [!TIP]
> Use the `envoy-external` gateway on `HTTPRoutes` to make applications public
> to the internet. These are also accessible on your private network once you
> set up split DNS.

The `external-dns` application created in the `network` namespace will handle
creating public DNS records. By default, `echo` and the `flux-webhook` are the
only subdomains reachable from the public internet. In order to make additional
applications public you must **set the correct gateway** like in the HelmRelease
for `echo`.

### 🏠 Home DNS

> [!TIP]
> Use the `envoy-internal` gateway on `HTTPRoutes` to make applications private
> to your network. If you're having trouble with internal DNS resolution check
> out [this](https://github.com/onedr0p/cluster-template/discussions/719) GitHub
> discussion.

`k8s_gateway` will provide DNS resolution to external Kubernetes resources (i.e.
points of entry to the cluster) from any device that uses your home DNS server.
For this to work, your home DNS server must be configured to forward DNS queries
for `${cloudflare_domain}` to `${gateways_dns}` instead of the upstream DNS
server(s) it normally uses. This is a form of **split DNS** (aka split-horizon
DNS / conditional forwarding).

_... Nothing working? That is expected, this is DNS after all!_

### 🪝 GitHub Webhook

By default Flux will periodically check your git repository for changes.
In-order to have Flux reconcile on `git push` you must configure GitHub to send
`push` events to Flux.

1. Obtain the webhook path:

    📍 _Hook id and path should look like
    `/hook/12ebd1e363c641dc3c2e430ecf3cee2b3c7a5ac9e1234506f6f5f3ce1230e123`_

    ```sh
    kubectl -n flux-system get receiver github-webhook --output=jsonpath='{.status.webhookPath}'
    ```

2. Piece together the full URL with the webhook path appended:

    ```text
    https://flux-webhook.${cloudflare_domain}/hook/12ebd1e363c641dc3c2e430ecf3cee2b3c7a5ac9e1234506f6f5f3ce1230e123
    ```

3. Navigate to the settings of your repository on GitHub, under
   "Settings/Webhooks" press the "Add webhook" button. Fill in the webhook URL
   and your token from `github-push-token.txt`, Content type:
   `application/json`, Events: Choose Just the push event, and save.

## 🧹 Tidy up

Once your cluster is fully configured and you no longer need to run
`just configure`, it's a good idea to clean up the repository by removing the
[template](./template) directory and any files related to the templating
process. This will help eliminate unnecessary clutter from the upstream template
repository and resolve any "duplicate registry" warnings from Renovate.

1. Tidy up your repository:

    ```sh
    just template tidy
    ```

2. Push your changes to git:

    ```sh
    git add -A
    git commit -m "chore: tidy up :broom:"
    git push
    ```

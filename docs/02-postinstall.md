# Post

I am working toward reducing the amount of things to look at post bootstrap so
that it gets the whole cluster going right away.

This cluster is IPv6 FIRST, but still dualstack supported for the most part.
There are certain aspects of the cluster that are currently broken due to lack
of innate dualstack support in applications. Often the applications are set to
only listen on IPv4 as the default, but can be tuned to be dualstack.

### ✅ Verifications

1. Check the status of Cilium:

    ```sh
    # repository external
    cilium status
    ```

    ```sh
    # pod internal
    kubectl -n kube-system exec ds/cilium --container cilium-agent -- cilium status
    ```

2. Check the status of Flux and its resources:

    📍 _Run `just kube::sync-all-*` to force Flux to sync your Git repository
    state_

    ```sh
    flux check
    flux get sources git flux-system
    flux get ks -A
    flux get hr -A
    ```

3. Check TCP connectivity to both the internal and external gateways:

    ```sh
    nmap -Pn -n -p 443 ${gateways_internal} ${gateways_external} -vv
    ```

4. Check DNS resolution for `echo`, this should resolve to
   `${gateways_external}`:

    ```sh
    dig @${gateways_dns} echo.${cloudflare_domain}
    ```

5. Check the wildcard `Certificate` status:

    ```sh
    kubectl -n network describe certificates
    ```

## External DNS

External dns is a kubernetes SIG project that looks at managing dns records
based off of kubernetes resource presence or lack of. It handles updates for
specific resources either via pre-set automatic ingest, or via annotation
watches. This cluster has two primary gateways that get used for dns resource
creation determination. The two gateway handle routing for internally sourced
traffic, and externally sourced traffic and have different routes for
applications.

### Cloudflare DNS

Applications that link to the external-gateway have their dns resources managed
in Cloudflare creating public DNS records. Resources in cloudflares dns
management are effectively CNAMED to a cloudflare tunnel running in cluster and
have their access granted that way.

### OPNSense DNS

Applications that link to the internal-gateway have the dns resources managed by
OPNSense. When running internally on the LAN, there is a rule that all of my
external domains never query upstream dns servers, instead, requests redirect to
`k8s_gateway` to provide DNS resolution. This is a form of split-horizon DNS.

## Flux

By default Flux checks watched git repositories for updates. It is also
configured to update on push, but this is not out of the box and requires
configuration.

### Git Webhook

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

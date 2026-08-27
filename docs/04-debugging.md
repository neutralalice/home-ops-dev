## 🐛 Debugging

General tips to debug issues

1. Check if the Flux resources are up-to-date and in a ready state:

    📍 _Run `just kube reconcile` to force Flux to sync your Git repository
    state_

    ```sh
    flux get sources git -A
    flux get ks -A
    flux get hr -A
    ```

2. Do you see the pod of the workload you are debugging:

    ```sh
    kubectl -n <namespace> get pods -o wide
    ```

3. Check the logs of the pod if it's there:

    ```sh
    kubectl -n <namespace> logs <pod-name> -f
    ```

4. If a resource exists, try to describe it to see what problems it might have:

    ```sh
    kubectl -n <namespace> describe <resource> <name>
    ```

5. Check the namespace events:

    ```sh
    kubectl -n <namespace> get events --sort-by='.metadata.creationTimestamp'
    ```

Resolving problems that you have could take some tweaking of your YAML manifests
in order to get things working, other times it could be a external factor like
permissions on a NFS server. If you are unable to figure out your problem see
the support sections below.

# 🐛 Debugging

General tips to debug issues

1. Check if the Flux resources are up-to-date and in a ready state:

    📍 _Run `just kube reconcile` to force Flux to sync your Git repository
    state_

    ```sh
    just kube::reconcile
    flux get sources git -A
    flux get ks -A
    flux get hr -A
    ```

2. Check if the pod is there:

    ```sh
    kubectl -n <namespace> get pods -o wide
    ```

- If it's not here check if there are replicaset or controller events

3. Check if the pod has logs:

    ```sh
    kubectl -n <namespace> logs <pod-name> -f
    ```

4. Check the resource events:

    ```sh
    kubectl -n <namespace> describe <resource> <name>
    ```

5. Check the namespace events:

    ```sh
    kubectl -n <namespace> get events --sort-by='.metadata.creationTimestamp'
    ```

## debugging volumes

1. make a custom profile for kubectl debug

```json
{ "volumeMounts": [{ "mountPath": "/config", "name": "config" }] }
```

2. launch the profile in target pod with

```sh
kubectl debug <pod> -n <ns> -it --profile=<base/restricted> --custom <json/yaml profile> --image=busybox --target=<targe>
```

## Privileged actions

If firmware needs to be updated, one of the possible ways to do it is with a
privileged pod

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: firmware-flash
    namespace: default
spec:
    hostNetwork: true
    hostIPC: true
    hostPID: true
    containers:
        - name: ubuntu
          image: ubuntu:26.04
          command: ["/bin/bash", "-c", "sleep infinity"]
          securityContext:
              privileged: true
              runAsUser: 0
              capabilities:
                  add: ["ALL"]
          volumeMounts:
              - mountPath: /host
                name: host-root
                readOnly: true
              - mountPath: /dev
                name: dev-devices
              - mountPath: /sys
                name: sys-bus
    volumes:
        - name: host-root
          hostPath:
              path: /
        - name: dev-devices
          hostPath:
              path: /dev
        - name: sys-bus
          hostPath:
              path: /sys
    nodeSelector:
        kubernetes.io/hostname: "k8s-01"
```

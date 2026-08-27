## 💥 Reset

> [!CAUTION]
> **Resetting** the cluster **multiple times in a short period of time** could
> lead to being **rate limited by DockerHub or Let's Encrypt**.

There might be a situation where you want to destroy your Kubernetes cluster.
The following command will reset your nodes back to maintenance mode.

```sh
just talos reset
```

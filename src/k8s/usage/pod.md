<!-- toc -->
# Kubernetes 的 Pod 操作技巧 

## Pod 的信息以环境变量的方式注入容器

参考 [Expose Pod Information to Containers Through Environment Variables][2]。

```yaml
env:
- name: PODIP
  valueFrom:
    fieldRef:
      fieldPath: status.podIP
```

## 在 Pod 内用 ServcieAccount 访问 APIServer

```sh
KUBE_TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -sSk -H "Authorization: Bearer $KUBE_TOKEN"  https://kubernetes.default.svc/api/v1/nodes/172.29.128.12/proxy/metrics/cadvisor
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://kubernetes.io/docs/tasks/inject-data-application/environment-variable-expose-pod-information/ "Expose Pod Information to Containers Through Environment Variables"

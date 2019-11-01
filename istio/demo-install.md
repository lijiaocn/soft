<!-- toc -->
# Istio 预览版部署

在 Kubernetes 集群中部署 istio 是最方便的，这里介绍 istio demo 的部署方法。

用这种方式部署的 istio 不能用于生产，只用来了解 istio 的用法和特性，参考 [Quick Start Evaluation Install][2]。

这里会使用前面下载 istio 的部署文件（见 [下载 istio](./install.md#下载-istio)）。

## 创建 istio 的 crd 

Istio 使用了大量的 CRD，用下面的命令在 kubernetes 集群中创建这些 CRD：

```sh
$ for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl apply -f $i; done
customresourcedefinition.apiextensions.k8s.io/virtualservices.networking.istio.io created
customresourcedefinition.apiextensions.k8s.io/destinationrules.networking.istio.io created
customresourcedefinition.apiextensions.k8s.io/serviceentries.networking.istio.io created
customresourcedefinition.apiextensions.k8s.io/gateways.networking.istio.io created
customresourcedefinition.apiextensions.k8s.io/envoyfilters.networking.istio.io created
customresourcedefinition.apiextensions.k8s.io/clusterrbacconfigs.rbac.istio.io created
customresourcedefinition.apiextensions.k8s.io/policies.authentication.istio.io created
customresourcedefinition.apiextensions.k8s.io/meshpolicies.authentication.istio.io created
customresourcedefinition.apiextensions.k8s.io/httpapispecbindings.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/httpapispecs.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/quotaspecbindings.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/quotaspecs.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/rules.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/attributemanifests.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/rbacconfigs.rbac.istio.io created
customresourcedefinition.apiextensions.k8s.io/serviceroles.rbac.istio.io created
customresourcedefinition.apiextensions.k8s.io/servicerolebindings.rbac.istio.io created
customresourcedefinition.apiextensions.k8s.io/adapters.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/instances.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/templates.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/handlers.config.istio.io created
customresourcedefinition.apiextensions.k8s.io/sidecars.networking.istio.io created
customresourcedefinition.apiextensions.k8s.io/authorizationpolicies.rbac.istio.io created
customresourcedefinition.apiextensions.k8s.io/clusterissuers.certmanager.k8s.io created
customresourcedefinition.apiextensions.k8s.io/issuers.certmanager.k8s.io created
customresourcedefinition.apiextensions.k8s.io/certificates.certmanager.k8s.io created
customresourcedefinition.apiextensions.k8s.io/orders.certmanager.k8s.io created
customresourcedefinition.apiextensions.k8s.io/challenges.certmanager.k8s.io created
```

## 部署 istio 

这里用到的 istio-demo.yaml/istio-demo-auth.yaml，istio 项目代码是没有的，它们在 istio 的 release 文件中，到 [istio/releases][5] 页面下载对应版本的 release 文件：

```sh
$ wget https://github.com/istio/istio/releases/download/1.2.5/istio-1.2.5-linux.tar.gz
$ tar -xvf istio-1.2.5-linux.tar.gz
$ ls istio-1.2.5/install/kubernetes/istio-demo*
istio-1.2.5/install/kubernetes/istio-demo-auth.yaml
istio-1.2.5/install/kubernetes/istio-demo.yaml
```

istio-demo.yaml 和 istio-demo-auth.yaml 是两种部署配置，后者强制加密 client 与 server 之间的通信。文件内容是 istio 的 deployment、config、crd 等，这两个文件巨大无比，接近两万行！

用下面的命令创建，可以看到在 kubernetes 中创建了大量的资源：

```sh
$ kubectl create -f istio-1.2.5/install/kubernetes/istio-demo.yaml
namespace/istio-system created
secret/kiali created
configmap/istio-galley-configuration created
configmap/istio-grafana-custom-resources created
configmap/istio-grafana-configuration-dashboards-galley-dashboard created
configmap/istio-grafana-configuration-dashboards-istio-mesh-dashboard created
configmap/istio-grafana-configuration-dashboards-istio-performance-dashboard created
configmap/istio-grafana-configuration-dashboards-istio-service-dashboard created
configmap/istio-grafana-configuration-dashboards-istio-workload-dashboard created
....省略...
```

## 部署完成

当 istio-system 中的所有 pod 都是 Running 状态时，部署完成：

```sh
$ kubectl get pods -n istio-system
NAME                                      READY   STATUS      RESTARTS   AGE
grafana-6575997f54-d8ltw                  1/1     Running     1          32m
istio-citadel-555dbdfd6b-kfd86            1/1     Running     1          32m
istio-cleanup-secrets-1.2.5-kv7jf         0/1     Completed   0          32m
istio-egressgateway-79f5b5b958-vr5hf      0/1     Running     1          32m
istio-galley-6855ffd77f-glwtd             1/1     Running     0          47s
istio-grafana-post-install-1.2.5-894k2    0/1     Completed   0          32m
istio-ingressgateway-585b9b66b8-9mwzs     0/1     Running     1          32m
istio-pilot-6d4dcbd54b-8p2fs              1/2     Running     2          32m
istio-policy-56588bf46d-8ft9v             2/2     Running     7          6m58s
istio-security-post-install-1.2.5-wzbkd   0/1     Completed   0          32m
istio-sidecar-injector-74f597fb84-p5n5p   1/1     Running     3          6m58s
istio-telemetry-76c5645cd9-k2r7n          2/2     Running     6          6m58s
istio-tracing-555cf644d-kcwll             1/1     Running     2          32m
kiali-6cd6f9dfb5-rvqg2                    1/1     Running     1          32m
prometheus-7d7b9f7844-hngc6               1/1     Running     4          32m
```

如果使用的 [minikube](../minikube/index.md) 创建的 kubernetes ，可以用 minikube 查看 istio 服务地址：

```sh
$ minkube service list
|---------------|------------------------|--------------------------------|
|   NAMESPACE   |          NAME          |              URL               |
|---------------|------------------------|--------------------------------|
| istio-system  | grafana                | No node port                   |
| istio-system  | istio-citadel          | No node port                   |
| istio-system  | istio-egressgateway    | No node port                   |
| istio-system  | istio-galley           | No node port                   |
| istio-system  | istio-ingressgateway   | http://192.168.99.100:31270    |
|               |                        | http://192.168.99.100:31380    |
|               |                        | http://192.168.99.100:31390    |
|               |                        | http://192.168.99.100:31400    |
|               |                        | http://192.168.99.100:31248    |
|               |                        | http://192.168.99.100:30079    |
|               |                        | http://192.168.99.100:30269    |
|               |                        | http://192.168.99.100:30249    |
|               |                        | http://192.168.99.100:31829    |
| istio-system  | istio-pilot            | No node port                   |
| istio-system  | istio-policy           | No node port                   |
| istio-system  | istio-sidecar-injector | No node port                   |
| istio-system  | istio-telemetry        | No node port                   |
| istio-system  | jaeger-agent           | No node port                   |
| istio-system  | jaeger-collector       | No node port                   |
| istio-system  | jaeger-query           | No node port                   |
| istio-system  | kiali                  | No node port                   |
| istio-system  | prometheus             | No node port                   |
| istio-system  | tracing                | No node port                   |
| istio-system  | zipkin                 | No node port                   |
| kube-system   | kube-dns               | No node port                   |
|---------------|------------------------|--------------------------------|
```

## 卸载

反向操作即可：

```sh
$ kubectl delete -f istio-1.2.5/install/kubernetes/istio-demo.yaml
$ for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl delete -f $i; done
```

## 参考

[1]: https://istio.io/docs/setup/kubernetes/ "Installing on Kubernetes"
[2]: https://istio.io/docs/setup/kubernetes/install/kubernetes/ "Quick Start Evaluation Install"
[3]: https://github.com/istio/istio/releases "istio/releases"
[4]: https://github.com/istio/istio/tree/1.2.5/install/kubernetes "1.2.5/install/kubernetes "
[5]: https://github.com/istio/istio/releases "istio/releases"

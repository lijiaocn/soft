<!-- toc -->
# Istio 预览版部署

在 Kubernetes 集群中部署 istio 是最方便的，这里介绍 istio demo 版本的部署方法。

用这种方式部署的 istio 不能用于生产，适合用来了解 istio  的用法和特性，参考文档 [Quick Start Evaluation Install][2]。

需要事先下载 istio 的部署文件，见 [下载 istio](./install.md#下载-istio)。

## 创建 istio 的 crd 

Istio 使用了大量的 CRD，用下面的命令在 kubernetes 集群中创建：

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

需要注意这里将要使用的 istio-demo.yaml 或者 istio-demo-auth.yaml，不在 istio repo 中，在 istio 的relase 文件中。到 [istio/releases][5] 页面下载对应版本的 release 文件：

```sh
$ wget https://github.com/istio/istio/releases/download/1.2.5/istio-1.2.5-linux.tar.gz
$ tar -xvf istio-1.2.5-linux.tar.gz
$ ls istio-1.2.5/install/kubernetes/istio-demo*
istio-1.2.5/install/kubernetes/istio-demo-auth.yaml
istio-1.2.5/install/kubernetes/istio-demo.yaml
```

istio-demo.yaml 和 istio-demo-auth.yaml 两个 istio 部署文件，后者强制加密 client 与 server 之间的通信。文件中是 istio 的 deployment、config、crd 等，比较麻烦的是这两个文件巨大无比，接近两万行！

执行下面命令创建，可以看到在 kubernetes 中创建了大量的资源：

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
configmap/istio-grafana-configuration-dashboards-mixer-dashboard created
configmap/istio-grafana-configuration-dashboards-pilot-dashboard created
configmap/istio-grafana created
configmap/kiali created
configmap/prometheus created
configmap/istio-security-custom-resources created
configmap/istio created
configmap/istio-sidecar-injector created
serviceaccount/istio-galley-service-account created
serviceaccount/istio-egressgateway-service-account created
serviceaccount/istio-ingressgateway-service-account created
serviceaccount/istio-grafana-post-install-account created
clusterrole.rbac.authorization.k8s.io/istio-grafana-post-install-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-grafana-post-install-role-binding-istio-system created
job.batch/istio-grafana-post-install-1.2.5 created
serviceaccount/kiali-service-account created
serviceaccount/istio-mixer-service-account created
serviceaccount/istio-pilot-service-account created
serviceaccount/prometheus created
serviceaccount/istio-cleanup-secrets-service-account created
clusterrole.rbac.authorization.k8s.io/istio-cleanup-secrets-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-cleanup-secrets-istio-system created
job.batch/istio-cleanup-secrets-1.2.5 created
serviceaccount/istio-security-post-install-account created
clusterrole.rbac.authorization.k8s.io/istio-security-post-install-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-security-post-install-role-binding-istio-system created
job.batch/istio-security-post-install-1.2.5 created
serviceaccount/istio-citadel-service-account created
serviceaccount/istio-sidecar-injector-service-account created
serviceaccount/istio-multi created
clusterrole.rbac.authorization.k8s.io/istio-galley-istio-system created
clusterrole.rbac.authorization.k8s.io/kiali created
clusterrole.rbac.authorization.k8s.io/kiali-viewer created
clusterrole.rbac.authorization.k8s.io/istio-mixer-istio-system created
clusterrole.rbac.authorization.k8s.io/istio-pilot-istio-system created
clusterrole.rbac.authorization.k8s.io/prometheus-istio-system created
clusterrole.rbac.authorization.k8s.io/istio-citadel-istio-system created
clusterrole.rbac.authorization.k8s.io/istio-sidecar-injector-istio-system created
clusterrole.rbac.authorization.k8s.io/istio-reader created
clusterrolebinding.rbac.authorization.k8s.io/istio-galley-admin-role-binding-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-kiali-admin-role-binding-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-mixer-admin-role-binding-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-pilot-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/prometheus-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-citadel-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-sidecar-injector-admin-role-binding-istio-system created
clusterrolebinding.rbac.authorization.k8s.io/istio-multi created
role.rbac.authorization.k8s.io/istio-ingressgateway-sds created
rolebinding.rbac.authorization.k8s.io/istio-ingressgateway-sds created
service/istio-galley created
service/istio-egressgateway created
service/istio-ingressgateway created
service/grafana created
service/kiali created
service/istio-policy created
service/istio-telemetry created
service/istio-pilot created
service/prometheus created
service/istio-citadel created
service/istio-sidecar-injector created
deployment.apps/istio-galley created
deployment.apps/istio-egressgateway created
deployment.apps/istio-ingressgateway created
deployment.apps/grafana created
deployment.apps/kiali created
deployment.apps/istio-policy created
deployment.apps/istio-telemetry created
deployment.apps/istio-pilot created
deployment.apps/prometheus created
deployment.apps/istio-citadel created
deployment.apps/istio-sidecar-injector created
deployment.apps/istio-tracing created
service/jaeger-query created
service/jaeger-collector created
service/jaeger-agent created
service/zipkin created
service/tracing created
mutatingwebhookconfiguration.admissionregistration.k8s.io/istio-sidecar-injector created
poddisruptionbudget.policy/istio-galley created
poddisruptionbudget.policy/istio-egressgateway created
poddisruptionbudget.policy/istio-ingressgateway created
poddisruptionbudget.policy/istio-policy created
poddisruptionbudget.policy/istio-telemetry created
poddisruptionbudget.policy/istio-pilot created
poddisruptionbudget.policy/istio-sidecar-injector created
attributemanifest.config.istio.io/istioproxy created
attributemanifest.config.istio.io/kubernetes created
handler.config.istio.io/stdio created
instance.config.istio.io/accesslog created
instance.config.istio.io/tcpaccesslog created
rule.config.istio.io/stdio created
rule.config.istio.io/stdiotcp created
instance.config.istio.io/requestcount created
instance.config.istio.io/requestduration created
instance.config.istio.io/requestsize created
instance.config.istio.io/responsesize created
instance.config.istio.io/tcpbytesent created
instance.config.istio.io/tcpbytereceived created
instance.config.istio.io/tcpconnectionsopened created
instance.config.istio.io/tcpconnectionsclosed created
handler.config.istio.io/prometheus created
rule.config.istio.io/promhttp created
rule.config.istio.io/promtcp created
rule.config.istio.io/promtcpconnectionopen created
rule.config.istio.io/promtcpconnectionclosed created
handler.config.istio.io/kubernetesenv created
rule.config.istio.io/kubeattrgenrulerule created
rule.config.istio.io/tcpkubeattrgenrulerule created
instance.config.istio.io/attributes created
destinationrule.networking.istio.io/istio-policy created
destinationrule.networking.istio.io/istio-telemetry created
```

## 部署完成

istio-system 中的所有 pod 都是 Running 状态时，部署完成：

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

如果是在用 [minikube](../minikube/index.md) 部署的 kubernetes 中部署的，可以用 minikube 查看 istio 的服务：

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

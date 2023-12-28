<!-- toc -->
# Kubernetes 的用户交互 API

用户可以向 kubernetes 提交各种类型的任务和数据，我们把 kubernetes 接受的任务或数据类型称为 API。经过这几年的飞速发展，API 的种类非常丰富。

最常见的 API：[concepts][2]、[tasks][3]。

API 完整列表：[api-index][4]。

在 kubernetes 项目中，API 的定义代码位于 [staging/src/k8s.io/api][6]，staging/src/k8s.io/api 已经作为独立项目发布，通过 [k8s.io/api][7] 引用。

## API 的版本与作用域

Resource 是有版本的：

```yaml
apiVersion: v1
kind: Namespace
...
apiVersion: extensions/v1beta1
kind: DaemonSet
```

有一些 resource 是 cluster 级别的，不属于任何 namespace，全局有效，后面的 cluster apis。

## CLUSTER APIS

```sh
APIService v1 apiregistration.k8s.io
AuditSink v1alpha1 auditregistration.k8s.io
Binding v1 core
CertificateSigningRequest v1beta1 certificates.k8s.io
ClusterRoleBinding v1 rbac.authorization.k8s.io
ComponentStatus v1 core
Lease v1 coordination.k8s.io
LocalSubjectAccessReview v1 authorization.k8s.io
Namespace v1 core
Node v1 core
PersistentVolume v1 core
ResourceQuota v1 core
Role v1 rbac.authorization.k8s.io
RoleBinding v1 rbac.authorization.k8s.io
RuntimeClass v1beta1 node.k8s.io
SelfSubjectAccessReview v1 authorization.k8s.io
SelfSubjectRulesReview v1 authorization.k8s.io
ServiceAccount v1 core
SubjectAccessReview v1 authorization.k8s.io
TokenRequest v1 authentication.k8s.io
TokenReview v1 authentication.k8s.io
NetworkPolicy v1 networking.k8s.io
```

## WORKLOADS APIS

```sh
Container v1 core
CronJob v1beta1 batch
DaemonSet v1 apps
Deployment v1 apps
Job v1 batch
Pod v1 core
ReplicaSet v1 apps
ReplicationController v1 core
StatefulSet v1 apps
```

## SERVICE APIS

```sh
Endpoints v1 core
EndpointSlice v1alpha1 discovery.k8s.io
Ingress v1beta1 networking.k8s.io
Service v1 core
```

## CONFIG AND STORAGE APIS

```sh
ConfigMap v1 core
CSIDriver v1beta1 storage.k8s.io
CSINode v1beta1 storage.k8s.io
Secret v1 core
PersistentVolumeClaim v1 core
StorageClass v1 storage.k8s.io
Volume v1 core
VolumeAttachment v1 storage.k8s.io
```

## METADATA APIS

```sh
ControllerRevision v1 apps
CustomResourceDefinition v1 apiextensions.k8s.io
Event v1 core
LimitRange v1 core
HorizontalPodAutoscaler v1 autoscaling
MutatingWebhookConfiguration v1 admissionregistration.k8s.io
ValidatingWebhookConfiguration v1 admissionregistration.k8s.io
PodTemplate v1 core
PodDisruptionBudget v1beta1 policy
PriorityClass v1 scheduling.k8s.io
PodPreset v1alpha1 settings.k8s.io
PodSecurityPolicy v1beta1 policy
```

## 参考

1. [李佶澳的博客][1]

[1]: https://www.lijiaocn.com "李佶澳的博客"
[2]: https://kubernetes.io/docs/concepts/ "concepts"
[3]: https://kubernetes.io/docs/tasks/ "tasks"
[4]: https://kubernetes.io/docs/reference/kubernetes-api/api-index/ "api-index"
[5]: https://github.com/kubernetes/kubernetes/tree/master/pkg/apis  "kubernetes/pkg/apis"
[6]: https://github.com/kubernetes/kubernetes/tree/v1.16.3/staging/src/k8s.io/api "staging/src/k8s.io/api"
[7]: https://github.com/kubernetes/api "kubernetes/api"

## ReplicaSet



### ReplicaSet 作用

ReplicaSet 的目的是维护一组在任何时候都处于运行状态的 Pod 副本的稳定集合。 因此，它通常用来保证给定数量的、完全相同的 Pod 的可用性。



### ReplicaSet 的工作原理

RepicaSet 是通过一组字段来定义的，包括一个用来识别可获得的 Pod 的集合的选择算符、一个用来标明应该维护的副本个数的数值、一个用来指定应该创建新 Pod 以满足副本个数条件时要使用的 Pod 模板等等。

ReplicaSet 通过 Pod 上的 [metadata.ownerReferences](https://kubernetes.io/zh/docs/concepts/workloads/controllers/garbage-collection/#owners-and-dependents) 字段连接到附属 Pod，该字段给出当前对象的属主资源。 ReplicaSet 所获得的 Pod 都在其 ownerReferences 字段中包含了属主 ReplicaSet 的标识信息。正是通过这一连接，ReplicaSet 知道它所维护的 Pod 集合的状态， 并据此计划其操作行为。



### 何时使用 ReplicaSet

ReplicaSet 确保任何时间都有指定数量的 Pod 副本在运行。 然而，Deployment 是一个更高级的概念，它管理 ReplicaSet，并向 Pod 提供声明式的更新以及许多其他有用的功能。 因此，我们建议使用 Deployment 而不是直接使用 ReplicaSet，除非 你需要自定义更新业务流程或根本不需要更新。

这实际上意味着，你可能永远不需要操作 ReplicaSet 对象：而是使用 Deployment，并在 spec 部分定义你的应用。



## ReplicaSet 的替代方案

### Deployment （推荐）

[`Deployment`](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/) 是一个 可以拥有 ReplicaSet 并使用声明式方式在服务器端完成对 Pods 滚动更新的对象。 尽管 ReplicaSet 可以独立使用，目前它们的主要用途是提供给 Deployment 作为 编排 Pod 创建、删除和更新的一种机制。当使用 Deployment 时，你不必关心 如何管理它所创建的 ReplicaSet，Deployment 拥有并管理其 ReplicaSet。 因此，建议你在需要 ReplicaSet 时使用 Deployment。

### 裸 Pod

与用户直接创建 Pod 的情况不同，ReplicaSet 会替换那些由于某些原因被删除或被终止的 Pod，例如在节点故障或破坏性的节点维护（如内核升级）的情况下。 因为这个原因，我们建议你使用 ReplicaSet，即使应用程序只需要一个 Pod。 想像一下，ReplicaSet 类似于进程监视器，只不过它在多个节点上监视多个 Pod， 而不是在单个节点上监视单个进程。 ReplicaSet 将本地容器重启的任务委托给了节点上的某个代理（例如，Kubelet 或 Docker）去完成。
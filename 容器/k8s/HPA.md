### HPA 

hpa 扩缩容灵敏度：https://www.cnblogs.com/tencent-cloud-native/p/14245238.html



### HPA metric server 安装

根据集群版本（kubectl version)找到适配的metric server版本

wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.6.1/components.yaml

| Metrics Server | Metrics API group/version | Supported Kubernetes version |
| -------------- | ------------------------- | ---------------------------- |
| 0.6.x          | metrics.k8s.io/v1beta1    | *1.19+                       |
| 0.5.x          | metrics.k8s.io/v1beta1    | *1.8+                        |
| 0.4.x          | metrics.k8s.io/v1beta1    | *1.8+                        |
| 0.3.x          | metrics.k8s.io/v1beta1    | 1.8-1.21                     |

**修改kubernetes参数**

--**enable-aggregator-routing**=true

ps -ef | grep apiserver |grep enable-aggregator-routing

有输出则不用关注

**修改参数部署metrics-server**

*低于 v1.16 的 Kubernetes 版本需要传递`--authorization-always-allow-paths=/livez,/readyz`命令行标志

```plain
[root@k8s-master ~]# vim components.yaml
132 spec:
133     containers:
134      - args:
135      - --cert-dir=/tmp
136      - --secure-port=4443
137      - --kubelet-preferred-address-types=InternalIP
138      - --kubelet-use-node-status-port
139      - --metric-resolution=15s
140      - --kubelet-insecure-tls #不经过认证
141      image: bitnami/metrics-server:0.6.1

[root@k8s-master ~]# kubectl apply -f components.yaml
```

部署完毕后结果验证

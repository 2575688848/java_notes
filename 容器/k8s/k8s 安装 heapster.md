1、下载安装

```
wget https://github.com/kubernetes-sigs/metrics-server/archive/v0.3.6.tar.gz
tar -zxvf v0.3.6.tar.gz
cd metrics-server-0.3.6/deploy/1.8+/
# 添加配置，见下图
vi metrics-server-deployment.yaml
kubectl  apply -f ./
```



metrics-server-deployment.yaml 添加的配置如下：

```
        image: rancher/metrics-server:v0.3.6 #修改镜像
        imagePullPolicy: Always
        command:              # 添加这几行
        - /metrics-server   
        - --metric-resolution=30s
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP,Hostname,InternalDNS,ExternalDNS,ExternalIP
        volumeMounts:
        - name: tmp-dir
          mountPath: /tmp
```



由于网络原因，导致默认配置中的镜像版本 k8s.gcr.io/metrics-server-amd64:v0.3.6 拉取失败，故修改镜像地址为 rancher/metrics-server:v0.3.6 。

2、问题验证

```
[root@beta-test-v4-dubbo02 1.8+]# kubectl  top node
NAME                   CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
beta-autoceshi-1       3134m        78%    2110Mi          13%       
beta-test-v4-dubbo02   310m         31%    1814Mi          47%       
beta-vediocloud2-1     235m         5%     1437Mi          9%        
beta-vediocloud2-2     186m         4%     818Mi           5%        
beta-vediocloud2-3     270m         6%     3147Mi          19%       
beta-vediocloudddd-1   208m         5%     1522Mi          9%        
beta-vediocloudddd-2   228m         5%     3867Mi          24%       
beta-vediocloudddd-3   270m         6%     2171Mi          13%       
beta-vediocloudddd-4   190m         4%     1436Mi          9%        
beta-vediocloudddd-5   3578m        89%    2378Mi          14%       
beta-vediocloudddd-6   228m         5%     2011Mi          12%       
beta-vediocloudddd-7   191m         4%     2139Mi          13%       
beta-vediocloudddd-9   150m         3%     1822Mi          11% 
```
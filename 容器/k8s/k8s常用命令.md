### 常用命令

#### 查看

kubectl get deployments -n ns1 ：	查看创建的部署

kubectl get rs ： 								   查看副本集（一个副本对应一个 pod）

kubectl get pod -n ns1 -o wide | grep v2x：  查看 pod

kubectl logs -f pod名 -n ns1

kubectl get svc(services) -n ns1 | grep mysql (查看 svc 信息)

kubectl describe pods kube-amd64-2d6tb -n ns1 : 					查看 pod 信息

kubectl describe deployments ns1-v2x-websocket -n ns1：	 查看 deployment 信息

kubectl get deployment ns1-v2x-test -o yaml -n ns1:				  查看 deployment yml

kubectl rollout status deployment/nginx-deployment -n ns1： 		  查看部署的状态，输出类似于（deployment "nginx-deployment" successfully rolled out）



#### 删除

在k8s master 节点：

kubectl delete pod -n ns1 上一步得到的pod名（使用此命令 delete 后，k8s 会重新创建一个 pod）

kubectl delete deployment ns1-v2x-vehicle-access -n ns1（删除部署，只有删除部署才是真正的删除）



#### 交互操作

** 进入 pod 内部

kubectl exec -it -n ns1 mariadb-master-0 /bin/bash



**  导入数据库数据

kubectl exec -i -n ns1 数据库pod名 -- mysql -u用户名 -p密码 数据库名 < sql文件.sql



** 导出数据库
kubectl exec -it -n ns1 mariadb-master-0 -- mysqldump -uroot -prootPassword v2x_platform > v2x_platform.sql;
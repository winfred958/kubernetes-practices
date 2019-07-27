# 1. 下载 yaml 配置文件 apply
 - [官网教程](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/#deploying-the-dashboard-ui)
```
curl -O https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta1/aio/deploy/recommended.yaml
mv recommended.yaml kubernetes-dashboard.yaml
kubectl apply -f kubernetes-dashboard.yaml
```
# 2. 添加用户
 - [Creating-sample-user](https://github.com/kubernetes/dashboard/wiki/Creating-sample-user)
 
 创建用户配置文件 kubernetes-dashboard-adminuser.yaml
 ```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: kube-system
```
kubectl apply -f kubernetes-dashboard-adminuser.yaml

# 3. 生成token
 - 生成token
 ```
 kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
 ```
  - 使用token登录
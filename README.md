# treafik-profile
由于 Traefik 2.X 版本和之前的 1.X 版本不兼容，我们这里选择功能更加强大的 2.X 版本来和大家进行讲解，我们这里使用的镜像是 traefik:2.1.1。

在 Traefik 中的配置可以使用两种不同的方式：

动态配置：完全动态的路由配置
静态配置：启动配置
静态配置中的元素（这些元素不会经常更改）连接到 providers 并定义 Treafik 将要监听的 entrypoints。

在 Traefik 中有三种方式定义静态配置：在配置文件中、在命令行参数中、通过环境变量传递

动态配置包含定义系统如何处理请求的所有配置内容，这些配置是可以改变的，而且是无缝热更新的，没有任何请求中断或连接损耗。

安装 Traefik 到 Kubernetes 集群中的资源清单文件我这里提前准备好了，直接执行下面的安装命令即可：
```
$ kubectl apply -f https://www.qikqiak.com/k8strain/network/manifests/traefik/crd.yaml
$ kubectl apply -f https://www.qikqiak.com/k8strain/network/manifests/traefik/rbac.yaml
$ kubectl apply -f https://www.qikqiak.com/k8strain/network/manifests/traefik/deployment.yaml
$ kubectl apply -f https://www.qikqiak.com/k8strain/network/manifests/traefik/dashboard.yaml
```
其中 deployment.yaml 我这里是固定到 master 节点上的，如果你需要修改可以下载下来做相应的修改即可。我们这里是通过命令行参数来做的静态配置：

```
args:
- --entryPoints.web.address=:80
- --entryPoints.websecure.address=:443
- --api=true
- --api.dashboard=true
- --ping=true
- --providers.kubernetesingress
- --providers.kubernetescrd
- --log.level=INFO
- --accesslog
```
  其中前两项配置是来定义 web 和 websecure 这两个入口点的，--api=true 开启,=，就会创建一个名为 api@internal 的特殊 service，在 dashboard 中可以直接使用这个 service 来访问，然后其他比较重要的就是开启 kubernetesingress 和 kubernetescrd 这两个 provider。

dashboard.yaml 中定义的是访问 dashboard 的资源清单文件，可以根据自己的需求修改。

```
$ kubectl get pods -n kube-system                       
NAME                                  READY   STATUS    RESTARTS   AGE
traefik-867bd6b9c-lbrlx               1/1     Running   0          6m17s
......
$ kubectl get ingressroute
NAME                 AGE
traefik-dashboard    30m
```
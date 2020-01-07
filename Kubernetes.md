## Kubernetes Learning

### 创建一个k8s集群
- 查看 minikube 版本，检查其是否已安装
```
$ minikube version

minikube version: v1.6.2
commit: 54f28ac5d3a815d1196cd5d57d707439ee4bb392
```
- 启动 k8s 集群
```
$ minikube start

* minikube v1.6.2 on Ubuntu 18.04
* Selecting 'none' driver from user configuration (alternates: [])
* Running on localhost (CPUs=2, Memory=2461MB, Disk=47990MB) ...
* OS release is Ubuntu 18.04.3 LTS
* Preparing Kubernetes v1.17.0 on Docker '18.09.7' ...
  - kubelet.resolv-conf=/run/systemd/resolve/resolv.conf
* Pulling images ...
* Launching Kubernetes ...
* Configuring local host environment ...
* Waiting for cluster to come online ...
* Done! kubectl is now configured to use "minikube"
```
至此，Minikube 已为你启动了一个虚拟机，且k8s集群已在该虚拟机中运行。

- 集群版本，检查 kubectl 是否安装
```
$ kubectl version

Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.0", GitCommit:"70132b0f130acc0bed193d9ba59dd186f0e634cf", GitTreeState:"clean", BuildDate:"2019-12-07T21:20:10Z", GoVersion:"go1.13.4", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.0", GitCommit:"70132b0f130acc0bed193d9ba59dd186f0e634cf", GitTreeState:"clean", BuildDate:"2019-12-07T21:12:17Z", GoVersion:"go1.13.4", Compiler:"gc", Platform:"linux/amd64"}
```
Client Version 是kubectl版本，Server Version 是主安装Kubernetes版本

- 集群详细信息
```
$ kubectl cluster-info

Kubernetes master is running at https://172.17.0.56:8443
KubeDNS is running at https://172.17.0.56:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

- 查看集群中的Nodes节点
```
$ kubectl get nodes

NAME       STATUS   ROLES    AGE    VERSION
minikube   Ready    master   119s   v1.17.0
```
显示只有一个节点，且可以看到其状态为 ready （准备接受要部署的应用程序），Kubernetes将根据Node可用资源选择将我们的应用程序部署到何处

### k8s部署 - Kubernetes Deployments
拥有运行中的Kubernetes集群后，您可以在其之上部署容器化的应用程序。为此，您将创建一个Kubernetes部署配置。部署指示Kubernetes如何创建和更新应用程序实例。创建部署后，Kubernetes主服务器将提到的应用程序实例调度到集群中的各个节点上。

创建应用程序实例后，Kubernetes部署控制器将持续监视这些实例。如果承载实例的节点发生故障或被删除，则部署控制器将实例替换为群集中另一个节点上的实例。这提供了一种自我修复机制来解决机器故障或维护。

创建部署时，需要为应用程序指定容器映像以及要运行的副本数（replicas）。您可以稍后通过更新部署来更改该信息。

- 部署APP

通过 ```kubectl creat deployment``` 命令行部署app。需要提供部署名称和应用程序映像位置（包括Docker Hub外部托管的映像的完整存储库URL）。

```
$ kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1

deployment.apps/kubernetes-bootcamp created
```
这个操作执行了一下几件事：
> 搜索可以在其中运行应用程序实例的合适节点（示例中我们只有1个可用节点）

> 安排应用程序在该节点上运行

> 配置集群以在需要时在新节点上重新安排实例

- 列出部署的 app

```
$ kubectl get deployments

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1/1     1            1           13m
```

- 查看app应用

Kubernetes内部运行的Pod在私有的隔离网络上运行。默认情况下，它们在同一kubernetes集群中的其他Pod和服务中可见，但在该网络外部不可见。当使用```kubectl```时，我们通过API端点进行交互以与我们的应用程序进行通信。

kubectl命令可以创建一个代理，该代理会将通信转发到群集范围的专用网络中。可以通过按Ctrl-C终止代理，并且在运行时不显示任何输出。

开另一个terminal，运行：
```
$ kubectl proxy

Starting to serve on 127.0.0.1:8001
```

```
$ curl http://localhost:8001/version

{
  "major": "1",
  "minor": "15",
  "gitVersion": "v1.15.0",
  "gitCommit": "e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529",
  "gitTreeState": "clean",
  "buildDate": "2019-06-19T16:32:14Z",
  "goVersion": "go1.12.5",
  "compiler": "gc",
  "platform": "linux/amd64"
}
```

为了在不使用 Proxy（代理）的情况下可以访问新部署，可以通过 Service（服务）实现。
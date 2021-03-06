---
title: Istio流量管理之TCP流量转移
author: olzhy
type: post
date: 2020-12-26T08:48:52+08:00
url: /posts/istio-tcp-traffic-shifting.html
categories:
  - 计算机
tags:
  - 工具使用
keywords:
  - 工具使用
  - 服务网格
  - Service Mesh
  - 云原生
  - Istio
description: Istio流量管理之TCP流量转移 (TCP Traffic Shifting of Istio Traffic Management)

---
在上文“[Istio流量管理之流量转移](https://olzhy.github.io/posts/istio-traffic-shifting.html)”中，我们使用Istio为7层HTTP应用作了流量按比例分配测试。本文使用Istio自带的tcp-echo样例对4层TCP应用作一下测试。

关于Istio安装等环境准备，请参阅“[Istio安装使用](https://olzhy.github.io/posts/istio-get-started.html)”。

### 1 tcp-echo源码解析

tcp-echo是一个4层应用。其启动后会一直监听所暴露的端口，并等待TCP连接，连接成功后提供ping/pong请求响应。从源码可以看到，其接收到一串字符后会拼上一个前缀并返回给客户端。

[main.go](https://github.com/istio/istio/blob/master/samples/tcp-echo/src/main.go#L63)

```go
func main() {...}

func serve(addr, prefix string) {...}

func handleConnection(conn net.Conn, prefix string) {
	defer conn.Close()
	reader := bufio.NewReader(conn)
	for {
		// read client request data
		bytes, err := reader.ReadBytes(byte('\n'))
		...
		// prepend prefix and send as response
		line := fmt.Sprintf("%s %s", prefix, bytes)
		conn.Write([]byte(line))
	}
}
```

下面我们在本地启动运行一下该程序。暴露端口为9000，前缀为“hello”。

```shell
$ go run main.go 9000 hello

listening on [::]:9000, prefix: hello
```

服务端起来了，我们使用nc命令发起TCP连接请求并发送字符串“world”。

```shell
$ nc localhost 9000

world
hello world
```

可以看到服务端拼接了前缀“hello”，返回“hello world”。

本地测试完成，下面我们尝试使用Istio samples文件夹下自带的部署文件将其部署到Docker Desktop Kubernetes集群。

### 2 tcp-echo Kubernetes部署

使用samples文件夹下自带的tcp-echo描述文件将其部署至Kubernetes集群。

```shell
$ cd /usr/local/istio-1.8.1
$ kubectl apply -n istio-demo -f samples/tcp-echo/tcp-echo-services.yaml
```

可以看到，该部署文件有两个Deployment，对应两个版本的tcp-echo，版本v1的输出前缀为“one”，版本v2的输出前缀为“two”。每个Deployment暴露两个端口9000与9001，通过同一个Service对外提供服务。访问Service时，会轮训两个版本的tcp-echo。

[samples/tcp-echo/tcp-echo-services.yaml](https://raw.githubusercontent.com/istio/istio/release-1.8/samples/tcp-echo/tcp-echo-services.yaml)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: tcp-echo
  labels:
    app: tcp-echo
    service: tcp-echo
spec:
  ports:
  - name: tcp
    port: 9000
  - name: tcp-other
    port: 9001
  selector:
    app: tcp-echo
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tcp-echo-v1
  labels:
    app: tcp-echo
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tcp-echo
      version: v1
  template:
    metadata:
      labels:
        app: tcp-echo
        version: v1
    spec:
      containers:
      - name: tcp-echo
        image: docker.io/istio/tcp-echo-server:1.2
        imagePullPolicy: IfNotPresent
        args: [ "9000,9001,9002", "one" ]
        ports:
        - containerPort: 9000
        - containerPort: 9001
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tcp-echo-v2
  labels:
    app: tcp-echo
    version: v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tcp-echo
      version: v2
  template:
    metadata:
      labels:
        app: tcp-echo
        version: v2
    spec:
      containers:
      - name: tcp-echo
        image: docker.io/istio/tcp-echo-server:1.2
        imagePullPolicy: IfNotPresent
        args: [ "9000,9001,9002", "two" ]
        ports:
        - containerPort: 9000
        - containerPort: 9001
```

tcp-echo部署完成，因为我们需要一个带nc命令的Pod来测试tcp-echo。所以下面部署一下Istio自带的sleep应用，该应用包含基础命名curl、nc等，就是用来辅助我们做测试的。

```shell
$ cd /usr/local/istio-1.8.1
$ kubectl apply -n istio-demo -f samples/sleep/sleep.yaml
```

部署完成，进入sleep Pod，执行测试命令。

```shell
$ kubectl exec -ti sleep-854565cb79-pcgjv -c sleep -n istio-demo -- sh -c 'for i in $(seq 1 10); do echo hello | nc tcp-echo 9000; done'

two hello
two hello
one hello
one hello
two hello
two hello
one hello
one hello
two hello
two hello
```

请求tcp-echo 10次，前缀有时为“one”，有时为“two”，说明有时请求到版本v1，有时请求到版本v2。

因Kubernetes无法做流量按比例分配，下面使用Istio来尝试实现一下。

### 3 使用Istio对tcp-echo作流量分配

使用Istio自带的描述文件为tcp-echo配置Gateway，Virtual Service，Destination Rule。

描述文件[samples/tcp-echo/tcp-echo-all-v1.yaml](https://raw.githubusercontent.com/istio/istio/release-1.8/samples/tcp-echo/tcp-echo-all-v1.yaml)内容如下，tcp-echo会通过Gateway以31400端口提供v1版本的TCP服务。

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: tcp-echo-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 31400
      name: tcp
      protocol: TCP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: tcp-echo-destination
spec:
  host: tcp-echo
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: tcp-echo
spec:
  hosts:
  - "*"
  gateways:
  - tcp-echo-gateway
  tcp:
  - match:
    - port: 31400
    route:
    - destination:
        host: tcp-echo
        port:
          number: 9000
        subset: v1
```

应用该配置文件：

```shell
$ kubectl apply -n istio-demo -f samples/tcp-echo/tcp-echo-all-v1.yaml
```

查看Gateway外部访问IP及端口，本文使用Docker Desktop内置Kubernetes，所以外部访问IP即为localhost。

```shell
$ kubectl get service/istio-ingressgateway -n istio-system
```

以Gateway地址请求tcp-echo 10次，发现输出前缀均为“one”，命令如下。

```shell
$ for i in $(seq 1 10); do echo hello | nc localhost 31400; done

one hello
one hello
one hello
one hello
one hello
one hello
one hello
one hello
one hello
one hello
```

下面尝试将80%的流量打到v1，20%的流量打到v2。描述文件[samples/tcp-echo/tcp-echo-20-v2.yaml](https://raw.githubusercontent.com/istio/istio/release-1.8/samples/tcp-echo/tcp-echo-20-v2.yaml)内容如下：

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: tcp-echo
spec:
  hosts:
  - "*"
  gateways:
  - tcp-echo-gateway
  tcp:
  - match:
    - port: 31400
    route:
    - destination:
        host: tcp-echo
        port:
          number: 9000
        subset: v1
      weight: 80
    - destination:
        host: tcp-echo
        port:
          number: 9000
        subset: v2
      weight: 20
```

应用配置文件命令如下：

```shell
$ kubectl apply -n istio-demo -f samples/tcp-echo/tcp-echo-20-v2.yaml
```

然后再次以Gateway地址请求tcp-echo 10次，发现前缀大概率为“one”，命令如下。

```shell
$ for i in $(seq 1 10); do echo hello | nc localhost 31400; done

two hello
one hello
two hello
one hello
one hello
one hello
one hello
one hello
one hello
one hello
```

测试结束，使用如下命令删除sleep，tcp-echo应用，及路由配置。

```shell
$ kubectl delete -n istio-demo -f samples/tcp-echo/tcp-echo-services.yaml
$ kubectl delete -n istio-demo -f samples/sleep/sleep.yaml
$ kubectl delete -n istio-demo -f samples/tcp-echo/tcp-echo-all-v1.yaml
```

总结本文，首先介绍了Istio除了作7层流量转移外，还支持4层流量转移。然后对tcp-echo样例分别进行了本地测试，Kubernetes部署，及Istio流量转移测试。


> 参考资料
>
> [1] [Istio TCP Traffic Shifting](https://istio.io/latest/docs/tasks/traffic-management/tcp-traffic-shifting/)
>
> [2] [Istio Traffic Management](https://istio.io/latest/docs/concepts/traffic-management/)
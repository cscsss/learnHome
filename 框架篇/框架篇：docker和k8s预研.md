
# 一、 docker简介
## 环境配置
软件开发最大的麻烦事之一，就是环境配置。用户计算机的环境都不相同，你怎么知道自家的软件，能在那些机器跑起来？

用户必须保证两件事：操作系统的设置，各种库和组件的安装。只有它们都正确，软件才能运行。举例来说，安装一个 Python 应用，计算机必须有 Python 引擎，还必须有各种依赖，可能还要配置环境变量。

如果某些老旧的模块与当前环境不兼容，那就麻烦了。开发者常常会说："它在我的机器可以跑了"（It works on my machine），言下之意就是，其他机器很可能跑不了

## 虚拟机（virtual machine）
-  就是带环境安装的一种解决方案。它可以在一种操作系统里面运行另一种操作系统，比如在 Windows 系统里面运行 Linux 系统。应用程序对此毫无感知，因为虚拟机看上去跟真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通文件，不需要了就删掉，对其他部分毫无影响。虽然用户可以通过虚拟机还原软件的原始环境。但是，这个方案有几个缺点。
- （1）资源占用多
虚拟机会独占一部分内存和硬盘空间。它运行的时候，其他程序就不能使用这些资源了。哪怕虚拟机里面的应用程序，真正使用的内存只有 1MB，虚拟机依然需要几百 MB 的内存才能运行。
- （2）冗余步骤多
虚拟机是完整的操作系统，一些系统级别的操作步骤，往往无法跳过，比如用户登录
- （3）启动慢
启动操作系统需要多久，启动虚拟机就需要多久。可能要等几分钟，应用程序才能真正运行
## Linux 容器
由于虚拟机存在这些缺点，Linux 发展出了另一种虚拟化技术：Linux 容器（Linux Containers，缩写为 LXC）。
- Linux 容器不是模拟一个完整的操作系统，而是对进程进行隔离。或者说，在正常进程的外面套了一个保护层。对于容器里面的进程来说，它接触到的各种资源都是虚拟的，从而实现与底层系统的隔离。由于容器是进程级别的，相比虚拟机有很多优势。
- （1）启动快
容器里面的应用，直接就是底层系统的一个进程，而不是虚拟机内部的进程。所以，启动容器相当于启动本机的一个进程，而不是启动一个操作系统，速度就快很多。
- （2）资源占用少
容器只占用需要的资源，不占用那些没有用到的资源；虚拟机由于是完整的操作系统，不可避免要占用所有资源。另外，多个容器可以共享资源，虚拟机都是独享资源。
- （3）体积小
容器只要包含用到的组件即可，而虚拟机是整个操作系统的打包，所以容器文件比虚拟机文件要小很多。

## Docker
-  docker属于 Linux 容器的一种封装，使用go语言开发，基于Linux内核的cgroups，namespace，Union FS等技术，对应用进程进行封装隔离，并且独立于宿主机与其他进程。

Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker，就不用担心环境问题。

总体来说，Docker 的接口相当简单，用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样

# 二、docker带来的好处（解决了什么问题）
-  （1）职责的逻辑分离
使用docker，开发人员只需要关心容器中运行的应用程序，而运维人员只需要关心如何管理容器。Docker设计的目的就是加强开发人员写代码的开发环境与应用程序要部署的生产环境的一致性，从而降低那种“开发时一切正常，肯定是运维的问题”的风险。
-  （2）快速高效的开发生命周期
Docker 的目标之一就是缩短代码从开发，测试到部署，上线运行的周期，让你的应用程序具备可移植性，易于构建，并易于协作。
-  （3）响应式部署和扩展
Docker 是基于容器的平台，允许高度可移植的工作负载。Docker 容器可以在开发人员的本机上，数据中心的物理或虚拟机上，云服务上或混合环境中运行。
Docker 的可移植性和轻量级的特性，还可以使您轻松地完成动态管理的工作负担，并根据业务需求指示，实时扩展或拆除应用程序和服务。
-  （4）在同一硬件上运行更多工作负载
Docker 轻巧快速。它为基于虚拟机管理程序的虚拟机提供了可行、经济、高效的替代方案，因此您可以利用更多的计算能力来实现业务目标。Docker 非常适合于高密度环境以及中小型部署，而您可以用更少的资源做更多的事情。

## Docker 的主要用途，目前有三大类。

- （1）提供一次性的环境。比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。
- （2）提供弹性的云服务。因为 Docker 容器可以随开随关，很适合动态扩容和缩容。
- （3）组建微服务架构。通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构

## 镜像 & 容器 & 仓库

![](https://img-blog.csdnimg.cn/img_convert/6ce0c6a82fec42e56dec61319eef4d95.png)

镜像和容器的关系就像类和类的实例，一个镜像可以同时跑多个容器，单个容器实例又可以创建新的镜像。然后docker仓库就像maven储存jar一样，提供保存镜像功能

# 三、Kubernetes简介
-  docker提供容器化技术，然后它并不具备编排能力，如果想在多台机子需要运行则每台机子都需要操作一遍，不方便。docker的缺点
    -  单机使用，无集群
    -  容器数量上升，管理成本成指数增加
    -  没有容灾和自愈机制
    -  没有编排模板，无法大规模容器调度（上线下线）
    -  没有统一的配置中心
    -  没有图形管理功能
-  因此我需要容器编排工具
    - docker swarm(凉了) 
    - mesosphere + marathon (少人用)
    - kubenetes (k8s)
-  k8s 的优势
    -  自动化容器的部署 和 扩缩容
    -  相同服务容器有组的概念，可以提供服务发现和负载均衡
    -  可自我修复：当某一个node节点关机或挂掉后，node节点上的服务会自动转移到另一个node节点上
    -  滚动更新: 更新服务不中断，一次更新一个pod，而不是同时删除整个服务
    -  集中化配置管理和秘钥管理
    -  任务批处理
    -  扩展性好: 支持模块化、插件化、可挂载、可组合
-  k8s缺点
    - 学习成本高
# 四 k8s需要安装的模块和组件
![](https://img-blog.csdnimg.cn/img_convert/b15ee8b3bddfdb3f42cb624653a5e443.png)
- master节点
    - 集群拥有一个Kubernetes Master。Kubernetes Master提供集群的独特视角，并且拥有一系列组件，比如Kubernetes API Server。API Server提供可以用来和集群交互的REST端点。master节点包括用来创建和复制Pod的Deployment
-  Node节点
    - 节点是物理或者虚拟机器，作为Kubernetes worker，通常需要安装docker,kubelet，kube Proxy
## k8s组件
-  etcd (组件)
    - 用于持久化存储集群中所有的资源对象，如Node、Service、Pod、RC、Namespace等
- API Server (组件)
    - 提供了资源对象的唯一操作入口，其他所有组件都必须通过它提供的API来操作资源数据
- Controller Manager 
    -   集群内部的管理控制中心，其主要目的是实现Kubernetes集群的故障检测和恢复的自动化工作，比如根据RC的定义完成Pod的复制或移除，以确保Pod实例数符合RC副本的定义；根据Service与Pod的管理关系
- Scheduler 
    - 集群中的调度器，负责Pod在集群节点中的调度分配
- Kubelet 
    - 负责本Node节点上的Pod的创建、修改、监控、删除等全生命周期管理
- kube Proxy 
    - 实现了Service的代理与软件模式的负载均衡器

## k8s集群的三种安装方式
-  minikube (个人学习使用)
-  二进制文件下载安装（适合生产，需要熟练人员）
-  kubeadm 安装（适合生产）

# 五、Kubernetes基本概念
-  Pod
    -  Pod是最小部署单元，Pod有一个或多个容器组成，Pod中容器共享存储和网络，在同一台Docker主机上运行
    -  pod包含的容易建议只运行一个服务进程 
    -  生产环境中很少单独启动一个pod直接使用，而是常用Depolyment、DaemonSet、StatefulSet等方式调度管理Pod
-  Deployment、Replication Controller 和 Replicat Set、
    - Replication Controller和 Replicat Set 是两种部署Pod的方式。可以确保任意时间都有指定数量的Pod“副本”在运行。如果为某个Pod创建了Replication Controller并且指定3个副本，它会创建3个Pod，并且持续监控它们
    - 生产会可以使用更高级的Deployment进行pod的管理和部署
    -  Deployment示例，字段解析
```java
apiVersion: apps/v1  # 指定api版本，此值必须在kubectl api-versions中  
kind: Deployment  # 指定创建资源的角色/类型   
metadata:  # 资源的元数据/属性 
  name: demo  # 资源的名字，在同一个namespace中必须唯一
  namespace: default # 部署在哪个namespace中
  labels:  # 设定资源的标签
    app: demo
    version: stable
spec: # 资源规范字段
  replicas: 1 # 声明副本数目
  revisionHistoryLimit: 3 # 保留历史版本
  selector: # 选择器
    matchLabels: # 匹配标签
      app: demo
      version: stable
  strategy: # 策略
    rollingUpdate: # 滚动更新
      maxSurge: 30% # 最大额外可以存在的副本数，可以为百分比，也可以为整数
      maxUnavailable: 30% # 示在更新过程中能够进入不可用状态的 Pod 的最大值，可以为百分比，也可以为整数
    type: RollingUpdate # 滚动更新策略
  template: # 模版
    metadata: # 资源的元数据/属性 
      annotations: # 自定义注解列表
        sidecar.istio.io/inject: "false" # 自定义注解名字
      labels: # 设定资源的标签
        app: demo
        version: stable
    spec: # 资源规范字段
      containers:
      - name: demo # 容器的名字   
        image: demo:v1 # 容器使用的镜像地址   
        imagePullPolicy: IfNotPresent # 每次Pod启动拉取镜像策略，三个选择 Always、Never、IfNotPresent
                                      # Always，每次都检查；Never，每次都不检查（不管本地是否有）；IfNotPresent，如果本地有就不检查，如果没有就拉取 
        resources: # 资源管理
          limits: # 最大使用
            cpu: 300m # CPU，1核心 = 1000m
            memory: 500Mi # 内存，1G = 1000Mi
          requests:  # 容器运行时，最低资源需求，也就是说最少需要多少资源容器才能正常运行
            cpu: 100m
            memory: 100Mi
        livenessProbe: # pod 内部健康检查的设置
          httpGet: # 通过httpget检查健康，返回200-399之间，则认为容器正常
            path: /healthCheck # URI地址
            port: 8080 # 端口
            scheme: HTTP # 协议
            # host: 127.0.0.1 # 主机地址
          initialDelaySeconds: 30 # 表明第一次检测在容器启动后多长时间后开始
          timeoutSeconds: 5 # 检测的超时时间
          periodSeconds: 30 # 检查间隔时间
          successThreshold: 1 # 成功门槛
          failureThreshold: 5 # 失败门槛，连接失败5次，pod杀掉，重启一个新的pod
        readinessProbe: # Pod 准备服务健康检查设置
          httpGet:
            path: /healthCheck
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 30
          timeoutSeconds: 5
          periodSeconds: 10
          successThreshold: 1
          failureThreshold: 5
        #也可以用这种方法   
        #exec: 执行命令的方法进行监测，如果其退出码不为0，则认为容器正常   
        #  command:   
        #    - cat   
        #    - /tmp/health   
        #也可以用这种方法   
        #tcpSocket: # 通过tcpSocket检查健康  
        #  port: number 
        ports:
          - name: http # 名称
            containerPort: 8080 # 容器开发对外的端口 
            protocol: TCP # 协议
      imagePullSecrets: # 镜像仓库拉取密钥
        - name: harbor-certification
      affinity: # 亲和性调试
        nodeAffinity: # 节点亲和力
          requiredDuringSchedulingIgnoredDuringExecution: # pod 必须部署到满足条件的节点上
            nodeSelectorTerms: # 节点满足任何一个条件就可以
            - matchExpressions: # 有多个选项，则只有同时满足这些逻辑选项的节点才能运行 pod
              - key: beta.kubernetes.io/arch
                operator: In
                values:
                - amd64
```
-  Label
    - 标签用于区分对象，（比如Pod，Service）键/值对存在；每个对象可以	有多个标签，通过标签关联对象 。然后可通过Pod里的Selector属性选择 
-  StatefulSet
    -  有状态集，常用于部署有状态且需要顺序启动的应用程序。可用于部署es集群，mongoDB集群、redis集群或者zookeeper
-  DaemonSet
    -  守护进程集，和守护进程类似，它会在符合条件的节点部署一个pod。当有新节点加入集群时，会为它新加一个pod,当移除时，则回收pod。适合部署日志收集deaemon
-  ConfigMap
    -  用于管理程序的配置文件
-  Secret
    -  用于保存敏感信息，如密码，令牌，SSH秘钥
-  Storage （持久化）
    - 容器内的磁盘文件都是短暂的，如果容器崩溃重启，其文件会丢失。因此k8s提供了Volums（数据卷），可以将数据挂载到主机上或者其他文件系统上（Glustter\NFS等）
    - Volums资源的管理，可以使用PersistentVolume和PersistentVolume和Clain管理
-  Service
    -  因为Pod可能在不断的创建和死亡的，其的地址是不断变化的。那它们之间是怎么访问的？答案：service主要用于Pod之间的通信。对于Pod来说，Service是提前定义好的且不变的资源
    -  Service资源的名词解析
```java
apiVersion: v1 # 指定api版本，此值必须在kubectl api-versions中 
kind: Service # 指定创建资源的角色/类型 
metadata: # 资源的元数据/属性
  name: demo # 资源的名字，在同一个namespace中必须唯一
  namespace: default # 部署在哪个namespace中
  labels: # 设定资源的标签
    app: demo
spec: # 资源规范字段
  type: ClusterIP # ClusterIP 类型
  ports:
    - port: 8080 # service 端口
      targetPort: http # 容器暴露的端口
      protocol: TCP # 协议
      name: http # 端口名称
  selector: # 选择器
    app: demo
```
-  Ingress
    -  Ingess 为k8s集群的服务提供了入口，可以提供负载均衡，SSL
    -  Ingree 用与 集群外部到集群内部的Service的http 和 https路由
-  RBAC（Role-Base Access Control,基于角色访问控制）
    - 提供了Role,ClusterRole,RoleBinding,ClusterRoleBinding四种资源。role和ClusterRole区别是role是作用于命名空间的，ClusterRole是作用与集群的
-  CronJob
    - cronJob可以用于周期行的执行任务，这些自动化任务和运行在linux和unix系统上的cronJob一样。crobJob对定期和重复任务非常有用，如执行备份任务，周期性调度程序接口

# 六、目标
Jenkins+Docker+K8S+GitLab+Harbor搭建持续集成交付环境

整套环境的搭建包含：Docker环境的搭建、docker-compose环境的搭建、K8S集群的搭建、GitLab代码仓库的搭建、Jenkins自动化部署环境的搭建、Harbor私有仓库的搭
![](https://img-blog.csdnimg.cn/img_convert/48ef9f01a21ece5b1bef0c2b4c18a1ed.png)


docker整理来自阮一峰博文，如有侵权，联系我删除
---
阮一峰docker博文链接：http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html
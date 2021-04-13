为了解决每次部署到服务器上要配置一堆环境

利用docker包装项目，配置所需环境

部署到远程时，docker从远程仓库拉取所需环境的镜像，而且不同docker可以互相解耦（jkd8和jdk11不会互相干扰）





但是每个应用都要手动部署一个docker，查看适合的配置，很麻烦，于是用k8s编排容器，用于管理。

一台机器为master，其余为worker，k8s只要与master交互就可以了。

node  service  pod

node中有多个service，service负责对外暴露接口。

service中有多个pod，pod中的环境配置都是一样的。pod中可以放进多个应用（容器），他们所需的环境都是相同的，于是共享其中的环境资源。


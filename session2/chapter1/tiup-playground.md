TiDB 集群是由多个组件构成的分布式系统，一个典型的 TiDB 集群至少由 3 个 PD 节点、3 个 TiKV 节点和 2 个 TiDB 节点构成。通过手工来部署这么多组件对于想要体验TiDB 的用户甚至是 TiDB 的开发人员来说都是非常耗时且头疼的事情。在上一章节我们介绍了 TiUP 的基础用法，在本章中，我们将介绍 TiUP 中的 playground 和 client 组件，并且将通过这两个组件快速搭建起一套本地的 TiDB 测试环境。

**playground**

通过playground 可以用户指定的组件数量快速搭建、启动本地集群，主要步骤如下：

(1) 安装playground

```
# tiup install playground:v0.0.5
```
如果不指定playground版本信息，默认将安装最新版本。版本信息列表使用以下命令查询：

```
# tiup list playground
```

(2) 通过playground部署本地集群
接着我们可以通过 install 来安装最新版本的 playground 的，相关的命令如下：

```
# tiup --tag=tidb-cluster playground v3.0.9
```
在默认情况中，playground 会启动由一个TiDB，一个 TiKV 和一个 PD 构成的集群。如果不指定 ```--tag``` 的选项，TiUP 会随机生成一个名称。如果不指定版本，默认playground会安装最新版本
。

(3) 实际部署example

作为一个分布式系统，一个最基础的 TiDB 测试集群通常由 2 个 TiDB 组件，3 个 TiKV 组件和3个PD组件来构成。

```
# tiup run playground --db=2 --kv=3 --pd=3
```
相比于之前需要手动搭建各个组件，修改各种配置，playground 功能极大的减少了搭建时间和成本。同时，我们还可以通过 monitor 选项来为测试集群增加监控功能，相关的命令如下：
```
# tiup --tag=tidb-cluster playground --db=2 --kv=3 --pd=3 --monitor
```
在集群搭建成功后，playground 会提供 mysql 对接的连接信息：
```
CLUSTER START SUCCESSFULLY, Enjoy it ^-^
To connect TiDB: mysql --host 127.0.0.1 --port 4000 -u root
```

**client**

client 功能来连接到测试集群,其会自动探测TiDB的端口进行连接，命令如下：
```
# tiup  client

```
当遇到需要连接指定集群的时候，client 同样支持通过名称来连接集群，命令如下：
```
# tiup  client NAME
```

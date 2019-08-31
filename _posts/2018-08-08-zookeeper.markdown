---
layout:     post
title:      "zookeeper"
subtitle:   " \"zookeeper搭建记录\""
date:       2018-07-08 20:44:00
author:     "wmf"
header-img: "img/in-post/bigdata.jpg"
catalog: true
tags:
    - java
---
## zk是什么
zookeeper是一个数据库<br/>
zookeeper是一个文件系统特点的数据库<br/>
zookeeper解决了数据一致性问题的分布式数据库<br/>
zookeeper是一个具有发布和订阅功能的分布式数据库<br/>
## build
su - *更换root账号*<br/>
service iptables stop *关闭防火墙*<br/>
chkconfig iptables off *永久关闭防火墙*<br/>
vi /etc/ssh/sshd_config *打开sshd配置*<br/>
service sshd restart *重启sshd*<br/>
sudo passwd *修改密码*<br/>
yum -y install lrzsz *安装lrzsz*<br/>
**上传jdk.tar.gz**
mkdir -p /export/server<br/>
tar -zxvf jdk.tar.gz *解压压缩包*<br/>
rm -rf jdk.tar.gz *删除压缩包*<br/>
## ENV
vi /etc/profile
```
export JAVA_HOME=/export/server/jdk1.8.0
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```

source /etc/profile *重新加载环境变量*
## zookeeper
tar -zxvf zoopkeeper.tar.gz *解压*<br/>
mv zookeeper.1.11 zookeeper *重命名*<br/>
cd /zookeeper/conf<br/>
cp cp zoo_sample.cfg zoo.cfg *复制文件*<br/>
vi zoo.cfg *修改配置文件*
```
dataDir=/export/data/zkdata
server.1=主机名:2888:3888
```
cd /export/data/zkdata *打开之前配置的地址*<br/>
echo 1 > myid *新建myid文件并写入id*
## start
/export/server/zookeeper/bin/zkServer.sh start
## 数据模型
1. Znode 兼具文件和目录两种属性
2. Znode 具有原子性
3. Znode 存储大小有限，一般以KB及以下为单位
4. Znode 通过路径引用(绝对路径)

Znode的属性
1. status
2. data
3. children
## 节点类型
##### 永久节点和临时节点
临时节点，客户端不连接就自动消失， 所以临时节点不能有子节点
节点不可被修改
永久节点不多解释
##### Znode 的序列化特性
可以指定是否序列化，可以用于指导Znode的先后
## 节点属性
##### dataVersion
数据版本号,每次set是都会+1
##### cVersion
子节点版本号，只要子节点变化就+1
##### cZxid
Znode创建的事务ID
##### mZxid
Znode修改的事务ID,即每次对znode的修改都会更新mzxid
对于zk来说每次变化都会产生唯一的事务ID,通过zxid可以确定操作发生的先后顺序
##### ctime
节点创建的时间
##### mtime
节点修改的时间
##### ephemeraOwner
如果为0说明改节点是一个永久节点， 如果有值则说明这是一个临时节点且存储了客户端连接的session_id
## shell基本操作
##### 客户端连接zookeeper
cd /export/server/zookeeper/bin<br/>
./zkCli.sh<br/>
ls /<br/>
<font color="red">注意: 服务端zkServer.sh, 客户端zkCli.sh, 因为这点破事你捅咕了一晚上</font>
##### 创建节点
create [-s] [-e] path data acl<br/>
-s 序列化<br/>
-e 临时节点<br/>
##### 读取节点
ls path [watch]<br/>
ls 可以列举出指定目录下的子节点，只有一级<br/>
get path [watch]<br/>
get可以查看数据和事务ID、创建时间等信息<br/>
ls2 path [watch]<br/>
节点信息+子节点，但是它查看不到数据<br/>
##### 更新节点
set path data [version]<br/>
##### 删除节点
delete path [version]<br/>
Rmr path 递归删除<br/>
##### QUOTA
setquota -n|-b val path  *(add limit to a node)*<br/>
-n 子节点的最大个数<br/>
-b 数据值的长度<br/>
val 所设的值<br/>
path 路径<br/>
##### history
之前执行的命令,命令前有编号<br/>
可以通过redo+编号重新执行<br/>
## watch
##### 一次性触发
事件触发监听，一个watch event会被发送到设置监听的客户端，这种效果是一次性的，后续再发生相同的事件，不会再次触发
##### 事件封装
Zookeeper使用WatchedEvent对象来封装服务端事件并传递
WatchedEvent包含三个基本属性
通知状态(keeperState), 事件类型(EventType)和节点路径(path)
##### 异步发送
异步发送
##### 先注册先触发
先注册先触发
## 事件类型
EventType.NodeCreated:节点创建<br/>
EventType.NodeDataChanged:节点数据变更<br/>
EventType.NodeChildrenChanged:子节点变更<br/>
EventType.NodeDeleted:节点删除## 
## 状态类型
KeeperState.Disconnected:未连接<br/>
KeeperState.SyncConnected:已连接<br/>
KeeperState.AuthFailed:认证失败<br/>
KeeperState.Expired:会话失效
## shell操作watch
create /watchtest 123 *(创建一个path存储123)*<br/>
get /watchtest watch *(后面跟了一个watch代表设置监听，ls/get/ls2都可以设置watch，此时当节点发生变化，该客户端可收到信息)*
## zookeeper JAVA API
##### 连接并创建节点
```java
public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
    // 构建java的zk客户端
    ZooKeeper zk = new ZooKeeper("129.28.89.201:2181", 30000, new Watcher() {
        // 回调方法
        public void process(WatchedEvent watchedEvent) {
            System.out.println(watchedEvent.getState());
            System.out.println(watchedEvent.getType());
            System.out.println(watchedEvent.getPath());
        }
    });
    zk.create("/girls", "sex".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT_SEQUENTIAL);
    zk.close();
}
```
##### 获取并监听
```java
//获取并监听
zk.getData("/hello", true, null);
//对节点数据修改，监听就会触发, 第三个参数版本号，-1是交给zookeeper自己维护
zk.setData("/hello", "hello".getBytes(), -1);
```
输出
```
SyncConnected
None
null
SyncConnected
NodeDataChanged
/hello
```
注: 监听是一次性的
## 选举
FastLeaderElection算法
## 典型应用
1.数据发布与订阅(配置中心)
2.命名服务(可以用一个path存储一个服务的地址)
3.分布式锁(临时节点不可重复，创建成功的相当于获取了锁，其它未成功的监听该节点，节点删除后通过创建获得锁)
## 例子1: 监听配置文件修改
```java
import java.util.concurrent.CountDownLatch;
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.Watcher.Event.EventType;
import org.apache.zookeeper.Watcher.Event.KeeperState;
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.data.Stat;
/**
 * 分布式配置中心demo
 * @author
 *
 */
public class Test2 implements Watcher {

    private static CountDownLatch connectedSemaphore = new CountDownLatch(1);
    private static ZooKeeper zk = null;
    private static Stat stat = new Stat();

    public static void main(String[] args) throws Exception {
        //zookeeper配置数据存放路径
        String path = "/username";
        //连接zookeeper并且注册一个默认的监听器
        zk = new ZooKeeper("129.28.89.201:2181", 5000, //
                new Test2());
        //等待zk连接成功的通知
        connectedSemaphore.await();
        //获取path目录节点的配置数据，并注册默认的监听器
        System.out.println("获取配置信息成功，值为：" + new String(zk.getData(path, true, stat)));// true为继续监听该节点

        Thread.sleep(Integer.MAX_VALUE);
    }

    public void process(WatchedEvent event) {
        if (KeeperState.SyncConnected == event.getState()) {  //zk连接成功通知事件
            if (EventType.None == event.getType() && null == event.getPath()) {
                connectedSemaphore.countDown();
            } else if (event.getType() == EventType.NodeDataChanged) {  //zk目录节点数据变化通知事件
                try {
                    System.out.println("配置已修改，新值为：" + new String(zk.getData(event.getPath(), true, stat)));// true为继续监听该节点
                } catch (Exception e) {
                }
            }
        }
    }
}
```
输出
```
获取配置信息成功，值为：wmf
配置已修改，新值为：wmf2
配置已修改，新值为：wmf3
```
这个小程序会一直获取配置信息的变动
## Zookeeper集群模式安装
本例搭建的是伪集群模式，即一台机器上启动三个zookeeper实例组成集群，真正的集群模式无非就是实例IP地址不同，搭建方法没有区别 

[https://blog.csdn.net/java_66666/article/details/81015302](https://blog.csdn.net/java_66666/article/details/81015302) <br/>
在conf目录下创建三个cfg文件，zoo-1.cfg，zoo-2.cfg，zoo-2.cfg
```
dataDir=/export/data/zkdata-1 // 数据存放地址
clientPort=2181 //端口号
server.1=127.0.0.1:2888:3888 //这里就是集群,一共三台服务器
server.2=127.0.0.1:2888:3889
server.3=127.0.0.1:2888:3890
```
```
dataDir=/export/data/zkdata-2 // 数据存放地址
clientPort=2182 //端口号
server.1=127.0.0.1:2888:3888 //这里就是集群,一共三台服务器
server.2=127.0.0.1:2888:3889
server.3=127.0.0.1:2888:3890
```
```
dataDir=/export/data/zkdata-3 // 数据存放地址
clientPort=2183 //端口号
server.1=127.0.0.1:2888:3888 //这里就是集群,一共三台服务器
server.2=127.0.0.1:2888:3889
server.3=127.0.0.1:2888:3890
```
然后在三个dataDir分别创建三个myid文件，存放1,2,3
```
echo 1 > myid
echo 2 > myid
echo 3 > myid
```
这三个文件夹就是存放数据的地址(zookeeper是一个数据库) <br/>
三个myid就是存放主键的文件 <br/>
分别启动三个zookeeper服务(启动zookeeper可以选择配置文件)
```
bin/zkServer.sh start  conf/zoo-1.cfg
bin/zkServer.sh start  conf/zoo-2.cfg
bin/zkServer.sh start  conf/zoo-3.cfg
```
连接指定端口的zookeeper服务
```
bin/zkCli.sh -server localhost:2181
bin/zkCli.sh -server localhost:2182
bin/zkCli.sh -server localhost:2183
```



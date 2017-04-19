title: 关于RocketMQ-第二篇-安装及启动
date: 2017/02/13 21:51:15
categories:
- JAVA服务端-RocketMQ
tags:
- RocketMQ
---

# 一.安装RocketMQ
## 1.安装环境
- linux 64位 我用的centos7
- JDK 1.7 自带了，可以 yum install java（命令默认安装的是open jdk，也可以安装oracle的）
- Git 1.8.X yum install git
- screen yum install screen
- Maven 3.X yum install apache-maven(如果报错，则按下面的方式)
```
wget http://repos.fedorapeople.org/repos/dchen/apache‐maven/epel‐apache‐maven.repo ‐O
/etc/yum.repos.d/epel‐apache‐maven.repo
yum ‐y install apache‐maven
```

## 2.clone RocketMQ
- 进入建好的文件夹
- git clone https://github.com/alibaba/RocketMQ.git
- bash install.sh

安装成功后可以看到devenv文件夹，内含所有必须的文件和库。其实这个文件是
/RocketMQ/target/alibaba-xxxx的快捷方式。

## 3.设置环境变量
- 切记java环境变量设置正确，保证jdk单一。
- 设置ROCKETMQ_HOME环境变量：

```
cd devenv
  echo "ROCKETMQ_HOME=`pwd`" >> ~/.bash_profile //将当前路径赋予全局常量写到.bash_profile中，方便后面调用
  source ~/.bash_profile //使新的环境变量生效
```


# 二.启动RocketMQ
## 1.启动NameServer
进入devenv目录的bin目录
```
screen -S 'nameserver' //建个窗口取名叫nameserver
bash mqnamesrv
如果您看到The Name Server boot success. serializeType=JSON ，这意味着名称服务器成功启动。按Ctrl +
A ，然后按D 断开会话。
```
## 2.启动broker服务
同样在这个目录

```
screen bash mqbroker ‐n localhost:9876
//‐n 是mqbroker服务一个可选参数，意为选择nameserver地址，此处默认本机ip，端口9876
```
如果看到如下输出：
The broker[localhost.localdomain, 192.168.1.72:10911] boot success. serializeType=JSON and name server is localhost:9876 表示启动成功

## 3.发送及接收消息
在发送/接收消息之前，我们需要告诉客户端名称服务器所在的位置。为了简单起见，我们使用环境变量
NAMESRV_ADDR ，（producer/consumer初始化时会调用NAMESRV_ADDR，设置成环境变量或者代码里直接声明
都可以）

```
export NAMESRV_ADDR = localhost:9876
```
**发送消息**

bash tools.sh com.alibaba.rocketmq.example.quickstart.Producer

**接收消息**

bash tools.sh com.alibaba.rocketmq.example.quickstart.Consumer


# 三.遇到问题
- **1.在虚拟机中遇到"VM warning: INFO: OS::commit_memory(0x00000006c0000000, 2147483648, 0) faild; error='Cannot allocate memory'"**
- 原因是内存不够，要加大内存，我加到了3G；还需要修改下启动脚本 runserver.sh 、runbroker.sh 中对于内存的限制：
```
runserver.sh
JAVA_OPT="${JAVA_OPT} -server -Xms512m -Xmx512m -Xmn256m -XX:PermSize=512m -XX:MaxPermSize=512m"
runbroker.sh
JAVA_OPT="${JAVA_OPT} -server -Xms2g -Xmx2g -Xmn512m -XX:PermSize=512m -XX:MaxPermSize=512m"
```
- **2.启动Broker遇到提示cannot stat '/root/rmq_bk_gc.log':No such file or directory**
- 解决方法：多等一会，直到出现成功。









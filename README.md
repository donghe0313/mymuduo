# mymuduo

基于c++11的muduo库核心功能，与boost库解耦
运行autobuild.sh可完成编译

## #muduo网络库的核心代码模块

## #Channel
fd、events、revents、callbacks 两种channel listenfd-acceptorChannel connfd-connectionChannel

## #Poller和EPollPoller - Demultiplex

std::unordered_map<int, Channel*> channels

## #EventLoop - Reactor

ChannelList activeChannels_;
std::unique_ptr poller_;
int wakeupFd; -> loop
std::unique_ptr wakeupChannel ;

## #Thread和EventLoopThread

### #EventLoopThreadPool

getNextLoop() : 通过轮询算法获取下一个subloop baseLoop
一个thread对应一个loop => one loop per thread

### #Socket

### #Acceptor

主要封装了listenfd相关的操作 socket bind listen baseLoop
#Buffer
缓冲区 应用写数据 -> 缓冲区 -> Tcp发送缓冲区 -> send
prependable readeridx writeridx
TcpConnection
一个连接成功的客户端对应一个TcpConnection Socket Channel 各种回调 发送和接收缓冲
区

### #TcpServer

Acceptor EventLoopThreadPool
ConnectionMap connections_;

### #待写

1、TcpClient编写客户端类
2、支持定时事件TimerQueue 链表/队列 时间轮（libevent） nginx定时器（红黑树）
3、DNS、HTTP、RPC
4、丰富的使用示例examples目录
5、服务器性能测试 - QPS 涉及linux上进程socketfd的设置相关
wrk linux上，需要单独编译安装 只能测试http服务的性能
Jmeter JDK Jmeter 测试http服务 tcp服务 生成聚合报告

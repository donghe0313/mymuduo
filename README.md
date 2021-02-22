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

##### 遇到的问题（1）

切换到使用自己仿写的muduo库进行聊天时，因为客户端因为Json::parse()错误中断，但登录时返回的信息能正常返回。

原因：

登录时返回的是同一个线程内的，但聊天时，若两个用户不在一个loop内，则是不同的线程。问题就出在mymuduo的TcpConnection::send(const std::string &buf)这个函数中。c_str（）返回的是string中的char*；

因此在此线程中传完回调函数后

 loop_->runInLoop(std::bind(&TcpConnection::sendInLoop,this,buf.c_str(),buf.size())); ，把内存空间释放掉，因此在执行此函数的loop中的指针为空悬指针；

```c++
void TcpConnection::send(const std::string &buf)
{
    if (state_ == kConnected)
    {
        if (loop_->isInLoopThread())
        {
            sendInLoop(buf.c_str(), buf.size());
        }
        else
        {
            loop_->runInLoop(std::bind(
                &TcpConnection::sendInLoop,
                this,
                buf.c_str(),
                buf.size()
            ));
        }
    }
}
```

解决方案：

在类中新加一个成员变量std::string otherbuf ,且修改send函数，把传入的字符拷贝到此connection的otherbuf中

void TcpConnection::send(const std::string &buf)

    void TcpConnection::send(const std::string &buf)
    {
        if (state_ == kConnected)
        {
            if (loop_->isInLoopThread())
            {
                sendInLoop(buf.c_str(), buf.size());
            }
            else
            {
                otherbuf = buf;
                loop_->runInLoop(std::bind(
                    &TcpConnection::sendInLoop,
                    this,
                    otherbuf.c_str(),
                    otherbuf.size()
                ));
            }
        }
    }


### #待写

1、TcpClient编写客户端类
2、支持定时事件TimerQueue 链表/队列 时间轮（libevent） nginx定时器（红黑树）
3、DNS、HTTP、RPC
4、丰富的使用示例examples目录
5、服务器性能测试 - QPS 涉及linux上进程socketfd的设置相关
wrk linux上，需要单独编译安装 只能测试http服务的性能
Jmeter JDK Jmeter 测试http服务 tcp服务 生成聚合报告

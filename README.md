# my-muduo

从零手写的 Reactor 模式多线程网络库，参照 chenshuo/muduo。

不是为了造轮子。目的是通过「读源码 → 破坏性实验 → 合上源码重写」练三件事：
one loop per thread、epoll ET 下的事件分发、跨线程任务投递与连接生命周期管理。

## 进度
- [ ] EventLoop / Channel / EPollPoller
- [ ] TimerQueue（timerfd，处理时间回拨）
- [ ] Buffer（readv + 64KB 栈上缓冲）
- [ ] TcpConnection（shared_ptr 生命周期、connectEstablished/Destroyed）
- [ ] TcpServer + EventLoopThreadPool
- [ ] 优雅退出
- [ ] 与原版 muduo 的压测对比

## 构建与测试

```
cmake -B build -DCMAKE_BUILD_TYPE=Debug -DENABLE_ASAN=ON
cmake --build build -j$(nproc)
ctest --test-dir build --output-on-failure
```

默认开 ASan + UBSan；TSan 用 `-DENABLE_TSAN=ON` 单独建一个 build 目录。
CI：GitHub Actions，每次 push 跑编译 + ctest。

## 压测（待填）
环境：<机器型号 / 核数 / 内核版本>，`wrk -t4 -c100 -d30s`

| 版本 | QPS | P99 | 备注 |
|---|---|---|---|
| 原版 muduo | | | 基线 |
| my-muduo v0.x | | | 目标 ≥ 原版 80% |

## docs/
每条核心链路一篇笔记，每篇必须含一节「破坏性实验」：
把这个设计去掉会发生什么（实测数据，不是推测）。

- 01-eventloop.md
- 02-buffer.md
- 03-timerqueue.md
- 04-tcpconnection.md

## 自我约束
- 不堆功能，总量 ≤ 3000 行。写完超了说明设计有问题，不是功能不够。
- 重写阶段不看 muduo 源码，只看自己的笔记 + UNP + 陈硕的书。

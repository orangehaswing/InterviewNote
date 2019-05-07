# Redis 和 I/O 多路复用

最近在看 UNIX 网络编程并研究了一下 Redis 的实现，感觉 Redis 的源代码十分适合阅读和分析，其中 I/O 多路复用（mutiplexing）部分的实现非常干净和优雅，在这里想对这部分的内容进行简单的整理。

## 几种 I/O 模型

为什么 Redis 中要使用 I/O 多路复用这种技术呢？

首先，Redis 是跑在单线程中的，所有的操作都是按照顺序线性执行的，但是由于读写操作等待用户输入或输出都是阻塞的，所以 I/O 操作在一般情况下往往不能直接返回，这会导致某一文件的 I/O 阻塞导致整个进程无法对其它客户提供服务，而 **I/O 多路复用**就是为了解决这个问题而出现的。

### Blocking I/O

先来看一下传统的阻塞 I/O 模型到底是如何工作的：当使用 `read` 或者 `write` 对某一个**文件描述符（File Descriptor 以下简称 FD)**进行读写时，如果当前 FD 不可读或不可写，整个 Redis 服务就不会对其它的操作作出响应，导致整个服务不可用。

这也就是传统意义上的，也就是我们在编程中使用最多的阻塞模型：

![blocking-io](https://img.draveness.me/2016-11-26-blocking-io.png-1000width)

阻塞模型虽然开发中非常常见也非常易于理解，但是由于它会影响其他 FD 对应的服务，所以在需要处理多个客户端任务的时候，往往都不会使用阻塞模型。

### I/O 多路复用

> 虽然还有很多其它的 I/O 模型，但是在这里都不会具体介绍。

阻塞式的 I/O 模型并不能满足这里的需求，我们需要一种效率更高的 I/O 模型来支撑 Redis 的多个客户（redis-cli），这里涉及的就是 I/O 多路复用模型了：

![I:O-Multiplexing-Mode](https://img.draveness.me/2016-11-26-I:O-Multiplexing-Model.png-1000width)

在 I/O 多路复用模型中，最重要的函数调用就是 `select`，该方法的能够同时监控多个文件描述符的可读可写情况，当其中的某些文件描述符可读或者可写时，`select` 方法就会返回可读以及可写的文件描述符个数。

> 关于 `select` 的具体使用方法，在网络上资料很多，这里就不过多展开介绍了；
>
> 与此同时也有其它的 I/O 多路复用函数 `epoll/kqueue/evport`，它们相比 `select` 性能更优秀，同时也能支撑更多的服务。

## Reactor 设计模式

Redis 服务采用 Reactor 的方式来实现文件事件处理器（每一个网络连接其实都对应一个文件描述符）

![redis-reactor-pattern](https://img.draveness.me/2016-11-26-redis-reactor-pattern.png-1000width)

文件事件处理器使用 I/O 多路复用模块同时监听多个 FD，当 `accept`、`read`、`write` 和 `close` 文件事件产生时，文件事件处理器就会回调 FD 绑定的事件处理器。

虽然整个文件事件处理器是在单线程上运行的，但是通过 I/O 多路复用模块的引入，实现了同时对多个 FD 读写的监控，提高了网络通信模型的性能，同时也可以保证整个 Redis 服务实现的简单。

## I/O 多路复用模块

I/O 多路复用模块封装了底层的 `select`、`epoll`、`avport` 以及 `kqueue` 这些 I/O 多路复用函数，为上层提供了相同的接口。

![ae-module](https://img.draveness.me/2016-11-26-ae-module.jpg-1000width)

在这里我们简单介绍 Redis 是如何包装 `select` 和 `epoll` 的，简要了解该模块的功能，整个 I/O 多路复用模块抹平了不同平台上 I/O 多路复用函数的差异性，提供了相同的接口：

- `static int  aeApiCreate(aeEventLoop *eventLoop)`
- `static int  aeApiResize(aeEventLoop *eventLoop, int setsize)`
- `static void aeApiFree(aeEventLoop *eventLoop)`
- `static int  aeApiAddEvent(aeEventLoop *eventLoop, int fd, int mask)`
- `static void aeApiDelEvent(aeEventLoop *eventLoop, int fd, int mask) `
- `static int  aeApiPoll(aeEventLoop *eventLoop, struct timeval *tvp)`

同时，因为各个函数所需要的参数不同，我们在每一个子模块内部通过一个 `aeApiState` 来存储需要的上下文信息：

```
// select
typedef struct aeApiState {
    fd_set rfds, wfds;
    fd_set _rfds, _wfds;
} aeApiState;

// epoll
typedef struct aeApiState {
    int epfd;
    struct epoll_event *events;
} aeApiState;

```

这些上下文信息会存储在 `eventLoop` 的 `void *state` 中，不会暴露到上层，只在当前子模块中使用。

### 封装 select 函数

> `select` 可以监控 FD 的可读、可写以及出现错误的情况。

在介绍 I/O 多路复用模块如何对 `select` 函数封装之前，先来看一下 `select` 函数使用的大致流程：

```
int fd = /* file descriptor */

fd_set rfds;
FD_ZERO(&rfds);
FD_SET(fd, &rfds)

for ( ; ; ) {
    select(fd+1, &rfds, NULL, NULL, NULL);
    if (FD_ISSET(fd, &rfds)) {
        /* file descriptor `fd` becomes readable */
    }
}

```

1. 初始化一个可读的 `fd_set` 集合，保存需要监控可读性的 FD；
2. 使用 `FD_SET` 将 `fd` 加入 `rfds`；
3. 调用 `select` 方法监控 `rfds` 中的 FD 是否可读；
4. 当 `select` 返回时，检查 FD 的状态并完成对应的操作。

而在 Redis 的 `ae_select` 文件中代码的组织顺序也是差不多的，首先在 `aeApiCreate` 函数中初始化 `rfds` 和 `wfds`：

```
static int aeApiCreate(aeEventLoop *eventLoop) {
    aeApiState *state = zmalloc(sizeof(aeApiState));
    if (!state) return -1;
    FD_ZERO(&state->rfds);
    FD_ZERO(&state->wfds);
    eventLoop->apidata = state;
    return 0;
}

```

而 `aeApiAddEvent` 和 `aeApiDelEvent` 会通过 `FD_SET` 和 `FD_CLR` 修改 `fd_set` 中对应 FD 的标志位：

```
static int aeApiAddEvent(aeEventLoop *eventLoop, int fd, int mask) {
    aeApiState *state = eventLoop->apidata;
    if (mask & AE_READABLE) FD_SET(fd,&state->rfds);
    if (mask & AE_WRITABLE) FD_SET(fd,&state->wfds);
    return 0;
}

```

整个 `ae_select` 子模块中最重要的函数就是 `aeApiPoll`，它是实际调用 `select` 函数的部分，其作用就是在 I/O 多路复用函数返回时，将对应的 FD 加入 `aeEventLoop` 的 `fired` 数组中，并返回事件的个数：

```
static int aeApiPoll(aeEventLoop *eventLoop, struct timeval *tvp) {
    aeApiState *state = eventLoop->apidata;
    int retval, j, numevents = 0;

    memcpy(&state->_rfds,&state->rfds,sizeof(fd_set));
    memcpy(&state->_wfds,&state->wfds,sizeof(fd_set));

    retval = select(eventLoop->maxfd+1,
                &state->_rfds,&state->_wfds,NULL,tvp);
    if (retval > 0) {
        for (j = 0; j <= eventLoop->maxfd; j++) {
            int mask = 0;
            aeFileEvent *fe = &eventLoop->events[j];

            if (fe->mask == AE_NONE) continue;
            if (fe->mask & AE_READABLE && FD_ISSET(j,&state->_rfds))
                mask |= AE_READABLE;
            if (fe->mask & AE_WRITABLE && FD_ISSET(j,&state->_wfds))
                mask |= AE_WRITABLE;
            eventLoop->fired[numevents].fd = j;
            eventLoop->fired[numevents].mask = mask;
            numevents++;
        }
    }
    return numevents;
}

```

### 封装 epoll 函数

Redis 对 `epoll` 的封装其实也是类似的，使用 `epoll_create` 创建 `epoll` 中使用的 `epfd`：

```
static int aeApiCreate(aeEventLoop *eventLoop) {
    aeApiState *state = zmalloc(sizeof(aeApiState));

    if (!state) return -1;
    state->events = zmalloc(sizeof(struct epoll_event)*eventLoop->setsize);
    if (!state->events) {
        zfree(state);
        return -1;
    }
    state->epfd = epoll_create(1024); /* 1024 is just a hint for the kernel */
    if (state->epfd == -1) {
        zfree(state->events);
        zfree(state);
        return -1;
    }
    eventLoop->apidata = state;
    return 0;
}

```

在 `aeApiAddEvent` 中使用 `epoll_ctl` 向 `epfd` 中添加需要监控的 FD 以及监听的事件：

```
static int aeApiAddEvent(aeEventLoop *eventLoop, int fd, int mask) {
    aeApiState *state = eventLoop->apidata;
    struct epoll_event ee = {0}; /* avoid valgrind warning */
    /* If the fd was already monitored for some event, we need a MOD
     * operation. Otherwise we need an ADD operation. */
    int op = eventLoop->events[fd].mask == AE_NONE ?
            EPOLL_CTL_ADD : EPOLL_CTL_MOD;

    ee.events = 0;
    mask |= eventLoop->events[fd].mask; /* Merge old events */
    if (mask & AE_READABLE) ee.events |= EPOLLIN;
    if (mask & AE_WRITABLE) ee.events |= EPOLLOUT;
    ee.data.fd = fd;
    if (epoll_ctl(state->epfd,op,fd,&ee) == -1) return -1;
    return 0;
}

```

由于 `epoll` 相比 `select` 机制略有不同，在 `epoll_wait` 函数返回时并不需要遍历所有的 FD 查看读写情况；在 `epoll_wait` 函数返回时会提供一个 `epoll_event` 数组：

```
typedef union epoll_data {
    void    *ptr;
    int      fd; /* 文件描述符 */
    uint32_t u32;
    uint64_t u64;
} epoll_data_t;

struct epoll_event {
    uint32_t     events; /* Epoll 事件 */
    epoll_data_t data;
};

```

> 其中保存了发生的 `epoll` 事件（`EPOLLIN`、`EPOLLOUT`、`EPOLLERR` 和 `EPOLLHUP`）以及发生该事件的 FD。

`aeApiPoll` 函数只需要将 `epoll_event` 数组中存储的信息加入 `eventLoop` 的 `fired` 数组中，将信息传递给上层模块：

```
static int aeApiPoll(aeEventLoop *eventLoop, struct timeval *tvp) {
    aeApiState *state = eventLoop->apidata;
    int retval, numevents = 0;

    retval = epoll_wait(state->epfd,state->events,eventLoop->setsize,
            tvp ? (tvp->tv_sec*1000 + tvp->tv_usec/1000) : -1);
    if (retval > 0) {
        int j;

        numevents = retval;
        for (j = 0; j < numevents; j++) {
            int mask = 0;
            struct epoll_event *e = state->events+j;

            if (e->events & EPOLLIN) mask |= AE_READABLE;
            if (e->events & EPOLLOUT) mask |= AE_WRITABLE;
            if (e->events & EPOLLERR) mask |= AE_WRITABLE;
            if (e->events & EPOLLHUP) mask |= AE_WRITABLE;
            eventLoop->fired[j].fd = e->data.fd;
            eventLoop->fired[j].mask = mask;
        }
    }
    return numevents;
}

```

### 子模块的选择

因为 Redis 需要在多个平台上运行，同时为了最大化执行的效率与性能，所以会根据编译平台的不同选择不同的 I/O 多路复用函数作为子模块，提供给上层统一的接口；在 Redis 中，我们通过宏定义的使用，合理的选择不同的子模块：

```
#ifdef HAVE_EVPORT
#include "ae_evport.c"
#else
    #ifdef HAVE_EPOLL
    #include "ae_epoll.c"
    #else
        #ifdef HAVE_KQUEUE
        #include "ae_kqueue.c"
        #else
        #include "ae_select.c"
        #endif
    #endif
#endif

```

因为 `select` 函数是作为 POSIX 标准中的系统调用，在不同版本的操作系统上都会实现，所以将其作为保底方案：

![redis-choose-io-function](https://img.draveness.me/2016-11-26-redis-choose-io-function.jpg-1000width)

Redis 会优先选择时间复杂度为 $O(1)$ 的 I/O 多路复用函数作为底层实现，包括 Solaries 10 中的 `evport`、Linux 中的 `epoll` 和 macOS/FreeBSD 中的 `kqueue`，上述的这些函数都使用了内核内部的结构，并且能够服务几十万的文件描述符。

但是如果当前编译环境没有上述函数，就会选择 `select` 作为备选方案，由于其在使用时会扫描全部监听的描述符，所以其时间复杂度较差 $O(n)$，并且只能同时服务 1024 个文件描述符，所以一般并不会以 `select` 作为第一方案使用。

## 总结

Redis 对于 I/O 多路复用模块的设计非常简洁，通过宏保证了 I/O 多路复用模块在不同平台上都有着优异的性能，将不同的 I/O 多路复用函数封装成相同的 API 提供给上层使用。

整个模块使 Redis 能以单进程运行的同时服务成千上万个文件描述符，避免了由于多进程应用的引入导致代码实现复杂度的提升，减少了出错的可能性。

# Redis 中的事件循环

在目前的很多服务中，由于需要持续接受客户端或者用户的输入，所以需要一个事件循环来等待并处理外部事件，这篇文章主要会介绍 Redis 中的事件循环是如何处理事件的。

在文章中，我们会先从 Redis 的实现中分析事件是如何被处理的，然后用更具象化的方式了解服务中的不同模块是如何交流的。

## aeEventLoop

在分析具体代码之前，先了解一下在事件处理中处于核心部分的 `aeEventLoop` 到底是什么：

![reids-eventloop](https://img.draveness.me/2016-12-09-reids-eventloop.png-1000width)

`aeEventLoop` 在 Redis 就是负责保存待处理文件事件和时间事件的结构体，其中保存大量事件执行的上下文信息，同时持有三个事件数组：

- `aeFileEvent`
- `aeTimeEvent`
- `aeFiredEvent`

`aeFileEvent` 和 `aeTimeEvent` 中会存储监听的文件事件和时间事件，而最后的 `aeFiredEvent` 用于存储待处理的文件事件，我们会在后面的章节中介绍它们是如何工作的。

### Redis 服务中的 EventLoop

在 `redis-server` 启动时，首先会初始化一些 redis 服务的配置，最后会调用 `aeMain` 函数陷入 `aeEventLoop` 循环中，等待外部事件的发生：

```
int main(int argc, char **argv) {
    ...

    aeMain(server.el);
}

```

`aeMain` 函数其实就是一个封装的 `while` 循环，循环中的代码会一直运行直到 `eventLoop` 的 `stop` 被设置为 `true`：

```
void aeMain(aeEventLoop *eventLoop) {
    eventLoop->stop = 0;
    while (!eventLoop->stop) {
        if (eventLoop->beforesleep != NULL)
            eventLoop->beforesleep(eventLoop);
        aeProcessEvents(eventLoop, AE_ALL_EVENTS);
    }
}

```

它会不停尝试调用 `aeProcessEvents` 对可能存在的多种事件进行处理，而 `aeProcessEvents` 就是实际用于处理事件的函数：

```
int aeProcessEvents(aeEventLoop *eventLoop, int flags) {
    int processed = 0, numevents;

    if (!(flags & AE_TIME_EVENTS) && !(flags & AE_FILE_EVENTS)) return 0;

    if (eventLoop->maxfd != -1 ||
        ((flags & AE_TIME_EVENTS) && !(flags & AE_DONT_WAIT))) {
        struct timeval *tvp;

        #1：计算 I/O 多路复用的等待时间 tvp

        numevents = aeApiPoll(eventLoop, tvp);
        for (int j = 0; j < numevents; j++) {
            aeFileEvent *fe = &eventLoop->events[eventLoop->fired[j].fd];
            int mask = eventLoop->fired[j].mask;
            int fd = eventLoop->fired[j].fd;
            int rfired = 0;

            if (fe->mask & mask & AE_READABLE) {
                rfired = 1;
                fe->rfileProc(eventLoop,fd,fe->clientData,mask);
            }
            if (fe->mask & mask & AE_WRITABLE) {
                if (!rfired || fe->wfileProc != fe->rfileProc)
                    fe->wfileProc(eventLoop,fd,fe->clientData,mask);
            }
            processed++;
        }
    }
    if (flags & AE_TIME_EVENTS) processed += processTimeEvents(eventLoop);
    return processed;
}

```

上面的代码省略了 I/O 多路复用函数的等待时间，不过不会影响我们对代码的理解，整个方法大体由两部分代码组成，一部分处理文件事件，另一部分处理时间事件。

> Redis 中会处理两种事件：时间事件和文件事件。

### 文件事件

在一般情况下，`aeProcessEvents` 都会先**计算最近的时间事件发生所需要等待的时间**，然后调用 `aeApiPoll` 方法在这段时间中等待事件的发生，在这段时间中如果发生了文件事件，就会优先处理文件事件，否则就会一直等待，直到最近的时间事件需要触发：

```
numevents = aeApiPoll(eventLoop, tvp);
for (j = 0; j < numevents; j++) {
    aeFileEvent *fe = &eventLoop->events[eventLoop->fired[j].fd];
    int mask = eventLoop->fired[j].mask;
    int fd = eventLoop->fired[j].fd;
    int rfired = 0;

    if (fe->mask & mask & AE_READABLE) {
        rfired = 1;
        fe->rfileProc(eventLoop,fd,fe->clientData,mask);
    }
    if (fe->mask & mask & AE_WRITABLE) {
        if (!rfired || fe->wfileProc != fe->rfileProc)
            fe->wfileProc(eventLoop,fd,fe->clientData,mask);
    }
    processed++;
}

```

文件事件如果绑定了对应的读/写事件，就会执行对应的对应的代码，并传入事件循环、文件描述符、数据以及掩码：

```
fe->rfileProc(eventLoop,fd,fe->clientData,mask);
fe->wfileProc(eventLoop,fd,fe->clientData,mask);

```

其中 `rfileProc` 和 `wfileProc` 就是在文件事件被创建时传入的函数指针：

```
int aeCreateFileEvent(aeEventLoop *eventLoop, int fd, int mask, aeFileProc *proc, void *clientData) {
    aeFileEvent *fe = &eventLoop->events[fd];

    if (aeApiAddEvent(eventLoop, fd, mask) == -1)
        return AE_ERR;
    fe->mask |= mask;
    if (mask & AE_READABLE) fe->rfileProc = proc;
    if (mask & AE_WRITABLE) fe->wfileProc = proc;
    fe->clientData = clientData;
    if (fd > eventLoop->maxfd)
        eventLoop->maxfd = fd;
    return AE_OK;
}

```

需要注意的是，传入的 `proc` 函数会在对应的 `mask` 位事件发生时执行。

### 时间事件

在 Redis 中会发生两种时间事件：

- 一种是定时事件，每隔一段时间会执行一次；
- 另一种是非定时事件，只会在某个时间点执行一次；

时间事件的处理在 `processTimeEvents` 中进行，我们会分三部分分析这个方法的实现：

```
static int processTimeEvents(aeEventLoop *eventLoop) {
    int processed = 0;
    aeTimeEvent *te, *prev;
    long long maxId;
    time_t now = time(NULL);

    if (now < eventLoop->lastTime) {
        te = eventLoop->timeEventHead;
        while(te) {
            te->when_sec = 0;
            te = te->next;
        }
    }
    eventLoop->lastTime = now;

```

由于对系统时间的调整会影响当前时间的获取，进而影响时间事件的执行；如果系统时间先被设置到了未来的时间，又设置成正确的值，这就会导致**时间事件会随机延迟一段时间执行**，也就是说，时间事件不会按照预期的安排尽早执行，而 `eventLoop` 中的 `lastTime` 就是用于检测上述情况的变量：

```
typedef struct aeEventLoop {
    ...
    time_t lastTime;     /* Used to detect system clock skew */
    ...
} aeEventLoop;

```

如果发现了系统时间被改变（小于上次 `processTimeEvents` 函数执行的开始时间），就会强制所有时间事件尽早执行。

```
    prev = NULL;
    te = eventLoop->timeEventHead;
    maxId = eventLoop->timeEventNextId-1;
    while(te) {
        long now_sec, now_ms;
        long long id;

        if (te->id == AE_DELETED_EVENT_ID) {
            aeTimeEvent *next = te->next;
            if (prev == NULL)
                eventLoop->timeEventHead = te->next;
            else
                prev->next = te->next;
            if (te->finalizerProc)
                te->finalizerProc(eventLoop, te->clientData);
            zfree(te);
            te = next;
            continue;
        }

```

Redis 处理时间事件时，不会在当前循环中直接移除不再需要执行的事件，而是会在当前循环中将时间事件的 `id` 设置为 `AE_DELETED_EVENT_ID`，然后再下一个循环中删除，并执行绑定的 `finalizerProc`。

```
        aeGetTime(&now_sec, &now_ms);
        if (now_sec > te->when_sec ||
            (now_sec == te->when_sec && now_ms >= te->when_ms))
        {
            int retval;

            id = te->id;
            retval = te->timeProc(eventLoop, id, te->clientData);
            processed++;
            if (retval != AE_NOMORE) {
                aeAddMillisecondsToNow(retval,&te->when_sec,&te->when_ms);
            } else {
                te->id = AE_DELETED_EVENT_ID;
            }
        }
        prev = te;
        te = te->next;
    }
    return processed;
}

```

在移除不需要执行的时间事件之后，我们就开始通过比较时间来判断是否需要调用 `timeProc` 函数，`timeProc` 函数的返回值 `retval` 为时间事件执行的时间间隔：

- `retval == AE_NOMORE`：将时间事件的 `id` 设置为 `AE_DELETED_EVENT_ID`，等待下次 `aeProcessEvents` 执行时将事件清除；
- `retval != AE_NOMORE`：修改当前时间事件的执行时间并重复利用当前的时间事件；

以使用 `aeCreateTimeEvent` 一个创建的简单时间事件为例：

```
aeCreateTimeEvent(config.el,1,showThroughput,NULL,NULL)

```

时间事件对应的函数 `showThroughput` 在每次执行时会返回一个数字，也就是该事件发生的时间间隔：

```
int showThroughput(struct aeEventLoop *eventLoop, long long id, void *clientData) {
    ...
    float dt = (float)(mstime()-config.start)/1000.0;
    float rps = (float)config.requests_finished/dt;
    printf("%s: %.2f\r", config.title, rps);
    fflush(stdout);
    return 250; /* every 250ms */
}

```

这样就不需要重新 `malloc` 一块相同大小的内存，提高了时间事件处理的性能，并减少了内存的使用量。

我们对 Redis 中对时间事件的处理以流程图的形式简单总结一下：

![process-time-event](https://img.draveness.me/2016-12-09-process-time-event.png-1000width)

创建时间事件的方法实现其实非常简单，在这里不想过多分析这个方法，唯一需要注意的就是时间事件的 `id` 跟数据库中的大多数主键都是递增的：

```
long long aeCreateTimeEvent(aeEventLoop *eventLoop, long long milliseconds,
        aeTimeProc *proc, void *clientData,
        aeEventFinalizerProc *finalizerProc) {
    long long id = eventLoop->timeEventNextId++;
    aeTimeEvent *te;

    te = zmalloc(sizeof(*te));
    if (te == NULL) return AE_ERR;
    te->id = id;
    aeAddMillisecondsToNow(milliseconds,&te->when_sec,&te->when_ms);
    te->timeProc = proc;
    te->finalizerProc = finalizerProc;
    te->clientData = clientData;
    te->next = eventLoop->timeEventHead;
    eventLoop->timeEventHead = te;
    return id;
}

```

## 事件的处理

> 上一章节我们已经从代码的角度对 Redis 中事件的处理有一定的了解，在这里，我想从更高的角度来观察 Redis 对于事件的处理是怎么进行的。

整个 Redis 服务在启动之后会陷入一个巨大的 while 循环，不停地执行 `processEvents` 方法处理文件事件 fe 和时间事件 te 。

> 有关 Redis 中的 I/O 多路复用模块可以看这篇文章 [Redis 和 I/O 多路复用](https://draveness.me/redis-io-multiplexing/)。

当文件事件触发时会被标记为 “红色” 交由 `processEvents` 方法处理，而时间事件的处理都会交给 `processTimeEvents` 这一子方法：

![redis-eventloop-proces-event](https://img.draveness.me/2016-12-09-redis-eventloop-proces-event.png-1000width)

在每个事件循环中 Redis 都会先处理文件事件，然后再处理时间事件直到整个循环停止，`processEvents` 和 `processTimeEvents` 作为 Redis 中发生事件的消费者，每次都会从“事件池”中拉去待处理的事件进行消费。

### 文件事件的处理

由于文件事件触发条件较多，并且 OS 底层实现差异性较大，底层的 I/O 多路复用模块使用了 `eventLoop->aeFiredEvent` 保存对应的文件描述符以及事件，将信息传递给上层进行处理，并抹平了底层实现的差异。

整个 I/O 多路复用模块在事件循环看来就是一个输入事件、输出 `aeFiredEvent` 数组的一个黑箱：

![eventloop-file-event-in-redis](https://img.draveness.me/2016-12-09-eventloop-file-event-in-redis.png-1000width)

在这个黑箱中，我们使用 `aeCreateFileEvent`、 `aeDeleteFileEvent` 来添加删除需要监听的文件描述符以及事件。

在对应事件发生时，当前单元格会“变色”表示发生了可读（黄色）或可写（绿色）事件，调用 `aeApiPoll` 时会把对应的文件描述符和事件放入 `aeFiredEvent` 数组，并在 `processEvents` 方法中执行事件对应的回调。

### 时间事件的处理

时间事件的处理相比文件事件就容易多了，每次 `processTimeEvents` 方法调用时都会对整个 `timeEventHead` 数组进行遍历：

![process-time-events-in-redis](https://img.draveness.me/2016-12-09-process-time-events-in-redis.png-1000width)

遍历的过程中会将时间的触发时间与当前时间比较，然后执行时间对应的 `timeProc`，并根据 `timeProc` 的返回值修改当前事件的参数，并在下一个循环的遍历中移除不再执行的时间事件。

## 总结

> 笔者对于文章中两个模块的展示顺序考虑了比较久的时间，最后还是觉得，目前这样的顺序更易于理解。

Redis 对于事件的处理方式十分精巧，通过传入函数指针以及返回值的方式，将时间事件移除的控制权交给了需要执行的处理器 `timeProc`，在 `processTimeEvents` 设置 `aeApiPoll` 超时时间也十分巧妙，充分地利用了每一次事件循环，防止过多的无用的空转，并且保证了该方法不会阻塞太长时间。

事件循环的机制并不能时间事件准确地在某一个时间点一定执行，往往会比实际约定处理的时间稍微晚一些。












# Libevent 

## 1. 简介

libevent包括事件管理、缓存管理、DNS、HTTP、缓存事件几大部分。事件管理包括各种IO（socket）、定时器、信号等事件；缓存管理是指evbuffer功能；DNS是libevent提供的一个异步DNS查询功能；HTTP是libevent的一个轻量级http实现，包括服务器和客户端。libevent也支持ssl，这对于有安全需求的网络程序非常的重要，但是其支持不是很完善，比如http server的实现就不支持ssl。



## 2. 核心框架

**Reactor**（反应堆）模式是libevent的核心框架，**libevent以事件驱动，自动触发回调功能**。



## 3. Event事件驱动状态转换图

事件驱动实际上是libevent的核心思想

![img](pic/wps4495.tmp.jpg)

**状态介绍**：

1. **Invalid ptr**:  此时仅仅是定义了 struct event *ptr
2. **non_pending：**相当于**创建了事件**，但是事件还**没有处于被监听状态**，类似于使用epoll的时候定义了struct epoll_event ev并且对ev的两个字段进行了赋值，但是此时尚未调用epoll_ctl。
3. **pending：**开始监听，但是还没事件产生

4. **active：**代表监听的事件已经产生，这时需要处理回调函数，相当于我们epoll所说的事件就绪

## 4. API

### 4.1 Event_base根节点

```c
创建根节点：
	struct event_base *event_base_new(void);
		返回值值就是event_base根节点地址
释放根节点：
	void event_base_free(struct event_base *);
```
### 4.2 event事件驱动结构体

初始化：

```c
struct event *event_new(struct event_base *base, evutil_socket_t fd, short events, event_callback_fn cb, void *arg);

参数说明：
    base：根节点
    fd: 上树的文件描述符
    events: 事件
    cb: call_back回调函数
        typedef void (*event_callback_fn)(evutil_socket_t fd, short events, void *arg);
    arg: 传给回调函数的参数
        
 返回值：
        succ: event节点
        error: NULL
events:
    #define  EV_TIMEOUT         0x01   //超时事件
    #define  EV_READ            0x02   //读事件
    #define  EV_WRITE           0x04   //写事件
    #define  EV_SIGNAL          0x08   //信号事件
    #define  EV_PERSIST         0x10   //周期性触发
    #define  EV_ET                     // 边沿触发
```

释放：

```c
void event_free(struct event *ev);
```

### 4.3 add del

上树：

```c
int event_add(struct event *ev, const struct timeval *timeout);
	参数说明：
		ev:上树节点地址
		timeout: NULL  永久监听，  固定时间 限时等待
```

下树：

```c
int event_del(struct event *ev);
```

### 4.4 循环监听

```c
int event_base_dispatch(struct event_base *base); 类似epoll_wait
 
```

```c
int event_base_loopexit(struct event_base *base, const struct timeval *tv); //等待固定时间之后退出

int event_base_loopbreak(struct event_base *base);//立即退出
```

### 4.5 server demo

```c
#include <stdio.h>
#include <stdlib.h>
#include <event.h>
#include "wrap.h"

#define _EVENT_SIZE 1024

typedef struct fd_event
{
	int fd;
	struct event *ev;
}xevs;

xevs xevent[_EVENT_SIZE];

void init_xevent()
{
	int i = 0;
	for (; i < _EVENT_SIZE; i++)
	{
		xevent[i].fd = -1;
		xevent[i].ev = NULL;
	}
}

void destroy_xevent()
{
	int i = 0;
	for (; i < _EVENT_SIZE; i++)
	{
		xevent[i].fd = -1;
		free(xevent[i].ev);
		xevent[i].ev = NULL;
	}
}

int find_index()
{
	int i = 0;
	for (i = 0; i < _EVENT_SIZE; i++)
	{
		if (xevent[i].fd < 0)
		{
			break;
		}
	}
    // check i ...
	return i;
}

void xevent_add(int fd,  struct event * ev)
{
	int i = find_index();
	if (i < 0)
	{
		printf("xevent full...\n");
		return;
	}
	xevent[i].fd = fd;
	xevent[i].ev = ev;

}

int find_fd(int fd)
{
	int i = 0;
	for (i = 0; i < _EVENT_SIZE; i++)
	{
		if (xevent[i].fd == fd)
		{
			break;
		}
	}
	
	// check ....
	return i;
}

// client callback funcation
void c_cb(int fd, short events, void *arg)
{
	printf("begin %d call %s...\n",fd,  __FUNCTION__);
	char buf[1500] = {'0'};

	int n = recv(fd, buf, sizeof(buf), 0);
	if (n <= 0)
	{
		Close(fd);
		// event_del
		event_del(xevent[find_fd(fd)].ev);
        
		xevent[find_fd(fd)].fd = -1;
		free(xevent[find_fd(fd)].ev);
        xevent[find_fd(fd)].ev = NULL;
		printf("client %d closed....\n", fd);
	}
	else 
	{
		write(STDOUT_FILENO, buf, n);
		send(fd, buf, n, 0);
	}
}

// server callback function
void s_cb(int fd, short events, void *arg)
{
	printf("begin %d call %s...\n",fd,  __FUNCTION__);
	struct event_base *base = (struct event_base *)arg;

	int cfd = Accept(fd, NULL, NULL);

	struct event *ev = event_new(base, cfd, EV_READ | EV_PERSIST, c_cb, NULL);
	event_add(ev, NULL);

	xevent_add(cfd, ev);
}


int main()
{
	// create socket and bind 
	int lfd = tcp4bind(8080, NULL);

	// listen 
	Listen(lfd, 128);

	// create event_base
	struct event_base *base = event_base_new();

	init_xevent();

	// init event 
	struct event *ev = event_new(base, lfd, EV_READ | EV_PERSIST, s_cb, base);
	
	// add ev 
	event_add(ev, NULL);

	// add lfd to xevent 
	xevent_add(lfd, ev);

	event_base_dispatch(base);

	Close(lfd);
	destroy_xevent();
	event_base_free(base);
	return 0;

}
```

# 5. BufferEvent

**Bufferevent**实际上也是一个event，只不过比普通的event高级一些，它的内部有两个缓冲区，以及一个文件描述符（网络套接字）。我们都知道一个网络套接字有读和写两个缓冲区，**bufferevent同样也带有两个缓冲区**，还有就是libevent事件驱动的核心回调函数，那么**四个缓冲区**以及触发回调的关系如下：

![img](pic/wps6C55.tmp.jpg)

BufferEvent有**三个回调函数**：

1. **读回调** – 当bufferevent将**底层读缓冲区**的数据**读到自身的读缓冲区**时触发读事件回调
2. **写回调** – 当bufferevent将自**身写缓冲**的数据写到**底层写缓冲**区的时候出发写事件回调
3. **事件回调** – 当bufferevent绑定的socket连接，断开或者异常的时候触发事件回调

## 5.1 API

### 5.1.1 创建bufferevent事件

```c
struct bufferevent * bufferevent_socket_new(struct event_base *base：
                                            , evutil_socket_t fd, int options);
参数说明：
    base：根节点
    fd: 文件句柄
	options:
		BEV_OPT_CLOSE_ON_FREE    释放bufferevent自动关闭底层接口
        BEV_OPT_THREADSAFE      使bufferevent能够在多线程下是安全的
```

### 5.1.2 释放bufferevent事件

```c
void bufferevent_free(struct bufferevent *bufev);
```

### 5.1.3 客户端创建socket及连接

```c
int bufferevent_socket_connect(struct bufferevent *bev, struct sockaddr *serv, int socklen);
```

### 5.1.4 回调函数

```c
void bufferevent_setcb(struct bufferevent *bufev,  // bufferevent节点
   					  bufferevent_data_cb readcb,  // 读回调
                      bufferevent_data_cb writecb, // 写回调
					  bufferevent_event_cb eventcb, // 事件回调
                      void *cbarg);                 // 回调函数参数
读写回调原型：
    typedef void (*bufferevent_data_cb)(struct bufferevent *bev, void *ctx);
	void  (struct bufferevent *bev, void *ctx);
事件回调原型：
	typedef void (*bufferevent_event_cb)(struct bufferevent *bev, short what, void *ctx);
		what:代表 对应的事件
			BEV_EVENT_EOF,  对方关闭
			BEV_EVENT_ERROR， 异常
            BEV_EVENT_TIMEOUT, 超时
			BEV_EVENT_CONNECTED 建立连接
      
```

### 5.1.5 回调函数使能

bufferevent_enable与bufferevent_disable是**设置事件是否生效**，如果设置为disable，事件回调将不会被触发。

```c
int bufferevent_enable(struct bufferevent *bufev, short event);
int bufferevent_disable(struct bufferevent *bufev, short event);
```





### 5.1.6 读写

**写**：

```c
int bufferevent_write(struct bufferevent *bufev, const void *data, size_t size);
	bufferevent_write是将data的数据写到bufferevent的写缓冲区
        
int bufferevent_write_buffer(struct bufferevent *bufev, struct evbuffer *buf);
	bufferevent_write_buffer 是将数据写到写缓冲区另外一个写法，实际上bufferevent的内部的两个缓冲区结构就是struct evbuffer。
```

**读**：

```c
size_t bufferevent_read(struct bufferevent *bufev, void *data, size_t size);
	bufferevent_read 是将bufferevent的读缓冲区数据读到data中，同时将读到的数据从bufferevent的读缓冲清除。
        
int bufferevent_read_buffer(struct bufferevent *bufev, struct evbuffer *buf);
```

### 5.1.7 链接监听器evconnlistener

链接监听器封装了底层的socket通信相关函数，比如socket，bind，listen，accept这几个函数。链接监听器创建后实际上相当于调用了socket，bind，listen，此时等待新的客户端连接到来，如果有新的客户端连接，那么内部先进行accept处理，然后调用用户指定的回调函数

```c
#include <event2/bufferevent.h>
#include <event2/listener.h>

struct evconnlistener *evconnlistener_new_bind(struct event_base *base,evconnlistener_cb cb, void *ptr, unsigned flags, int backlog, const struct sockaddr *sa, int socklen);

参数：
    base： 根节点
    cb: Accpet回调函数
    ptr: 回调函数参数
    flags：
        LEV_OPT_LEAVE_SOCKETS_BLOCKING   文件描述符为阻塞的
        LEV_OPT_CLOSE_ON_FREE            关闭时自动释放
        LEV_OPT_REUSEABLE                端口复用 reuseable
        LEV_OPT_THREADSAFE               分配锁，线程安全
    backlog：如果backlog是-1，那么监听器会自动选择一个合适的值，如果填0，那么监听器会认为listen函数已经被调用过了
    sa, socklen: bind函数相关参数
```

**evconnlistener_new**函数与前一个函数不同的地方在与后2个参数，使用本函数时，认为socket已经初始化好，并且bind完成，甚至也可以做完listen，所以大多数时候，我们都可以使用第一个函数

```c
struct evconnlistener *evconnlistener_new(struct event_base *base,
    evconnlistener_cb cb, void *ptr, unsigned flags, int backlog,
evutil_socket_t fd);
```

**回调函数原型**：

```c
typedef void (*evconnlistener_cb)(struct evconnlistener *evl, evutil_socket_t fd, struct sockaddr *cliaddr, int socklen, void *ptr);

主要回调函数fd参数会与客户端通信的描述符，并非是等待连接的监听的那个描述符，所以cliaddr对应的也是新连接的对端地址信息，已经是accept处理好的
```

**释放：**

```c
void evconnlistener_free(struct evconnlistener *lev);
```

**使链接监听器生/失效**

```c
int evconnlistener_enable(struct evconnlistener *lev);  使链接监听器生效

int evconnlistener_disable(struct evconnlistener *lev);  使链接监听器失效
```

## 5.2 demo

### 5.2.1 server

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <arpa/inet.h>
#include <sys/signal.h>
#include <sys/socket.h>

#include <event.h>
#include <event2/listener.h>
#include <event2/util.h>

#define PORT 8888
static const char MESSAGE[] = "Hello, World!\n";

static void listener_cb(struct evconnlistener*, evutil_socket_t, struct sockaddr *, int socklen, void *);
static void read_cb(struct bufferevent *bev, void *ctx);
static void write_cb(struct bufferevent *bev, void *ctx);
static void event_cb(struct bufferevent *bev, short what, void *ctx);
static void signal_cb(evutil_socket_t sig, short events, void *ptr);

// bufferevent实现server 回射功能
int main(int argc, char *argv[])
{
	// 创建根节点
	struct event_base *base = event_base_new();

	struct sockaddr_in server_addr;
	memset(&server_addr, 0, sizeof(server_addr));
	server_addr.sin_family = AF_INET;
	server_addr.sin_port = htons(PORT);

	// 创建、绑定、监听、提取连接
	// 创建bufferevent 链接监听器
	struct evconnlistener *listener = evconnlistener_new_bind(base, listener_cb, 
 (void *)base, LEV_OPT_CLOSE_ON_FREE | LEV_OPT_REUSEABLE, -1, 
 (struct sockaddr *)&server_addr, sizeof(server_addr));
    
	if (!listener)
	{
		fprintf(stderr, "Could not create evconnlistener\n");
		return 1;
	}

	// 创建信号节点 ctrl +c 
	struct event *signal_event = evsignal_new(base, SIGINT, signal_cb, (void *)base);
	if (!signal_event || event_add(signal_event, NULL) < 0)
	{
		fprintf(stderr, "Could not create/add signal event\n");
		return 1;
	}

	// 循环监听
	event_base_dispatch(base);

	event_free(signal_event);
	evconnlistener_free(listener);
	event_base_free(base);

	printf("done\n");
	return 0;
}

static void listener_cb(struct evconnlistener *evl, evutil_socket_t fd, struct sockaddr *cliaddr, int socklen, void *ptr)
{
	struct event_base *base = (struct event_base *)ptr;

	// 新建bufferevent 
	struct bufferevent *bev = bufferevent_socket_new(base, fd, BEV_OPT_CLOSE_ON_FREE);
	if (!bev)
	{
		fprintf(stderr, "Erroring Constructing Bufferevent, %s\n", strerror(errno));
		event_base_loopbreak(base);
		return;
	}

	// 设置回调
	bufferevent_setcb(bev, read_cb, write_cb, event_cb, (void *)base);
	bufferevent_enable(bev, EV_READ | EV_WRITE);

	// 给客户端发送消息
	bufferevent_write(bev, MESSAGE, sizeof(MESSAGE));
	struct sockaddr_in *sin = (struct sockaddr_in*)cliaddr;
	printf("client %d, connect successfully\n", ntohs(sin->sin_port));
}



static void write_cb(struct bufferevent *bev, void *ctx)
{
	// 获取缓冲区类型
	struct evbuffer *output = bufferevent_get_output(bev);
	if (evbuffer_get_length(output) == 0)
	{
		printf("send succ\n");
	}
}

static void read_cb(struct bufferevent *bev, void *ctx)
{
	char buf[1500] = {'0'};
	bufferevent_read(bev, buf, sizeof(buf));
	printf("recv: %s\n", buf);

	// 回射
	bufferevent_write(bev, buf, sizeof(buf));
}

static void event_cb(struct bufferevent *bev, short what, void *ctx)
{
	if (what == BEV_EVENT_EOF)
	{
		printf("client closed connection\n");
	}
	else if (what == BEV_EVENT_ERROR)
	{
		printf("Got an error on the connection: %s\n", strerror(errno));
	}

	bufferevent_free(bev);

}

// 信号回调函数
static void signal_cb(evutil_socket_t sig, short events, void *ptr)
{
	struct event_base *base = (struct event_base *)ptr;
	struct timeval delay = { 2, 0 };

	printf("Caught an interrupt signal; exiting cleanly in two seconds.\n");

	event_base_loopexit(base, &delay);//退出循环监听
}
```

### 5.2.2 client

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <arpa/inet.h>
#include <sys/signal.h>
#include <sys/socket.h>
#include <unistd.h>
#include <event.h>
#include <event2/listener.h>


void cmd_msg_cb(evutil_socket_t fd, short events, void *arg);
void server_msg_cb(struct bufferevent *bev, void *ctx);
void event_cb(struct bufferevent *bev, short event, void *arg);

// 客户端功能
// 1. 从标准输入读取输入，发送给服务器 ----> 普通event事件
// 2. 接受服务器的消息                 ----> bufferevent 事件
int main()
{

	struct event_base 	*base;            // 根节点
	struct event      	*ev_cmd;          // 监听stdin
	struct bufferevent 	*bev;             // bufferevent 缓冲区
	struct sockaddr_in 	sin;              // server info 

	do
	{
        // 初始化根节点 
		if ((base = event_base_new()) == NULL)
		{
			fprintf(stderr, "Could not create event_base\n");
			break;
		}

        // 设置服务端 ip 端口 
		memset(&sin, 0, sizeof(sin));
		sin.sin_family = AF_INET;
		sin.sin_port = htons(8888);
		sin.sin_addr.s_addr = inet_addr("127.0.0.1");

		// 初始化bev缓冲区及与服务端建立连接
		bev = bufferevent_socket_new(base, -1, BEV_OPT_CLOSE_ON_FREE);
		if (!bev || bufferevent_socket_connect(bev, (struct sockaddr *)&sin, sizeof(sin)) < 0)
		{
			fprintf(stderr, "Could create bufferevent /connect server, errmsg: %s\n", strerror(errno));
			break;
		}

		//设置buffer的回调函数 主要设置了读回调 server_msg_cb ,传入参数是标准输入的读事件
		bufferevent_setcb(bev, server_msg_cb, NULL, event_cb, (void*)ev_cmd);
		bufferevent_enable(bev, EV_READ | EV_PERSIST);

		// 初始化 event事件
		if ((ev_cmd = event_new(base, STDIN_FILENO, EV_READ | EV_PERSIST, cmd_msg_cb, (void *)bev)) == NULL)
		{
			fprintf(stderr, "Could not create event\n");
			break;
		}
		// 节点上树
		event_add(ev_cmd, NULL);
		// 循环监听
		event_base_dispatch(base);
	}while(0);

	event_free(ev_cmd);
	bufferevent_free(bev);
	event_base_free(base);
	return 0;
}

// 从标准输入读取并发送数据到服务器
void cmd_msg_cb(evutil_socket_t fd, short events, void *arg)
{
	char buf[1500] = {'0'};

	int n = read(fd, buf, sizeof(buf));
	if (n < 0)
	{
		fprintf(stderr, "%s read error", __FUNCTION__);
		exit(-1);
	}
	
	struct bufferevent *bev = (struct bufferevent *)arg;
	bufferevent_write(bev, buf, sizeof(buf));

}

// 接受服务器端消息
void server_msg_cb(struct bufferevent *bev, void *ctx)
{
	char buf[1500] = {'0'};
	int len = bufferevent_read(bev, buf, sizeof(buf));
	buf[len] = '\0';
	printf("recv: %s\n", buf);
}

void event_cb(struct bufferevent *bev, short event, void *arg)
{
	if (event & BEV_EVENT_EOF)
		printf("connection closed\n");
	else if (event & BEV_EVENT_ERROR)
		printf("some other error\n");
	else if( event & BEV_EVENT_CONNECTED)
	{
		printf("the client has connected to server\n");
		return ;
	}


	//这将自动close套接字和free读写缓冲区
	bufferevent_free(bev);
	//释放event事件 监控读终端
    
    
	struct event *ev = (struct event*)arg;
	event_free(ev);
	exit(0);
}
```


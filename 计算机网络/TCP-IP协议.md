# TCP/IP协议

# TCP和UDP区别介绍

## TCP与UDP基本区别

1. 基于连接与无连接
2. TCP要求系统资源较多，UDP较少； 
3. UDP程序结构较简单 
4. 流模式（TCP）与数据报模式(UDP); 
5. TCP保证数据正确性，UDP可能丢包 
6. TCP保证数据顺序，UDP不保证 

## UDP应用场景：

1. 面向数据报方式
2. 网络数据大多为短消息 
3. 拥有大量Client
4. 对数据安全性无特殊要求
5. 网络负担非常重，但对响应速度要求高

## 具体编程时的区别

1. socket()的参数不同 
2. UDP Server不需要调用listen和accept 
3. UDP收发数据用sendto/recvfrom函数 
4. TCP：地址信息在connect/accept时确定 
5. UDP：在sendto/recvfrom函数中每次均 需指定地址信息 
6. UDP：shutdown函数无效

# UDP和TCP编程步骤不同

## TCP

TCP编程的服务器端一般步骤是： 

1. 创建一个socket，用函数socket()； 
2. 设置socket属性，用函数setsockopt(); * 可选 
3. 绑定IP地址、端口等信息到socket上，用函数bind(); 
4. 开启监听，用函数listen()； 
5. 接收客户端上来的连接，用函数accept()； 
6. 收发数据，用函数send()和recv()，或者read()和write(); 
7. 关闭网络连接； 
8. 关闭监听； 

TCP编程的客户端一般步骤是： 

1. 创建一个socket，用函数socket()； 
2. 设置socket属性，用函数setsockopt();* 可选 
3. 绑定IP地址、端口等信息到socket上，用函数bind();* 可选 
4. 设置要连接的对方的IP地址和端口等属性； 
5. 连接服务器，用函数connect()； 
6. 收发数据，用函数send()和recv()，或者read()和write(); 
7. 关闭网络连接；

## UDP

UDP编程的服务器端一般步骤是： 

1. 创建一个socket，用函数socket()； 
2. 设置socket属性，用函数setsockopt();* 可选 
3. 绑定IP地址、端口等信息到socket上，用函数bind(); 
4. 循环接收数据，用函数recvfrom(); 
5. 关闭网络连接； 

UDP编程的客户端一般步骤是： 

1. 创建一个socket，用函数socket()； 
2. 设置socket属性，用函数setsockopt();* 可选 
3. 绑定IP地址、端口等信息到socket上，用函数bind();* 可选 
4. 设置对方的IP地址和端口等属性; 
5. 发送数据，用函数sendto(); 
6. 关闭网络连接；


# TCP、UDP握手编程

## TCP

使用TCP套接字编程可以实现基于TCP/IP协议的面向连接的通信，它分为服务器端和客户端两部分，其主要实现过程如图1.1所示。

                                      ![img](http://hi.csdn.net/attachment/201112/2/0_1322811079mRzu.gif)     

参考程序

tcpserver.c

```
#define  PORT 1234
#define  BACKLOG 1
 
int main()
{
	int  listenfd, connectfd;
    struct  sockaddr_in server;
    struct  sockaddr_in client;
    socklen_t  addrlen;
    
    if((listenfd = socket(AF_INET, SOCK_STREAM, 0)) == -1)
    {
    perror("Creating  socket failed.");
    exit(1);
    }
    
    int opt =SO_REUSEADDR;
    
    setsockopt(listenfd,SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));
    bzero(&server,sizeof(server));
    server.sin_family=AF_INET;
    server.sin_port=htons(PORT);
    server.sin_addr.s_addr= htonl (INADDR_ANY);
       
    if(bind(listenfd, (struct sockaddr *)&server, sizeof(server)) == -1) {
    perror("Binderror.");
    exit(1);
    }   
       
    if(listen(listenfd,BACKLOG)== -1){  /* calls listen() */
    perror("listen()error\n");
    exit(1);
    }
       
    addrlen =sizeof(client);
       
    if((connectfd = accept(listenfd,(struct sockaddr*)&client,&addrlen))==-1) {
    perror("accept()error\n");
    exit(1);
    }
       
	printf("Yougot a connection from cient's ip is %s, prot is %d\n",inet_ntoa(client.sin_addr),htons(client.sin_port));
       
    send(connectfd,"Welcometo my server.\n",22,0);
	close(connectfd);
	close(listenfd);
	return 0;
}
```

tcpclient.c

```
#define  PORT 1234
#define  MAXDATASIZE 100
 
int main(int argc, char *argv[])
{
       int  sockfd, num;
       char  buf[MAXDATASIZE];
       struct hostent *he;
       struct sockaddr_in server;
       
       if (argc!=2) {
       printf("Usage:%s <IP Address>\n",argv[0]);
       exit(1);
       }
       
       if((he=gethostbyname(argv[1]))==NULL){
       printf("gethostbyname()error\n");
       exit(1);
       }
       
       if((sockfd=socket(AF_INET, SOCK_STREAM, 0))==-1){
       printf("socket()error\n");
       exit(1);
       }
       
       bzero(&server,sizeof(server));
       server.sin_family= AF_INET;
       server.sin_port = htons(PORT);
       server.sin_addr =*((struct in_addr *)he->h_addr);
       
       if(connect(sockfd,(struct sockaddr *)&server,sizeof(server))==-1){
       printf("connect()error\n");
       exit(1);
       }
       
       if((num=recv(sockfd,buf,MAXDATASIZE,0)) == -1){
       printf("recv() error\n");
       exit(1);
       }
       
       buf[num-1]='\0';
       printf("Server Message: %s\n",buf);
       close(sockfd);
return 0;
}
```

## UDP

建立UDP套接口时socket函数的第二个参数应该是SOCK_DGRAM，说明是建立一个UDP套接口；由于UDP是无连接的，所以服务器端并不需要listen或accept函数。

使用UDP套接字编程可以实现基于TCP/IP协议的面向无连接的通信，它分为服务器端和客户端两部分，其主要实现过程如图3.1所示。

​                                    ![img](http://hi.csdn.net/attachment/201112/9/0_1323398200rzV7.gif)

参考程序

udpserver.c

```
#define PORT 1234
#define MAXDATASIZE 100
 
main()
{
       int sockfd;
       struct sockaddr_in server;
       struct sockaddr_in client;
       socklen_t addrlen;
       int num;
       char buf[MAXDATASIZE];
 
       if((sockfd = socket(AF_INET, SOCK_DGRAM, 0)) == -1) 
       {
       perror("Creatingsocket failed.");
       exit(1);
       }
 
       bzero(&server,sizeof(server));
       server.sin_family=AF_INET;
       server.sin_port=htons(PORT);
       server.sin_addr.s_addr= htonl (INADDR_ANY);
       if(bind(sockfd, (struct sockaddr *)&server, sizeof(server)) == -1)
       {
       perror("Bind()error.");
       exit(1);
       }   
 
       addrlen=sizeof(client);
       while(1)  
       {
       num =recvfrom(sockfd,buf,MAXDATASIZE,0,(struct sockaddr*)&client,&addrlen);                                   
 
       if (num < 0)
       {
       perror("recvfrom() error\n");
       exit(1);
       }
 
       buf[num] = '\0';
       printf("You got a message (%s%) from client.\nIt's ip is%s, port is %d.\n",buf,inet_ntoa(client.sin_addr),htons(client.sin_port)); 
       sendto(sockfd,"Welcometo my server.\n",22,0,(struct sockaddr *)&client,addrlen);
       if(!strcmp(buf,"bye"))
       break;
       }
       close(sockfd);  
}
```

udpclient.c

```
 #define PORT 1234
#define MAXDATASIZE 100
 
int main(int argc, char *argv[])
{
       int sockfd, num;
       char buf[MAXDATASIZE];
 
       struct hostent *he;
       struct sockaddr_in server,peer;
 
       if (argc !=3)
       {
       printf("Usage: %s <IP Address><message>\n",argv[0]);
       exit(1);
       }
 
       if ((he=gethostbyname(argv[1]))==NULL)
       {
       printf("gethostbyname()error\n");
       exit(1);
       }
 
       if ((sockfd=socket(AF_INET, SOCK_DGRAM,0))==-1)
       {
       printf("socket() error\n");
       exit(1);
       }
 
       bzero(&server,sizeof(server));
       server.sin_family = AF_INET;
       server.sin_port = htons(PORT);
       server.sin_addr= *((struct in_addr *)he->h_addr);
       sendto(sockfd, argv[2],strlen(argv[2]),0,(struct sockaddr *)&server,sizeof(server));
       socklen_t  addrlen;
       addrlen=sizeof(server);
       while (1)
       {
       if((num=recvfrom(sockfd,buf,MAXDATASIZE,0,(struct sockaddr *)&peer,&addrlen))== -1)
       {
       printf("recvfrom() error\n");
       exit(1);
       }
       if (addrlen != sizeof(server) ||memcmp((const void *)&server, (const void *)&peer,addrlen) != 0)
       {
       printf("Receive message from otherserver.\n");
       continue;
       }
 
       buf[num]='\0';
       printf("Server Message:%s\n",buf);
       break;
       }
 
       close(sockfd);
}
```

                                             

















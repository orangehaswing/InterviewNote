Python网络编程



编写一个最简单的Client/Server程序：

server:

python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
127.0.0.1 - - [09/Jun/2018 14:56:35] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [09/Jun/2018 15:00:20] "GET / HTTP/1.1" 200 -

client:

import requests
r = requests.get('http://127.0.0.1:8000/')
print(r)

result:

wing@wing-virtual-machine:~/Code/Python$ python3 pro_http.py 
<Response [200]>
wing@wing-virtual-machine:~/Code/Python$ python3 pro_http.py 
<Response [200]>

可以看到，服务器返回了一个200成功响应。

总结请求过程：

1. 客户端向服务器端发起了一个HTTP(GET)请求。
2. 服务器端向客户端返回了一个HTTP(200)响应。



tcpdump监听：tcpdump -i lo0 port 8000 -w result.cap

出现：

tcpdump: lo0: You don't have permission to capture on that device
(socket: Operation not permitted)

solve：

Add a capture group and add yourself to it:

```
sudo groupadd pcap
sudo usermod -a -G pcap $USER

```

Next, change the group of `tcpdump` and set permissions:

```
sudo chgrp pcap /usr/sbin/tcpdump
sudo chmod 750 /usr/sbin/tcpdump

```

Finally, use `setcap` to give `tcpdump` the necessary permissions:

```
sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump
```



出现：

bash: /usr/sbin/tcpdump: Permission denied

slove：

先查看处在那个模式：

grep tcpdump /sys/kernel/security/apparmor/profiles
/usr/sbin/tcpdump (enforce)

果然不是complain模式。

修改为complain模式：

aa-complain /usr/sbin/tcpdump



tcpdump来监听本地网卡的tcp连接结果：

..........



看服务器端的状态，通过lsof命令来查看绑定8000端口的描述符信息：

wing@wing-virtual-machine:~/Code/Python$ lsof -n -i:8000
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
python3 1905 wing    3u  IPv4  34690      0t0  TCP *:8000 (LISTEN)

通过结果可以观察到服务器的进程的一些信息，服务器进程处于**LISTEN**阶段，说明服务器处于保持着监听连接的状态：

现在用刚才的例子来解释TCP中状态迁移的概念，这时候，如果从客户端到来一个请求：

1. 服务器端接收到客户端的SYN报文，返回SYN+ACK报文，服务器端进入**SYN_RCVD**状态。
2. 服务器端收到客户端返回的ACK应答后，连接建立，进入**ESTABLISHED**状态。
3. 服务器端的数据传输完毕，给客户端发送FIN报文，进入**FIN_WAIT_1**状态。
4. 服务器端接收到客户端返回的ACK应答后，进入**FIN_WAIT_2**状态。
5. 服务器端接收到客户端的FIN报文，接着返回一个ACK应答，等待连接关闭，进入**TIME_WAIT**状态。
6. 服务器端经过**2MSL**时间后进入**CLOSED**状态，此时连接关闭。



至于客户端，在每个阶段也有各自的状态，下图表示了TCP状态迁移的过程：

![img](https://pic1.zhimg.com/80/v2-11a83cab93a101daf0b0db542aca7588_hd.jpg)



socket

socket在上图中处于第三层和第四层之间，把socket理解为在传输层和应用层之间的一组通信接口，或者是一个抽象的通信设备，应用程序借助socket就能方便地与其他应用程序进行交流。

**socket的域**

在上面的程序中我们建立socket对象都是使用了AF_INET这个参数，它表示这个socket是通过IPV4的方式进行通信的。

**socket的协议**

在protocol上我们使用了SOCK_STREAM，表示这是个流式套接字（即TCP），除此之外我们还可以把它指定为SOCK_DGRAM，表示这是个数据报套接字（即UDP）。

**socket的通道**

一般来说，socket的信道是双向的，即一个socket既能读又能写。有时候你需要建立一个半开放的socket，这时候就要使用socket的shutdown调用，它接收一个标记，其中：

- SHUT_RD代表关闭连接的读端。
- SHUT_WR代表关闭连接的写端。
- SHUT_RDWR代表关闭连接的读端跟写端。



服务器端的最简化代码：

import socket as sk

sock = sk.socket(sk.AF_INET, sk.SOCK_STREAM)
sock.setsockopt(sk.SOL_SOCKET, sk.SO_REUSEADDR,1)
sock.bind(('127.0.0.1',8000))
sock.listen(5)
while 1:

​	cli_sock,cli_addr = sock.accept()
​	req = cli_sock.recv(4096)
​	cli_sock.send(b'hello word')
​	cli_sock.close()

客户端的代码最简形式：

import socket

sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('127.0.0.1',8000))
sock.send(b'GET / HTTP/1.1\r\nHost:127.0.0.0.1\r\n\r\n')
data = sock.recv(4096)
print(data)
sock.close()

result:
wing@wing-virtual-machine:~/Code/Python$ python3 client_http.py 
b'hello word'

过程：

**服务器端：**

1. 调用socket.socket建立一个socket对象，指定域(domain)和协议(protocol)，此时一个文件描述符会绑定到这个socket对象。
2. 调用sock.setsockopt设置这个socket选项，本例中把socket.SO_REUSEADDR设置为1，表示服务器端进程终止后，操作系统会为它绑定的端口保留一段时间，以防其他进程在它结束后抢占这个端口。
3. 调用sock.bind为这个socket对象绑定到一个地址上，它需要一个主机地址和端口组成的元组作为参数。
4. 调用sock.listen通知系统开始侦听来自客户端的连接，参数是在队列中最大的未决连接数量。
5. 调用sock.accept阻塞调用直至返回一个元组，里面包含了用于与客户端进行对话的socket对象以及客户端的地址信息。
6. 调用cli_sock.recv方法接受来自客户端发来的数据，在这个例子中拿到的是**b’GET / HTTP/1.1\r\nHost: 127.0.0.1:8000\r\n\r\n’**。
7. 调用cli_sock.send方法把数据发送给客户端。
8. 调用cli_sock.close结束连接。

**客户端：**

1. 调用socket.socket建立一个socket对象，指定域(domain)和协议(protocol)，此时一个文件描述符会绑定到这个socket对象。
2. 调用sock.connect通过指定的主机和端口连接到对端的服务器进程。
3. 调用sock.send给服务器端发送数据。
4. 调用sock.recv接收服务器端发来的数据。
5. 调用sock.close关闭连接。



五种IO模型：

**阻塞IO模型**

recv->无数据报准备好->等待数据->数据报准备好->数据从内核复制到用户空间->复制完成->返回成功指示

**非阻塞IO模型**

recv->无数据报准备好->返回EWOULDBLOCK->recv->无数据报准备好->返回EWOULDBLOCK->数据报准备好->数据从内核复制到用户空间->复制完成->返回成功指示

特点：轮询操作，大量占用cpu时间。

**IO复用模型**

select->无数据报准备好->据报准备好->返回可读条件->recv->数据从内核复制到用户空间->复制完成->返回成功指示

**信号驱动模型**

建立信号处理程序(sigaction)->递交SIGIO->recv->数据从内核复制到用户空间->复制完成->返回成功指示

**异步IO模型**

aio_read->无数据准备好->数据报准备好->数据从内核复制到用户空间->复制完成->递交aio_read中指定的信号

特点：直到数据复制完成产生信号的过程中进程都不被阻塞。



用一张图总结5个IO模型是这样的：

![img](https://pic1.zhimg.com/80/v2-7e2c42ee86f13913646d13bc65582436_hd.jpg)





**HTTP**

以浏览器输入一个网址打开为例，看HTTP的请求过程：

1. 浏览器首先从URL中解析出主机名，端口等信息，URL的通用格式为：**://:@:/;?#**。
2. 浏览器把主机名转换为IP地址（DNS）。
3. 浏览器与服务器建立一条TCP连接。
4. 浏览器在TCP连接上发送一条HTTP请求报文。
5. 服务器在TCP连接上返回一条HTTP响应报文。
6. 关闭连接，浏览器渲染文档。

HTTP的请求信息包括几个要素：

1. 请求行，例如**GET /index.html HTTP/1.1**，表示要请求index.html这个文件。
2. 请求头（首部）。
3. 空行。
4. 消息体。



**DNS**

主机到IP的转换通常要经过DNS查询，DNS是一个庞大的分布式数据库，它将主机名组织在一个层级的空间中，一个节点的域名由该节点到根的路径所有节点组成的名字连接而成。

![img](https://pic4.zhimg.com/80/v2-4575b995a28ba241392f8058cb7c4c9e_hd.jpg)

使用dnspython包可以方便地进行dns查询，代码如下：

import dns.resolver

domain = 'baidu.com'
A = dns.resolver.query(domain,'A')
for answer in A.response.answer:

​	for item in answer.items:

​		print(item.address)

result：

wing@wing-virtual-machine:~/Code/Python$ python3 DNS_http.py 
123.125.115.110
220.181.57.216



操作系统通过对四元组来为主动TCP连接命名的，接收到TCP数据包，操作系统会检测他们的源地址和目标地址是否与系统中的某一主动套接字相符。

唯一标识主动套接字的四元组：（local_ip, local_port, remote_ip, remote_port）

下面直接上代码，这是一个简单的TCP客户端和服务器模型：

import argparse,socket

def recvall (sock,length):
    data = b''
    while len(data) < length:
        more = sock.recv(length - len(data))
        if not more:
            raise EOFError('was expecting %d bytes but only received'
                            ' %d bytes before the socket closed'
                            % (length,len(data)))
        data += more
    return data

def server(interface,port):
    
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR,1)
    sock.bind((interface, port))
    sock.listen(1)
    print('listening at',sock.getsockname)
    while True:
        print('waiting to accept a new connection')
        sc, sockname = sock.accept()
        print('we have accepted a connection from',sockname)print('socket name', sc.getsockname())
            print('socket peer', sc.getpeername())
            message = recvall(sc, 16)
            print('incoming sixteen-octet message:',repr(message))
            sc.sendall(b'farewell,client')
            sc.close()
            print('reply sent, socket closed')
    
    def client(host,port):
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.connect((host, port))
        print('client has been assigned socket name',sock.getsockname())
        sock.sendall(b'Hi there, server')
        reply = recvall(sock, 15)
        print('The server said',repr(reply))
        sock.close()
    
    if __name__ == '__main__':
        choices = {'client':client,'server':server}
        parser = argparse.ArgumentParser(description = 'Send and receive over TCP')
        parser.add_argument('role',choices = choices, help = 'which role to play')
        parser.add_argument('host',help = 'interface the server listens at;'
                            'host the client sends to')
        parser.add_argument('-p',metavar='PORT',type = int,default = 1060,
                            help = 'TCP port (default 1060)')
        args = parser.parse_args()
        function = choices[args.role]
        function(args.host,args.p)
    



server：

wing@wing-virtual-machine:~/Code/Python$ python3 TCP_protocol.py server 127.0.0.1
listening at <built-in method getsockname of socket object at 0x7fbfcc97d048>
waiting to accept a new connection
we have accepted a connection from ('127.0.0.1', 37056)
socket name ('127.0.0.1', 1060)
socket peer ('127.0.0.1', 37056)
incoming sixteen-octet message: b'Hi there, server'
reply sent, socket closed
waiting to accept a new connection



client：

wing@wing-virtual-machine:~/Code/Python$ python3 TCP_protocol.py client 127.0.0.1
client has been assigned socket name ('127.0.0.1', 37056)
The server said b'farewell,client'












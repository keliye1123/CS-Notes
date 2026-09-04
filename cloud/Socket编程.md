# Socket编程

- [Socket编程](#Socket编程)
  - [一、基础数据结构](#一基础数据结构)
  - [二、Linux的socket编程](#二Linux的socket编程)
    - [socket编程的本质：文件操作](#socket编程的本质文件操作)
    - [socket函数详解](#socket函数详解)
    - [主机字节序与网路字节序](#主机字节序与网路字节序)
    - [网络通信流程](#网络通信流程)
    - [客户端与服务端封装例子](#客户端与服务端封装例子)
  - [三、windows的socket编程](#三windows的socket编程)


## 一、基础数据结构

## 1.sockaddr
`sockaddr` 是**通用基类地址结构体**，不常被定义使用而是用来被强转使用,客户端的connect()函数和服务端的bind()函数需要这个结构体。
~~~cpp
struct sockaddr {
    sa_family_t sa_family;//地址族
    char        sa_data[14];//存放协议专属地址（端口+ip）
};
~~~

### 2.sockaddr_in
`sockaddr_in`是IP地址和port捆绑关系的数据结构，sockaddr结构体是为了统一地址结构的表示方法，统一接口函数，但是，操作不方便，所以定义了
等价的sockaddr_in结构体，它的大小与sockaddr相同，作为参数调用时要强制转换成sockaddr。
~~~cpp
struct sockaddr_in {
    unsigned short sin_family; //AF_INET（地址族）
    unsigned short sin_port; // port（端口号）
    struct in_addr sin_addr ;// IP address, unsigned long（IP地址，IP地址不是唯一的，一台电脑如果有不同网卡，就会有不同的ip地址）
    char sin_zero[8]; // align（用于对齐结构体大小，方便强转为sockaddr类型）
};
struct in_addr {
  unsigned int s_addr;//32位的地址。
}
~~~
- 注意：给sin_port赋值时要用htons(port_)转为网络字节序。
- 给sin_addr赋值的三种种例子:
~~~
//例子1
struct hostent* h;  //用于存放IP的数据结构。
h=gethostbyname("localhost");
struct sockaddr_in server_addr{};
memcpy(&server_addr.sin_addr,h -> h_addr,h -> h_length);
//例子2
server_addr.sin_addr.s_addr = htonl(INADDR_ANY);    //INADDR_ANY表示本地任意网卡的ip都能使用
//例子3
server_addr.sin_addr.s_addr = inet_addr("192.168.1.10");//输入指定ip要用inet_addr()函数。
/*
- INADDR_ANY:与(0.0.0.0)等价
- 127.0.0.1:只监听本机回环网卡只有自己本机程序能访问，外部设备连不上
*/
~~~

### 3.struct hostent
~~~
struct hostent {
    char  *h_name;        // 主机正式规范域名
    char **h_aliases;     // 别名数组，以NULL结尾
    int    h_addrtype;    // 地址族 AF_INET（IPv4）
    int    h_length;      // IP地址字节长度（IPv4=4）
    char **h_addr_list;   // IP列表，**网络字节序二进制IP**
};
#define h_addr h_addr_list[0] // 取第一个IP地址（兼容宏）
~~~


## 二、Linux的socket编程

### socket编程的本质：文件操作

#### 1.文件描述符/句柄
在Linux中操作文件时，会给将一个int型数字作为文件的句柄，后面操作文件都要通过这个数字。
- 代码示例
```
//demo3.cpp，本程序演示了Linux底层文件的操作-创建文件并写入数据。
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>
int main() {
    int fd;//定义一个文件描述符/文件句柄

    fd=open("data.txt",O_CREAT|O_RDWR|O_TRUNC); // 打开文件。
    if (fd==-1) {
        perror ("open(data. txt) ") ;
        return -1;
    }
    printf("文件描述符fd=%d\n",fd);

    char buffer[1024];
    strcpy(buffer,"我是一只傻傻鸟。\n");

    if (write(fd,buffer, strlen(buffer))==-1) {//把数据写入文件。
        perror("write() ") ;
        return -1;
    }
    sleep(100);
    close(fd); // 关闭文件。
}
``` 
- 输出结果
![cloud_image1.png](Image/cloud/cloud_image1.png)

#### 2.文件描述符/句柄的分配规则
- 在/proc/进程id/fd 中，存放着每个进程打开的fd文件描述符/句柄,当我们运行demo3时，可打开其fd，如下图操作
![cloud_image2.png](Image/cloud/cloud_image2.png)
我们使用`ls fd`可以看到4个文件描述符，分别是0,1,2,3,3是我们打开文件时创建的文件描述符。 Linux进程默认打开了三个文件描述符：0-标准输入（键盘），1-标准输出（显示器），2-标准错误（显示器）。
正是因为Linux打开了这三个文件描述符，我们才能调用cin，cout，cerr进行输入和输出。
- 测试代码
```
#include <unistd.h>
#include <iostream>
int main() {
    close(0);   //关闭输入文件的句柄
    int a = 0;
    std::cin >> a;
    std::cout <<"cout:"<< a << std::endl;
    std::cerr <<"cerr:"<< a << std::endl;
    sleep(100);
    return 0;
}
```  
- 输出结果
![cloud_image3.png](Image/cloud/cloud_image3.png)
![cloud_image4.png](Image/cloud/cloud_image4.png)
会直接输出，不会给你输入的机会。当我们查看其打开的文件描述符时，0就没有了。

- 测试代码 
```  
//demo3.cpp，本程序演示了Linux底层文件的操作-创建文件并写入数据。
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>
int main() {
    close(0);   //关闭输入
    int fd;//定义一个文件描述符/文件句柄。

    fd=open("data.txt",O_CREAT|O_RDWR|O_TRUNC); // 打开文件。
    if (fd==-1) {
        perror ("open(data. txt) ") ;
        return -1;
    }
    printf("文件描述符fd=%d\n",fd);

    char buffer[1024];
    strcpy(buffer,"我是一只傻傻鸟。\n");

    if (write(fd,buffer, strlen(buffer))==-1) {//把数据写入文件。
        perror("write() ") ; return -1;
    }
    sleep(100);
    close(fd); // 关闭文件。
}
```  
- 输出结果
![cloud_image5.png](Image/cloud/cloud_image5.png)
![cloud_image6.png](Image/cloud/cloud_image6.png)
- 由图可知本来为输入文件描述符的0变为了我们创建文件的文件描述符。
- 文件描述符的分配规则是：找到最小的，没有被占用的文件描述符。


#### 3.文件与socket的区别
- socket和文件本质是一样，都可以用write和read进行读写操作。
- 服务端测试代码(测试代码后面会讲，现在只用于演示)
```  
/*
*程序名：client.cpp，此程序用于演示socket的客户端
*/
#include <iostream>
#include <cstdio>
#include <cstring>
#include <cstdlib>
#include <unistd.h>
#include <netdb.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
using namespace std;

int main(int argc,char *argv[]) {
    if (argc!=3) {
        cout << "Using:./demo1 服务端的IP 服务端的端口\nExample:./demo1 8.156.69.189 5005\n\n";
        return -1 ;
    }

    //第1步：创建客户端的socket。
    int socket_c = socket(AF_INET,SOCK_STREAM,0);
    if (socket_c==-1) {
        perror("socket");
        return -1;
    }

    //第二步：向服务器发起请求。
    struct hostent* h;  //用于存放服务端IP的数据结构。
    if ((h=gethostbyname(argv[1])) == nullptr) {    //指定服务端的ip地址
        cout << "gethostbyname failed.\n" << endl;
        close(socket_c);
        return -1;
    }
    struct sockaddr_in server_addr; //用于存放服务端的IP地址。
    memset(&server_addr,0,sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    memcpy(&server_addr.sin_addr,h -> h_addr,h -> h_length);    //指定服务端IP地址
    server_addr.sin_port = htons(atoi(argv[2]));    //指定服务端的通信端口
    if (connect(socket_c,(struct sockaddr*)&server_addr,sizeof(server_addr)) != 0) {//向服务端发起连接请求
        perror("connect");
        close(socket_c);
        return -1;
    }

    //第三步：与服务端通信，客户发送一个请求报文后等待服务端的恢复，收到回复后，再发下一个请求报文
    char buffer[1024];  //存储报文内容
    for (int i = 0;i < 3;i++) {
        int iret;
        memset(buffer,0,sizeof(buffer));
        sprintf(buffer,"this is the %d",i+1);   //生成报文内容
        //向服务端发送请求报文
        //用send函数发送报文可以改为write
        if ((iret = write(socket_c,buffer,strlen(buffer)))<=0) {
        // if ((iret = send(socket_c,buffer,strlen(buffer),0))<=0) {
            perror("send");
            break;
        }
        cout << "send:"<< buffer << endl;

        memset(buffer,0,sizeof(buffer));
        // 接受服务端的回应报文，如果服务端没有发送回应报文，recv()函数阻塞等待。
        //用recv接收报文可以改为read
        if ((iret = read(socket_c,buffer,sizeof(buffer))) <= 0) {
        // if ((iret = recv(socket_c,buffer,sizeof(buffer),0)) <= 0) {
            cout << "iret="<< iret << endl;
            break;
        }
        cout <<"接受："<<buffer << endl;
        sleep(1);
    }
    //第四步：关闭socket，释放资源
    close(socket_c);
}
```
- 测试结果  
![cloud_image7.png](Image/cloud/cloud_image7.png)
用write和read也是可以正常通信的

#### 4.总结
- /proc/进程id/fd目录中，存放了每个进程打开的fd。
- Linux进程默认打开了三个文件描述符：0-标准输入（键盘），1-标准输出（显示器），2-标准错误（显示器）。cin cout cerr
- 文件描述符的分配规则是：找到最小的，没有被占用的文件描述符。
- 对Linux来说，socket操作与文件操作没有区别。
- 在网络传输数据的过程中，可以使用文件的I/O函数。
- 文件描述符是Linux分配给文件或socket的整数。


### socket函数详解

#### 1.创建socket
~~~cpp
//创建socket函数
int socket(int domain,int type,int protocol);
~~~
socket创建成功返回一个有效的socket，失败则返回-1，全部网络编程的函数，失败时基本上都是返回-1。
- 1.domain：通信的协议家族
  - AF_INET :IPv4互联网协议族
  - AF_INET6:IPv6互联网协议族
  - AF_LOCAL:本地通信协议族
  - AF_PACKET：内核底层的协议族
  - AF_IPX:IPX Novell协议族

- 2.type:数据传输的类型
  - SOCK_STREAM:面向连接的socket，tcp套接字
  - SOCK_DGRAM:无连接的socket，udp套接字

- 3.protocol：最终使用的协议
  - 在IPv4 网络协议家族中，数据传输方式为SOCK STREAM的协议只有IPPROTO_TCP，数据传输方式为SOCK_DGRAM的协议只有IPPROTO_UDP。
  - 本参数可以填0，编译器可以自动识别
  - 例子：socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);// 创建 tcp 的 sock
  - 例子：socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);// 创建 udp 的 sock  
  
#### 2.发送连接请求
~~~cpp
//发送连接请求
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
~~~
- sockefd:本地要发送连接请求socket的句柄
- *addr:存了对端服务器地址信息的指针，通常由struct sockaddr_in强转而来。
- addrlen:地址结构体的字节大小，**直接写 `sizeof(struct sockaddr_in)`**
- 返回值:`0`：成功（TCP 完成三次握手）,`-1`：失败，`errno` 保存错误码（`ECONNREFUSED` 拒绝连接、`ETIMEDOUT` 超时）。
#### 3.发送报文
~~~cpp
int send(int sockfd, const void *buf, size_t len, int flags);
int write(int sockfd, const void *buf, size_t len);//与send在flags = 0时作用一样
~~~
- sockfd:本地要发送报文的socket的句柄
- *buf:地址，通常是char*类型的首地址以'/0'结尾
- len:打算发送多少字节（buf 中有效数据长度）
- int flags:发送标志，填 0 = 默认行为（和 write 基本一样）
- 返回值:**>0**：实际成功发送的字节数,**0**：连接优雅关闭,**-1**：出错，errno 保存错误码.

#### 4.接受报文
~~~cpp
int recv(int sockfd, void *buf, size_t len, int flags);
int read(int sockfd, void *buf, size_t len);//与recv的flags=0时作用相同
~~~
- sockfd:本地要接受报文的socket的句柄
- *buf:地址，通常是char*类型的首地址以'/0'结尾
- len:缓冲区最多能接收多少字节（buf 的可用字节大小）
- flags:接收选项，日常默认填0
- 返回值:0>：本次实际读到的字节数,=0:对端正常关闭连接（TCP 断开）,**-1**：出错，查看 `errno`.

#### 5.将ip和端口绑定到socket上
~~~cpp
bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
~~~
- sockfd:本地要绑定的socket的句柄，服务端必须 bind；客户端一般不用手动 bind，操作系统自动分配随机端口。
- *addr:**本机要绑定的地址（IP + 端口）日常使用定义struct sockaddr_in填充，再强制转型
- addrlen:地址结构体的大小，一般写sizeof(struct sockaddr_in)
- 返回值:0：绑定成功,`-1`：失败，查看errno。

#### 6.关闭socket
~~~cpp
int close(sockfd);
~~~
- sockfd:本地要关闭的socket的句柄
- 返回值:0：关闭成功,-1：失败，errno 保存错误码

#### 7.受理连接
~~~cpp
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
~~~
- sockfd:等待接受连接请求的socket
- addr:2. 用来保存客户端的地址信息（IP + 端口）（输出参数）。
- addrlen:调用前：赋值为地址结构体的大小sizeof(struct sockaddr_in),函数返回后：内核修改它，存入实际客户端地址的字节长度,如果第二个参数 addr 是 NULL，这个参数也填NULL。
- 返回值:成功：返回一个全新的 int 文件描述符 connfd，这个connfd才是用来和这个客户端send/recv通信的套接字；监听 sockfd 继续等待下一个连接，失败：返回 -1，errno 记录错误。

#### 8.gethostbyname函数
用域名/主机名/字符串IP获取大端序IP，用于网络通信的客户端程序中。
~~~cpp
struct hostent *gethostbyname(const char *name);
~~~
- *name:传入：域名 / 主机名字符串，例 "www.baidu.com"、"localhost",也可以直接传入点分十进制 IPv4 字符串。
- 返回值:**成功**：返回 `struct hostent *` 静态缓冲区指针,**失败**：返回 `NULL`。



### 主机字节序与网路字节序

#### 1.大端序/小端序
如果数据类型占用的内存空间大于1字节，CPU把数据存放在内存中的方式有两种：
- 大端序（Big Endian）：低位字节存放在高位，高位字节存放在低位。
- 小端序（Little Endian）：低位字节存放在低位，高位字节存放在高位。

Intel 系列的 CPU 以小端序方式保存数据，其它型号的CPU 不一定。操作文件的本质是把内存中的数据写入磁盘，在网络编程中，传输数据的本质也是把数据写入文件
(socket 也是文件描述符)。这样的话，字节序不同的计算机之间传输数据，可能会出现问题。

#### 2.网络字节序
为了解决不同字节序的计算机之间传输数据的问题，约定采用网络字节序（大端序）。
- C语言提供了四个函数，用于在主机字节序和网络字节序之间的转换：
  - uint16_t htons(uint16_t hostshort); // uint16_t 2 字节的整数 unsigned short
  - uint32_t htonl(uint32_t hostlong);     // uint32_t 4字节的整数 unsigned int
  - uint16_t ntohs(uint16_t netshort);
  - uint32_t ntohl(uint32_t netlong);

- h：host(主机)；
- to：转换；
- n：network(网络）；
- S：short(2字节，16 位的整数)；
- l:long(4字节，32位的整数)；

#### 3.如何处理大小端
在网络编程中，数据收发的时候有自动转换机制，不需要程序员手动转换，只有向sockaddr_in结
体成员变量填充数据时，才需要考虑字节序的问题。

### 网络通信流程
![cloud_image8.png](Image/cloud/cloud_image8.png)

### 客户端与服务端封装例子
#### 1.封装客户端socket
```cpp  
/*
*程序名：client.cpp，此程序用于演示socket的客户端
*/
// Using:./demo1 服务端的IP 服务端的端口\nExample:./demo1 8.156.69.189 5005\n\n
#include <iostream>
#include <cstring>
#include <cstdlib>
#include <unistd.h>
#include <netdb.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
using namespace std;

class TcpClient {       //TCP通信的客户端类
    //属性
private:
    int socket_c_ = -1;       //客户端的socket，-1表示未连接或连接已断开，>=0表示有效的socket
    std::string ip_;     //服务端的IP/域名/本地设置的dns
    unsigned short port_ = 0;//通讯端口

    //方法
public:

    //关闭Socket
    void CloseSocket_() const {
        close(socket_c_);
    }

    //向服务端发送连接请求，成功返回true，失败返回false
    bool Connect_(const std::string &ip,const unsigned short& port) {
        if (socket_c_ >= 0) {//如果socket已连接，直接返回失败
            std::cout << "错误：客户端socket已被连接" << std::endl;
            return false;
        }

        ip_ = ip;
        port_ = port;

        //第1步：创建客户端的socket。
        if (( socket_c_ = socket(AF_INET,SOCK_STREAM,0)) == -1) {
            std::cout << "错误：客户端socket创建失败" << std::endl;
            return false;
        }

        //第二步：向服务器发起请求。
        struct hostent* h;  //用于存放服务端IP的数据结构。
        if ((h=gethostbyname(ip_.data())) == nullptr) {    //指定服务端的ip地址
            close(socket_c_);
            socket_c_ = -1;
            std::cout << "错误：客户端ip地址无效" << std::endl;
            return false;
        }
        struct sockaddr_in server_addr{}; //用于存放服务端的IP地址。
        memset(&server_addr,0,sizeof(server_addr));
        server_addr.sin_family = AF_INET;
        memcpy(&server_addr.sin_addr,h -> h_addr,h -> h_length);    //指定服务端IP地址
        server_addr.sin_port = htons(port_);    //指定服务端的通信端口
        if (connect(socket_c_,(struct sockaddr*)&server_addr,sizeof(server_addr)) == -1) {//向服务端发起连接请求
            close(socket_c_);
            socket_c_ = -1;
            std::cout << "错误：客户端连接到服务器失败" << std::endl;
            return false;
        }
        return true;
    }

    [[nodiscard]] bool Send_(const std::string &buffer) const{
        if (socket_c_ == -1) { //如果socket是未连接状态，直接返回false
            std::cout << "错误：客户端socket是未连接状态" << std::endl;
            return false;
        }

        //第三步：与服务端通信，客户发送一个请求报文后等待服务端的恢复，收到回复后，再发下一个请求报文
        //向服务端发送请求报文
        //用send函数发送报文可以改为write
        // if (send(socket_c_,buffer.data(),buffer.size(),0)<=0)
        if (send(socket_c_,buffer.data(),buffer.size(),0) == -1) {
            std::cout << "错误：客户端发送报文失败" << std::endl;
            return false;
        }
        return true;
    }

    //buffer接受报文
    bool Receive_(std::string &buffer,size_t maxlen) const {
        if (socket_c_ == -1) { //如果socket是未连接状态，直接返回false
            std::cout << "错误：客户端socket是未连接状态" << std::endl;
            return false;
        }

        //第四步，接收
        buffer.clear();
        buffer.resize(maxlen);
        // if (read(socket_c_,buffer.data(),buffer.size()) <= 0)
        //recv失败返回-1，socket断开连接返回0，成功返回字符串长度，只能用&buffer[0],buffer.data()和buffer.str_c()都不能修改本身
        int readn = recv(socket_c_,&buffer[0],buffer.size(),0);
        if (readn <= 0) {
            buffer.clear();
            close(socket_c_);
            return false;
        }
        buffer.resize(readn);
        return true;
    }


};

int main(int argc,char *argv[]) {
    if (argc!=3) {
        cout << "Using:./demo1 服务端的IP 服务端的端口\nExample:./demo1 8.156.69.189 5005\n\n";
        return -1 ;
    }

    TcpClient client;
    if (!client.Connect_(argv[1],atoi(argv[2]))) return -1;

    for (int i = 0;i < 1;i++) {
        std::string buffer;
        buffer = "这是第"+std::to_string(i+1)+"个消息";

        if (!client.Send_(buffer)) return -1;
        std::cout <<"发送："<< buffer << std::endl;

        std::string res;
        if (!client.Receive_(res,10)) return -1;
        std::cout <<"收到："<< res << std::endl;
        sleep(1);
    }

    client.CloseSocket_();
}
```  

#### 2.封装服务端socket
```cpp  
/*
*程序名：serve.cpp，此程序用于演示socket通信的服务端
*/
#include <iostream>
#include <cstdio>
#include <cstring>
#include <cstdlib>
#include <unistd.h>
#include <netdb.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
using namespace std;


class TcpServe {       //TCP通信的服务端类

    //属性
private:
    int socket_s_ = 0;       //服务端的socket，-1表示未连接或连接已断开，>=0表示有效的socket
    int socket_c_ = 0;       //客户端的socket，-1表示未连接或连接已断开，>=0表示有效的socket
    unsigned short port_ = 0;//通讯端口
    std::string client_ip_;  //存放客户端Ip

    //方法
public:
    std::string GetClientIP_() const {
        return client_ip_;
    }

    bool Listen_(const unsigned short& port) {
        //第一步创建服务端socket
        socket_s_ = socket(AF_INET, SOCK_STREAM, 0);
        if (socket_s_ == -1) {
            std::cout << "错误：服务端socket创建失败" << std::endl;
            return false;
        }

        port_ = port;

        //第二步：把服务端用于通信的ip和端口绑定到socket上
        //把服务端用于通信的ip地址和端口绑定到socket上
        struct sockaddr_in server_addr;     //用于存放服务端ip地址和端口的数据结构
        memset(&server_addr,0,sizeof(server_addr));
        server_addr.sin_family = AF_INET;   //指定协议
        server_addr.sin_addr.s_addr = htonl(INADDR_ANY);    //服务端任意网卡的ip都可以用于通信
        server_addr.sin_port = htons(port_);//指定通信端口，普通用户只能用1024以上的端口
        //绑定服务端的ip和端口
        //绑定成功返回0，失败返回-1
        if (bind(socket_s_,(struct sockaddr*)&server_addr,sizeof(server_addr)) == -1 ) {
            close(socket_s_);
            std::cout << "错误：服务端绑定ip与端口失败" << std::endl;
            return false;
        }

        //第三步：把socket设置为可连接（监听）的状态。
        //返回-1错误，返回0为成功监听
        if (listen(socket_s_,5) == -1) {
            std::cout << "错误：服务端监听失败" << std::endl;
            close(socket_s_);
            return false;
        }

        return true;
    }

    bool Accept_() {
        //第四步：受理客户端的连接请求，如果没有客户端连上来，accept()函数将阻塞等待。
        struct sockaddr_in client_addr;
        socklen_t client_addr_len = sizeof(client_addr);
        socket_c_=accept(socket_s_,(struct sockaddr*)&client_addr,&client_addr_len);//接收客户端的IP地址
        client_ip_ = inet_ntoa(client_addr.sin_addr);

        if (socket_c_ == -1) {
            close(socket_s_);
            std::cout << "错误：服务端报文接收失败" << std::endl;
            return false;
        }

        return true;
    }

    bool Receive_(std::string &ac,unsigned int maxlen) {
        if (socket_c_ == -1) return false;
        //接收客户端的请求报文，如果客户端没有发送请求报文，recv()函数将阻塞等待。
        //如果客户端已断开连接，recv()函数将返回0.
        //recv收到消息返回消息长度，网络异常返回-1,四次挥手正常断开连接返回0
        ac.clear();
        ac.resize(maxlen);
        int readn = recv(socket_c_,&ac[0],maxlen,0);
        if (readn <= 0) {
            ac.clear();
            return false;
        }
        ac.resize(readn);
        return true;
    }

    bool Send_(std::string rs) {
        if (socket_c_ == -1) return false;
        //向客户端发送回应报文。
        //send返回-1为失败，大于等于0为写入字节数，
        if (send(socket_c_,rs.data(),rs.size(),0) == -1) return false;
        return true;
    }

    void CloseSocket_() const{
        //第六步：关闭socket，释放资源
        close(socket_c_);    //关闭客户端连上来的socket
        close(socket_s_);    //关闭服务端用于监听的socket
    }

};

int main(int argc,char *argv[]) {
    if (argc!=2) {
        cout << "Using:./demo2 通讯端口\nExample:./demo2 5005\n\n";
        cout <<"注意：运行服务端程序的Linux系统的防火墙必须要开通5005端口。\n";
        cout <<"如果是云服务器，还要开通云平台的访问策略。\n\n";
        return -1;
    }

    TcpServe serve;

    serve.Listen_(atoi(argv[1]));

    while (true) {
        //受理客户端的连接，如果没收到连接则阻塞
        if ( serve.Accept_() == false) {
            perror("accept error");
            return -1;
        }

        int pid = fork();
        if (pid == -1) {
            perror("fork error");
            return -1;
        }
        if (pid == 0) continue;

        std::string ac;
        std::string rs = "hello!";
        while (true) {
            //接收报文，如果未发送报文则阻塞，发送报文返回true，发送四次挥手报文则返回false
            if (serve.Receive_(ac,1024) == false) break;

            std::cout << "收到："<< ac << std::endl;
            serve.Send_(rs);
            std::cout << "发送："<< rs << std::endl;
            std::cout << "主机ip地址为："<< serve.GetClientIP_() << std::endl;
        }
        serve.CloseSocket_();

    }



}  
```

## 三、windows的socket编程
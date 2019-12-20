 ![img](https://img2018.cnblogs.com/blog/1169746/201809/1169746-20180929212723880-1930317638.png) 

**客户端工作流程：**

- 调用 socket() 函数创建套接字
- 调用 connect() 函数连接服务端
- 调用write()/read() 函数或者 send()/recv() 函数进行数据的读写
- 关闭socket(close)

**服务器端工作流程：**

- 调用 socket() 函数创建套接字 用 bind() 函数将创建的套接字与服务端IP地址绑定
- 调用listen()函数监听socket() 函数创建的套接字，等待客户端连接 当客户端请求到来之后
- 调用 accept()函数接受连接请求，返回一个对应于此连接的新的套接字，做好通信准备
- 调用 write()/read() 函数和 send()/recv()函数进行数据的读写，通过 accept() 返回的套接字和客户端进行通信 关闭socket（close）

 ![img](https://img-blog.csdnimg.cn/20190225144200721.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIyNjEzNzU3,size_16,color_FFFFFF,t_70) 



##### 1、socket函数

  int socket(int domain, int type, int protocol);

 功能说明：
 调用成功，返回socket文件描述符；失败，返回－1，并设置errno

 参数说明：

 　domain指明所使用的协议族，通常为PF_INET，表示TCP/IP协议；

 　type参数指定socket的类型，基本上有三种：数据流套接字、数据报套接字、原始套接字

 　protocol通常赋值"0"。

 　两个网络程序之间的一个网络连接包括五种信息：通信协议、本地协议地址、本地主机端口、远端主机地址和远端协议端口。socket数据结构中包含这五种信息。

##### 2、bind函数

  int bind(int sock_fd,struct sockaddr_in *my_addr, int addrlen);

 功能说明：

 将套接字和指定的端口相连。成功返回0，否则，返回－1，并置errno.

 参数说明：

 sock_fd是调用socket函数返回值,

 　my_addr是一个指向包含有本机IP地址及端口号等信息的sockaddr类型的指针；

 　struct sockaddr_in结构类型是用来保存socket信息的：

 　struct sockaddr_in {

 　short int sin_family;

 　unsigned short int sin_port;

 　struct in_addr sin_addr;

 　unsigned char sin_zero[8];

 　};

  addrlen为sockaddr的长度。

 ##### 3、connect函数

   int connect(int sock_fd, struct sockaddr *serv_addr,int addrlen);
 客户端发送服务请求。成功返回0，否则返回－1，并置errno。

 参数说明：

  sock_fd
 是socket函数返回的socket描述符；serv_addr是包含远端主机IP地址和端口号的指针；addrlen是结构sockaddr_in的长度。

##### 4、listen函数

  int listen(int sock_fd， int backlog);

 功能说明：


 等待指定的端口的出现客户端连接。调用成功返回0，否则，返回－1，并置errno.

 参数说明：

  sock_fd 是socket()函数返回值；

  backlog指定在请求队列中允许的最大请求数

##### 5、accept函数

  int accept(int sock_fd, struct sockadd_in* addr, int addrlen);

 功能说明：


  用于接受客户端的服务请求，成功返回新的套接字描述符，失败返回－1，并置errno。

 参数说明：

  sock_fd是被监听的socket描述符，
 addr通常是一个指向sockaddr_in变量的指针，


  addrlen是结构sockaddr_in的长度。

##### 6、write函数

   ssize_t write(int fd,const void *buf,size_t nbytes)

 功能说明：
 write函数将buf中的nbytes字节内容写入文件描述符fd.成功时返回写的字节数.失败时返回-1. 并设置errno变量.
 在网络程序中,当我们向套接字文件描述符写时有俩种可能：

 1)write的返回值大于0,表示写了部分或者是全部的数据.

 2)返回的值小于0,此时出现了错误.需要根据错误类型来处理.


 如果错误为EINTR表示在写的时候出现了中断错误.


 如果错误为EPIPE表示网络连接出现了问题.

 ##### 7、read函数

   ssize_t  read(int fd,void *buf,size_t nbyte)

 函数说明：
 read函数是负责从fd中读取内容.当读成功时,read返回实际所读的字节数,如果返回的值是0
 表示已经读到文件的结束了,小于0表示出现了错误.

 如果错误为EINTR说明读是由中断引起的,
 如果错误是ECONNREST表示网络连接出了问题.

 ##### 8、close函数

 int close(sock_fd);

 说明:

 当所有的数据操作结束以后，你可以调用close()函数来释放该socket，从而停止在该socket上的任何数据操作：

 函数运行成功返回0，否则返回-1

二、socket编程的其他函数说明
 1、 网络字节顺序及其转换函数
 1） 网络字节顺序
 每一台机器内部对变量的字节存储顺序不同，而网络传输的数据是一定要统一顺序的。所以对内部字节表示顺序与网络字节顺序不同的机器，

一定要对数据进行转换，从程序的可移植性要求来讲，就算本机的内部字节表示顺序与网络字节顺序相同也应该在传输数据以前先调用数据转换函数，

以便程序移植到其它机器上后能正确执行。真正转换还是不转换是由系统函数自己来决定的。

 2） 有关的转换函数

- unsigned short int htons（unsigned short int hostshort）：

   主机字节顺序转换成网络字节顺序，对无符号短型进行操作4bytes

- unsigned long int htonl（unsigned long int hostlong）：

   主机字节顺序转换成网络字节顺序，对无符号长型进行操作8bytes

- unsigned short int ntohs（unsigned short int netshort）：

   网络字节顺序转换成主机字节顺序，对无符号短型进行操作4bytes

- unsigned long int ntohl（unsigned long int netlong）：

   网络字节顺序转换成主机字节顺序，对无符号长型进行操作8bytes

   注：以上函数原型定义在netinet/in.h里

   2、IP地址转换

   有三个函数将数字点形式表示的字符串IP地址与32位网络字节顺序的二进制形式的IP地址进行转换

   （1） unsigned long int inet_addr(const char *
   cp)：该函数把一个用数字和点表示的IP地址的字符串转换成一个无符号长整型，如：struct sockaddr_in
   ina

   ina.sin_addr.s_addr=inet_addr(“202.206.17.101”)

   该函数成功时：返回转换结果；失败时返回常量INADDR_NONE，该常量=-1，二进制的无符号整数-1相当于255.255.255.255，这是一个广播地址，所以在程序中调用iner_addr（）时，一定要人为地对调用失败进行处理。由于该函数不能处理广播地址，所以在程序中应该使用函数inet_aton（）。

（2）int inet_aton（const char * cp,struct in_addr *
 inp）：此函数将字符串形式的IP地址转换成二进制形式的IP地址；成功时返回1，否则返回0，转换后的IP地址存储在参数inp中。

（3） char * inet_ntoa（struct in-addr
 in）：将32位二进制形式的IP地址转换为数字点形式的IP地址，结果在函数返回值中返回，返回的是一个指向字符串的指针。

 3、字节处理函数

 Socket地址是多字节数据，不是以空字符结尾的，这和C语言中的字符串是不同的。Linux提供了两组函数来处理多字节数据，一组以b（byte）开头，是和BSD系统兼容的函数，另一组以mem（内存）开头，是ANSI
 C提供的函数。

 以b开头的函数有：

 （1） void bzero（void * s,int
 n）：将参数s指定的内存的前n个字节设置为0，通常它用来将套接字地址清0。

 （2） void bcopy（const void * src，void * dest，int
 n）：从参数src指定的内存区域拷贝指定数目的字节内容到参数dest指定的内存区域。

 （3） int bcmp（const void * s1，const void * s2，int
 n）：比较参数s1指定的内存区域和参数s2指定的内存区域的前n个字节内容，如果相同则返回0，否则返回非0。

 注：以上函数的原型定义在strings.h中。

 以mem开头的函数有：

 （1） void * memset（void * s，int c，size_t n）：将参数s指定的内存区域的前n个字节设置为参数c的内容。

 （2） void * memcpy（void * dest，const void * src，size_t n）：功能同bcopy（），区别：函数bcopy（）能处理参数src和参数dest所指定的区域有重叠的情况，memcpy（）则不能。

（4） int memcmp（const void * s1，const void * s2，size_t n）：比较参数s1和参数s2指定区域的前n个字节内容，如果相同则返回0，否则返回非0。

 注：以上函数的原型定义在string.h中。
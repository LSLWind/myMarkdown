### 阻塞I/O

```java
ServerSocket serverSocket = new ServerSocket(portNumber);//创建一个新的ServerSocket，用以监听指定端口上的连接请求
Socket clientSocket = serverSocket.accept();//❶ 对accept()方法的调用将被阻塞，直到一个连接建立
BufferedReader in = new BufferedReader(   
    new InputStreamReader(clientSocket.getInputStream()));
PrintWriter out =
    new PrintWriter(clientSocket.getOutputStream(), true);//❷这些流对象都派生于该套接字的流对象
String request, response;
while ((request = in.readLine()) != null) {//❸ 处理循环开始
    if ("Done".equals(request)) {
        break;   //如果客户端发送了“Done”，则退出处理循环
    }
    response = processRequest(request); //❹ 请求被传递给服务器的处理方法
    out.println(response);//服务器的响应被发送给了客户端
}  //继续执行处理循环
```

- `ServerSocket`上的`accept()`方法将会一直阻塞到一个连接建立❶，随后返回一个新的`Socket`用于客户端和服务器之间的通信。该`ServerSocket`将继续监听传入的连接。
- `BufferedReader`和`PrintWriter`都衍生自`Socket`的输入输出流❷。前者从一个字符输入流中读取文本，后者打印对象的格式化的表示到文本输出流。
- `readLine()`方法将会阻塞，直到在❸处一个由换行符或者回车符结尾的字符串被读取。
- 客户端的请求已经被处理❹。

这段代码片段将只能同时处理一个连接，要管理多个并发客户端，需要为每个新的客户端`Socket`创建一个新的`Thread`

[<img src="https://s2.ax1x.com/2020/02/15/1zRPN6.png" alt="1zRPN6.png" style="zoom:50%;" />](https://imgchr.com/i/1zRPN6)

阻塞式调用很耗资源，效率不高

### NIO

`java.nio.channels.Selector`是Java的非阻塞I/O实现的关键。它使用了事件通知API以确定在一组非阻塞套接字中有哪些已经就绪能够进行I/O相关的操作。因为可以在任何的时间检查任意的读操作或者写操作的完成状态，使一个单一的线程便可以处理多个并发的连接。


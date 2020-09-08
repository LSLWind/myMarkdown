### 基础概念

阻塞IO：IO请求时一直阻塞直到有返回结果。

非阻塞IO：即资源不可用时，IO请求立即返回

同步IO：应用阻塞在发送或接受数据的状态，直到数据成功传输或返回失败

异步IO：应用在发送或接受数据后立刻返回，实际处理时异步的

### BIO

即阻塞式IO，一般的网络编程都要使用net包与io包，BIO基本步骤为

**1.建立Socket连接**

```java
public class HttpServer {
    private boolean shutdown = false;

    public static void main(String[] args) {
        HttpServer server = new HttpServer();
        server.await();
    }

    public void await() {
        ServerSocket serverSocket = null;
        int port = 8080;
        try {
            serverSocket = new ServerSocket(port, 1, InetAddress.getByName("127.0.0.1"));
        } catch (IOException e) {
            e.printStackTrace();
            System.exit(1);
        }
        // Loop waiting for a request
        while (!shutdown) {
            Socket socket = null;
            InputStream input = null;
            OutputStream output = null;
            try {
                socket = serverSocket.accept();
                /**
                *此处执行任务
                */
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

**2.返回响应**

此处BIO即阻塞式IO，开启一个线程池，将任务交给线程池执行并返回响应，但处理IO时阻塞，会导致一次只执行一个任务

### NIO

非阻塞IO

#### Buffer缓冲区

本质是一个可以写入数据的内存块，包含在Buffer对象中，内部维护一个byte[]基本步骤为

* 将数据写入缓冲区
* 调用flip()方法转换为读取模式
* 缓冲区读取数据
* 嗲用clear()或compact()清除缓冲区

三个重要属性：

* capacity：容量，内存数据块的固定大小
* position：位置，写入模式时代表写数据的位置，读取下代表读取数据的位置
* limit：限制，写入时限制等于buffer的容量，读取下等于写入的数据量

<img src="https://i.loli.net/2020/05/13/ChTBDoP1pS8Gy5a.png" alt="image.png" style="zoom:50%;" />

```java
public class BufferDemo {
    public static void main(String[] args) {
        // 构建一个byte字节缓冲区，容量是4
        ByteBuffer byteBuffer = ByteBuffer.allocateDirect(4);
        // 默认写入模式，查看三个重要的指标
        System.out.println(String.format("初始化：capacity容量：%s, position位置：%s, limit限制：%s", byteBuffer.capacity(),
                byteBuffer.position(), byteBuffer.limit()));
        // 写入2字节的数据
        byteBuffer.put((byte) 1);
        byteBuffer.put((byte) 2);
        byteBuffer.put((byte) 3);
        // 再看数据
        System.out.println(String.format("写入3字节后，capacity容量：%s, position位置：%s, limit限制：%s", byteBuffer.capacity(),
                byteBuffer.position(), byteBuffer.limit()));

        // 转换为读取模式(不调用flip方法，也是可以读取数据的，但是position记录读取的位置不对)
        System.out.println("#######开始读取");
        byteBuffer.flip();
        byte a = byteBuffer.get();
        System.out.println(a);
        byte b = byteBuffer.get();
        System.out.println(b);
        System.out.println(String.format("读取2字节数据后，capacity容量：%s, position位置：%s, limit限制：%s", byteBuffer.capacity(),
                byteBuffer.position(), byteBuffer.limit()));

        // 继续写入3字节，此时读模式下，limit=3，position=2.继续写入只能覆盖写入一条数据
        // clear()方法清除整个缓冲区。compact()方法仅清除已阅读的数据。转为写入模式
        byteBuffer.compact(); // buffer : 1 , 3
        byteBuffer.put((byte) 3);
        byteBuffer.put((byte) 4);
        byteBuffer.put((byte) 5);
        System.out.println(String.format("最终的情况，capacity容量：%s, position位置：%s, limit限制：%s", byteBuffer.capacity(),
                byteBuffer.position(), byteBuffer.limit()));

        // rewind() 重置position为0
        // mark() 标记position的位置
        // reset() 重置position为上次mark()标记的位置

    }
}
```


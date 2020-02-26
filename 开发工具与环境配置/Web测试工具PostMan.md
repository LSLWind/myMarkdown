### 基础

**Params**：表示url上的附带传输数据

**Headers**：表示报文头部数据

* Content-Type：表示传输的数据类型
  * text/html：html页面
  * application/json：json数据

**Body**：表示报文内容，下面有多种格式

* raw：原始数据，经常用于传递json

### Restful接口测试

**Headers**的Content-Type为application/json，在**Body**中的raw写请求的json，返回json响应结果
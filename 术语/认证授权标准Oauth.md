OAuth是一个关于授权（authorization）的开放网络标准，在全世界得到广泛应用，目前的版本是2.0版。 OAuth常用于为第三方认证登录提供标准，基本流程为：

<img src="https://pic2.zhimg.com/80/5e27c9064ca22c7e54a1d395d679f8b5_hd.jpg" style="zoom: 80%;" />

上图中所涉及到的对象分别为：

- Client 第三方应用，用户想要使用使用Client但又不想注册时可在Client中选择第三方登录
- Resource Owner 资源所有者，即用户
- Authorization Server 授权服务器，即提供第三方登录服务的服务器，如Github
- Resource Server 拥有资源信息的服务器，通常和授权服务器属于同一应用

根据上图的信息，我们可以知道OAuth2的基本流程为：

1. 第三方应用请求用户授权。
2. 用户同意授权，并返回一个凭证（code）
3. 第三方应用通过第二步的凭证（code）向授权服务器请求授权
4. 授权服务器验证凭证（code）通过后，同意授权，并返回一个资源访问的凭证（Access Token）。
5. 第三方应用通过第四步的凭证（Access Token）向资源服务器请求相关资源。
6. 资源服务器验证凭证（Access Token）通过后，将第三方应用请求的资源返回。

具体的，基本的第三方认证的流程实现为：
**1.用户访问网站，点击第三方登录**

**2.用户授权**

用户可选的由授权方式由授权服务器提供，授权服务器会告诉用户该第三方在授权服务器中提交的相关信息（如果需要实现第三方登录功能，第三方应用需要向Github、QQ等应用中提交应用的相关信息，不同服务可能会需要审核等不同的步骤），以及授权后第三方应用能够获取哪些资源。在Github中，最基础的认证可以访问用户的公共信息。 

**3.返回用户凭证code**

当客户端向授权服务器提交应用信息时，通常需要填写一个redirect_uri。之后当客户端引导用户进入授权页面时（第三方登录页面实际上就是第三方授权页面，由授权服务器提供），就会附带这个redirect_uri的信息，授权服务器验证两个URL一致时并且用户同意授权时（其实就是第三方登录验证成功），会通知浏览器跳转到第三方应用的redirect_uri，同时，在redirect_uri后附加用户凭证（code）的相关信息，此时，浏览器返回第三方应用同时携带用户凭证（code）的相关信息。

也就是用户点击了第三方登录按钮，跳转到第三方登录页面，此时发送时包含的信息为：

- response_type：表示授权类型，必选项，此处的值固定为"code"
- client_id：表示客户端的ID，必选项
- redirect_uri：表示重定向URI，可选项
- scope：表示申请的权限范围，可选项
- state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值。

如

> ```http
> GET /authorize?response_type=code&client_id=s6BhdRkqt3&state=xyz
>         &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
> Host: server.example.com
> ```

授权服务器比对client_id与redirect_uri，返回给第三方应用的redirect_uri携带code,如：

```text
http://tianmaying.com/oauth/github/callback?code=9e3efa6cea739f9aaab2&state=XXX
```

这样，返回用户凭证（code）就完成了，用户也就进入了第三方应用界面

从这一步开始，OAuth2 的授权将有两个重大变化的发生：

- 用户主动行为结束，用户理论上可以不需要再做任何主动的操作，作为第三方应用的我们可以在后台拿到资源服务器上的资源而对用户是不可见的，当用户的浏览器跳到下一个页面时，整个OAuth2的流程已经结束
- 浏览器端行为结束，从OAuth2 的基本流程可以看出，接下来的流程已经不需要和用户进行交互，接下来的行为都在第三方应用与授权服务器、资源服务器之间的交互。

我们知道浏览器端的行为实际上是不安全的，甚至安全凭证的传递都是通过URL直接传递的。但是由于用户凭证（code）不是那么敏感，其他攻击者拿到用户凭证（code）后依然无法获取到相应的用户资源，所以之前的行为是允许的

**4.请求授权服务器授权**

要拿到授权服务器的授权，需要以下几个信息：

- client_id 标识第三方应用的id，由授权服务器（Github）在第三方应用提交时颁发给第三方应用
- client_secret 第三方应用和授权服务器之间的安全凭证，由授权服务器（Github）在第三方应用提交时颁发给第三方应用
- code 第一步中返回的用户凭证
- redirect_uri 第一步生成用户凭证后跳转到第二步时的地址
- state 由第三方应用给出的随机码

可以看到，上述信息还涉及到第三方应用的安全凭证（client_secret），因此OAuth要求该请求必须是POST请求，同时，还必须是HTTPS服务，以此保证获取到的验证凭证（Access Token）的安全性。 

 当授权服务器拿到上述的所有信息，跳转到验证通过后，会将Access Token返回给第三方应用。

**5.第三方应用可请求访问用户资源**

拿到验证凭证（Access Token）后，剩下的事情就很简单了，资源服务器会提供一系列关于用户资源的API，拿验证凭证（Access Token）访问相应的API即可，例如，在GIthub中，如果你想拿到用户信息，可以访问以下API：

```text
GET https://api.github.com/user?access_token=...
```

注意，此时的访问不是通过浏览器进行的，而是服务器直接发送HTTP请求，因此其安全性是可以保证的。

如果验证凭证（Access Token）是正确的，此时资源服务器就会返回资源信息，此时整个OAuth流程就结束了。

授权服务器的具体的回复信息为

```http
  HTTP/1.1 200 OK
     Content-Type: application/json;charset=UTF-8
     Cache-Control: no-store
     Pragma: no-cache

     {
       "access_token":"2YotnFZFEjr1zCsicMWpAA",
       "token_type":"example",
       "expires_in":3600,
       "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
       "example_parameter":"example_value"
     }
```

- access_token：表示访问令牌，必选项。
- token_type：表示令牌类型，该值大小写不敏感，必选项，可以是bearer类型或mac类型。
- expires_in：表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间。
- refresh_token：表示更新令牌，用来获取下一次的访问令牌，可选项。
- scope：表示权限范围，如果与客户端申请的范围一致，此项可省略。
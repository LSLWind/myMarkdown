### 数据库设计

```mysql
create database yasuo;
use yasuo
```

#### User

```mysql
create table user(
	id int not null auto_increment,
	name varchar(30),
	password varchar(30),
	token varchar(60),
	is_delete tinyint,
    gmt_create DATETIME,
	gmt_modified DATETIME,
	primary key(id)
)default charset=utf8;
```

#### Article

```mysql
create table article(
	id BIGINT not null auto_increment,
	title CHAR not null,
	content TEXT,
	upload_user_id INT not null,
	gmt_create DATETIME,
	gmt_modified DATETIME,
	primary key(id)
)default charset=utf8;
```

#### Picture

```mysql
create table picture(
	id bigint not null auto_increment,
	path char not null,
	remark char comment "图片备注",
	gmt_create DATETIME,
	gmt_modified DATETIME,
	primary key(id)
)default charset=utf8;
```

对于某些管理系统，不允许注册，因此先增加一个管理员，管理员有权限增加管理员并修改信息



### 实体entity

根据表结构建实体，如：

```java
import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.Date;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer id;
    private String name;
    private String password;
    private String token;
    private Boolean hadDeleted;//是否注销

    @JsonFormat(pattern = "yyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date gmtCreate;

    @JsonFormat(pattern = "yyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date gmtModified;
    
}
```

### 前端

#### 引入前端框架

根据需要自行引入，插件，如库文件放在resources/static/plugins下，css、fonts、js

img等文件夹放在resources/static/dist下，静态html页面直接卸载static目录下

#### 约定返回数据json格式



#### login

登录页，点击登录时调用函数login()，使用ajax。

API: /users/login  method=post

登录验证成功则在cookie中设置token，跳转到首页 "/"

```javascript
var data = {"userName": userName, "password": password}
    $.ajax({
        type: "POST",//方法类型
        dataType: "json",//预期服务器返回的数据类型
        url: "users/login",
        contentType: "application/json; charset=utf-8",
        data: JSON.stringify(data),
        success: function (result) {
            if (result.resultCode == 200) {
                $('.alert-danger').css("display", "none");
                setCookie("token", result.data.token);
                window.location.href = "/";
            }
            ;
            if (result.resultCode == 500) {
                showErrorInfo(result.message);
                return;
            }
        },
        error: function () {
            $('.alert-danger').css("display", "none");
            showErrorInfo("接口异常，请联系管理员！");
            return;
        }
    });
```

### 后端

#### 定义返回数据类型

位于/common，定义返回信息状态码与json数据格式

Constants：定义返回状态码常量

```java
public class Constants {
    public static final int RESULT_CODE_SUCCESS = 200;  // 成功处理请求
    public static final int RESULT_CODE_BAD_REQUEST = 412;  // 请求错误
    public static final int RESULT_CODE_NOT_LOGIN = 402;  // 未登录
    public static final int RESULT_CODE_PARAM_ERROR = 406;  // 传参错误
    public static final int RESULT_CODE_SERVER_ERROR = 500;  // 服务器错误

    public final static String FILE_PRE_URL = "http://localhost:8080";//上传文件的默认url前缀
    public final static String FILE_UPLOAD_PATH = "/home/project/upload/";//上传文件的保存地址
}

```

Result<T\>：定义返回数据的json格式对应的Class

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Result<T> implements Serializable {
    private static final long serialVersionUID = 1L;
    private int resultCode;
    private String message;
    private T data;
    
    public Result(int resultCode,String message){
        this.resultCode=resultCode;
        this.message=message;
    }

}
```

#### 配置数据源、日志

在application.properties中进行配置

```properties
#配置数据库数据源
spring.datasource.url=jdbc:mysql://106.52.17.68:3306/yasuo?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=true
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.username=lsl
spring.datasource.password=liang1942557346
#配置日志
logging.file.path = running/logs
logging.file.name=lsl.log
logging.pattern.file= %d{yyyy-MMM-dd HH:mm:ss.SSS} %-5level [%thread] %logger{15} - %msg%n
```

#### 控制层controller

##### UserController

#### 数据库与实体的ORM mapper

#### userDAO

基于myBatis，使用注解完成映射，定义映射接口与sql语句与映射方法，同时要在主Application上加上Mapper扫描@MapperScan("com.lsl.yasuo.mapper")







#### 数据访问层service

##### userService/userServiceImpl

实现放在包impl下面





#### 其它utils

##### 用户数据页面PageResult

每个数据页面都显示不同的内容，即分页显示用户的数据（文章、上传图片等）

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class PageResult {
    private int totalCount;//作品总数
    private int pageSize;//每页的记录数
    private int totalPage;//总页数
    private int currentPage;//当前页数
    private List<?> list;//作品列表

    //初始化用户的作品信息显示页面，总作品数，页面大小，当前页数
    public PageResult(List<?> list, int totalCount, int pageSize, int currentPage) {
        this.list = list;
        this.totalCount = totalCount;
        this.pageSize = pageSize;
        this.currentPage = currentPage;
        this.totalPage = (int)Math.ceil((double)totalCount/pageSize);
    }
}
```

##### 签署token，提供校验

每次登陆时都签署发布一个token，请求时校验token获取用户数据

```xml
        <!--基于jwt的token验证依赖-->
        <dependency>
            <groupId>com.auth0</groupId>
            <artifactId>java-jwt</artifactId>
            <version>3.8.2</version>
        </dependency>
```

```java
public class TokenUtil {
    //设置token过期时间7200s
    private static final long EXPIRE_TIME=1000*7200;
    //设置token的私钥，随便设置一个
    private static final String TOKEN_SECRET="652a32f032e56239652320ff326c2365e0";

    /**
     * 基于私钥、过期时间，生成token，在token中附带用户名与用户id
     * @param userName token携带用户名信息
     * @param password token携带用户密码做校验
     * @return token
     */
    public static String sign(String userName,String password){
        Date date = new Date(System.currentTimeMillis() + EXPIRE_TIME);//设置过期时间
        Algorithm algorithm = Algorithm.HMAC256(TOKEN_SECRET);//私钥以及加密算法
        //设置头部信息
        Map<String, Object> header = new HashMap<>(2);
        header.put("type", "JWT");
        header.put("alg", "HS256");
        //设置头部，附带信息，设置过期时间，加上算法生成token
        return JWT.create().withHeader(header)
                .withClaim("loginName", userName)
                .withClaim("password", password)
                .withExpiresAt(date)
                .sign(algorithm);
    }

    /**
     * 对给定 token进行校验
     * @param token 给定token
     * @return token是否正确
     */
    public static boolean verify(String token){
        try{
            Algorithm algorithm=Algorithm.HMAC256(TOKEN_SECRET);
            JWTVerifier verifier=JWT.require(algorithm).build();//校验器
            DecodedJWT jwt =verifier.verify(token);//解码token校验，出错则抛出异常
            return true;
        }catch (Exception e){
            return false;
        }
    }
}
```


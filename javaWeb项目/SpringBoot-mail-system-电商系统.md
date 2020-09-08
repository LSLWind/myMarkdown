## 分层结构

<img src="https://i.loli.net/2020/03/22/eGIi78gkfdzDbJO.png" alt="image.png" style="zoom: 80%;" />

### 数据模型

**dataobject**

数据实体，与数据库中的表完全一对一映射，简写为DO，如UserDO。DO是DAO从数据库中取出数据时映射的对象。

**model**

领域模型，处理的业务逻辑所需要的数据实体，如多个相关表中的数据汇集在一起构成model，简写为Model，如UserModel。Model组合多个表中所需要的数据为一个类，提供给业务逻辑所需要的所有数据。

**viewmodel**

显示给视图层中的数据实体，传递给前台的数据应该是有限且不同于model的，位于controller包下，当获取到model时，应该只返回给前台有限必要的数据，简写为VO，如UserLoginVO。VO是映射给前台的数据对象。

![image.png](https://i.loli.net/2020/03/22/OmKTZVPnDG64dH7.png)

## 数据访问层

### MyBatis生成器

使用MyBatis时经常会重复写许多模板方法，在maven使用MyBatis提供的生成器插件可以快速生成mapper文件，对应的DO类已经映射接口

**在pom.xml中设置生成器插件**

```xml
            <!--mybatis映射生成工具插件-->
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.2</version>
                <dependencies>
                    <dependency>
                        <groupId>org.mybatis.generator</groupId>
                        <artifactId>mybatis-generator-core</artifactId>
                        <version>1.3.2</version>
                    </dependency>
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>8.0.13</version>
                    </dependency>
                </dependencies>
                <!--插件执行目的-->
                <executions>
                    <execution>
                        <id>mybatis generator</id>
                        <phase>package</phase>
                        <goals>
                            <goal>generate</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <!--允许移动生成的文件-->
                    <verbose>true</verbose>
                    <!--允许自动覆盖文件-->
                    <overwrite>true</overwrite>
                    <configurationFile><!--插件配置文件-->
                        src/main/resources/mybatis-generator.xml
                    </configurationFile>
                </configuration>
            </plugin>
```

该插件完成的目标是在package阶段进行（文件）生成（generate）工作，需要多去配置文件，指定的配置文件路径为

```xml
                    <configurationFile><!--插件配置文件-->
                        src/main/resources/mybatis-generator.xml
                    </configurationFile>
```

**插件配置文件**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>

    <context id="default" targetRuntime="MyBatis3">
        <!--jdbc的数据库连接 -->
        <jdbcConnection
                driverClass="com.mysql.cj.jdbc.Driver"
                connectionURL="jdbc:mysql://127.0.0.1:3306/express?serverTimezone=Hongkong"
                userId="root"
                password="123456">
        </jdbcConnection>
        <!-- Model模型生成器,指定Model类存放位置
            targetPackage     指定生成的model生成所在的包名
            targetProject     指定在该项目下所在的路径
        -->
        <javaModelGenerator targetPackage="lsl.widn.mailsystem.dataobject" targetProject="./src/main/java">
            <!-- 是否允许子包，即targetPackage.schemaName.tableName -->
            <property name="enableSubPackages" value="true"/>
            <!-- 是否对model添加 构造函数 -->
            <property name="constructorBased" value="true"/>
            <!-- 是否对类CHAR类型的列的数据进行trim操作 -->
            <property name="trimStrings" value="true"/>
            <!-- 建立的Model对象是否 不可改变  即生成的Model对象不会有 setter方法，只有构造方法 -->
            <property name="immutable" value="false"/>
        </javaModelGenerator>
        <!--mapper映射文件生成所在的目录 为每一个数据库的表生成对应的SqlMap文件 -->
        <sqlMapGenerator targetPackage="mapping" targetProject="./src/main/resources">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>
        <!-- 客户端代码，生成易于使用的针对Model对象和XML配置文件 的代码
        type="ANNOTATEDMAPPER",生成Java Model 和基于注解的Mapper对象
        type="MIXEDMAPPER",生成基于注解的Java Model 和相应的Mapper对象
        type="XMLMAPPER",生成SQLMap XML文件和独立的Mapper接口
-->
        <!-- targetPackage：mapper接口dao生成的位置 -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="lsl.widn.mailsystem.dao" targetProject="./src/main/java">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <!--生成对应表及列名-->
        <table tableName="user_info" domainObjectName="UserDo" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false" />
        <table tableName="user_password" domainObjectName="UserPasswordDo" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false" />

    </context>
</generatorConfiguration>
```

配置信息都位于<contex\>标签下

**执行maven命令**

指令以**插件名:目标名**的方式执行，运行插件完成工作，上述命令为

```
mybatis-generator:generate
```

## 返回结果与异常设计

### 定义通用的返回结果

```java
@Data
public class CommonReturnType {
    //返回处理结果, code为1表示成功，否则有出错码
    private int code;
    //fail使用通用的错误码格式
    private Object data;

    public static CommonReturnType create(Object result){
        return CommonReturnType.create(result,1);
    }
    public static CommonReturnType create(Object result, int code){
        CommonReturnType type=new CommonReturnType();
        type.setCode(code);
        type.setData(result);
        return type;
    }
}
```

注意重载的create方法，可以方面的设置返回结果与状态码

### 异常设计

对于异常设计，应该有通用错误码与错误信息，此时要使用枚举类。

先定义一个通用的出错接口

```java
public interface CommonError {
    public int getErrCode();
    public String getErrMsg();
    public CommonError setErrMsg(String errMsg);
}
```

错误枚举类

```java
public enum EmBusinessError implements CommonError {
    //通用错误类型
    PARAMETER_VALIDATION_ERROR(10001,"参数不合法"),
    UNKNOWN_ERROR(10002,"未知错误"),
    REQUEST_ILLEGAL_ERROR(10003,"请求非法"),
    SERVER_EXCEPTION_ERROR(10004,"服务器异常"),

    //错误状态码枚举
    //2000开头为用户信息错误
    USER_NOT_EXIST(20001,"用户不存在"),
    USERNAME_OR_PASSWORD_ERROR(20002,"用户名或密码错误"),
    DELETE_FAIL_ERROR(20003,"用户删除失败，请稍后重试"),

    //3000开头为微信错误
    WE_CHAT_LOGIN_STATUS_ERROR(30001,"微信登录异常")
    //4000开头为微信client端错误
    ;
    private EmBusinessError(int errCode, String errMsg){
        this.errCode=errCode;
        this.errMsg=errMsg;
    }
    private int errCode;
    private String errMsg;

    @Override
    public int getErrCode(){
        return errCode;
    }
    @Override
    public String getErrMsg(){
        return errMsg;
    }
    @Override
    public CommonError setErrMsg(String errMsg){
        this.errMsg=errMsg;
        return this;
    }
}
```

业务异常，内部维护通用错误设计，非常灵活

```java
//包装器业务异常实现
public class BusinessException extends Exception implements CommonError{
    private CommonError commonError;
    //直接接受EmBusinessError的传参用于构造业务异常
    public BusinessException(CommonError commonError){
        super();
        this.commonError=commonError;
    }

    //接受自定义errMsg的方式构造业务异常
    public BusinessException(CommonError commonError, String errMsg){
        super();
        this.commonError=commonError;
        this.commonError.setErrMsg(errMsg);
    }

    @Override
    public int getErrCode(){
        return this.commonError.getErrCode();
    }
    @Override
    public String getErrMsg(){
        return this.commonError.getErrMsg();
    }
    @Override
    public CommonError setErrMsg(String errMsg){
        this.commonError.setErrMsg(errMsg);
        return this;
    }
}
```

### 捕获异常

在controller中使用@ExceptionHandler(Exception.class)注解可以捕获异常并在捕获异常后执行注解上的方法

常见一个基类Controller作为通用的异常捕获处理

```java
public class BaseController {
    @Autowired
    HttpServletRequest request;

    //定义exceptionHandler解决未被controller层吸收的异常
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.OK)
    @ResponseBody
    public Object handlerException(HttpServletRequest request, Exception ex){
        Map<String,Object> responseData=new HashMap<>();
        if (ex instanceof BusinessException){
            BusinessException businessException=(BusinessException)ex;
            responseData.put("error",businessException.getErrMsg());
            return CommonReturnType.create(responseData,((BusinessException) ex).getErrCode());
        }
        responseData.put("error",EmBusinessError.UNKNOWN_ERROR.getErrMsg());
        return CommonReturnType.create(responseData,EmBusinessError.UNKNOWN_ERROR.getErrCode());
    }
}
```

## 数据库设计

设计表时一般有几个通用规则

* 列的值不应该给null，可设定默认值，防止经常抛出的空指针异常
* 用户表与密码表分开存放，密码应该存放加密值，如MD5值

### 用户表


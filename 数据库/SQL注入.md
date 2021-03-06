本文主要针对SQL注入的含义、以及如何进行SQL注入和如何预防SQL注入让小伙伴有个了解。适用的人群主要是测试人员，了解如何进行SQL注入，可以帮助我们测试登录、发布等模块的SQL攻击漏洞，至于如何预防SQL注入，按理说应该是开发该了解的事情~但是作为一个棒棒的测试，搞清楚原理是不是能让我们更加透彻地理解bug的产生原因呢~好啦，话不多说，进入正题~

如何理解SQL注入（攻击）？

SQL注入是一种将SQL代码添加到输入参数中，传递到服务器解析并执行的一种攻击手法。

SQL注入攻击是输入参数未经过滤，然后直接拼接到SQL语句当中解析，执行达到预想之外的一种行为，称之为SQL注入攻击。



SQL注入是怎么产生的？

1）WEB开发人员无法保证所有的输入都已经过滤

2）攻击者利用发送给SQL服务器的输入参数构造可执行的SQL代码（可加入到get请求、post请求、http头信息、cookie中）

3）数据库未做相应的安全配置



如何进行SQL注入攻击？

以php编程语言、mysql数据库为例，介绍一下SQL注入攻击的构造技巧、构造方法

1.数字注入

在浏览器地址栏输入：learn.me/sql/article.php?id=1，这是一个get型接口，发送这个请求相当于调用一个查询语句：

$sql = "SELECT * FROM article WHERE id =",$id
正常情况下，应该返回一个id=1的文章信息。那么，如果在浏览器地址栏输入：learn.me/sql/article.php?id=-1 OR 1 =1，这就是一个SQL注入攻击了，可能会返回所有文章的相关信息。为什么会这样呢？

这是因为，id = -1永远是false，1=1永远是true，所有整个where语句永远是ture，所以where条件相当于没有加where条件，那么查询的结果相当于整张表的内容

2.字符串注入

有这样一个用户登录场景：登录界面包括用户名和密码输入框，以及提交按钮。输入用户名和密码，提交。

这是一个post请求，登录时调用接口learn.me/sql/login.html，首先连接数据库，然后后台对post请求参数中携带的用户名、密码进行参数校验，即sql的查询过程。假设正确的用户名和密码为user和pwd123，输入正确的用户名和密码、提交，相当于调用了以下的SQL语句：

SELECT * FROM user WHERE username = 'user' ADN password = 'pwd123'
由于用户名和密码都是字符串，SQL注入方法即把参数携带的数据变成mysql中注释的字符串。mysql中有2种注释的方法：

1）'#'：'#'后所有的字符串都会被当成注释来处理

用户名输入：user'#（单引号闭合user左边的单引号），密码随意输入，如：111，然后点击提交按钮。等价于SQL语句：

SELECT * FROM user WHERE username = 'user'#'ADN password = '111'
'#'后面都被注释掉了，相当于：

SELECT * FROM user WHERE username = 'user' 
2）'-- ' （--后面有个空格）：'-- '后面的字符串都会被当成注释来处理

用户名输入：user'-- （注意--后面有个空格，单引号闭合user左边的单引号），密码随意输入，如：111，然后点击提交按钮。等价于SQL语句：

SELECT * FROM user WHERE username = 'user'-- 'AND password = '111'
SELECT * FROM user WHERE username = 'user'-- 'AND password = '1111'
'-- '后面都被注释掉了，相当于：

SELECT * FROM user WHERE username = 'user'
因此，以上两种情况可能输入一个错误的密码或者不输入密码就可登录用户名为'user'的账号，这是十分危险的事情。



如何预防SQL注入？

这是开发人员应该思考的问题，作为测试人员，了解如何预防SQL注入，可以在发现注入攻击bug时，对bug产生原因进行定位。

1）严格检查输入变量的类型和格式

对于整数参数，加判断条件：不能为空、参数类型必须为数字

对于字符串参数，可以使用正则表达式进行过滤：如：必须为[0-9a-zA-Z]范围内的字符串

2）过滤和转义特殊字符

在username这个变量前进行转义，对'、"、\等特殊字符进行转义，如：php中的addslashes()函数对username参数进行转义

3）利用mysql的预编译机制

把sql语句的模板（变量采用占位符进行占位）发送给mysql服务器，mysql服务器对sql语句的模板进行编译，编译之后根据语句的优化分析对相应的索引进行优化，在最终绑定参数时把相应的参数传送给mysql服务器，直接进行执行，节省了sql查询时间，以及mysql服务器的资源，达到一次编译、多次执行的目的，除此之外，还可以防止SQL注入。具体是怎样防止SQL注入的呢？实际上当将绑定的参数传到mysql服务器，mysql服务器对参数进行编译，即填充到相应的占位符的过程中，做了转义操作。
————————————————
版权声明：本文为CSDN博主「茵茵大可爱」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/github_36032947/article/details/78442189
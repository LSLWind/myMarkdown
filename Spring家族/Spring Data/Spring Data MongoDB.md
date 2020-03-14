## 配置

为了有效地使用Spring Data MongoDB，需要在Spring配置中添加几个必要的bean。首先，我们需要配置`MongoClient`，以便于访问MongoDB数据库。同时，我们还需要有一个`MongoTemplate` bean，实现基于模板的数据库访问。

<img src="https://i.loli.net/2020/03/07/KayvNWJxp87eXIL.png" alt="image.png" style="zoom: 80%;" />

除了直接声明这些bean，我们还可以让配置类扩展`AbstractMongo-Configuration`并重载`getDatabaseName()`和`mongo()`方法。

![image.png](https://i.loli.net/2020/03/07/Myr1uteL2TSCZbq.png)

上述配置假定mongodb数据库在本地，也可以配置为远程连接，在创建`MongoClient`的时候进行指定：

```java
public Mongo mongo() throws Exception {
  return new MongoClient("mongodbserver");//ip地址
}
```

另外，MongoDB服务器有可能监听的端口并不是默认的27017。如果是这样的话，在创建`MongoClient`的时候，还需要指定端口：

```java
public Mongo mongo() throws Exception {
  return new MongoClient("mongodbserver", 37017);
}
```

如果MongoDB服务器运行在生产配置上，你可能还启用了认证功能。在这种情况下，为了访问数据库，我们还需要提供应用的凭证。访问需要认证的MongoDB服务器稍微有些复杂，如下面的程序清单所示。

![image.png](https://i.loli.net/2020/03/07/m24GgQUve5ErsqR.png)

为了访问需要认证的MongoDB服务器，`MongoClient`在实例化的时候必须要有一个`MongoCredential`的列表，它们是通过注入的`Environment`对象解析得到的。

### 文档类型映射

MongoDB并没有提供对象-文档映射的注解。Spring Data MongoDB填补了这一空白，提供了一些将Java类型映射为MongoDB文档的注解。

| 注　　解    | 描　　述                                                     |
| ----------- | ------------------------------------------------------------ |
| `@Document` | 标示映射到MongoDB文档上的领域对象                            |
| `@Id`       | 标示某个域为ID域                                             |
| `@DbRef`    | 标示某个域要引用其他的文档，这个文档有可能位于另外一个数据库中 |
| `@Field`    | 为文档域指定自定义的元数据                                   |
| `@Version`  | 标示某个属性用作版本域                                       |

`@Document`和`@Id`注解类似于JPA的`@Entity`和`@Id`注解。我们将会经常使用这两个注解，对于要以文档形式保存到MongoDB数据库的每个Java类型都会使用这两个注解。例如，如下的程序清单展现了如何为Order类添加注解，它会被持久化到MongoDB中。

![image.png](https://i.loli.net/2020/03/09/oOUlgYA62MXCQcw.png)

## 基础使用

### 使用MongoTemplate访问MongoDB

我们已经在配置类中配置了`MongoTemplate` bean，不管是显式声明还是扩展`AbstractMongoConfiguration`都能实现相同的效果。接下来，需要做的就是将其注入到使用它的地方：

```java
@Autowired
MongoOperations mongo;
```

注意，在这里我们将`MongoTemplate`注入到一个类型为`MongoOperations`的属性中。`MongoOperations`是`MongoTemplate`所实现的接口，不使用具体实现是一个好的做法，尤其是在注入的时候。

`MongoOperations`暴露了多个使用MongoDB文档数据库的方法。在这里，我们不可能讨论所有的方法，但是可以看一下最为常用的几个操作，比如计算文档集合中有多少条文档。使用注入的`MongoOperations`，我们可以得到`Order`集合并调用`count()`来得到数量：

```java
long orderCount = mongo.getCollection("order").count();
```

现在，假设要保存一个新的Order。为了完成这个任务，我们可以调用`save()`方法：

```javascript
Order order = new Order();
... // set properties and add line items
mongo.save(order, "order");
```

save()方法的第一个参数是新创建的`Order`，第二个参数是要保存的文档存储的名称。

另外，我们还可以调用`findById()`方法来根据ID查找订单：

```java
String orderId = ...;
Order order = mongo.findById(orderId, Order.class);
```

对于更高级的查询，我们需要构造`Query`对象并将其传递给`find()`方法。例如，要查找所有`client`域等于“Chuck Wagon”的订单，可以使用如下的代码：

```java
List<Order> chucksOrders = mongo.find(Query.query(
    Criteria.where("client").is("Chuck Wagon")), Order.class);
```

在本例中，用来构造`Query`对象的Criteria只检查了一个域，但是它也可以用来构造更加有意思的查询。比如，我们想要查询Chuck所有通过Web创建的订单：

```java
List<Order> chucksWebOrders = mongo.find(Query.query(
    Criteria.where("customer").is("Chuck Wagon")
            .and("type").is("WEB")), Order.class);
```

如果你想移除某一个文档的话，那么就应该使用`remove()`方法：

```java
mongo.remove(order);
```

如我前面所述，`MongoOperations`有多个操作文档数据的方法。我建议你查看一下其JavaDoc文档，以了解通过`MongoOperations`都能完成什么功能。

通常来讲，我们会将`MongoOperations`注入到自己设计的Repository类中，并使用它的操作来实现Repository方法。但是，如果你不愿意编写Repository的话，那么Spring Data MongoDB能够自动在运行时生成Repository实现。下面，我们来看一下是如何实现的。

### 编写MongoDB Repository

与使用JPA一样，通过继承Repository接口完成DAO操作

```java
public interface OrderRepository
         extends MongoRepository<Order, String> {
}
```

接口中常用的方法有

| 方　　法                             | 描　　述                                         |
| ------------------------------------ | ------------------------------------------------ |
| `long count();`                      | 返回指定Repository类型的文档数量                 |
| `void delete(Iterable<? extends T);` | 删除与指定对象关联的所有文档                     |
| `void delete(T);`                    | 删除与指定对象关联的文档                         |
| `void delete(ID);`                   | 根据ID删除某一个文档                             |
| `void deleteAll();`                  | 删除指定Repository类型的所有文档                 |
| `boolean exists(Object);`            | 如果存在与指定对象相关联的文档，则返回true       |
| `boolean exists(ID);`                | 如果存在指定ID的文档，则返回true                 |
| `List findAll();`                    | 返回指定Repository类型的所有文档                 |
| `List findAll(Iterable);`            | 返回指定文档ID对应的所有文档                     |
| `List findAll(Pageable);`            | 为指定的Repository类型，返回分页且排序的文档列表 |
| `List findAll(Sort);`                | 为指定的Repository类型，返回排序后的所有文档列表 |
| `T findOne(ID);`                     | 为指定的ID返回单个文档                           |
| ` Iterable  save (Iterable ); `      | 保存指定Iterable中的所有文档                     |
| ` S save (s);`                       | 为给定的对象保存一条文档                         |

同JPA一样，指定查询方式可以为find...by


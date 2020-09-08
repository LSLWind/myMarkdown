### Callable

泛型类，与Runnable一样，但有返回值

* call()：本质是被Runnable的run()方法调用的，线程执行的任务，可直接提交给线程池（ExecutorService）去执行（调用submit()方法)

### FutureTask

表示执行任务，构造方法传递一个Callable进行包装，之后Thread的构造方法可以传递一个FutureTask构造线程，可以通过FutureTask获取Callable的执行结果

* get()：获取Callable的执行结果

### Future

Furure表示异步执行返回的结果

![image.png](https://i.loli.net/2020/05/13/sHqXxTREJ27SgWf.png)

### 基本逻辑

将任务放在Callable中，交给FutureTask构造任务，提交给线程池执行，通过get获取结果，返回Future
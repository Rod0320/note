# Thread和Runable的区别 

>  `Runable`是一个接口,是整个线程中最顶级的一个规范,用于在线程中实现我们的业务逻辑

> `Thread`是Runable的一个实现类,用于整个线程生命周期的第一步,创建状态,也就是创建线程对象

- 每一个线程都必须要有业务逻辑,也就是业务代码
- **Runable接口就是一个给规范**,规定了线程`必须实现这个接口,因为线程必须要有业务逻辑`
  - 当然JDK 1.5的时候还有一个`callable接口`,作用也是类似的

- 而整个线程的生命周期中,第一步是创建一个线程对象
- Thread类也是Java中`唯一的`可以创建线程对象的类,所有线程都逃不过要先创建一个thread实例
## Spring 能帮我们做什么

Spring 是为解决企业级应用开发的复杂性而设计的一款框架，Spring 的设计理念就是：简化开发。

在 Spring 框架中，一切对象都是 bean，所以其通过面向 bean 编程（BOP），结合其核心思想依赖注入（DI）和面向切面(（AOP）编程，Spring 实现了其伟大的简化开发的设计理念。

## 控制反转（IOC）

IOC 全称为：Inversion of Control。控制反转的基本概念是：不用创建对象，但是需要描述创建对象的方式。

简单的说我们本来在代码中创建一个对象是通过 new 关键字，而使用了 Spring 之后，我们不在需要自己去 new 一个对象了，而是直接通过容器里面去取出来，再将其自动注入到我们需要的对象之中，即：依赖注入。

也就说创建对象的控制权不在我们程序员手上了，全部交由 Spring 进行管理，程序要只需要注入就可以了，所以才称之为控制反转。

使用ApplicationContent， 他是BeanFactory的子类，。 ApplicationContent有两个具体的实现子类，ClassPathXmlApplicationContent - 从classpath中加载配置文件  ， FilesystemXmlApplicationContent - 从本地文件加载配置文件（不常用）。 


IOC的set()方法来注入的

## 依赖注入（DI）

依赖注入（Dependency Injection，DI）就是 Spring 为了实现控制反转的一种实现方式，所有有时候我们也将控制反转直接称之为依赖注入。

注入的方式： 接口注入，set注入， 构造器注入

## 面向切面编程（AOP）

AOP 全称为：Aspect Oriented Programming。AOP是一种编程思想，其核心构造是方面（切面），即将那些影响多个类的公共行为封装到可重用的模块中，而使原本的模块内只需关注自身的个性化行为。

AOP 编程的常用场景有：Authentication（权限认证）、Auto Caching（自动缓存处理）、Error Handling（统一错误处理）、Debugging（调试信息输出）、Logging（日志记录）、Transactions（事务处理）等。

### AOP的几种实现方式

1. AspectJ静态代理
   1. 编译期将aspectJ加入字节码，运行期直接是择增强之后的。
2. JDK动态代理
   1. 必须要实现接口
3. CGlib动态代理
   1. 不需要实现接口
---
comments: true
---

## AOP

`Spring AOP` 是通过动态代理实现的，实现动态代理有两种方式，一种是 `JDK` 自带的动态代理，另一种是 `cglib`。

### JDK 动态代理

类如果实现了接口，spring就会用这种方式实现动态代理。代理类的处理器实现了InvocationHandler接口重写invoke方法，在invoke方法里面调用原类的方法后再进行其他操作，Proxy类newProxyInstance()方法生成代理类后进行调用。

优点: 不需要依赖，生成代理类速度快。

缺点: 被代理的类必须实现了接口，否则无法代理，只能代理接口中的方法，方法执行的效率比较低。

### cglib 动态代理

需要代理的类没有实现接口，此时JDK的动态代理将没有办法使用，于是Spring会使用CGLib的动态代理来生成代理对象。底层采用了ASM字节码生成框架，直接对需要代理的类的字节码进行操作，生成这个类的一个子类，并重写了类的所有可以重写的方法，在重写的过程中，将我们定义的额外的逻辑（简单理解为Spring中的切面）织入到方法中，对方法进行了增强。而通过字节码操作生成的代理类，和我们自己编写并编译后的类没有太大区别。

优点: 不需要实现接口，方法执行的效率比较高。

缺点: 无法代理final类，final方法，private方法，生成代理类速度慢。

## 常用注解

`@Configuration` 应用于类，在spring启动之前执行，常用于读取配置文件。

`@EnableAutoConfiguration` 应用于类，开启springboot自动配置功能。

`@ComponentScan` 应用于类，定义自动扫描的包。

`@Bean` 应用于方法，将方法返回的对象实例化。

`@DependsOn` 应用于方法，表明当前bean所依赖的bean，当前bean会在依赖bean实例化之后再实例化。

`@ConditionalOnClass` 应用于方法，表明当前类路径下有指定类时，才实例化。

`@ConditionalOnMissingBean` 应用于方法，表明当前bean不存在时，才实例化。

`@ConditionalOnProperty` 应用于方法，表明指定属性的值为某个值时，才实例化。


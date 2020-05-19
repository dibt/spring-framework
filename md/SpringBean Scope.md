# Spring Bean 作用域
- 作用域  

|来源|说明|
|:---:|:---:|
|singleton|默认 Spring Bean 作用域，一个 BeanFactory 有且仅有一个实例|
|prototype|原型作用域，每次依赖查找和依赖注入生成新 Bean 对象|
|request|将 Spring Bean 存储在 ServletRequest 上下文中|
|session|将 Spring Bean 存储在 HttpSession 中|
|application|将 Spring Bean 存储在 ServletContext 中|
- Singleton Bean 无论依赖查找还是依赖注入，均为同一个对象
Prototype Bean 无论依赖查找还是依赖注入，均为新生成的对象  
- 如果依赖注入集合类型的对象，Singleton Bean 和 Prototype Bean 均会存在一个，Prototype Bean 有别于其他地方的依赖注入(也是新生成的)  
注意事项：  
Spring 容器没有办法管理 prototype Bean 的完整的声明周期，也没有办法记录实例的存在。销毁回调方法将不会执行，可以利用 BeanPostProcessor 进行清扫工作
(建议实现DisposableBean#destroy)。
Spring 容器有的是 prototype Bean 的 BeanDefinition，也就是 Bean 定义的元信息，而非有 Bean 完整的生命周期  
- 无论是 Singleton Bean 还是 Prototype Bean 均会执行初始化方法回调，不过仅仅只有 Singleton Bean 会执行销毁方法回调
### 面试题
1. Spring 内建的 Bean 作用域有几种
2. singleton Bean 是否在一个应用是唯一的
否，singleton bean 尽在当前 Spring Ioc 容器(BeanFactory)中是单例对象，因为一个应用可能包含多个应用上下文
延伸：一个静态字段在 JVM 里是否是唯一的
否，一个静态字段在一个 ClassLoader 里是唯一的，一个应用可能包含多个 ClassLoader，而 ClassLoader 之间是可以相互隔离的
3. application 作用域的 Bean 是否可以被其他方案替代
可以的，实际上 application 作用域的 Bean 和 singleton Bean没有本质区别
 
# 依赖注入来源
 * 普通的DefinitionBean，有完整的生命周期的管理
 * 单例对象，只能通过java API的方式调用registerSingleton()
 * ResolverDependency，非spring管理对象，只能通过ConfigurableListableBeanFactory#registerResolvableDependency()调用
 * 外部化配置 @Autowire、@Value都是通过AutowiredAnnotationBeanPostProcessor来处理的
 
 
### 面试题
1. 注入和查找的依赖来源是否相同？  
不相同，依赖查找的来源仅限于Spring BeanDefinition 以及单例对象，而依赖注入的来源还包括Resolvable Dependency 以及@Value标注的外部化配置  
2. 单例对象能在Ioc容器启动后注册吗？  
可以的，Ioc容器启动有两个阶段：1、注册BeanDefinition 
2、注册单例对象。注册单例对象的注册(DefaultSingletonBeanRegistry#registerSingleton())和BeanDefinition(DefaultListableBeanFactory#registerBeanDefinition())不同，BeanDefinition会被ConfigurableListableBeanFactory#freezeConfiguration()
方法影响，从而冻结注册，而单例对象没有这个限制。  
DefaultListableBeanFactory是DefaultSingletonBeanRegistry的子类  
3、依赖注入的来源都有哪些
 


 
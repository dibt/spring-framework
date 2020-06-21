# Spring Bean 生命周期
### Spring Bean 元信息解析
  - 面向资源 BeanDefinition 解析
    - BeanDefinitionReader
    - XML 解析器 - BeanDefinitionParser
  - 面向注解 BeanDefinition 解析
    - AnnotatedBeanDefinitionReader  

Bean 名称生成来自于 BeanNameGenerator，注解实现 AnnotationBeanNameGenerator
### Spring Bean 注册
  - BeanDefinition 注册接口
    - BeanDefinitionRegistry 默认实现 --> DefaultListableBeanFactory
### Spring BeanDefinition 合并阶段
  - BeanDefinition 合并
    - 父子 BeanDefinition 合并
      - 当前 BeanFactory 查找
      - 层次性 BeanFactory 查找 --> AbstractBeanFactory#getMergedBeanDefinition  
RootBeanDefinition 不需要合并，不存在 parent
GenericBeanDefinition 普通的 BeanDefinition，可以 setParent    
### Spring Bean Class 加载阶段 
  - ClassLoader 类加载
  - Java Security 安全控制
  - ConfigurableBeanFactory 临时 ClassLoader
BeanDefinition --> Class  是通过 ClassLoader 加载的
### Spring Bean 实例化前阶段
 - 非主流生命周期 - Bean 实例化前阶段
   - InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation  
### Spring Bean 实例化阶段
  - 实例化方法
    - 传统实例化方式
      - 实例化策略 - InstantiationStrategy  
    - 依赖注入 -> 构造器依赖注入
      - 构造器依赖注入按照类型注入，底层实现是 DefaultListableBeanFactory#resolveDependency
  - Bean 属性赋值 (Populate) 判断
    - InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation
  - Bean 属性赋值前阶段
    - Bean 属性值元信息
      - PropertyValues
    - Bean 属性赋值前回调
     - InstantiationAwareBeanPostProcessor#postProcessProperties
Spring Bean Aware 接口回调阶段 
  - Spring Aware 接口
    - BeanNameAware -> AbstractAutowireCapableBeanFactory#doCreateBean -> 
    AbstractAutowireCapableBeanFactory#initializeBean 
    ->AbstractAutowireCapableBeanFactory#invokeAwareMethods
    - BeanClassLoaderAware
    - BeanFactoryAware
    - EnvironmentAware -> AbstractApplicationContext#prepareBeanFactory -> 
    ApplicationContextAwareProcessor#postProcessBeforeInstantiation -> 
    ApplicationContextAwareProcessor#invokeAwareInterfaces
    - EmbeddedValueResolverAware
    - ResourceLoaderAware
    - ApplicationEventPublisherAware
    - MessageSourceAware
    - ApplicationContextAware
### 总结 Spring Bean 初始化前阶段
  - Bean 实例化
  - Bean 属性赋值
  - Bean Aware 接口回调
  - 方法回调 BeanPostProcessor#postProcessBeforeInstantiation
### Spring Bean 初始化阶段
  - Bean Initialization (回调顺序从上到下)
    - @PostConstruct 标注方法，基于注解的驱动，和 BeanFactory 没有直接的关系（CommonAnnotationBeanPostProcessor）
    - 实现 InitializingBean 接口的 afterPropertiesSet() 方法
    - 自定义初始化方法
### Spring Bean 初始化后阶段
  - 方法回调 BeanPostProcessor#postProcessAfterInitialization
### Spring Bean 销毁前阶段
  - 方法回调
    - DestructionAwareBeanPostProcessor#postProcessBeforeDestruction  
  - Bean Destroy
     - @PreDestroy 标注方法
     - 实现DisposableBean 接口的 destroy 方法
     - 自定义销毁方法
Spring Bean Destroy 实现是基于 AbstractBeanFactory#destroyBean -> DisposableBeanAdapter#destroy
### 面试题
1. BeanPostProcessor 的使用场景有哪些
  BeanPostProcessor 提供 Spring Bean 初始化前和初始化后的生命周期回调，分别对应 postProcessBeforeInstantiation 以及 
  postProcessAfterInitialization 方法，允许对相关 Bean 进行扩展甚至替换，其中， ApplicationContext 相关的 Aware 回调也是基于 BeanPostProcessor 
  实现，即 ApplicationContextAwareProcessor
2. BeanFactoryPostProcessor 与 BeanPostProcessor 的区别
  BeanFactoryPostProcessor 是 Spring BeanFactory(实际为 ConfigurationListableBeanFactory) 的后置处理器，用于扩展 BeanFactory 或者通过 
  BeanFactory 进行依赖查找和依赖注入
  BeanFactoryPostProcessor 必须有 Spring ApplicationContext 执行， BeanFactory 无法与其直接交互。
  而 BeanPostProcessor 是直接与 BeanFactory 关联的，属于 N 对 1 的关系。
3. BeanFactory 是怎样处理 Bean 生命周期的
  BeanFactory 的默认实现是 DefaultListableBeanFactory，其中 Bean 生命周期与方法映射如下：
    - BeanDefinition 注册阶段 - registerBeanDefinition
    - BeanDefinition 合并阶段 - getMergedBeanDefinition
    - Bean 实例化前阶段 - resolveBeforeInstantiation
    - Bean 实例化阶段 - createBeanInstance
    - Bean 实例化后阶段 - populateBean
    - Bean 属性赋值前阶段 - populateBean
    - Bean 属性赋值阶段 - populateBean
    - Bean Aware 接口回调阶段 - initializeBean
    - Bean 初始化前阶段 - initializeBean
    - Bean 初始化阶段 - initializeBean
    - Bean 初始化后阶段 - initializeBean
    - Bean 初始化完成阶段 - preInstantiateSingletons
    - Bean 销毁前阶段 - destroyBean
    - Bean 销毁阶段 - destroyBean

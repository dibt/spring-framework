# 依赖注入
### 依赖注入的模式和类型
- 手动模式 - 配置或者编程的方式，提前安排注入规则  
  - XML 资源配置元信息
  - Java 注解元信息
  - API 配置元信息  
  BeanDefinitionBuilder -> BeanDefinition -> ApplicationContext#registerBeanDefinition
- 自动模式 - 实现方提供依赖自动关联的方式，按照内建的注入规则(官方不推荐)
  - AutoWiring (自动绑定)
- 依赖注入类型
  - Setter 方法
  - 构造器
  - 字段 @Autowired、@Resource  
  @Autowired 会忽略掉静态字段
  - 方法 @Autowired、@Resource、@Bean
  - 接口回调    
  
|内建接口|说明|
|:----:|:----:|
| BeanFactoryAware |获取 Ioc 容器 - BeanFactory |
| ApplicationContextAware | 获取 Spring 应用上下文 - ApplicationContext 对象|
| EnvironmentAware | 获取 Environment 对象|
| ResourceLoaderAware |获取资源加载器对象 - ResourceLoader |
| BeanClassLoaderAware |获取加载当前 Bean Class 的 ClassLoader |
| BeanNameAware |获取当前 Bean 的名称|
| MessageSourceAware |获取 MessageSource 对象，用于 Spring 国际化|
| ApplicationEventPublisherAware |获取 ApplicationEventPublisherAware 对象，用于 Spring 事件 |
| EmbeddedValueResolverAware |获取 StringValueResolve 对象，用于占位符处理|  
限定注入 --使用注解 @Qualifier 限定，例子 SpringCloud 的 @LoadBalanced 注解  

- 延迟依赖注入
  - 单一类型
  - 集合类型
- 使用 API ObjectProvider 延迟注入（推荐）
   - 单一类型
   - 集合类型  
- 依赖注入处理过程
  - 入口 - DefaultListableBeanFactory#resolveDependency
  - 依赖描述符 - DependencyDescriptor
  - 自定绑定候选对象处理器 - AutowireCandidateResolver
@Autowired 实时注入+类型查找
# 依赖查找 等容器启动完成之后进行的初始化操作
 ## 依赖查找的类型
 1. 单一类型
 2. 集合类型 
 3. 层次类型 
### 单一类型依赖查找接口 - BeanFactory
  - 根据 Bean 名称查找 
    - getBean(String) **非类型安全**
  - 根据 Bean 类型查找
    - 实时查找  
      - Spring 3.0 getBean(Class) **非类型安全**
    - Spring 5.1 延迟查找
      - getBeanProvider(Class)
      - getBeanProvider(ResolvableType)
   - 根据 Bean 名称+类型查找
     - getBean(String,Class)
### 集合类型依赖查找接口 - ListableBeanFactory
  - 根据Bean类型查找
    - 获取同类型 Bean 名称列表，建议先通过名称的方式去查找
      - getBeanNamesForType(Class)
      - Spring 4.2 getBeanNamesForType(ResolvableType)
  - 获取同类型 Bean 实例列表，可能会提前触发一些 Bean 的初始化
    - getBeansOfType(Class) 以及重载方法 **类型安全**
  - 通过注解类型查找
    - Spring 3.0 获取标注类型 Bean 名称列表
      - getBeanNamesForAnnotation(Class<? extends Annotation>)
    - Spring 3.0 获取标注类型 Bean 实例列表
      - getBeansWithAnnotation(Class<? extends Annotation>)
    - Spring 3.0 获取指定名称+ 标注类型 Bean 实例
     -- findAnnotationOnBean(String, Class<? extend Annotation>)
### 层次类型依赖查找接口 - HierarchicalBeanFactory  
HierarchicalBeanFactory-->ConfigurableBeanFactory-->ConfigurableListableBeanFactory 类似于 ClassLoader 双亲委派模式  
HierarchicalBeanFactory#getParentBeanFactory  读  
子类ConfigurableBeanFactory#setParentBeanFactory  写
  - 双亲 BeanFactory：getParentBeanFactory()
  - 层次性查找
    - 根据 Bean 名称查找
      - 基于 HierarchicalBeanFactory#containsLocalBean方法实现
  - 根据 Bean 类型查找实例列表
    - 单一类型：BeanFactoryUtils#beanOfType
    - 集合类型：BeanFactoryUtils#beansOfTypeIncludingAncestors
  - 根据 Java 注解查找名称列表
    - BeanFactoryUtils#beanNamesForTypeIncludingAncestors
层次类型依赖查找的安全性取决于其扩展的单一或者集合类型的 BeanFactory 接口
### Bean 延迟依赖查找接口
  - ObjectFactory **非类型安全**
    - getObject
  - ObjectProvider  
    - Spring5 对 Java8 特性扩展--函数式接口
      - getIfAvailable(Supplier) **类型安全**
      - ifAvailable(Consumer)
   - Stream扩展 - stream() **类型安全**  
如果需要实现延时查找，推荐使用 ObjectProvider 既能表示单一类型查找，又能表示集合类型查找
DefaultListableBeanFactory 既是单一也是集合还是层次  
BeanFactory -> HierarchicalBeanFactory -> ConfigurableBeanFactory\ListableBeanFactory -> ConfigurableListableBeanFactory-> DefaultListableBeanFactory
 
**AbstractApplicationContext 内建可查询依赖**  

|Bean 名称|Bean 实例|使用场景|
|:----:|:----:|:----:|
| environment | Environment 对象 | 外部化配置以及Profiles |
| systemProperties | java.util.Properties 对象 | Java 系统属性 |
| systemEnvironment | java.util.Map 对象 | 操作系统环境变量 |
| messageSource | MessageSource 对象 | 国际化文案 |
| lifecycleProcessor | LifeProcessor 对象 | Lifecycle Bean 处理器 |
| applicationEventMulticaster | ApplicationEventMulticaster 对象 | Spring 事件广播器 |
### 面试题
1. 
 


 
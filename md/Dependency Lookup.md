# 依赖查找 等容器启动完成之后进行的初始化操作
 ## 依赖查找的类型
 1. 单一类型
 2. 集合类型 
 3. 层次类型 
### 单一类型依赖查找接口 - BeanFactory
  - 根据Bean名称查找
    - getBean(String)
  - 根据Bean类型查找
    - 实时查找  
      - Spring 3.0 getBean(Class)
    - Spring 5.1 延迟查找
      - getBeanProvider(Class)
      - getBeanProvider(ResolvableType)
   - 根据Bean 名称+类型查找
     - getBean(String,Class)
### 集合类型依赖查找接口 - ListableBeanFactory
  - 根据Bean类型查找
    - 获取同类型Bean名称列表，建议先通过名称的方式去查找
      - getBeanNamesForType(Class)
      - Spring 4.2 getBeanNamesForType(ResolvableType)
  - 获取同类型Bean实例列表，可能会提前触发一些Bean的初始化
    - getBeansOfType(Class)以及重载方法
  - 通过注解类型查找
    - Spring 3.0 获取标注类型Bean名称列表
      - getBeanNamesForAnnotation(Class<? extends Annotation>)
    - Spring 3.0 获取标注类型Bean实例列表
      - getBeansWithAnnotation(Class<? extends Annotation>)
    - Spring 3.0 获取指定名称+ 标注类型Bean 实例
     -- findAnnotationOnBean(String, Class<? extend Annotation>)
### 层次类型依赖查找接口 - HierarchicalBeanFactory
  - 双亲BeanFactory：getParentBeanFactory()
  - 层次性查找
    - 根据Bean 名称查找
      - 基于 containsLocalBean方法实现
  - 根据Bean类型查找实例列表
    - 单一类型：BeanFactoryUtils#beanOfType
    - 集合类型：BeanFactoryUtils#beansOfTypeIncludingAncestors
  - 根据Java 注解查找名称列表
    - BeanFactoryUtils#beanNamesForTypeIncludingAncestors
    
### 面试题
1. 
 


 
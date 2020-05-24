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




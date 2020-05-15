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
  - 字段
  - 方法
  - 接口回调
  

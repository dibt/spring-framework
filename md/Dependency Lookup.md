# 依赖查找
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
 
### 面试题
1. 
 


 
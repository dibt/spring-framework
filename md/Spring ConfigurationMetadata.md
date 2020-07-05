### Spring 配置元信息
 - Bean 配置元信息 - BeanDefinition
   - GenericBeanDefinition：通用型 BeanDefinition 只是简单继承了 AbstractBeanDefinition，实现比较一般
   - RootBeanDefinition：无 Parent 的 BeanDefinition 或者合并后的 BeanDefinition，提供的更全面的功能
   - AnnotatedBeanDefinition：注解标注的 BeanDefinition
     - AnnotationMetadata
       - 普通注解，StandardAnnotationMetadata 基于 Java 反射实现
       - AnnotationMetadataReadingVisitor 基于 ASM 动态字节码技术，性能会更高，不需要加载整个类，Spring5.2 之后已经被 @Deprecated，推荐使用 
       SimpleAnnotationMetadataReadingVisitor
### Spring 事务原理
TransactionInterceptor#invoke  
TransactionAspectSupport#invokeWithinTransaction  
PlatformTransactionManager -> DataSourceTransactionManager  
TransactionInfo -> AbstractPlatformTransactionManager#getTransaction  
当前线程已经在一个事务中，则 handleExistingTransaction  
当前线程如果不在一个事务中，则 newTransactionStatus  
判断当前线程是否在一个事务中 DataSourceTransactionManager#isExistingTransaction -> ConnectionHolder#transactionActive(默认是false)  
回滚事务 TransactionAspectSupport#completeTransactionAfterThrowing -> AbstractPlatformTransactionManager#rollback -> 
AbstractPlatformTransactionManager#processRollback  
提交事务 TransactionAspectSupport#commitTransactionAfterReturning -> AbstractPlatformTransactionManager#commit

TransactionStatus#getTransaction 默认实现  AbstractPlatformTransactionManager#getTransaction  
TransactionDefinition  

### AOP调用顺序
AbstractAutoProxyCreator#wrapIfNecessary -> AbstractAutoProxyCreator#createProxy -> ProxyFactory#getProxy -> 
ProxyCreatorSupport#createAopProxy -> DefaultAopProxyFactory#createAopProxy  

### JDK 动态代理
Proxy#newProxyInstance  
- 运行时获取需要生成代理对象的 Class 对象
- 通过 Class 对象获取所需构造器
- 将 InvocationHandler 实例作为参数，通过构造器创建代理对象  
Proxy#ProxyClassFactory -> ProxyGenerator.generateProxyClass -> Proxy#defineClass0  

Class#getDeclaredMethod  
- 其中 privateGetDeclaredMethods 方法从缓存或 JVM 中获取该 Class 中申明的方法列表
- searchMethods 方法将从返回的方法列表里找到一个匹配名称和参数的方法对象。
searchMethods -> Method#copy 所次每次调用 getDeclaredMethod 方法返回的 Method 对象其实都是一个新的对象，且新对象的 root 属性都指向原来的 Method
对象，如果需要频繁调用，最好把 Method 对象缓存起来  

privateGetDeclaredMethods 中的数据结构 ReflectionData，用来缓存从JVM中读取类的属性，reflectionData 
对象是 SoftReference 类型的，说明在内存紧张时可能会被回收，不过也可以通过 -XX:SoftRefLRUPolicyMSPerMB 参数控制回收的时机，只要发生 GC 就会将其回收，如果 reflectionData 
被回收之后，又执行了反射方法，那只能通过 newReflectionData 方法重新创建一个这样的对象了，通过 unsafe.compareAndSwapObject 方法重新设置 reflectionData 字段，在 
privateGetDeclaredMethods 方法中，如果通过 reflectionData 获得的 ReflectionData 对象不为空，则尝试从 ReflectionData 对象中获取 declaredMethods 
属性，如果是第一次，或则被GC回收之后，重新初始化后的类属性为空，则需要重新到 JVM 中获取一次，并赋值给 ReflectionData，下次调用就可以使用缓存数据了。  

Method#invoke 这里的 MethodAccessor 对象是 invoke 方法实现的关键，一开始 methodAccessor 
为空，需要调用 acquireMethodAccessor 生成一个新的 MethodAccessor 对象，在 
acquireMethodAccessor 方法中，会通过 ReflectionFactory 类的 newMethodAccessor 创建一个实现 MethodAccessor 接口的对象，MethodAccessor 
实现有两个版本  
- Java 实现的(DelegatingMethodAccessorImpl),Java实现的版本在初始化时需要较多时间，但长久来说性能较好；  
- native code实现的(NativeMethodAccessorImpl)  

为了权衡两个版本的性能，Sun的JDK使用了inflation的技巧：让Java方法在被反射调用时，开头若干次(ReflectionFactory 的 inflationThreshold 属性，默认为 15)
使用native版，等反射调用次数超过阈值（15次）时则生成一个专用的 MethodAccessor实现类，生成其中的 invoke() 方法的字节码，以后对该 Java 方法的反射调用就会使用 Java 版。

在 ReflectionFactory 类中，有两个重要的字段：noInflation (默认false)和 inflationThreshold (默认15)，在 checkInitted 方法中可以通过 -Dsun.reflect
.inflationThreshold=xxx和-Dsun.reflect.noInflation=true 对这两个字段重新设置，而且只会设置一次；如果 
noInflation 为 false，方法 newMethodAccessor 都会返回 MethodAccessorImpl  的一个子类对象，并设置到 DelegatingMethodAccessorImpl 对象里去了。  
其实 DelegatingMethodAccessorImpl 
对象就是一个代理对象，负责调用被代理对象 delegate 的 invoke 方法，其中 delegate 参数目前是 NativeMethodAccessorImpl 对象，所以最终 Method 的 invoke 
方法调用的是 NativeMethodAccessorImpl 对象 invoke 方法，这里用到了 
ReflectionFactory 类中的 inflationThreshold，当 delegate 调用了15次 invoke 方法之后，如果继续调用就通过 MethodAccessorGenerator 
类的 generateMethod 
方法生成 MethodAccessorImpl 对象，并设置为 delegate 对象，这样下次执行 Method.invoke 时，就调用新建的 MethodAccessor 对象的 invoke 方法了。
** 这里需要注意的是：generateMethod 方法在生成 MethodAccessorImpl 对象时，会在内存中生成对应的字节码，并调用 ClassDefiner.defineClass 创建对应的 class 对象，在 
ClassDefiner.defineClass 方法实现中，每被调用一次都会生成一个 DelegatingClassLoader 
类加载器对象，这里每次都生成新的类加载器，是为了性能考虑，在某些情况下可以卸载这些生成的类，因为类的卸载是只有在类加载器可以被回收的情况下才会被回收的，如果用了原来的类加载器，那可能导致这些新创建的类一直无法被卸载 **
### CGLib动态代理  
CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成某个类的子类，注意，CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。

MethodInterceptor#intercept
Enhancer#create -> Enhancer#createHelper -> AbstractClassGenerator#create(Object) -> ClassLoaderData#get -> 
AbstractClassGenerator#generate  
- 代理类字节码的默认生成策略 DefaultGeneratorStrategy.INSTANCE  
- 代理类的默认命名策略 DefaultNamingPolicy.INSTANCE  
DefaultGeneratorStrategy 类中的 generate 方法，这是真正生成代理类的地方  
MethodProxy类有两个主要的方法:  
- invoke：调用代理类中被调用的方法  
- invokeSuper: 直接在代理类中调用父类的方法
CGLIB 动态代理执行代理方法效率之所以比JDK 动态代理高，是因为 CGLIB 采用了 FastClass 机制。  
FastClass 的原理简单来说就是：为不需要反射 invoke 调用的原类型生成一个 FastClass 类，然后给原类型的方法分配一个 index，在生成的 FastClass 中的 invoke 方法中，先直接把 Object
 强制转换为原类型，然后根据这个 index，就可以直接定位要调用的方法直接进行调用，这样省去了反射调用，所以调用效率比 JDK 动态代理通过反射调用效率高。

区别
- JDK动态代理  代理类与委托类实现同一接口，主要是通过代理类实现 InvocationHandler 并重写 invoke 方法来进行动态代理的，在 invoke 方法中将对方法进行增强处理，底层使用反射机制进行方法的调用
- CGLIB动态代理  代理类将委托类作为自己的父类并为其中的非 final 委托方法创建两个方法，一个是与委托方法签名相同的方法，它在方法中会通过 
super 调用委托方法；另一个是代理类独有的方法。在代理方法中，它会判断是否存在实现了 MethodInterceptor 接口的对象，若存在则将调用intercept方法对委托方法进行代理，底层将方法全部存入一个数组中，通过数组索引直接进行方法调用

### Spring 解决循环依赖
DefaultListableBeanFactory#registerSingleton -> DefaultSingletonBeanRegistry#registerSingleton，其中 
DefaultSingletonBeanRegistry 的三个 Map 是解决循环依赖的重要的缓存。  
```  
        /** 一级缓存 缓存创建完成的单例对象的容器 */
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

	/** Cache of singleton factories: bean name to ObjectFactory. */
	/** 三级缓存 缓存创建单例对象所使用的 BeanFactory */
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

	/** Cache of early singleton objects: bean name to bean instance. */
	/** 二级缓存 缓存单例对象早期的引用，这里的 Object 只是一个 Instance，还没有完成 Init，不是一个完整的 Bean  */
	private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
```
AbstractApplicationContext#refresh -> AbstractApplicationContext#finishBeanFactoryInitialization -> 
**AbstractApplicationContext#getBean** -> AbstractBeanFactory#getBean -> AbstractBeanFactory#doGetBean -> 
AbstractAutowireCapableBeanFactory#createBean -> AbstractAutowireCapableBeanFactory#doCreateBean -> 
AbstractAutowireCapableBeanFactory#populateBean -> AbstractAutowireCapableBeanFactory#applyPropertyValues -> 
BeanDefinitionValueResolver#resolveValueIfNecessary -> BeanDefinitionValueResolver#resolveReference -> 
**AbstractApplicationContext#getBean**

### ThreadLocal  
  - InheritableThreadLocal 父子线程之间传递参数，入口为 Thread#init
  

                                             
                                         





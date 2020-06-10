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

### Java 反射
Class#getDeclaredMethod，其中 privateGetDeclaredMethods 方法从缓存或 JVM 中获取该 Class 中申明的方法列表，searchMethods 
方法将从返回的方法列表里找到一个匹配名称和参数的方法对象。
searchMethods -> Method#copy 所次每次调用 getDeclaredMethod 方法返回的 Method 对象其实都是一个新的对象，且新对象的 root 属性都指向原来的 Method
对象，如果需要频繁调用，最好把 Method 对象缓存起来  
privateGetDeclaredMethods 中的数据结构 ReflectionData，用来缓存从JVM中读取类的如下属性数，reflectionData 
对象是 SoftReference 类型的，说明在内存紧张时可能会被回收，不过也可以通过 -XX:SoftRefLRUPolicyMSPerMB 参数控制回收的时机，只要发生 GC 就会将其回收，如果 reflectionData 
被回收之后，又执行了反射方法，那只能通过 newReflectionData 方法重新创建一个这样的对象了，通过 unsafe.compareAndSwapObject 方法重新设置 reflectionData 字段；  
在 privateGetDeclaredMethods 方法中，如果通过 reflectionData 
获得的 ReflectionData 对象不为空，则尝试从 ReflectionData 对象中获取 declaredMethods 属性，如果是第一次，或则被GC回收之后，重新初始化后的类属性为空，则需要重新到 JVM 
中获取一次，并赋值给 ReflectionData，下次调用就可以使用缓存数据了。
Method#invoke 这里的 MethodAccessor 对象是 invoke 方法实现的关键，一开始 methodAccessor 
为空，需要调用 acquireMethodAccessor 生成一个新的 MethodAccessor 对象，在 
acquireMethodAccessor 方法中，会通过 ReflectionFactory 类的 newMethodAccessor 创建一个实现 MethodAccessor 接口的对象，MethodAccessor 
实现有两个版本  
- （1）一个是Java实现的。Java实现的版本在初始化时需要较多时间，但长久来说性能较好；  
- （2）另一个是native code实现的  

为了权衡两个版本的性能，Sun的JDK使用了inflation的技巧：让Java方法在被反射调用时，开头若干次(ReflectionFactory 的 inflationThreshold 属性，默认为 15)
使用native版，等反射调用次数超过阈值（15次）时则生成一个专用的 MethodAccessor实现类，生成其中的 invoke() 方法的字节码，以后对该 Java 方法的反射调用就会使用 Java 版。

在 ReflectionFactory 类中，有两个重要的字段：noInflation (默认false)和 inflationThreshold (默认15)，在 checkInitted 方法中可以通过 -Dsun.reflect
.inflationThreshold=xxx和-Dsun.reflect.noInflation=true 对这两个字段重新设置，而且只会设置一次；如果 
noInflation 为 false，方法 newMethodAccessor 都会返回 DelegatingMethodAccessorImpl 对象，其实 DelegatingMethodAccessorImpl 
对象就是一个代理对象，负责调用被代理对象 delegate 的 invoke 方法，其中 delegate 参数目前是 NativeMethodAccessorImpl 对象，所以最终 Method 的 invoke 
方法调用的是 NativeMethodAccessorImpl 对象 invoke 方法，这里用到了 
ReflectionFactory 类中的 inflationThreshold，当 delegate 调用了15次 invoke 方法之后，如果继续调用就通过 MethodAccessorGenerator 
类的 generateMethod 
方法生成 MethodAccessorImpl 对象，并设置为 delegate 对象，这样下次执行 Method.invoke 时，就调用新建的 MethodAccessor 对象的 invoke 方法了。
** 这里需要注意的是：generateMethod 方法在生成 MethodAccessorImpl 对象时，会在内存中生成对应的字节码，并调用 ClassDefiner.defineClass 创建对应的 class 对象，在 
ClassDefiner.defineClass 方法实现中，每被调用一次都会生成一个 DelegatingClassLoader 
类加载器对象，这里每次都生成新的类加载器，是为了性能考虑，在某些情况下可以卸载这些生成的类，因为类的卸载是只有在类加载器可以被回收的情况下才会被回收的，如果用了原来的类加载器，那可能导致这些新创建的类一直无法被卸载 **
                                             
                                         





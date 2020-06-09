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
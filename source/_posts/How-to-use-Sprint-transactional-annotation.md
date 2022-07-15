---
title: How to use Sprint transactional annotation
date: 2022-07-15 20:58:18
categories: BackEnd 
tags:
    - Spring 
top:
---
# How to use Sprint transactional annotation

# 1. Attributes

- `value` and `transactionManager`
    - used to provide a TransactionManager reference to be used when handling the transaction for the annotated block
- `propagation`
    - define how the transaction boundaries propagate to other methods that will be called either directly or indirectly from within the annotated block
    - default is `REQUIRED`
        - means a transaction is started if no transaction is already available
        - otherwise use the current running thread
- `timeout` and `timeoutString`
    - define the max number of seconds the current method is allowed to run
- `readOnly`
    - defines if the current transaction is read-only or read-write
- `rollbackFor` and `rollbackForClassName`
    - define one or more Throwable classes for which the current transaction will be rolled back
    - default is RuntimeException or an Error is thrown, but not for a checked Exception
- `noRollbackFor` and `noRollbackForClassName`
    - used for one or more RuntimeException

# 2. Transaction Override

- `addStatementReportOperation` is using the serializeble level, which override the class level readonly transaction

```java
@Service
@Transactional(readOnly = true)
public class OperationService {
 
    @Transactional(isolation = Isolation.SERIALIZABLE)
    public boolean addStatementReportOperation(
        String statementFileName,
        long statementFileSize,
        int statementChecksum,
        OperationType reportType) {
         
        ...
    }
}
```

- we use readOnly in class level because Spring could perform some read only optimization
    - we could save memory when loading read only entities since the loaded state is discarded right away, and not kept for the whole duration of the currently running persistence context
    - also, in the cluster, read only data source could be redirected to DB replica instead of DB primary, this could also reduce the burden for primary

1. [https://stackoverflow.com/questions/10394857/how-to-use-transactional-with-spring-data#:~:text=Thus we recommend using %40Transactional,re-decorated in that interface](https://stackoverflow.com/questions/10394857/how-to-use-transactional-with-spring-data#:~:text=Thus%20we%20recommend%20using%20%40Transactional,re%2Ddecorated%20in%20that%20interface) 
2. [https://vladmihalcea.com/spring-read-only-transaction-hibernate-optimization/](https://vladmihalcea.com/spring-read-only-transaction-hibernate-optimization/)  
3. [https://vladmihalcea.com/spring-transactional-annotation/](https://vladmihalcea.com/spring-transactional-annotation/)
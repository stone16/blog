---
title: Spring @Transactional 事务Tips
date: 2020-08-24 20:31:52
categories: BackEnd
tags:
    - 事务
top:
---
Spring针对Transaction APi, JDBC, Hibernate, Java Persistence API等事务API，实现了一致的编程模型，而Spring的声明式事务功能提供了非常方便的事务配置方式，使用`@Transactional`注解，就可以一键开启方法的事务性配置。

但是不是加上标注就能实现事务的，还是需要去关注事务是否有效，出错以后事务是否会正确回滚，当业务代码设计到多个子业务逻辑的时候，怎么正确处理事务。

# 1. 事务生效问题

```

@Entity
@Data
public class UserEntity {
    @Id
    @GeneratedValue(strategy = AUTO)
    private Long id;
    private String name;

    public UserEntity() { }

    public UserEntity(String name) {
        this.name = name;
    }
}



@Repository
public interface UserRepository extends JpaRepository<UserEntity, Long> {
    List<UserEntity> findByName(String name);
}


@Service
@Slf4j
public class UserService {
    @Autowired
    private UserRepository userRepository;

    //一个公共方法供Controller调用，内部调用事务性的私有方法
    public int createUserWrong1(String name) {
        try {
            this.createUserPrivate(new UserEntity(name));
        } catch (Exception ex) {
            log.error("create user failed because {}", ex.getMessage());
        }
        return userRepository.findByName(name).size();
    }

    //标记了@Transactional的private方法
    @Transactional
    private void createUserPrivate(UserEntity entity) {
        userRepository.save(entity);
        if (entity.getName().contains("test"))
            throw new RuntimeException("invalid username!");
    }

    //根据用户名查询用户数
    public int getUserCount(String name) {
        return userRepository.findByName(name).size();
    }
}
```

上述代码使用JPA做数据库访问，Entity定义在UserEntity当中，在服务层，声明了createUsr方法，当名字包含test的时候，希望抛出异常，然后实现数据库的回滚（只是例子，当然实际实现上将判断和数据库存储执行顺序换过来就能避开这里的问题了）

当调用的时候，发现即使用户名不合法，也能够调用成功，这是因为上述代码将注解定义到了private方法，因此不生效

> 只有定义在public方法上的@Transactional才能生效，因为Spring默认通过动态代理的方式实现AOP，对目标方法进行增强，private方法无法代理到，Spring也就无法使用动态增强事务处理的逻辑了。

然而就算把上述的private方法改为public transactional依旧不会生效，这是因为：

> Transactional需要通过代理过的类从外部调用目标方法才能生效

Spring通过AOP技术对方法进行增强，要调用增强过的方法必然是调用代理之后的对象

因此我们可以在controller层调用这个逻辑，来实现整个transactional的支持。即你需要使用Spring注入的类，通过代理调用才有机会来进行动态的增强。

# 2. 事务回滚问题

通过AOP锁实现的事务处理可以理解为使用try catch来包裹标记了`@Transactional`注解的方法，当方法出现了异常并且满足一定条件的时候，在catch里面我们可以设置事务回滚，没有异常则直接提交事务。

1. 只有异常传播出标记了注解的方法，事务才能回滚


```
try {
   // This is an around advice: Invoke the next interceptor in the chain.
   // This will normally result in a target object being invoked.
   retVal = invocation.proceedWithInvocation();
}
catch (Throwable ex) {
   // target invocation exception
   completeTransactionAfterThrowing(txInfo, ex);
   throw ex;
}
finally {
   cleanupTransactionInfo(txInfo);
}
```

2. 默认情况下，出现RuntimeException或者Error的时候，Spring才会回滚事务


在必要的时候，可以选择手动进行回滚，以及遇到所有的Exception都回滚事务

```

@Transactional
public void createUserRight1(String name) {
    try {
        userRepository.save(new UserEntity(name));
        throw new RuntimeException("error");
    } catch (Exception ex) {
        log.error("create user failed", ex);
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
    }
}


@Transactional(rollbackFor = Exception.class)
public void createUserRight2(String name) throws IOException {
    userRepository.save(new UserEntity(name));
    otherTask();
}

```

+ 有时我们会遇到嵌套逻辑，分别需要实现事务的问题，而子逻辑事务的回滚不希望影响到父逻辑，可以使用`@Transactional(propagation = Propagation.REQUIRES_NEW)`, 以此来设置事务传播策略，即执行到这个方法的时候需要开启新的事务，并挂起当前事务。
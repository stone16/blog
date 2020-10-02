---
title: 'Spring IoC, AOP浅析'
date: 2020-10-01 19:43:05
categories: BackEnd
tags: 
    - Spring
top:
---
# 1. IoC 

## 1.1 Overview

+ Inversion of control 
    + 控制反转
    + 将设计好的对象交给Spring容器来控制，而不是在对象内部控制

+ 好处
    + 可以无侵入的调整对象的关系
    + 同时可以无侵入的调整对象的属性，甚至实现对象的替换

## 1.2 单例Bean注入Prototype的Bean

Spring创建的Bean默认是单例的，但是当Bean遇到继承的时候，是会忽略这一点的。

```

@Slf4j
public abstract class SayService {
    List<String> data = new ArrayList<>();

    public void say() {
        data.add(IntStream.rangeClosed(1, 1000000)
                .mapToObj(__ -> "a")
                .collect(Collectors.joining("")) + UUID.randomUUID().toString());
        log.info("I'm {} size:{}", this, data.size());
    }
}


@Service
@Slf4j
public class SayHello extends SayService {
    @Override
    public void say() {
        super.say();
        log.info("hello");
    }
}

@Service
@Slf4j
public class SayBye extends SayService {
    @Override
    public void say() {
        super.say();
        log.info("bye");
    }
}
```

上述代码中，基类SayService是有状态的，dataList一直在增加。当SayHello类继承基类，并且声明为Service的时候，将其注册为Bean。这时候有状态的基类就很有可能造成内存泄露或者线程安全的问题了。

正确做法是在将类标记为`@Service`并交给容器进行管理之前，需要首先评估一下类是否有状态，然后为Bean设置合适的Scope。

```
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE, proxyMode = ScopedProxyMode.TARGET_CLASS)
```

让其以代理的方式注入，这样虽然controller还是单例的，但是每次都从代理那里获得Service，这样prototype范围的配置才会真正生效。



# 2. AOP 

+ 体现了松耦合，高内聚
+ 在切面集中实现横切关注点
    + 缓存
    + 权限
    + 日志

+ 然后通过切点配置将代码注入到合适的地方

+ 关键点
    + 连接点 Join Point 
        + 实现AOP的地方
        + 方法执行

    + 切点 PointCut
        + 告诉程序在哪里做切入
        + Spring中默认使用AspectJ查询表达式，通过在连接点运行查询表达式来匹配切入点

    + 增强 Advice
        + 定义了切入切点后增强的方式
            + 前
            + 后
            + 环绕

    + 切面 Aspect
        + 切面 = 切点 + 增强 
        + 实现整个AOP操作


## 2.1 实现整个日志记录，异常处理，方法耗时的统一切面
```
// 定义一个自定义注解Metrics 

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD, ElementType.TYPE})
public @interface Metrics {

    /**
     * 在方法成功执行后打点，记录方法的执行时间发送到指标系统，默认开启
     *
     * @return
     */
    boolean recordSuccessMetrics() default true;

    /**
     * 在方法成功失败后打点，记录方法的执行时间发送到指标系统，默认开启
     *
     * @return
     */
    boolean recordFailMetrics() default true;

    /**
     * 通过日志记录请求参数，默认开启
     *
     * @return
     */
    boolean logParameters() default true;

    /**
     * 通过日志记录方法返回值，默认开启
     *
     * @return
     */
    boolean logReturn() default true;

    /**
     * 出现异常后通过日志记录异常信息，默认开启
     *
     * @return
     */
    boolean logException() default true;

    /**
     * 出现异常后忽略异常返回默认值，默认关闭
     *
     * @return
     */
    boolean ignoreException() default false;
}
```

```
// 实现一个切面完成Metrics注解提供的功能

@Aspect
@Component
@Slf4j
public class MetricsAspect {


	@Autowired
	private ObjectMapper ObjectMapper;

	//实现一个返回Java基本类型默认值的工具。其实，你也可以逐一写很多if-else判断类型，然后手动设置其默认值。
	//这里为了减少代码量用了一个小技巧，即通过初始化一个具有1个元素的数组，然后通过获取这个数组的值来获取基本类型默认值 
	//Array.newInstance(Class<?> componentType, int length) 创建一个有特定的类的类型和长度的对象
	private static final Map<Class<?>, Object> DEFAULT_VALUES = 
		Stream.of(boolean.class, byte.class, char.class, double.class, float.class, int.class, long.class, short.class) 
			.collect(toMap(clazz -> (Class) clazz, clazz -> Array.get(Array.newInstance(clazz, 1), 0))); 

	public static T getDefaultValue(Class clazz) { 
		return (T) DEFAULT_VALUES.get(clazz); 
	}

	// 实现了对标记了Metrics注解的方法进行匹配
	@PointCut("within(@org.cleilei.commonmistakes.springpart1.aopmetrics.Metrics *)")
	public void withMetricsAnnotation() {
	}

	// 实现了匹配类型上标记了@RestController注解的方法
	@PointCut("within(@org.springframework.web.bind.annotation.RestController *)")
	public void controllerBean() {
	}

	@Around("controllerBean() || withMetricsAnnotation()")
	public Object metrics(ProceedingJoinPoint pjp) throws Throwable {

		// 通过连接点获取方法签名和方法上Metrics的注解，并根据方法签名生成日志中要输出的方法定义描述
		MethodSignature signature = (MethodSignature)pjp.getSignature();
		Metrics metrics = signature.getMethod().getAnnotation(Metrics.class);

		String name = String.format("%s %s", signature.getDeclaringType().toString(), signature.toLongString());

		if (metrics == null) {
			@Metrics
			final class c {} 
			metrics = c.class.getAnnotation(Metrics.class);
		}

		// 尝试从上下文获取请求的URL，来方便定位问题
		RequestAttributes RequestAttributes = RequestContextHolder.getRequestAttributes();

		if (requestAttributes != null) {
			HttpServletRequest request = ((ServletRequestAttributes) requestAttributes).getRequest();
			if (request != null) {
				name += String.format(" %s ", request.getRequestURL().toString());
			}
		}

		// 记录参数
		if (metrics.logParameters()) {
			log.info(String.format("Call method %s with parameters %s", name. objectMapper.writeValueAsString(pjp.getArgs())));
		}

		// 记录方法的执行，break points, 异常时记录
        Object returnValue;
        Instant start = Instant.now();
        try {
            returnValue = pjp.proceed();
            if (metrics.recordSuccessMetrics())
                //在生产级代码中，我们应考虑使用类似Micrometer的指标框架，把打点信息记录到时间序列数据库中，实现通过图表来查看方法的调用次数和执行时间
                log.info(String.format("Call method %s succeed，time used：%d ms", name, Duration.between(start, Instant.now()).toMillis()));
        } catch (Exception ex) {
            if (metrics.recordFailMetrics())
                log.info(String.format("Call method %s fail，time used：%d ms", name, Duration.between(start, Instant.now()).toMillis()));
            if (metrics.logException())
                log.error(String.format("Call method %s with exceptions", name), ex);

            //忽略异常的时候，使用一开始定义的getDefaultValue方法，来获取基本类型的默认值
            if (metrics.ignoreException())
                returnValue = getDefaultValue(signature.getReturnType());
            else
                throw ex;
        }
        //实现了返回值的日志输出
        if (metrics.logReturn())
            log.info(String.format("Call method %s with result: %s", name, returnValue));
        return returnValue;
	}
}
```
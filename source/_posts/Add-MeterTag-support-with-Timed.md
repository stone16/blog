---
title: Add @MeterTag support with @Timed
date: 2023-09-23 17:01:30
categories: BackEnd
tags:
    - micrometer
    - datadog
    - metrics 
    - springBoot
top:
---
# Background

Micrometer add @MeterTag support on 2023, we are integrating micrometer with datadog in a spring boot project, this blog document how we set @MeterTag with @Timed 

# What’s MeterTag

- With MeterTag annotation, we could add tags to `@Timed` with some dynamic value, e.g, sth parameters in the input. We could leverage on valueResolver, and use some expression language. For our case, Spel will be totally fine.

# Implementation

Import related dependencies, in gradle kotlin, it would contains snippets as below

```jsx
implementation("io.micrometer:micrometer-registry-statsd:1.11.4")
implementation("io.micrometer:micrometer-core:1.11.4")
implementation("io.micrometer:micrometer-commons:1.11.4")
implementation("org.springframework:spring-aspects")
```

In application class, enable the AspectJAutoProxy 

```jsx
@SpringBootApplication
@Import(HttpSecurityConfig::class)
@EnableScheduling
@EnableIntegrationManagement
@EnableAspectJAutoProxy(proxyTargetClass = true)
@Modulithic
class Application

fun main(args: Array<String>) {
    runApplication<Application>(args = args)
}
```

For the annotation you want to use, we need to define the corresponding aspect bean. As we want to enhance the `@Timed` with `@MeterTag` we need to set the meterTagAnnotationHandler 

```jsx
@Configuration
class MetricsConfig {
    @Bean
    fun timedAspect(meterRegistry: MeterRegistry): TimedAspect {
        val timedAspect = TimedAspect(meterRegistry)
        timedAspect.setMeterTagAnnotationHandler(
            MeterTagAnnotationHandler(
                { ValueResolver { p -> p.toString() } },
                { CachedSpelValueExpressionResolver() },
            ),
        )
        return timedAspect
    }
}
```

We mainly leverage on CachedSpelValueExpressionResolver to pass in an expression following spring expression language, the resolver is defined as follow

```jsx
open class SpelValueExpressionResolver : ValueExpressionResolver {

    private val log = KotlinLogging.logger {}
    override fun resolve(expression: String, parameter: Any): String {
        try {
            val context = SimpleEvaluationContext.forReadOnlyDataBinding().withInstanceMethods().build()
            return parseExpression(expression).getValue(context, parameter, String::class.java) ?: ""
        } catch (ex: Exception) {
            log.error("Exception occurred while trying to evaluate the SpEL expression [$expression]", ex)
        }
        return parameter.toString()
    }

    open fun parseExpression(expression: String): Expression =
        SpelExpressionParser().parseExpression(expression)
}

class CachedSpelValueExpressionResolver : SpelValueExpressionResolver() {
    private val expressionsCache: MutableMap<String, Expression> = ConcurrentHashMap()
    override fun parseExpression(expression: String): Expression =
        expressionsCache.computeIfAbsent(expression) {
            super.parseExpression(expression)
        }
}
```

After that, we could add metrics as we need: 

```jsx
@DgsData(parentType = DgsConstants.QUERY.TYPE_NAME, field = DgsConstants.QUERY.HelloWorldPing)
@Timed
fun helloWorld(@MeterTag(key="message.size", expression="this.size")message: String): String = message
```

For local testing, as datadog send out metrics in UDP, we could write some script with a UDP socket server to print out the content 

```jsx
const dgram = require("dgram");

const port = process.argv[2] || 8125;
const socket = dgram.createSocket({ type: "udp4", reuseAddr: true });

socket.on("message", msg => console.log(msg.toString()));
socket.on("error", error => {
  console.log("error", error);
  socket.close();
});
socket.bind(port);
```

You should be able to see metrics printed out successfully with tags you want then! 

# Reference

[https://github.com/micrometer-metrics/micrometer/pull/3727](https://github.com/micrometer-metrics/micrometer/pull/3727)
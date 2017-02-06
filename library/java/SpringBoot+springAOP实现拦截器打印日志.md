# Spring boot + spring AOP 实现拦截器打印日志

## 准备工作

1. ##### 首先加入依赖的jar包
   ```java
   <dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-aop</artifactId>  
   </dependency>
   ```
   如果不支持maven,可以直接从这里下载 [ [下载jar包](http://mvnrepository.com/search?q=spring-boot-starter-aop) ]

2. ##### 创建一个拦截器类
    ```java
    package com.qp.interceptor;

    import java.lang.reflect.Method;

    import org.apache.commons.logging.Log;
    import org.apache.commons.logging.LogFactory;
    import org.aspectj.lang.ProceedingJoinPoint;
    import org.aspectj.lang.annotation.Around;
    import org.aspectj.lang.annotation.Aspect;
    import org.aspectj.lang.annotation.Pointcut;
    import org.aspectj.lang.reflect.MethodSignature;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.stereotype.Component;

    /**
     * 拦截器：统计业务方法执行耗时，方便快速定位耗时长的业务方法进行合理优化
     */
    @Aspect
    @Component
    public class LogInterceptor {
    	private static final Log log = LogFactory.getLog(LogInterceptor.class);

    	@Value("${LOG_SWITCH}")
    	private String LOG_SWITCH;  /**日志开关*/

    	/**
    	 * 定义拦截规则：拦截com.qp.service.log包下面的所有类中，所有方法。
    	 */
    	@Pointcut("execution(* com.qp.service.log..*(..))")
    	public void serviceMethodPointcut() {
    	}

    	// @Around在切入点前后切入内容，并且可以自己控制何时执行切入点自身的内容
    	@Around("serviceMethodPointcut()")
    	public Object doAround(ProceedingJoinPoint joinPoint) {
    		long beginTime = System.currentTimeMillis();
    		// 获取被拦截的方法的签名
    		MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
    		Object rs = null;
    		try {
          // 继续执行被拦截的方法
    			rs = joinPoint.proceed();
    		} catch (Throwable e) {
    			e.printStackTrace();
    		}
    		long costTime = System.currentTimeMillis() - beginTime;
    		// 耗时大于50ms的方法才打印日志
    		if (costTime > 50 && isPrintInterceptorLOG != null && "OPEN".equals(LOG_SWITCH)) {
    			log.info("方法[" + method.getName() + "]执行耗时：" + costTime + "ms");
    		}
    		return rs;
    	}
    }
    ```
    需要特别注意的是：这里使用@Around注解修饰的方法doAround()必须得有一个返回值，我这里是返回的Object对象。不然在运行时会抛出一个空返回值的异常。
    而使用使用 @Before @After 注解时则不需要返回。

    添加好拦截器后，所有com.qp.service.log包的方法在执行时都会被这里定义的拦截器拦截，并执行doAround（）方法。

3. ##### 测试
   直接调用对应接口或单元测试如果控制台成功打印结果，则表示配置成功（为方便查看测试结果，可以去掉>50才打印日志这一条件）：
       [INFO ][2017-02-06 13:04:51][pool-4-thread-1]com.qp.interceptor.LogInterceptor[50]-方法[countMaleViewRegMail]执行耗时：140ms
       [INFO ][2017-02-06 13:04:51][pool-4-thread-1]com.qp.interceptor.LogInterceptor[50]-方法[countRegMaleViewSessionPerson]执行耗时：142ms

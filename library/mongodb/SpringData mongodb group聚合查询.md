# SpringData mongodb group聚合查询

1. ##### 前言
   由于最近系统的数据量激增，导致后台查询速度变慢，所以决定对现有的一些查询语句进行优化。检查了一下代码，发现以前的代码里很多都是一个请求里执行了很多条查询，所以想改成分组聚合减少查询语句，
   分组聚合，即将查询对象按一定条件分组，然后对每一个组进行聚合分析。这样的好处在于可以显著减少所需执行查询语句，从而大大加快查询速度。
   在以前用mysql时，有用过group by语句来实现聚合查询。同样mongodb也提供了group功能。

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

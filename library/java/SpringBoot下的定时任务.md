# Spring boot下的定时任务实现

1. ##### 创建一个定时任务
   ```java
   package com.demo;

   import java.util.Date;

   import org.apache.commons.logging.Log;
   import org.apache.commons.logging.LogFactory;
   import org.springframework.scheduling.annotation.Scheduled;
   import org.springframework.stereotype.Component;

   import com.dsy.utils.DateUtil;

   @Component
   public class ScheduledTask {
   	private static final Log log = LogFactory.getLog(ScheduledTask.class);

   	/**
   	 * 定时任务，每5秒钟执行一次
   	 */
   	@Scheduled(cron = "0/5 * * * * ?")
   	public void test() {
   		log.info(DateUtil.getDateStr("yyyy-MM-dd hh:mm:ss", new Date()));
   	}

   }
   ```

2. ##### 启用定时任务
    在Spring boot中定时任务不是默认开启的，所以这里需要在Application文件中添加@EnableScheduling注解来启用。
    ```java
    package com.demo;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.scheduling.annotation.EnableScheduling;
    @SpringBootApplication
    @EnableScheduling
    public class Application {
        public static void main(String[] args) throws Exception {
            SpringApplication.run(Application.class);
        }
    }
    ```


3. ##### 测试结果
   运行项目，查看执行结果：
   ```java
    [INFO ][03-27 11:55:25][pool-5-thread-1]c.d.s.ScheduledTask[21]-2017-03-27 11:55:25
    [INFO ][03-27 11:55:30][pool-5-thread-1]c.d.s.ScheduledTask[21]-2017-03-27 11:55:30
    [INFO ][03-27 11:55:35][pool-5-thread-1]c.d.s.ScheduledTask[21]-2017-03-27 11:55:35
    [INFO ][03-27 11:55:40][pool-5-thread-1]c.d.s.ScheduledTask[21]-2017-03-27 11:55:40
    [INFO ][03-27 11:55:45][pool-5-thread-1]c.d.s.ScheduledTask[21]-2017-03-27 11:55:45
   ```

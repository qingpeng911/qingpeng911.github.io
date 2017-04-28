# Freemarker配置全局变量

1. ##### 配置文件application.properties中加入变量base
   ```java
   #全局变量
   base=http://127.0.0.1
   ```

2. ##### 创建FreemarkerConfiguration类用作变量加载
    ```java
    package com.qp.config;

    import java.util.HashMap;
    import java.util.Map;

    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer;

    @Configuration
    public class FreemarkerConfiguration extends FreeMarkerAutoConfiguration.FreeMarkerWebConfiguration {

        @Value("${base}")
        private String base;

        @Override
        public FreeMarkerConfigurer freeMarkerConfigurer() {
            FreeMarkerConfigurer configurer = super.freeMarkerConfigurer();
            Map<String, Object> sharedVariables = new HashMap<>();
            //FreeMarker 全局变量
            sharedVariables.put("base", base);
            configurer.setFreemarkerVariables(sharedVariables);
            return configurer;
        }

    }
    ```
    这个时候在项目下的所有ftl文件中均可以通过 ${base} 来取到 配置文件application.properties中配置的base值

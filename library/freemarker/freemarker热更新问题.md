# freemarker热更新问题

 ##### 今天遇到了一个非常奇葩的问题，我在项目中修改了freemarker的模版文件保存后刷新了页面，但是没有起作用，只有重启了Tomcat修改的内容才会生效。    
   我首先检查了spring boot的配置文件application.properties，确认已经确认已经关闭了freemarker的模版缓存，这个没有问题。排除。
  ```java
  spring.freemarker.cache=false
  ```

  然后在网上找了一下解决办法，试着加上了spring-boot-devtools。发现还是没有效果。 参考[ [SpringBoot热加载](http://www.tianshouzhi.com/api/tutorials/springboot/106/) ]  
  ```java
  <dependency>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-devtools</artifactId>
  	<optional>true</optional>
  </dependency>
  ```

  最后在stackoverflow上看到了这个[ [SpringBoot热加载](https://stackoverflow.com/questions/33349456/how-to-make-auto-reload-with-spring-boot-on-idea-intellij?answertab=votes#tab-top) ]，
  然后我才知道原来是IDE的问题，因为之前改STS配置的时候把Build project automatically给取消掉了，所以造成这样的问题
  ```java
  verify that the option check-box File->Setting –> Build, Execution, Deployment –> Compiler–>Build project automatically is selected.
  ```

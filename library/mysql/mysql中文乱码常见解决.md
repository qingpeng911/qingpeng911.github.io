# mysql中文乱码常见解决

1. ##### mysql配置文件my.ini里设置default-character-set=utf8    

2. ##### 修改数据表的字符集编码为utf8_general_ci    

3. ##### html页面加上
   ````html
   	<meta charset="UTF-8">  
    ````

4. ##### 连接数据库的url上加上?useUnicode=true&characterEncoding=utf-8,如：
   ````java
   driver=com.mysql.jdbc.Driver
   url=jdbc:mysql://localhost:3306/ssm?useUnicode=true&characterEncoding=utf-8
   username=root
   password=root
   #定义初始连接数  
   initialSize=10
   #定义最大连接数  
   maxActive=100
   #定义最大空闲  
   maxIdle=20
   #定义最小空闲  
   minIdle=1
   #定义最长等待时间  
   maxWait=1800
   ````

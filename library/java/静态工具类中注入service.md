# 静态工具类中注入service

1. ##### 有时候需要在静态工具类中调用service，但直接注入则会报错，所以需要通过其他办法。参考[ [静态工具类中使用注解注入service](http://blog.csdn.net/p793049488/article/details/37819121) ]

2. ##### Demo
   ```java
   package com.demo;

   import javax.annotation.PostConstruct;
   import javax.servlet.http.HttpSession;

   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Component;

   import com.dsy.model.AdminUser;
   import com.dsy.service.AdminUserService;

   @Component
   public class AdminUserUtil {

   	@Autowired
   	private AdminUserService adminUserService;
   	private static AdminUserUtil adminUserUtil;

   	@PostConstruct
   	private void init() {
   		adminUserUtil = this;
   		adminUserUtil.adminUserService = this.adminUserService;
   	}

   	/**
   	 * 根据session中的userid获取User
   	 */
   	public static AdminUser getUser(HttpSession session) {
   		String userId = (String) session.getAttribute("uid");
   		return adminUserUtil.adminUserService.findById(userId);
   	}

   }
   ```

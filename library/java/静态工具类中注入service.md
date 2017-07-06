# 静态工具类中注入service

1. ##### 有时候需要在静态工具类中调用service，但直接注入则会报错，所以需要通过其他办法。参考[ [静态工具类中使用注解注入service](http://blog.csdn.net/p793049488/article/details/37819121) ]

2. ##### Demo
   ```java
   package com.demo;

   import javax.annotation.PostConstruct;
   import javax.servlet.http.HttpSession;

   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Component;

   import com.demo.constant.SessionConstant;
   import com.demo.model.AdminUser;
   import com.demo.service.AdminUserService;

   @Component
   public class AdminUserUtil {

    @Autowired
   	private HttpSession session;
   	@Autowired
   	private AdminUserService adminUserService;
   	private static AdminUserUtil adminUserUtil;

   	@PostConstruct
   	private void init() {
      adminUserUtil = this;
  		adminUserUtil.adminUserService = this.adminUserService;
  		adminUserUtil.session = this.session;
   	}

    /**
  	 * 根据session获取最新的AdminUser
  	 */
  	public static AdminUser getUser(){
  		String uid = ((AdminUser) adminUserUtil.session.getAttribute(SessionConstant.ADMIN)).getId();
  		return adminUserUtil.adminUserService.findById(uid);
  	}

    /**
     * 从session中移除已登录的AdminUser
     */
    public static void removeSessionAdminUser() {
      adminUserUtil.session.removeAttribute(SessionConstant.ADMIN);
    }

   }
   ```

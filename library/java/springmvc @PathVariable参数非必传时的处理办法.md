# springmvc @PathVariable参数非必传时的处理办法

1. ##### 这样写的目的在于可以保证即使在不传参数uid的情况下也能正常访问到api
   ```java
   /**
 	 * 今日充值数据
 	 */
 	@ResponseBody
 	@RequestMapping(value={"/order/api_get_stats_today/{uid}","/order/api_get_stats_today"})
 	public Response order_api_get_stats_today(
 			@PathVariable(value="uid",required=false) String uid) {
 		Admin admin = AdminUtil.getUser();
 		if (isNoAuth(admin,uid)) {
 			return Response.get(D.ERROR, new OrderSs());
 		}
 		List<String> adminIds = getAccessAdminIdsByUid(admin,uid);
 		OrderSs orderSs = orderLOGService.findToday(adminIds);
 		return Response.get(D.SUC, orderSs);
 	}
   ```

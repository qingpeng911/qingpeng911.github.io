# SpringData mongodb group聚合查询

1. ##### 前言
   由于最近系统的数据量激增，导致后台查询速度变慢，所以决定对现有的一些查询语句进行优化。  
   分组聚合，即将查询对象按一定条件分组，然后对每一个组进行聚合分析。这样的好处在于可以显著减少所需执行查询语句，从而大大加快查询速度。  
   在以前用mysql时，有用过group by语句来实现聚合查询。同样mongodb也提供了group功能。

2. ##### 正文
    1.首先创建测试用的实体类Order
    ```java
    package com.test.model;

    import java.util.Date;
    import org.springframework.data.annotation.Id;
    import org.springframework.data.mongodb.core.index.Indexed;
    import org.springframework.data.mongodb.core.mapping.Document;
    import com.test.model.eum.EntryType;
    import com.test.model.eum.OrderStats;
    import com.test.model.eum.OrderType;
    import com.test.vo.Enum.LoginType;

    /**
     * 订单（测试类）
     */
    @Document(collection = "Order")
    public class Order {
    	@Id
    	/** 订单号 */
    	private String id;
      /** 产品ID */
      private String productId;
      /** 价格 */
      private String amount;
    	/** 订单类型 */
    	private OrderType orderType;
    	/** 订单状态 */
    	private OrderStats orderStats;
    	/** User ID */
      @Indexed
    	private String uid;
    	/** 创建时间 */
    	private Date createTime;
    	/** 支付成功时间 */
      @Indexed
    	private Date paidTime;
      //....
    }
    ```

    相关枚举类 OrderType OrderStats
    ```java
    package com.test.model.eum;
    /**
     * 订单类型
     */
    public enum OrderType {
    	ALIPAY("支付宝"),
    	Wx("微信"),
    	UNION_PAY("银联");

    	String value;
    	private OrderType(String value) {
    		this.value = value;
    	}
    	public String getValue() {
    		return value;
    	}
    	public void setValue(String value) {
    		this.value = value;
    	}
    }
    ```

    ```java
    package com.test.model.eum;
    /**
    * 订单状态
    */
    public enum OrderStats {
    	NOT_PAY("未支付"),
    	PAY_VERIFIED("支付成功");

    	String value;

    	OrderStats(String v) {
    		this.value = v;
    	}
    	public String getValue() {
    		return value;
    	}
    	public void setValue(String value) {
    		this.value = value;
    	}
    }
    ```

    ```java
    package com.dsy.vo.aggregation;

    import com.dsy.model.eum.OrderType;
    /**
     * 结果集映射类
     */
    public class OrderAgg {

    	/** 支付类型 */
    	private OrderType orderType;
    	/** 汇总收入 */
    	private int tAmount;
    	/** 支付成功次数 */
    	private int paySucCount;
    	/** 平均价格 */
    	private int avg;
    	/** 成功人数 */
    	private int sucPerson;

    }
    ```

    2.插入测试数据
       db.getCollection('Order')
       .insert({"productId":"001,"amount":2000,"orderType":"ALIPAY","orderStats":"PAY_VERIFIED","uid":"111111","createTime":ISODate("2016-12-27T22:57:52.937+08:00"),"paidTime":ISODate("2016-12-27T23:57:52.937+08:00")})

       db.getCollection('Order')
       .insert({"productId":"001","amount":2000,"orderType":"ALIPAY","orderStats":"PAY_VERIFIED","uid":"111222","createTime":ISODate("2016-12-28T22:57:52.937+08:00"),"paidTime":ISODate("2016-12-28T23:57:52.937+08:00")})

       db.getCollection('Order')
       .insert({"productId":"001","amount":2000,"orderType":"Wx","orderStats":"NOT_PAY","uid":"111133","createTime":ISODate("2016-12-27T21:57:52.937+08:00"),"paidTime":ISODate("2016-12-28T23:57:52.937+08:00")})

       db.getCollection('Order').insert({"productId":"001","amount":2000,"orderType":"UNION_PAY","orderStats":"NOT_PAY","uid":"111144","createTime":ISODate("2016-12-27T22:57:52.937+08:00")})

       db.getCollection('Order')
       .insert({"productId":"001","amount":2000,"orderType":"UNION_PAY","orderStats":"PAY_VERIFIED","uid":"111144","createTime":ISODate("2016-12-27T22:57:52.937+08:00"),"paidTime":ISODate("2016-12-28T23:57:52.937+08:00")})

    3.查询
    ```java
    package com.dsy.repositories.custom.impl;

    import static org.springframework.data.mongodb.core.aggregation.Aggregation.*;

    import java.util.ArrayList;
    import java.util.Arrays;
    import java.util.Date;
    import java.util.List;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.data.domain.Sort;
    import org.springframework.data.domain.Sort.Direction;
    import org.springframework.data.mongodb.core.MongoTemplate;
    import org.springframework.data.mongodb.core.aggregation.Aggregation;
    import org.springframework.data.mongodb.core.aggregation.AggregationResults;
    import org.springframework.data.mongodb.core.query.Criteria;

    import com.dsy.model.Order;
    import com.dsy.model.eum.OrderStats;
    import com.dsy.model.eum.OrderType;
    import com.dsy.repositories.custom.OrderRepositoryCustom;
    import com.dsy.utils.DateUtil;
    import com.dsy.utils.StringUtil;
    import com.dsy.vo.aggregation.OrderAgg;

    public class OrderRepositoryImpl implements OrderRepositoryCustom {

    	@Autowired
    	private MongoTemplate mongoTemplate;

      public List<OrderAgg> aggregatOrder(Date from, Date to) {
      Aggregation agg = newAggregation(match(Criteria.where("paidTime").gte(from).lt(to)), //查询指定日期之前成功的订单集合
          group("orderType") //group:按orderType分组
            .count().as("paySucCount") //统计各orderType下的订单数量，用paySucCount字段保存
            .sum("amount").as("tAmount"), //汇总各orderType下的总金额，用tAmount字段保存
          project("paySucCount", "tAmount") //project:选择要展示的结果字段"paySucCount","tAmount"
              .and("orderType").previousOperation() //声明在聚合结果集中保留orderType列
              .avg("amount").as("avg"), //订单商品的平均价格
              //.andExpression("tAmount / paySucCount").as("tAmount"));
          sort(Direction.DESC, "tAmount")  //排序
      AggregationResults<OrderAgg> groupRes = mongoTemplate.aggregate(agg, Order.class, OrderAgg.class);
      List<OrderAgg> rs = groupRes.getMappedResults();

      // 聚合按用户id去重
      // Aggregation agg = newAggregation(
      //     match(criteria),
      //     group("orderType", "uid"),
      //     group("orderType")
      //       .count().as("sucPerson"),
      //     project("sucPerson")
      //       .and("orderType").previousOperation());
      // AggregationResults<OrderAgg> groupRes = mongoTemplate.aggregate(agg, Order.class, OrderAgg.class);
      // List<OrderAgg> rs = groupRes.getMappedResults();
      return rs;
    }
  }
  ```
3. ##### 参考文档
   1.[ [mongodb官方文档](https://docs.mongodb.com/v3.2/introduction/) ]  
   2.[ [Spirng data mongodb官方文档](http://docs.spring.io/spring-data/data-mongodb/docs/current/reference/html/) ]  
   2.[ [Spring data mongodb - aggregation framework integration](http://stackoverflow.com/questions/15624473/spring-data-mongodb-aggregation-framework-integration) ]  
   3.[ [Spring data equivalent of below aggregation operation in mongodb](http://stackoverflow.com/questions/24865893/spring-data-equivalent-of-below-aggregation-operation-in-mongodb) ]

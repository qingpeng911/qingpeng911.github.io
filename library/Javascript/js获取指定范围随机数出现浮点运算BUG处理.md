# js获取指定范围随机数出现浮点运算BUG处理

1. ##### js获取随机数时经常会出现类似的问题
  ```javascript
  $(function(){
    var num = parseInt(Math.random() * 100) * 0.01;
    console.log(num);    	   
  })  
  //经常会出现类似 0.7000000000000001 这样的结果
  ```

1. ##### 解决办法
   ```javascript
   <!--获取一个在[min,max]区间的随机数，保留decimal位的小数-->
   function getRandomNumEq(min,max,decimal){
   	  var Range = max - min;
   	  //0-1之间的随机数
       var Rand = Math.random();
       var result = Number(min) + Number((Rand * Range).toFixed(decimal)) + "";
       return result.substr(0,result.indexOf('.')+decimal+1);
   }
   ```

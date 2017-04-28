# js+freemark实现select下拉框回显

1. ##### Html代码
  ```html
  <html lang="en">
  <head>
   <meta charset="UTF-8">
   <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
      <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0">
   <title>select下拉框回显</title>
  </head>
  <body>
    <div class="form-group">
      <label class="col-sm-3 control-label no-padding-right red" for="form-field-1"> 选择你所在的城市 </label>

      <div class="col-sm-9">
        <select name="city" required="city" data-default="${city!}">
          <option value="">--请选择--</option>
          <option value="hz">杭州</option>
          <option value="cd">成都</option>
          <option value="bj">北京</option>
        </select>
      </div>
    </div>
  </body>
  <!-- 百度静态资源公共库 -->
  <script type="text/javascript" src="http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
  <script type="text/javascript" src="js/selectUtil.js"></script>
  </body>
  </html>
  ```

1. ##### selectUtil.js 依赖jquery
   ```javascript
   $(function(){

     $("select option").each(function(){
         //+""是为了将int类型转为String
         if($(this).val() == $(this).parent().data("default")+""){
           $(this).attr("selected",true);
         }
     });

   })
   ```

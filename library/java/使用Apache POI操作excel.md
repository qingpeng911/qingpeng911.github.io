# 使用Apache POI操作excel

1. ##### 前言
   在实际业务中经常需要会涉及到数据的导入导出，尤其是财务相关的系统，而数据的导出格式一般都是excel表格。java中主流的操作excel的开源工具之一就是Apache POI[ [官方下载](https://poi.apache.org/download.html) ]。
2. ##### 加入maven依赖，在项目的pom.xml文件中加入下面的代码。（这里使用的是3.14版本）
   ```java
   <dependency>
       <groupId>org.apache.poi</groupId>
       <artifactId>poi</artifactId>
       <version>3.14</version>
   </dependency>
   <dependency>
       <groupId>org.apache.poi</groupId>
       <artifactId>poi-ooxml</artifactId>
       <version>3.14</version>
   </dependency>
   ```
2. ##### 基本操作
   参考资料：  
   1.[ [Apache POI使用详解](http://www.360doc.com/content/15/1112/17/110467_512619930.shtml) ]  
   2.[ [Apache POI使用手册](http://www.journaldev.com/2562/apache-poi-tutorial) ]  
   3.[ [Apache POI官方快速入门](http://poi.apache.org/spreadsheet/quick-guide.html) ]  

3. ##### 整理的工具类 ExcelUtil.java
   ```java
    package com.test.utils;

    import java.lang.reflect.Field;
    import java.util.List;
    import java.util.Map;

    import org.apache.commons.logging.Log;
    import org.apache.commons.logging.LogFactory;
    import org.apache.poi.ss.usermodel.Cell;
    import org.apache.poi.ss.usermodel.CellStyle;
    import org.apache.poi.ss.usermodel.Font;
    import org.apache.poi.ss.usermodel.Row;
    import org.apache.poi.ss.usermodel.Sheet;
    import org.apache.poi.ss.usermodel.Workbook;
    import org.apache.poi.xssf.usermodel.XSSFRichTextString;

    import com.test.constant.D.Desc;
    import com.test.vo.excel.MyExcel;

    /**
     * 操作Excel的工具类
     */
    public class ExcelUtil {
    	private static final Log log = LogFactory.getLog(ExcelUtil.class);

    	/**
    	 * 创建多个Sheet的Excel
    	 */
    	public static void createExcelWithMutiSheet(Workbook workbook,Map<String, List<MyExcel>> map){
    		for (Map.Entry<String, List<MyExcel>> entry : map.entrySet()) {
    			createExcelWithSingleSheet(workbook, entry.getKey(), entry.getValue());
    		}
    	}

    	/**
    	 * 创建单个Sheet的Excel
    	 */
    	public static void createExcelWithSingleSheet(Workbook workbook,String sheetName,List<MyExcel> list) {
    		//BUILD DOC
    		Sheet sheet = workbook.createSheet(sheetName);
    		sheet.setDefaultColumnWidth(28);
    		int currentRow = 0;
    		int currentColumn = 0;
    		//CREATE STYLE FOR HEADER
    		CellStyle headerStyle = workbook.createCellStyle();
    	    Font font = workbook.createFont();
    	    font.setBold(true);
    	    font.setBoldweight(Font.BOLDWEIGHT_BOLD);
    	    headerStyle.setFont(font);
    	    //POPULATE HEADER COLUMNS
    	    Row headerRow = sheet.createRow(currentRow);
    	    if(list.size() == 0) {
    	    	return ;
    	    }
          //通过反射获取Object实体对象的所有字段属性
    	    for(Field field : list.get(0).getClass().getDeclaredFields()){
            //获取字段上@Desc的注解内容
    	    	Desc desc = field.getAnnotation(Desc.class);
    	    	if(desc == null) {
    	    		continue;
    	    	}
            //根据注解内容作为单元格标题
    	    	XSSFRichTextString text  = new XSSFRichTextString(desc.value());
    	    	Cell cell = headerRow.createCell(currentColumn);
    	    	cell.setCellStyle(headerStyle);
    	    	cell.setCellValue(text);
    	    	currentColumn++;
    	    }
    	    //POPULATE VALUE ROWS/COLUMNS
    	    if(list == null || list.size() == 0) {
    			return ;
    		}
    	    currentRow++;//exclude header
    	    try {
    	    	for(Object obj : list){
    		    	currentColumn = 0;
    		    	Row row = sheet.createRow(currentRow++);
    		    	Field[] fields = obj.getClass().getDeclaredFields();
    		    	for(Field field : fields) {
    		    		Desc desc = field.getAnnotation(Desc.class);
    			    	if(desc == null) {
    			    		continue;
    			    	}
    		    		Cell cell = row.createCell(currentColumn++);
    		    		field.setAccessible(true);
    		    		if(field.get(obj) == null) {
    		    			continue;
    		    		}
                //根据字段的类型分别赋值
    		    		if(field.getType() == int.class
    		    				|| field.getType() == Integer.class
    		    				|| field.getType() == long.class
    		    				|| field.getType() == Long.class
    		    				|| field.getType() == double.class
    		    				|| field.getType() == Double.class
    		    				|| field.getType() == float.class
    		    				|| field.getType() == Float.class) {
    		    			//数字类型
    		    			cell.setCellType(Cell.CELL_TYPE_NUMERIC);
    		    			cell.setCellValue(Double.valueOf(field.get(obj).toString()));
    		    		}else {
    		    			//非数字类型
    		    			XSSFRichTextString text  = new XSSFRichTextString(field.get(obj).toString());
    		    			cell.setCellValue(text);
    		    		}
    		    	}
    		    }
    		} catch (Exception e) {
    			log.error(e);
    		}
    	}
    }
   ```
4. ##### 待导出的excel对应的实体类对象 MyExcel.java
   ```java
    package com.test.vo.excel;

    import com.test.constant.D.Desc;
    import com.test.model.SrcData;
    import com.test.utils.DateUtil;

    public class MyExcel implements Comparable<Object>{

    	/** 日期 */
    	@Desc("日期")
    	private String date;
    	/** 注册数 */
    	@Desc("注册数")
    	private int reg = 0;
    	/** 总收入 */
    	@Desc("总收入 ")
    	private int tIncome = 0;

      //getter/setter...

    	public MyExcel(SrcData data) {
    		super();
    		this.date = DateUtil.getDateStr("yyyy-MM-dd", data.getDate());
    		this.reg = data.getnReg();
    		this.tIncome = data.gettIncome() / 100;
    	}

    	public MyExcel() {
    		super();
    	}

    	@Override
    	public int compareTo(Object o) {
    		return this.date.compareTo(((MyExcel)o).getDate());
    	}

    }

   ```

5. ##### 测试
   ```java
    package com.test;

    import org.bson.types.ObjectId;
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.test.context.junit4.SpringRunner;
    import org.apache.commons.logging.LogFactory;
    import org.apache.poi.xssf.usermodel.XSSFWorkbook;

    import com.dsy.utils.ExcelUtil;
    import com.test.vo.excel;

    @RunWith(SpringRunner.class)
    @SpringBootTest
    public class Test {

    	@Test
    	public void excelTest() {
        //待导出的数据map,这里的key是导出excel表格多个sheet的名称
        Map<String, List<MyExcel>> map = new HashMap<>();

        XSSFWorkbook workbook = new XSSFWorkbook();
        if (workbook != null) {
            ExcelUtil.createExcelWithMutiSheet(workbook, map);			
        }
        FileOutputStream out;
        // 20170117-20170122.xlsx
        String fileName = DateUtil.getDateStr("yyyyMMdd", from)+"-"+DateUtil.getDateStr("yyyyMMdd", to);
        if (!StringUtil.isEmpty(channel)) {
            fileName = fileName+"("+channel+")";
        }
        try {
          //导出excel文件的路径:项目根目录下的public文件夹下
          //PS：在spring boot项目中根目录下的public文件夹里的内容是可以直接通过连接访问的，这里的excel可以直接通过url下载。但需要做好访问的身份验证，免得重要数据泄漏
          out = new FileOutputStream("public/"+fileName+".xlsx");
          workbook.write(out);
          out.close();
          workbook.close();
          //返回文件的url连接
          return "redirect:/"+fileName+".xlsx";
        } catch (FileNotFoundException e) {
          e.printStackTrace();
        } catch (IOException e) {
          e.printStackTrace();
        }
     	}

    }

6. ##### 测试结果
   ![amWiki logo](https://qingpeng911.github.io/amWiki/images/article/excel_test.png)  

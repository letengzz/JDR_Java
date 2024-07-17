# EasyExcel

Java领域解析，生成Excel比较有名的框架有Apache poi、jxl等，但他们都存在一个严重的问题就是非常的耗内存，如果你的系统并发量不大的话可能还行，但是一旦并发上来后一定会OOM或者JVM频繁的full gc。

EasyExcel是一个基于Java的**简单**、**省内存**的读写Excel的阿里**开源项目**。在尽可能节约内存的情况下支持**读写百兆**的Excel。

EasyExcel**采用一行一行的解析模式**，并将一行的解析结果以观察者的模式通知处理 (AnalysisEventListener)。

在项目中，涉及到Excel文件、CVS文件大多数的读写操作，均可使用EasyExcel。

**官方文档**：https://easyexcel.opensource.alibaba.com

## 创建项目

创建Maven项目，并导入依赖：

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>easyexcel</artifactId>
    <version>3.3.3</version>
</dependency>
<dependency>
     <groupId>org.projectlombok</groupId>
     <artifactId>lombok</artifactId>
     <version>1.18.22</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
	<scope>compile</scope>
</dependency>
```

## 简单写操作

**官方示例**：https://github.com/alibaba/easyexcel/blob/master/easyexcel-test/src/test/java/com/alibaba/easyexcel/test/demo/write/WriteTest.java

**官方教程**：https://easyexcel.opensource.alibaba.com/docs/current/quickstart/write

**说明**：此方式在数据量不大的情况下可以使用 (5000以内，具体也要看实际情况)，数据量大参照 [重复多次写入](#写入海量数据)

**操作步骤**：

1. 编写与Excel文件相对应的模型类并加入注解：

   ```java
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   public class Employee {
       //成员变量
       //ExcelProperty 表示标题
       @ExcelProperty("员工编号")
       private int id;
       @ExcelProperty("员工姓名")
       private String name;
       @ExcelProperty("入职日期")
       private Date date;
       @ExcelProperty("员工工资")
       private double salary;
       // 忽略这个字段
       @ExcelIgnore
       private String ignore;
   }
   ```

2. 编写获取测试数据的方法：

   ```java
   private List<Employee> data(int count) {
       List<Employee> list = ListUtils.newArrayList();
       for (int i = 0; i < count; i++) {
           Employee data = new Employee(i,"测试数据"+i,new Date(),6.6*i,"忽略字段"+i);
           list.add(data);
       }
       return list;
   }
   ```

3. 调用官方API完成写操作：

   `EasyExcel.write(fileName,DemoData.class).sheet("测试").doWrite(data(10000));`

   ```java
   @Test
   public void write() {
       // 写入文件名
       String fileName = "D:/simpleWrite.xlsx";
       // 调用write方法写入 按照指定的某个class去写
       // excelType(): 指定写入的类型 如xlsx、xls、cvs
       // sheet(): 指定sheet的名字
       // doWrite(): 写入的方法
       EasyExcel.write(fileName, Employee.class).excelType(ExcelTypeEnum.XLSX).sheet("模板").doWrite(data(10));
   }
   ```
   

## 简单读操作

**官方示例**：https://github.com/alibaba/easyexcel/blob/master/easyexcel-test/src/test/java/com/alibaba/easyexcel/test/demo/read/ReadTest.java

**官方教程**：https://easyexcel.opensource.alibaba.com/docs/current/quickstart/read

**操作步骤**：

1. 编写与Excel文件相对应的模型类并加入注解：

   ```java
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   public class Employee {
       //成员变量
       private int id;
       private String name;
       private Date date;
       private double salary;
   }
   ```

2. 使用PageReadListener监听器，调用官方API完成写操作：

   `EasyExcel.read(fileName,DemoData.class,new PageReadListener<DemoData>(list -> System.out.println(list))).sheet().doRead();`

   ```java
   @Slf4j
   public class SimpleRead {
       @Test
       public void read(){
           // 读入文件名
           String fileName = "D:/simpleWrite.xlsx";
           // 这里默认每次会读取100条数据 然后返回过来 直接调用使用数据就行
           // 指定读用哪个class去读，然后读取第一个sheet 文件流会自动关闭
           // 具体需要返回多少行可以在`PageReadListener`的构造函数设置
           EasyExcel.read(fileName, Employee.class, new PageReadListener<Employee>(dataList -> {
               for (Employee demoData : dataList) {
                   log.info("读取到一条数据{}", demoData);
               }
           })).sheet().doRead();
       }
   }
   ```

## 写入海量数据

**官方示例**：https://github.com/alibaba/easyexcel/blob/master/easyexcel-test/src/test/java/com/alibaba/easyexcel/test/demo/write/WriteTest.java

**官方教程**：https://easyexcel.opensource.alibaba.com/docs/current/quickstart/write#%E4%BB%A3%E7%A0%81

**操作步骤**：

1. 编写与Excel文件相对应的模型类并加入注解：

   ```java
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   public class Employee {
       //成员变量
       //ExcelProperty 表示标题
       @ExcelProperty("员工编号")
       private int id;
   
       @ExcelProperty("员工姓名")
       private String name;
   
       @ExcelProperty("入职日期")
       private Date date;
   
       @ExcelProperty("员工工资")
       private double salary;
   
       // 忽略这个字段
       @ExcelIgnore
       private String ignore;
   }
   ```

2. 调用官方API完成写功能：

   `try (ExcelWriter excelWriter = EasyExcel.write(fileName, Employee.class).build()) {
       WriteSheet writeSheet = EasyExcel.writerSheet("模板").build();
       excelWriter.write(data(10000), writeSheet);
   }`

   ```java
   @Test
   public void write() {
       // 写入文件名
       String fileName = "D:/manyWrite.xlsx";
       // 指定文件
       try (ExcelWriter excelWriter = EasyExcel.write(fileName, Employee.class).build()) {
           long t1 = System.currentTimeMillis();
           //调用写入
           for (int i = 0; i < 100; i++) {
               // 写入sheet 必须指定sheetNo 并且sheetName必须不同
               WriteSheet writeSheet = EasyExcel.writerSheet(i,"模板"+i).build();
               // 写入数据
               excelWriter.write(data(10000), writeSheet);
           }
           long t2 = System.currentTimeMillis();
           System.out.println(t2 - t1);
       }
   }
   ```

## 使用模板写入

可以使用阿里提供的模板写入，模板设置的样式新添加的数据会自动包含样式。

**官方示例**：https://github.com/alibaba/easyexcel/blob/master/easyexcel-test/src/test/java/com/alibaba/easyexcel/test/demo/fill/FillTest.java

**官方教程**：https://easyexcel.opensource.alibaba.com/docs/current/quickstart/fill

**操作步骤**：

1. 编写与Excel文件相对应的模型类并加入注解：

   ```java
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   public class Employee {
       //成员变量
       //ExcelProperty 表示标题
       @ExcelProperty("员工编号")
       private int id;
   
       @ExcelProperty("员工姓名")
       private String name;
   
       @ExcelProperty("入职日期")
       private Date date;
   
       @ExcelProperty("员工工资")
       private double salary;
   
       // 忽略这个字段
       @ExcelIgnore
       private String ignore;
   }
   ```

2. 编写模板文件：

   ![image-20240225163745413](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402251637118.png)

3. 调用官方API完成写功能：

   `EasyExcel.write(fileName).withTemplate(templateFileName).sheet().doFill(fillData)`

   ```java
   @Test
   public void write() {
       // 写入文件名
       String fileName = "D:/templateWrite.xlsx";
       // 模板文件
       String templateFileName = "D:/template.xlsx";
       // 分多次 填充 会使用文件缓存（省内存）
       try (ExcelWriter excelWriter = EasyExcel.write(fileName).withTemplate(templateFileName).build()) {
           long t1 = System.currentTimeMillis();
           WriteSheet writeSheet = EasyExcel.writerSheet().build();
           for (int i = 0; i < 100; i++) {
               excelWriter.fill(data(1000), writeSheet);
           }
           long t2 = System.currentTimeMillis();
           System.out.println(t2 - t1);
       }
   }
   ```

## 读取海量数据

使用自定义监听器读海量(百万级别)数据并监控内存消耗

**操作步骤**：

1. 编写与Excel文件相对应的模型类并加入注解：

   ```java
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   public class Employee {
       //成员变量
       private int id;
       private String name;
       private Date date;
       private double salary;
   }
   ```

2. 编写Dao层，模拟数据库操作：

   ```
   public class EmployeeDao {
       public void save(List<Employee> list){
           System.out.println(list.size() + "模拟操作数据库....");
       }
   }
   ```

3. 编写自定义监听器：

   ```java
   public class EmployeeListener implements ReadListener<Employee> {
       private int count = 100;
       private ArrayList<Employee> list  = new ArrayList<>(count);
   
       private EmployeeDao dao;
   
       public EmployeeListener(EmployeeDao dao) {
           this.dao = dao;
       }
       /**
        *  每解析一行数据，都会调用该方法
        * @param employee
        * @param analysisContext
        */
       @Override
       public void invoke(Employee employee, AnalysisContext analysisContext) {
           //将读取到的一行数据添加到集合
           list.add(employee);
           //判断是不是到达换存量
           if(list.size() >= count){
               //操作数据库
               dao.save(list);
               list = new ArrayList<>(count);
           }
       }
   
       /**
        *  所有数据解析完成了 都会来调用该方法
        * @param analysisContext
        */
       @Override
       public void doAfterAllAnalysed(AnalysisContext analysisContext) {
           //余下的数据量
           if(!list.isEmpty()){
               //操作数据库
               dao.save(list);
               list = new ArrayList<>(count);
           }
       }
   }
   ```

4. 调用官方API完成写功能：

   `EasyExcel.read(fileName,IndexOrNameData.class,new IndexOrNameDataListener()).sheet().doRead();`

   ```java
   @Test
   public void read(){
       // 读入文件名
       String fileName = "D:/templateWrite.xlsx";
       // 这里默认每次会读取100条数据 然后返回过来 直接调用使用数据就行
       // 指定读用哪个class去读，然后读取第一个sheet 文件流会自动关闭
       // 具体需要返回多少行可以在`PageReadListener`的构造函数设置
       ExcelReader reader = EasyExcel.read(fileName, Employee.class, new EmployeeListener(new EmployeeDao())).build();
       ReadSheet sheet = EasyExcel.readSheet().build();
       reader.read(sheet);
   }
   ```

## Web 文件导入操作

**官方示例**：https://github.com/alibaba/easyexcel/blob/master/easyexcel-test/src/test/java/com/alibaba/easyexcel/test/demo/web/WebTest.java

**官方教程**：https://easyexcel.opensource.alibaba.com/docs/current/quickstart/read#web%E4%B8%AD%E7%9A%84%E8%AF%BB

## Web 文件导出操作

**官方示例**：https://github.com/alibaba/easyexcel/blob/master/easyexcel-test/src/test/java/com/alibaba/easyexcel/test/demo/web/WebTest.java

**官方教程**：https://easyexcel.opensource.alibaba.com/docs/current/quickstart/write#web%E4%B8%AD%E7%9A%84%E5%86%99

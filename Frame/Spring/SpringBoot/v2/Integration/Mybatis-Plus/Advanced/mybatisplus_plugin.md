# Mybatis-Plus插件

## 分页

### 自带分页插件

MyBatis Plus自带分页插件，只要简单的配置即可实现分页功能

**配置插件功能**：

> config/PluginConfig.java

```java
@Configuration
//扫描mapper接口所在的包
@MapperScan("com.hjc.demo.mapper")
public class PluginConfig {
    //配置mybatisplus的插件 需要在插件中添加一个内部插件(分页插件叫做paginationInnerceptor)
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(){
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        //添加分页拦截器到MybatisPlusInterceptor中
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```

**测试**：

```java
@SpringBootTest
class UserMapperTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    void testSelectList() {
        Page<User> page = new Page<>(1,3);
        //selectList()根据MP内置的条件构造器查询一个list集合，null表示没有条件，即查询所有
        Page<User> userPage = userMapper.selectPage(page, null);
        //当前页的数据
        userPage.getRecords().forEach(System.out::println);
        //获取当前页的页码
        System.out.println(userPage.getCurrent());
        //获取每页的条数
        System.out.println(userPage.getSize());
        //获取总页数
        System.out.println(userPage.getTotal());
        //是否存在上一页
        System.out.println(userPage.hasPrevious());
        //是否存在下一页
        System.out.println(userPage.hasNext());
        //当前分页总页数
        System.out.println(userPage.getPages());
        //添加新的排序条件
        OrderItem orderItem = new OrderItem();
        orderItem.setAsc(true);
        userPage.addOrder(orderItem);
    }
}
```

**xml自定义分页**：

UserMapper中定义接口方法：

```java
@Repository
public interface UserMapper extends BaseMapper<User> {
    /**
     * 根据年龄查询用户列表，分页显示
     * @param page 分页对象,xml中可以从里面进行取值,传递参数 Page 即自动分页,必须放在第一位
     * @param age 年龄
     * @return
     */
    IPage<User> selectPageVo(@Param("page") Page<User> page, @Param("age") Integer age);

}

```

UserMapper.xml中编写SQL：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.hjc.demo.mapper.UserMapper">
    <!--SQL片段，记录基础字段-->
    <sql id="BaseColumns">id,name,age,email</sql>
    <select id="selectPageVo" resultType="com.hjc.demo.pojo.User">
        SELECT <include refid="BaseColumns"/> FROM t_user WHERE age > #{age}
    </select>
</mapper>
```

测试：

```java
@SpringBootTest
class UserMapperTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    void testSelectList2() {
        Page<User> page = new Page<>(1,3);
        //selectList()根据MP内置的条件构造器查询一个list集合，null表示没有条件，即查询所有
        userMapper.selectPageVo(page,10);
        //当前页的数据
        page.getRecords().forEach(System.out::println);
        //获取当前页的页码
        System.out.println(page.getCurrent());
        //获取每页的条数
        System.out.println(page.getSize());
        //获取总页数
        System.out.println(page.getTotal());
        //是否存在上一页
        System.out.println(page.hasPrevious());
        //是否存在下一页
        System.out.println(page.hasNext());
        //当前分页总页数
        System.out.println(page.getPages());
        //添加新的排序条件
        OrderItem orderItem = new OrderItem();
        orderItem.setAsc(true);
        page.addOrder(orderItem);
    }
}
```

### PageHelper插件

使⽤PageHelper的分⻚拦截器与自带的分页插件，⼆者冲突，推荐后者PageHelper进行分页

**导入依赖**：

> pom.xml

```xml
<dependency>
	<groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.2.0</version>
</dependency>
```

**配置插件功能**：

```java
@Configuration
//扫描mapper接口所在的包
@MapperScan("com.hjc.demo.mapper")
public class PluginConfig {

    //配置pageHelper
    @Bean
    public PageInterceptor pageInterceptor(){
        return new PageInterceptor();
    }
}
```

**测试**：

```java
@SpringBootTest
class UserMapperTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    void testPageHelper() {
        PageInfo<User> page = PageHelper.startPage(1,2).doSelectPageInfo(()->{
            userMapper.selectList(null);
        });
        page.getList().forEach(System.out :: println);
        System.out.println("总行数=" + page.getTotal());
        System.out.println("当前页=" + page.getPageNum());
        System.out.println("每页行数=" + page.getPageSize());
        System.out.println("总页数=" + page.getPages());
        System.out.println("起始行数=" + page.getStartRow());
        System.out.println("是第一页=" + page.isIsFirstPage());
        System.out.println("是最后页=" + page.isIsLastPage());
        System.out.println("还有下一页=" + page.isHasNextPage());
        System.out.println("还有上一页=" + page.isHasPreviousPage());
        System.out.println("页码列表=" + Arrays.toString(page.getNavigatepageNums()));
    }
}
```

## 乐观锁

**场景**：

​	一件商品，成本价是80元，售价是100元。老板先是通知小李，说你去把商品价格增加50元。小 李正在玩游戏，耽搁了一个小时。正好一个小时后，老板觉得商品价格增加到150元，价格太 高，可能会影响销量。又通知小王，你把商品价格降低30元。 此时，小李和小王同时操作商品后台系统。小李操作的时候，系统先取出商品价格100元；小王 也在操作，取出的商品价格也是100元。小李将价格加了50元，并将100+50=150元存入了数据 库；小王将商品减了30元，并将100-30=70元存入了数据库。是的，如果没有锁，小李的操作就 完全被小王的覆盖了。 现在商品价格是70元，比成本价低10元。几分钟后，这个商品很快出售了1千多件商品，老板亏1 万多。

**乐观锁与悲观锁**：

​	如果是乐观锁，小王保存价格前，会检查下价格是否被人修改过了。如果被修改过 了，则重新取出的被修改后的价格，150元，这样他会将120元存入数据库。 

​	如果是悲观锁，小李取出数据后，小王只能等小李操作完之后，才能对价格进行操作，也会保证最终的价格是120元。

**乐观锁实现流程**：

数据库中添加version字段：

- 取出记录时，获取当前version：

  ```sql
  SELECT id,`name`,price,`version` FROM product WHERE id=1
  ```

- 更新时，version+1，如果where语句中的version版本不对，则更新失败

  ```sql
  UPDATE product SET price=price+50, `version`=`version` + 1 WHERE id=1 AND
  `version`=1
  ```

### 修改冲突

**数据库中添加商品表**：

```sql
CREATE TABLE t_product
(
	id BIGINT(20) NOT NULL COMMENT '主键ID',
	NAME VARCHAR(30) NULL DEFAULT NULL COMMENT '商品名称',
	price INT(11) DEFAULT 0 COMMENT '价格',
	VERSION INT(11) DEFAULT 0 COMMENT '乐观锁版本号',
	PRIMARY KEY (id)
);
```

**添加数据**：

```sql
INSERT INTO t_product (id, NAME, price) VALUES (1, '外星人笔记本', 100);
```

**添加实体类**：

> Product.java

```java
@Data
public class Product {
    private Long id;
    private String name;
    private Integer price;
    private Integer version;
}
```

**添加mapper**：

```java
public interface ProductMapper extends BaseMapper<Product> {
}
```

**测试**：

```java
@SpringBootTest
class ProductMapperTest {
    @Autowired
    private ProductMapper productMapper;

    @Test
    void test1() {
        //1、小李
        Product p1 = productMapper.selectById(1L);
        System.out.println("小李取出的价格：" + p1.getPrice());
        //2、小王
        Product p2 = productMapper.selectById(1L);
        System.out.println("小王取出的价格：" + p2.getPrice());
        //3、小李将价格加了50元，存入了数据库
        p1.setPrice(p1.getPrice() + 50);
        int result1 = productMapper.updateById(p1);
        System.out.println("小李修改结果：" + result1);
        //4、小王将商品减了30元，存入了数据库
        p2.setPrice(p2.getPrice() - 30);
        int result2 = productMapper.updateById(p2);
        System.out.println("小王修改结果：" + result2);
        //最后的结果
        Product p3 = productMapper.selectById(1L);
        //价格覆盖，最后的结果：70
        System.out.println("最后的结果：" + p3.getPrice());
    }
}
```

### 使用乐观锁

![image-20230226104932989](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202302261314410.png)

**修改实体类**：

```java
@Data
public class Product {
    private Long id;
    private String name;
    private Integer price;
    @Version
    private Integer version;
}
```

**添加乐观锁插件**：

```java
@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor(){
MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
//添加分页插件
interceptor.addInnerInterceptor(new
PaginationInnerInterceptor(DbType.MYSQL));
//添加乐观锁插件
interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
return interceptor;
}
```

**测试**：

```java
@SpringBootTest
class ProductMapperTest {
    @Autowired
    private ProductMapper productMapper;

    @Test
    void test2() {
        //小李取数据
        Product p1 = productMapper.selectById(1L);
        //小王取数据
        Product p2 = productMapper.selectById(1L);
        //小李修改 + 50
        p1.setPrice(p1.getPrice() + 50);
        int result1 = productMapper.updateById(p1);
        System.out.println("小李修改的结果：" + result1);
        //小王修改 - 30
        p2.setPrice(p2.getPrice() - 30);
        int result2 = productMapper.updateById(p2);
        System.out.println("小王修改的结果：" + result2);

        if (result2 == 0) {
            //失败重试，重新获取version并更新
            p2 = productMapper.selectById(1L);
            p2.setPrice(p2.getPrice() - 30);
            result2 = productMapper.updateById(p2);
        }
        System.out.println("小王修改重试的结果：" + result2);
        //老板看价格
        Product p3 = productMapper.selectById(1L);
        System.out.println("老板看价格：" + p3.getPrice());
    }
}
```


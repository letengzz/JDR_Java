# MyBatis-Plus基本操作

## 基本功能

MyBatis-Plus中的基本CRUD在内置的BaseMapper中都已得到了实现：

```java
/**
 * Mapper 继承该接口后，无需编写 mapper.xml 文件，即可获得CRUD功能
 * <p>这个 Mapper 支持 id 泛型</p>
 */
public interface BaseMapper<T> extends Mapper<T> {

    /**
     * 插入一条记录
     *
     * @param entity 实体对象
     */
    int insert(T entity);

    /**
     * 根据 ID 删除
     *
     * @param id 主键ID
     */
    int deleteById(Serializable id);

    /**
     * 根据实体(ID)删除
     *
     * @param entity 实体对象
     * @since 3.4.4
     */
    int deleteById(T entity);

    /**
     * 根据 columnMap 条件，删除记录
     *
     * @param columnMap 表字段 map 对象
     */
    int deleteByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);

    /**
     * 根据 entity 条件，删除记录
     *
     * @param queryWrapper 实体对象封装操作类（可以为 null,里面的 entity 用于生成 where 语句）
     */
    int delete(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

    /**
     * 删除（根据ID或实体 批量删除）
     *
     * @param idList 主键ID列表或实体列表(不能为 null 以及 empty)
     */
    int deleteBatchIds(@Param(Constants.COLL) Collection<?> idList);

    /**
     * 根据 ID 修改
     *
     * @param entity 实体对象
     */
    int updateById(@Param(Constants.ENTITY) T entity);

    /**
     * 根据 whereEntity 条件，更新记录
     *
     * @param entity        实体对象 (set 条件值,可以为 null)
     * @param updateWrapper 实体对象封装操作类（可以为 null,里面的 entity 用于生成 where 语句）
     */
    int update(@Param(Constants.ENTITY) T entity, @Param(Constants.WRAPPER) Wrapper<T> updateWrapper);

    /**
     * 根据 ID 查询
     *
     * @param id 主键ID
     */
    T selectById(Serializable id);

    /**
     * 查询（根据ID 批量查询）
     *
     * @param idList 主键ID列表(不能为 null 以及 empty)
     */
    List<T> selectBatchIds(@Param(Constants.COLL) Collection<? extends Serializable> idList);

    /**
     * 查询（根据 columnMap 条件）
     *
     * @param columnMap 表字段 map 对象
     */
    List<T> selectByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);

    /**
     * 根据 entity 条件，查询一条记录
     * <p>查询一条记录，例如 qw.last("limit 1") 限制取一条记录, 注意：多条数据会报异常</p>
     *
     * @param queryWrapper 实体对象封装操作类（可以为 null）
     */
    default T selectOne(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper) {
        List<T> ts = this.selectList(queryWrapper);
        if (CollectionUtils.isNotEmpty(ts)) {
            if (ts.size() != 1) {
                throw ExceptionUtils.mpe("One record is expected, but the query result is multiple records");
            }
            return ts.get(0);
        }
        return null;
    }

    /**
     * 根据 Wrapper 条件，判断是否存在记录
     *
     * @param queryWrapper 实体对象封装操作类
     * @return 是否存在记录
     */
    default boolean exists(Wrapper<T> queryWrapper) {
        Long count = this.selectCount(queryWrapper);
        return null != count && count > 0;
    }

    /**
     * 根据 Wrapper 条件，查询总记录数
     *
     * @param queryWrapper 实体对象封装操作类（可以为 null）
     */
    Long selectCount(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

    /**
     * 根据 entity 条件，查询全部记录
     *
     * @param queryWrapper 实体对象封装操作类（可以为 null）
     */
    List<T> selectList(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

    /**
     * 根据 Wrapper 条件，查询全部记录
     *
     * @param queryWrapper 实体对象封装操作类（可以为 null）
     */
    List<Map<String, Object>> selectMaps(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

    /**
     * 根据 Wrapper 条件，查询全部记录
     * <p>注意： 只返回第一个字段的值</p>
     *
     * @param queryWrapper 实体对象封装操作类（可以为 null）
     */
    List<Object> selectObjs(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

    /**
     * 根据 entity 条件，查询全部记录（并翻页）
     *
     * @param page         分页查询条件（可以为 RowBounds.DEFAULT）
     * @param queryWrapper 实体对象封装操作类（可以为 null）
     */
    <P extends IPage<T>> P selectPage(P page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

    /**
     * 根据 Wrapper 条件，查询全部记录（并翻页）
     *
     * @param page         分页查询条件
     * @param queryWrapper 实体对象封装操作类
     */
    <P extends IPage<Map<String, Object>>> P selectMapsPage(P page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
}
```

### 新增功能

MyBatis-Plus在实现插入数据时，会默认基于雪花算法的策略生成id：

```java
@SpringBootTest
class UserMapperTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    void testInsert() {
        User user = new User(null, "张三", 23, "zhangsan@qq.com");
        //INSERT INTO user ( id, name, age, email ) VALUES ( ?, ?, ?, ? )
        int result = userMapper.insert(user);
        System.out.println("受影响行数："+result);
        //id自动获取：1629070359040888833
        System.out.println("id自动获取："+user.getId());
    }
}
```

### 删除功能

```java
@SpringBootTest
class UserMapperTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    void testDelete() {
        //通过id删除用户信息
        int result = userMapper.deleteById(1L);
        System.out.println("result = " + result);

        //根据map集合中所设置的条件删除用户信息
        HashMap<String, Object> map = new HashMap<>();
        map.put("name","张三");
        map.put("age",23);
        int result2 = userMapper.deleteByMap(map);
        System.out.println("result2 = " + result2);

    }
}
```

### 修改功能

```java
@SpringBootTest
class UserMapperTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    void testUpdate() {
       User user = userMapper.selectById(2L);
       user.setAge(45);
       //根据id修改信息
       int result = userMapper.updateById(user);
       System.out.println("result = " + result);
    }
}
```

### 查询功能

```java
@SpringBootTest
class UserMapperTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    void testSelect() {
        //根据id查询用户
        User user = userMapper.selectById(5L);
        System.out.println("user = " + user);

        //根据map集合中的条件查询用户信息
        HashMap<String, Object> map = new HashMap<>();
        map.put("name","张三");
        map.put("age",23);
        userMapper.selectByMap(map).forEach(System.out::println);
    }
}
```

### 自定义功能

> UserMapper.java

```java
/**
 * mapper接口继承BaseMapper 包含了基本的CRUD操作
 * IDEA在 userMapper 处报错，因为找不到注入的对象，因为类是动态创建的，但是程序可以正确的执行。
 * 为了避免报错，可以在mapper接口上添加 @Repository 注解
 * @author hjc
 */
@Repository
public interface UserMapper extends BaseMapper<User> {
    /**
     * 根据id查询用户信息为map集合
     * @param id l
     * @return l
     */
    @MapKey("id")
    Map<String,Object> selectMapById(Long id);
}
```

> resources/mapper/UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.hjc.demo.mapper.UserMapper">

    <select id="selectMapById" resultType="java.util.Map">
        select * from tb_user where  id =#{id}
    </select>
</mapper>
```

> UserMapperTest.java

```java
@SpringBootTest
class UserMapperTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    void testCustomer() {
        //测试自定义
        Map<String, Object> map = userMapper.selectMapById(2L);
        System.out.println("map = " + map);
    }
}
```

## 批量操作

### 批量删除

```java
@SpringBootTest
class UserMapperTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    void testBatchDelete() {
        //通过多个id实现批量删除
        List<Long> list = Arrays.asList(2L, 3L);
        int result = userMapper.deleteBatchIds(list);
        System.out.println("result = " + result);
    }
}
```

### 批量查询

```java
@SpringBootTest
class UserMapperTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    void testBatchSelect() {
        //根据多个id查询多个用户信息
        List<Long> list = Arrays.asList(1L, 2L, 3L);
        userMapper.selectBatchIds(list).forEach(System.out::println);

        //通过条件构造器查询一个list集合，若没有条件，则可以设置null为参数
        //查询所有数据
        userMapper.selectList(null).forEach(System.out::println);
    }
}
```

## 通用Service

通用 Service CRUD 封装IService接口，进一步封装 CRUD。

采用：get 查询单行、remove 删除、list 查询集合、page 分页。前缀命名方式区分 Mapper 层避免混淆， 泛型 T 为任意实体对象 建议如果存在自定义通用 Service 方法的可能，请创建自己的 IBaseService 继承 Mybatis-Plus 提供的基类。

官网地址：https://baomidou.com/pages/49cc81/#service-crud-%E6%8E%A5%E5%8F%A3

### IService

MyBatis-Plus中有一个接口 IService和其实现类 ServiceImpl，封装了常见的业务层逻辑。

![image-20230225150753143](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202302261313729.png)

![image-20230225150819726](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202302261313888.png)

### 创建Service接口和实现类

> service/UserService.java

```java
/**
 * UserService继承IService模板提供的基础功能
 * @author hjc
 */
public interface UserService extends IService<User> {
}
```

> service/Imp/UserServiceImpl

```java
/**
 * ServiceImpl实现了IService，提供了IService中基础功能的实现
 * 若ServiceImpl无法满足业务需求，则可以使用自定的UserService定义方法，并在实现类中实现
 * @author hjc
 */
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
}
```

### 测试插入

```java
@SpringBootTest
class UserServiceTest {
    @Autowired
    private UserService userService;
    @Test
    void testSave() {
        //插入一条数据
        User user = new User();
        user.setName("李贝");
        user.setEmail("2331@qq.com");
        user.setAge(15);
        boolean result = userService.save(user);
        System.out.println("result = " + result);

        //批量插入数据
        ArrayList<User> users = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            User user1 = new User(null, "测试" + i, 10 + i, "111");
            users.add(user1);
        }
        boolean saveBatch = userService.saveBatch(users);
        System.out.println("saveBatch = " + saveBatch);

        //批量插入数据(按照插入批次数量)
        ArrayList<User> users1 = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            User user1 = new User(null, "批量" + i, 10 + i, "111");
            users1.add(user1);
        }
        boolean saveBatch1 = userService.saveBatch(users1, 1);
        System.out.println("saveBatch1 = " + saveBatch1);

        //saveOrUpdateBatch既有修改功能又有修改功能
        //区分：看实体类有没有id 有id是修改 没有id是添加
        User user1 = new User();
        user1.setName("李贝");
        user1.setEmail("2331@qq.com");
        user1.setAge(15);
        boolean saveOrUpdate = userService.saveOrUpdate(user1);
        System.out.println("saveOrUpdate = " + saveOrUpdate);
        //规则同上 执行批量添加或更改
//        userService.saveOrUpdateBatch();
    }
}
```

### 测试删除

```java
@SpringBootTest
class UserServiceTest {
    @Autowired
    private UserService userService;

    @Test
    void testRemove() {
        User user = userService.getById(1L);
        userService.removeById(user);
        userService.removeById(2L);
        //是否启用填充(为true的情况,会将入参转换实体进行delete删除)
        userService.removeById(3L,true);
        userService.removeByIds(Arrays.asList(1L,2L));
        //是否填充(为true的情况,会将入参转换实体进行delete删除)
        userService.removeByIds(Arrays.asList(3L,4L),true);
        userService.removeBatchByIds(Arrays.asList(3L,4L));
        userService.removeBatchByIds(Arrays.asList(3L,4L),true);
        userService.removeBatchByIds(Arrays.asList(3L,4L),1);
        userService.removeBatchByIds(Arrays.asList(3L,4L),1,true);
        HashMap<String, Object> map = new HashMap<>();
        map.put("id",1);
        userService.removeByMap(map);
    }
}
```

### 测试修改

```java
@SpringBootTest
class UserServiceTest {
    @Autowired
    private UserService userService;

    @Test
    void testUpdate() {
        userService.updateById(new User(1L,"孙策",10,"222"));
        ArrayList<User> list = new ArrayList<>();
        list.add(new User(2L,"孙策",12,"123"));
        list.add(new User(3L,"孙权",44,"456"));
        userService.updateBatchById(list);
        userService.updateBatchById(list,2);
    }
}
```

### 测试查询

```java
@SpringBootTest
class UserServiceTest {
    @Autowired
    private UserService userService;

    @Test
    void testSelect() {
        //查询总记录数
        long count = userService.count();
        System.out.println("count = " + count);

        userService.list().forEach(System.out::println);

        List<Map<String, Object>> maps = userService.listMaps();
        System.out.println("maps = " + maps);

        userService.listByIds(Arrays.asList(2L,3L)).forEach(System.out::println);

        userService.listObjs().forEach(System.out::println);

        HashMap<String, Object> map = new HashMap<>();
        map.put("name","李贝");
        userService.listByMap(map).forEach(System.out::println);

        User user = userService.getById(1L);
        System.out.println("user = " + user);

        BaseMapper<User> baseMapper = userService.getBaseMapper();
        baseMapper.selectList(null).forEach(System.out::println);
    }
}
```


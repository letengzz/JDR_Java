# Java 操作 MinIO

## 导入依赖

```xml
<dependency>
    <groupId>io.minio</groupId>
    <artifactId>minio</artifactId>
    <version>8.5.9</version>
</dependency>
```

## 创建客户端对象

当使用SpringBoot时，可以将客户端对象注入到容器中：

```java
@Configuration
public class MyConfig {
  	//单例 没有线程安全问题 只用配置一个bean即可
    @Bean
    public MinioClient minioClient(){
        //通过构造器 使用链式编程 构建客户端对象
        return MinioClient.builder()
                .endpoint("http://192.168.100.200:9000")  //构建一个端点
                .credentials("admin","password")  //访问的账号和密码
                .build();
    }
}
```

通过注入客户端 即可操作服务器：

```java
@SpringBootTest
class MinioSpringbootApplicationTests {
    @Resource
    private MinioClient minioClient;
    
    @Test
    void test() {
        System.out.println(minioClient);
    }
}
```

当不使用SpirngBoot时：

```java
@Test
void contextLoads() {
  	//通过构造器 使用链式编程 构建客户端对象
    MinioClient minioClient = MinioClient.builder()
            .endpoint("http://192.168.100.200:9000")  //构建一个端点
            .credentials("admin", "password")  //访问的账号和密码
            .build();
    System.out.println(minioClient);
}
```

## MinioClient 方法

![minio (1)](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405051326957.png)

## Bucket 操作

Bucket 是存储Object的逻辑空间，每个Bucket之间的数据是相互隔离的，对用户而言，相当于存放文件的顶层**文件夹**

常用API：

- `bucketExists()`：检查指定的存储桶是否存在，返回布尔值，表示存储桶是否存在
- `makeBucket()`：创建一个新的存储桶(bucket)，需要指定存储桶的名称
- `listBuckets()`：列出用户有权访问的所有的存储桶，返回存储桶的列表
- `removeBucket()`：删除一个已存在的存储桶(bucket)，删除失败会抛出异常

### 判断存储桶是否存在

```java
@Test
void existsBucketTest() throws Exception {
    boolean exists = minioClient.bucketExists(BucketExistsArgs.builder().bucket("test").build());
    System.out.println("test目录是否存在：" + exists);
}
```

![image-20240505154303591](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405051543554.png)

### 创建存储桶

```java
@Test
void makeBucketTest() throws Exception {
    //判断是否存在
    boolean exists = minioClient.bucketExists(BucketExistsArgs.builder().bucket("test").build());
    if (!exists){
        minioClient.makeBucket(MakeBucketArgs.builder().bucket("test").build());
        System.out.println("bucket创建成功！");
    }else {
        System.out.println("bucket已经存在，无需再次创建！");
    }
}
```

![image-20240505154652506](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405051546181.png)

![image-20240505154759143](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405051548164.png)

### 列出所有存储桶

```java
@Test
void bucketListTest() throws Exception {
    //获取所有的存储桶
    List<Bucket> buckets = minioClient.listBuckets();
    buckets.forEach(bucket -> {
        System.out.println(bucket.name() + "---"+ bucket.creationDate());
    });
}
```

![image-20240505155102973](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405051551200.png)

### 删除已存在的存储桶

```java
@Test
void removeBucketTest() throws Exception {
    try {
        minioClient.removeBucket(RemoveBucketArgs.builder().bucket("test").build());
        System.out.println("删除成功");
    } catch (Exception e) {
        System.out.println("删除失败");
    }
}
```

删除成功：

![image-20240505155601476](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405051556068.png)

删除失败：

![image-20240505155622842](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405051556116.png)

## Object 操作

Object是存储到MinIO的基本对象，对用户而言，相当于**文件**

常用API：

- `putObject()`：上传文件到指定的存储桶
- `statObject()`：获取指定的对象(文件)信息，不存在时抛出异常
- `getPresignedObjectUrl()`：生成一个对象(文件)的URL，以便可以通过HTTP直接访问
- `getObject()`：从指定的存储桶中下载文件
- `listObjects()`：列出指定存储桶中的所有对象(文件)
- `removeObject()`：删除指定存储桶中的对象，需要指定存储桶名称和对象键

### 上传文件

方式一：

```java
@Test
void uploadFileTest() throws Exception {
    File file = new File("/Users/cuifendemac/minio.png");
    ObjectWriteResponse writeResponse = minioClient.putObject(PutObjectArgs.builder()
            .bucket("test") //存储到指定的存储桶
            .object("test.png") //存储到指定的对象
            .stream(new FileInputStream(file), file.length(), -1) //上传的文件流  objectSize:文件大小 partSize: 缓冲区大小 一般为-1
            .contentType("image/jpeg") //上传的文件类型
            .build());
    System.out.println("上传成功："+writeResponse);
}
```

方式二：

```java
@Test
void uploadFileTest2() throws Exception {
    ObjectWriteResponse uploadedObject = minioClient.uploadObject(UploadObjectArgs.builder()
            .bucket("test") //存储到指定的存储桶
            .object("test1.png") //存储到指定的对象
            .filename("/Users/cuifendemac/minio.png") //上传的文件路径
            .build());
    System.out.println("上传成功："+uploadedObject);
}
```

![image-20240505210519471](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405052105856.png)

### 检查文件是否存在

通过文件状态来判断文件是否存在：

```java
@Test
void existsObjectTest() throws Exception {
    try {
        StatObjectResponse statObject = minioClient.statObject(StatObjectArgs.builder()
                .bucket("test") //存储到指定的存储桶
                .object("test.png") //存储到指定的对象
                .build());
        System.out.println(statObject);
    } catch (Exception e) {
        System.out.println("文件不存在");
    }
}
```

存在则返回文件信息：

![image-20240505214125654](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405052141409.png)

不存在则会抛出异常：

![image-20240505214531327](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405052145265.png)

### 下载文件

```java
@Test
void objectDownTest() throws Exception {
    GetObjectResponse getObjectResponse = minioClient.getObject(GetObjectArgs.builder()
            .bucket("test")
            .object("test.png")
            .build());
    System.out.println(getObjectResponse.transferTo(new FileOutputStream("/Users/cuifendemac/minio2.png")));
}
```

### 列出存储桶所有文件

```java
@Test
void objectListTest() throws Exception {
    Iterable<Result<Item>> listedObjects = minioClient.listObjects(ListObjectsArgs.builder()
            .bucket("test")
            .build());
    for (Result<Item> result : listedObjects) {
        Item item = result.get();
        System.out.println(item.objectName() +" -- "+item.size() + " -- " + item.lastModified());
    }
}
```

![image-20240506230317282](./assets/image-20240506230317282.png)

### 删除文件

```java
@Test
void removeObjectTest() throws Exception {
    minioClient.removeObject(RemoveObjectArgs.builder()
            .bucket("test")
            .object("test1.png")
            .build());
    System.out.println("删除成功");
}
```

### 生成URL

```java
@Test
void getUrlTest() throws Exception {
    String url = minioClient.getPresignedObjectUrl(GetPresignedObjectUrlArgs.builder()
            .bucket("test") //存储到指定的存储桶
            .object("test.png") //存储到指定的对象
            .expiry(3, TimeUnit.MINUTES) //过期时间 超过三分钟就会失效
            .method(Method.GET) //请求方式
            .build());
    System.out.println("url = " + url);
}
```

![image-20240506225400406](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405062254342.png)

## 开放公共URL

默认情况下，上传到MinIO服务器上的文件不能通过 <http://ip:9000/存储桶/文件名> 直接访问的。

可以通过以下方式来进行修改：

1. 在Web管理后台修改访问策略为public，此时不用签名即可访问，公共的访问策略可以任何人上传下载删除：

   ![image-20240507124557648](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405071246135.png)

2. 自定义访问策略：

   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "PublicRead",
               "Effect": "Allow",
               "Principal": {
                   "AWS": [
                       "*"
                   ]
               },
               "Action": [
                   "s3:GetObject"
               ],
               "Resource": [
                   "arn:aws:s3:::test2/*"
               ]
           }
       ]
   }
   ```

   ![image-20240507125330592](https://cdn.jsdelivr.net/gh/letengzz/tc2@main/img202405071253316.png)

3. 通过客户端API修改： 创建存储桶，修改文件访问策略为公开的读权限

   ```java
   @Test
   void publicUrlTest() throws Exception {
       String bucketName = "test2";
       //判断是否存在
       boolean exists = minioClient.bucketExists(BucketExistsArgs.builder().bucket(bucketName).build());
       if (!exists) {
           minioClient.makeBucket(MakeBucketArgs.builder().bucket(bucketName).build());
           System.out.println("bucket创建成功！");
       } else {
           System.out.println("bucket已经存在，无需再次创建！");
       }
   
       String policyJsonString = "{\n" +
           "    \"Version\": \"2012-10-17\",\n" +
           "    \"Statement\": [\n" +
           "        {\n" +
           "            \"Sid\": \"PublicRead\",\n" +
           "            \"Effect\": \"Allow\",\n" +
           "            \"Principal\": {\n" +
           "                \"AWS\": [\n" +
           "                    \"*\"\n" +
           "                ]\n" +
           "            },\n" +
           "            \"Action\": [\n" +
           "                \"s3:GetObject\"\n" +
           "            ],\n" +
           "            \"Resource\": [\n" +
           "                \"arn:aws:s3:::"+bucketName+"/*\"\n" +
           "            ]\n" +
           "        }\n" +
           "    ]\n" +
           "}";
     
       //创建存储桶的时候，设置该存储桶的文件的访问策略，运行公开的读
       minioClient.setBucketPolicy(
               SetBucketPolicyArgs.builder()
                       .bucket(bucketName)
                       .config(policyJsonString) //访问策略
                       .build()
       );
   }
   ```

   
# 接口设计

设计一个接口需要包括以下几个方面：

1. 协议：

   通常协议采用HTTP，查询类接口通常为get或post，查询条件较少的使用get，较多的使用post。
   本接口使用 http post。
   还要确定content-type，参数以什么数据格式提交，结果以什么数据格式响应。
   一般情况没有特殊情况结果以json 格式响应。

2. 分析请求参数：

   根据前边对数据模型的分析，请求参数为：课程名称、课程审核状态、当前页码、每页显示记录数。
   根据分析的请求参数定义模型类。

3. 分析响应结果：

   根据前边对数据模型的分析，响应结果为数据列表加一些分页信息（总记录数、当前页、每页显示记录数）。
   数据列表中数据的属性包括：课程id、课程名称、任务数、创建时间、审核状态、类型。
   注意：查询结果中的审核状态为数据字典中的代码字段，前端会根据审核状态代码 找到对应的名称显示。
   根据分析的响应结果定义模型类。

4. 分析完成，使用SpringBoot注解开发一个Http接口。
5. 使用接口文档工具查看接口的内容。
6. 接口中调用Service方法完成业务处理。

接口请求示例：

> POST /content/course/list?pageNo=2&pageSize=1 
> Content-Type: application/json  
> {  
>     "auditStatus": "202002", 
>     "courseName": "",  
>     "publishStatus":"" 
> }

 **成功响应结果 **：

```json
{
    "items": [{
        "id": 26,
        "companyId": 1232141425,
        "companyName": null,
        "name": "spring cloud实战",
        "users": "所有人",
        "tags": null,
        "mt": "1-3",
        "mtName": null,
        "st": "1-3-2",
        "stName": null,
        "grade": "200003",
        "teachmode": "201001",
        "description": "本课程主要从四个章节进行讲解： 1.微服务架构入门 2.spring cloud 基础入门 3.实战Spring Boot 4.注册中心eureka。",
        "pic": "https://cdn.baidu.com/academy/wp-content/uploads/2018/08/Spring-BOOT-Interview-questions.jpg",
        "createDate": "2019-09-04 09:56:19",
        "changeDate": "2021-12-26 22:10:38",
        "createPeople": null,
        "changePeople": null,
        "auditStatus": "202002",
        "auditMind": null,
        "auditNums": 0,
        "auditDate": null,
        "auditPeople": null,
        "status": 1,
        "coursePubId": null,
        "coursePubDate": null
    }],
    "counts": 23,
    "page": 2,
    "pageSize": 1
}
```

 

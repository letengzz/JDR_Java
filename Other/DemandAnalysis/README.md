# 需求分析

在百度百科中对需求分析的定义：

> 需求分析也称为软件需求分析、系统需求分析或需求分析工程等，是开发人员经过深入细致的调研和分析，准确理解用户和项目的功能、性能、可靠性等具体要求，将用户非形式的需求表述转化为完整的需求定义，从而确定系统必须做什么的过程。

简单理解就是要搞清楚问题域，问题域就是用户的需求，软件要为用户解决什么问题，实现哪些业务功能，满足什么样的性能要求。

**如何作需求分析？**

- **第一：首先确认用户需求**：

  用户需求即用户的原始需求。

  通过用户访谈、问卷调查、开会讨论、查阅资料等调研手段梳理用户的原始需求。

  产品人员根据用户需求会绘制界面原型，通过界面原型再和用户确认需求。

- **第二：确认关键问题**：

  用户的原始需求可能 是含糊不清的，需求分析要从繁杂的问题中梳理出关键问题。

  比如：教学机构的老师想要将课程发布到网上，这是原始需求，根据这个用户需求需要进行扩展分析，扩展出几下几点：

  1. 课程发布需要发布哪些信息
  2. 如果发布了不良信息怎么办？
  3. 课程发布后用户怎么查看课程？

  根据以上几点继续延伸性分析：课程发布需要发布哪些信息：课程名称、课程介绍、课程价格、课程图片、师资等信息

  继续延伸分析：这么多课程信息进行归类，方便用户编辑，分为课程基本信息、课程营销信息、课程师资等信息。

  按照这样的思路对用户需求逐项分析，梳理出若干问题，再从中找到关键问题。比如：上边对课程信息分类后，哪些是关键信息，课程名称、课程图片、课程介绍等基本信息为关键信息，所以发布课程的第一步要编写课程基本信息。

  找到了关键问题，下一步就可以进行数据建模，创建课程基本信息表，并设计其中的字段。

- **第三：梳理业务流程**

  业务流程是由一个或多个用户参与完成为了完成一个目标所进行的一系列的业务操作，不论是整个系统还是一个模块通常首先分析核心的业务流程，比如：内容管理模块的核心业务流程是课程发布，本项目的核心业务流程是学生选课学习流程。

- **第四：数据建模**

  数据建模要根据分析的关键问题将其相关的信息全部建模。比如：根据发布课程的用户需求，可创建课程基本信息表、课程营销信息表、课程师资表、课程发布记录表、课程审核记录表等。

- **第五：编写需求规格说明书**

  需求分析阶段的成果物是需求分析规格说明书，针对每一个问题编写需求用例，需求用例包括：功能名称、功能描述、参与者、基本事件流程、可选事件流、数据描述、前置条件、后置条件等内容。

  比如：添加课程的需求用例：

  ![image-20231217142734973](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312171427116.png)

## **模块介绍**

内容管理这个词存在于很多软件系统，什么是内容管理 ？

内容管理系统（content management system，CMS），是一种位于WEB前端（Web 服务器）和后端办公系统或流程（内容创作、编辑）之间的软件系统。内容的创作人员、编辑人员、发布人员使用内容管理系统来提交、修改、审批、发布内容。这里指的“内容”可能包括文件、表格、图片、数据库中的数据甚至视频等一切你想要发布到Internet、Intranet以及Extranet网站的信息。

## **业务流程**

**例**：内容管理的业务由教学机构人员和平台的运营人员共同完成。

教学机构人员的业务流程：

1. 登录教学机构。
2. 维护课程信息，添加一门课程需要编辑课程的基本信息、上传课程图片、课程营销信息、课程计划、上传课程视频、课程师资信息等内容。
3. 课程信息编辑完成，通过课程预览确认无误后提交课程审核。
4. 待运营人员对课程审核通过后方可进行课程发布。

运营人员的业务流程：

1. 查询待审核的课程信息。
2. 审核课程信息。
3. 提交审核结果。

*课程编辑与发布的整体流程：

![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312152338213.jpg#id=pDuKh&originHeight=483&originWidth=567&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

## **界面原型**

产品工程师根据用户需求制作产品界面原型，开发工程师除了根据用户需求进行需求分析以外，还会根据界面原型上的元素信息进行需求分析。

例：**内容管理模块的界面原型**

- 课程列表 ：

  ![image.png](https://cdn.nlark.com/yuque/0/2023/png/35537136/1702654365312-51661ebd-a2ab-46b0-b49a-b7d5553bb026.png#averageHue=%23fcfcfb&clientId=u18eafb1a-7fad-4&from=paste&id=u3f01f576&originHeight=246&originWidth=567&originalType=url&ratio=1.25&rotation=0&showTitle=false&size=50326&status=done&style=none&taskId=ua7efb0a6-7b5a-42ab-aa19-512df12bd8e&title=)

- 点击添加课程：

  ![image.png](https://cdn.nlark.com/yuque/0/2023/png/35537136/1702654366811-269b37ac-394a-4517-a240-dd920d391fdc.png#averageHue=%23fdfbfa&clientId=u18eafb1a-7fad-4&from=paste&id=u04ea7b1f&originHeight=450&originWidth=567&originalType=url&ratio=1.25&rotation=0&showTitle=false&size=48978&status=done&style=none&taskId=u3719b59c-115e-4fae-9191-a7d7dcbf170&title=)

- 选择录播课程 ，填写课程信息：

  ![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312152340929.jpg#id=BwXJn&originHeight=416&originWidth=567&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

- 填写课程计划信息：

  ![image.png](https://cdn.nlark.com/yuque/0/2023/png/35537136/1702654355993-1abb7e46-aebb-4b28-a056-e1c3385fd1c5.png#averageHue=%23f8f8f8&clientId=u18eafb1a-7fad-4&from=paste&id=u37a65969&originHeight=268&originWidth=567&originalType=url&ratio=1.25&rotation=0&showTitle=false&size=35527&status=done&style=none&taskId=udccc770c-943a-4533-b338-754eff7f00d&title=)

- 填写课程师资信息：

  ![image.png](https://cdn.nlark.com/yuque/0/2023/png/35537136/1702654354716-5fdbd516-2cd6-466c-8189-f1b2aab43181.png#averageHue=%23fcfcfc&clientId=u18eafb1a-7fad-4&from=paste&id=ub225c893&originHeight=223&originWidth=567&originalType=url&ratio=1.25&rotation=0&showTitle=false&size=28215&status=done&style=none&taskId=ua3456090-1580-46b6-b92a-9f7c63965ac&title=)

- 课程填写完毕进行课程 发布：当审核状态为通过时发布按钮点亮，点击发布按钮 即可对该课程进行发布：

  ![image.png](https://cdn.nlark.com/yuque/0/2023/png/35537136/1702654357807-9c8da12a-0188-458f-af06-be00f4a05688.png#averageHue=%23fafaf9&clientId=u18eafb1a-7fad-4&from=paste&id=ue7b0f348&originHeight=149&originWidth=567&originalType=url&ratio=1.25&rotation=0&showTitle=false&size=35058&status=done&style=none&taskId=u8e0dd4a7-9c23-4e8d-962a-ac81d02d75b&title=)

## **数据模型**

使用PowerDesigner建立模型：

 ![](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202312152333092.jpg#id=zyNng&originHeight=273&originWidth=567&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
# Git 其他操作

## git config 自定义配置颜色

Git会适当地显示不同的颜色，比如 git status 命令

```bash
git config --global color.ui true
```

![image-20230514004027556](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140113887.png)

## 忽略特殊文件

一般情况下工作区中的文件都是要交给Git管理的，但有些文件并不想交给Git管理。但是又由于该文件处理工作区，所有在执行git status命令的时候会给出xx文件Untracked (未被跟踪)

在Git工作区的根目录下创建一个特殊的 .gitignore 文件，然后把要忽略的文件名填进去，Git就会自动忽略这些文件。 

GitHub已经准备了各种配置文件，只需要组合一下就可以使用了。

所有配置文件可以直接在线浏览：https://github.com/github/gitignore

### 添加文件，用 -f 强制添加到Git

```bash
git add -f App.class
```

### 检查忽略文件

```bash
git check-ignore 
```

### 忽略文件的原则

- 忽略操作系统自动生成的文件，比如缩略图等
- 忽略编译生成的中间文件、可执行文件等，也就是如果一个文件是通过另一个文件自动生成的，那自动生成的文件就没必要放进版本库，比如Java编译产生的 .class 文件；
- 忽略你自己的带有敏感信息的配置文件，比如存放口令的配置文件。

### 忽略文件的案例

假设在Windows下进行Python开发，Windows会自动在有图片的目录下生成隐藏的缩略图文件，如果有自定义目录，目录下就会有 Desktop.ini 文件，因此需要忽略Windows自动生成的垃圾文件：

```bash
# Windows: 
Thumbs.db 
ehthumbs.db 
Desktop.ini 
```

继续忽略Python编译产生的 .pyc 、 .pyo 、 dist 等文件或目录：

```bash
# Python:
*.py[cod]
*.so
*.egg
*.egg-info
dist
build
```

加上你自己定义的文件，最终得到一个完整的 .gitignore 文件：

```bash
# Windows:
Thumbs.db
ehthumbs.db
Desktop.ini
# Python:
*.py[cod]
*.so
*.egg
*.egg-info
dist
build
# My configurations:
db.ini
deploy_key_rsa
```

 最后一步就是把 .gitignore 也提交到Git，就完成了

## 配置别名

- 将 `status` 设 st 为别名： 

  ```bash
  git config --global alias.st status
  ```

- 配置一个 unstage 别名： 

  ```bash
  git config --global alias.unstage 'reset HEAD'
  ```

  当使用 `git unstage test.py` 实际上是  `git reset HEAD test.py`

- 配置一个 `git last` ，让其显示最后一次提交信息

  ```bash
  git config --global alias.last 'log -1'
  ```

配置Git的时候，加上 `--global` 是针对当前用户起作用的，如果不加，那只针对当前的仓库起作用。

每个仓库的Git配置文件都放在 .git/config 文件中

- 当前用户的Git配置文件放在用户主目录下的一个隐藏文件 .gitconfig 中

- 配置别名也可以直接修改这个文件，如果改错了，可以删掉文件重新通过命令配置。

## 高效使用GitHub

### 如何找项目

#### 搜索spring boot

##### 根据名字搜索

名字含有spring boot的项目：`in:name spring boot`

![image-20230514005018629](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140113480.png)

##### 根据 starts和forks搜索

搜索名字含有spring boot同时starts > 3000，forks>3000：`in:name spring boot stars:>3000 forks:>3000`

![image-20230514005047099](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140113519.png)

##### 根据 readme和language和pushed搜索

搜索readme中含有微服务 语言为java push时间为2019-09-01之后的：`in:readme 微服务 language:java pushed:>2019-09-01`

![image-20230514005106159](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140113396.png)

##### 根据 description 搜索

搜索描述包含爬虫，语言为python，stars >1000 ,2020-1-1有更新的：`in:description 爬虫 language:python stars:>1000 pushed:>2020-01-01`

![image-20230514005136561](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140113112.png)

#### 组合搜索

搜索文档中包含spring security的开源项目

![image-20230514004955567](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202305140113040.png)

### 优秀的开源项目

#### 博客

##### 蘑菇博客

- https://gitee.com/moxi159753/mogu_blog_v2

##### solo

- https://github.com/88250/solo

##### OneBlog

- https://github.com/zhangyd-c/OneBlog

##### halo

- https://github.com/halo-dev/halo

#### 快速开发平台

##### jeecg-boot

JeecgBoot 是一款基于代码生成器的 低代码 开发平台！前后端分离架构 SpringBoot2.x，SpringCloud，Ant Design&Vue，Mybatis-plus，Shiro，JWT，支持微服务。强大的代码生成器让前后端代码一键生成，实现低代码开发! JeecgBoot 引领新的低代码开发模式(OnlineCoding-> 代码生成器-> 手工MERGE)， 帮助解决Java项目70%的重复工作，让开发更多关注业务。既能快速提高效率，节省研发成本，同时又不失灵活性！

- https://github.com/zhangdaiscott/jeecg-boot

##### el-admin

一个基于 Spring Boot 2.1.0 、 Spring Boot Jpa、 JWT、Spring Security、Redis、Vue的前后端分离的后台管理系统，有丰富的文档和教程

- https://github.com/elunez/eladmin

#### 电商项目

##### mall

mall 项目是一套电商系统，包括前台商城系统及后台管理系统，基于SpringBoot+MyBatis实现，采用Docker容器化部署。前台商城系统包含首页门户、商品推荐、商品搜索、商品展示、购物车、订单流程、会员中心、客户服务、帮助中心等模块。后台管理系统包含商品管理、订单管理、会员管理、促销管理、运营管理、内容管理、统计报表、财务管理、权限管理、设置等模块。

- https://github.com/macrozheng/mall

##### xmall

- https://github.com/Exrick/xmall

##### litemall

Spring Boot后端 + Vue管理员前端 + 微信小程序用户前端 + Vue用户移动端

- https://github.com/linlinjava/litemall

#### 其他项目

##### V部落

V部落是一个多用户博客管理平台，采用Vue+SpringBoot开发。

- https://github.com/lenve/VBlog

##### 微人事

微人事是一个前后端分离的人力资源管理系统，项目采用 SpringBoot+Vue 开发，项目加入常见的企业级应用所涉及到的技术点，例如 Redis、RabbitMQ 等。

- https://github.com/lenve/vhr

##### NiceFish

NiceFish（美人鱼） 是一个系列项目，目标是示范前后端分离的开发模式:前端浏览器、移动端、Electron 环境中的各种开发模式；后端有两个版本：SpringBoot 版本和 SpringCloud 版本

- https://github.com/damoqiongqiu/NiceFish
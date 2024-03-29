# 物理视图和逻辑视图

## 物理视图

在Servlet中，将请求转发到一个HTML页面文件时，使用的完整的转发路径就是**物理视图**。

> /pages/user/login_success.html

如果把所有的HTML页面都放在某个统一的目录下，那么转发地址就会呈现出明显的规律：

> /pages/user/login.html 
>
> /pages/user/login_success.html 
>
> /pages/user/regist.html
>
>  /pages/user/regist_success.html
>
> ……

路径的开头都是：`/pages/user/`

路径的结尾都是：`.html`

所以，路径开头的部分称之为**视图前缀**，路径结尾的部分称之为**视图后缀**。

## 逻辑视图

物理视图=视图前缀+逻辑视图+视图后缀

![image-20230731212342772](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202307312123752.png)
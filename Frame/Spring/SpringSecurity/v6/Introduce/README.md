# SpringSecurity 概述

安全是开发者永远绕不开的话题，一个不安全的网站，往往存在着各种致命漏洞，只要被不法分子稍加利用，就能直接击溃整个网站，甚至破坏网站宝贵的用户数据。而用户的授权校验，则是网站安全系统的典型代表，这也是用户访问网站的第一关，我们需要一个更加安全和可靠的授权校验框架，才能让我们的网站更加稳定。

SpringSecurity是一个基于Spring开发的非常强大的**权限验证安全框架**，提供一系列的功能保护应用程序的安全性，其核心功能包括：

- 身份认证 (Authentication 用户登录)：身份认证是验证 **谁正在访问系统资源** 判断用户是否为合法用户 。认证用户的常见方式是要求用户输入用户名和密码  

- 授权 (Authorization 此用户能够做哪些事情)：用户进行身份认证后，系统会控制**谁能访问哪些资源** 这个过程叫做授权。用户无法访问没有权限的资源

- 攻击防护 (防止伪造身份攻击 protection against common attacks)：

  防御常见的攻击：

  - CSRF
  - HTTP Headers
  - HTTP Request


官方文档：https://docs.spring.io/spring-security/reference/index.html

****

[网络安全基础](../../../../../Other/NetworkSecurity/README.md)

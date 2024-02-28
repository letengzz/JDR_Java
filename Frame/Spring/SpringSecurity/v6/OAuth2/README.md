# OAuth 2

OAuth2：开放标准的授权协议。用于在不暴露用户凭证的情况下进行身份验证和授权。

## OAuth2 流程

首先要有用户的数据，有一个资源服务器负责管理用户数据，需要有一个客户应用需要访问用户应用。给资源服务器暴露用户数据称为API ，客户应用通过API访问资源服务器来返回用户的数据。

![image-20240227205942188](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272059017.png)

如果来恶意用户应用访问资源服务器并且没有认证和校验，那么恶意客户也能访问到用户数据。就需要一种机制来保护用户的数据。

![image-20240227210002261](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272100586.png)

业界实践是提前给客户应用颁发一个Access Token，表示用户应用被授权可以访问用户数据，当访问用户数据时，给出Access Token。客户端应用每次发送的请求，都要携带Access Token( 一般在请求头中携带)。

![image-20240227210113309](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272101071.png)

接下来，资源服务器会取出请求中的Access Token并校验Access Token确认客户应用有访问用户数据的权限，校验通过后，资源服务器返回用户数据。

![image-20240227210139302](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272101857.png)

这个机制可以工作的前提是必须提前给客户应用颁发Access Token，需要颁发Access Token的角色，这个角色就是授权服务器

授权服务器负责生成Access Token 并给客户应用颁发Access Token，客户应用带上Access Token访问用户数据。

![image-20240227210248379](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272102785.png)

资源服务器会从请求中取出Access Token 并且校验Access Token具有访问用户数据的权限校验成功就会给客户应用了。

![image-20240227210402452](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272104767.png)

在真实流程中，在颁发Token前要征询用户同意

![image-20240227210426020](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272104475.png)

如果用户同意授权服务器颁发token ，授权服务器会生成一个Access Token 并将token 颁发给客户应用

![image-20240227210455072](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272105629.png)



OAuth 2.0标准化了Access Token的请求和响应部分， OAuth2.0的细节在[RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749) (OAuth 2.0授权框架)中描述。

**OAuth2 最简向导：**[The Simplest Guide To OAuth 2.0](https://darutk.medium.com/the-simplest-guide-to-oauth-2-0-8c71bd9a15bb)

## OAuth2 角色

OAuth 2协议包含以下角色：

1. 资源所有者(`Resource Owner`)：即用户，资源的拥有人，想要通过客户应用访问资源服务器上的资源。
2. 客户应用(`Client`)：通常是一个Web或者无线应用，它需要访问用户的受保护资源。客户应用和资源服务器之间可以是两个完全不同的应用程序系统也可以是微服务中的两个微服务。
3. 资源服务器 (`Resource Server`)：存储受保护资源的服务器或定义了可以访问到资源的API，接收并验证客户端的访问令牌，以决定是否授权访问资源。
4. 授权服务器 (`Authorization Server`)：负责验证资源所有者的身份并向客户端颁发访问令牌。

![image-20231222124053994](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272027866.png)

## OAuth2 使用场景

### 开放系统间授权

#### 社交登录

在传统的身份验证中，用户需要提供用户名和密码，还有很多网站登录时，允许使用第三方网站的身份，这称为"第三方登录"。所谓第三方登录，实质就是 OAuth 授权。用户想要登录 A 网站，A 网站让用户提供第三方网站的数据，证明自己的身份。获取第三方网站的身份数据，就需要 OAuth 授权。

![image-20231222131233025](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272027683.png)

#### 开放API

例如云冲印服务的实现：资源拥有者在云存储服务上存储照片，通过云冲印服务来在线冲印照片需要云存储服务授权给云冲印服务。此时需要授权服务器给云冲印颁发一个令牌，云冲印服务携带着令牌就可以访问云存储服务的照片。

![image-20231222131118611](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272027145.png)

### 现代微服务安全

#### 单块应用安全

由于应用程序服务器通常情况下都是单体应用部署到一台服务器中，所以在登录的过程中通常使用用户名和密码的方式来校验用户名和密码是否正确，如果正确的话，会存储一个session，给客户端返回一个携带sessionid 的Cookie，这个Cookie会存储在浏览器中。下一次登录时，会自动将请求中携带着Cookie去访问服务器，通过过滤器判断Cookie的Sessionid的用户信息，如果能够获取就是合法登录的可以访问业务，如果没有获取到就可能是没有登录或登录过期的，就不能访问业务

![image-20231222152734546](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272028436.png)



#### 微服务安全

在微服务架构下，不是之前的单体应用了，面临着拆分粒度很小，每个项目都会拆分成多个微服务，微服务之间需要认证和授权。此外，前端的应用程序变得多种多样了，不是简单的基于浏览器的应用，因此，单体应用这种传统的方式就不能满足需求，使用OAuth2就可以解决该问题。

![image-20231222152557861](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272028208.png)



### 企业内部应用认证授权

企业内部应用认证授权，包含：

- **单点登录** (`SSO`、`Single Sign On`) 

- **身份识别与访问管理** (`IAM`、`Identity and Access Management` )

## OAuth2 四种授权模式

RFC6749：[RFC 6749 - The OAuth 2.0 Authorization Framework (ietf.org)](https://datatracker.ietf.org/doc/html/rfc6749)

阮一峰：[OAuth 2.0 的四种方式 - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html)

****

四种模式：

- **授权码** (`authorization-code`)
- **隐藏式** (`implicit`)
- **密码式** (`password`)
- **客户端凭证** (`client credentials`)

### 授权码

**授权码 (`authorization code`)，指的是第三方应用先申请一个授权码，然后再用该码获取令牌**

这种方式是最常用，最复杂，也是最安全的，它适用于那些有后端的 Web 应用。授权码通过前端传送，令牌则是储存在后端，而且所有与资源服务器的通信都在后端完成。这样的前后端分离，可以避免令牌泄漏。

![image-20231220180422742](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272030332.png)

注册客户应用：客户应用如果想要访问资源服务器需要有凭证，需要在授权服务器上注册客户应用。注册后会**获取到一个ClientID和ClientSecrets**

![image-20231222203153125](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272030328.png)

### 隐藏式

**隐藏式 (`implicit`)，也叫简化模式，有些 Web 应用是纯前端应用，没有后端。这时就不能用上面的方式了，必须将令牌储存在前端**

RFC 6749 规定了这种方式，允许直接向前端颁发令牌。这种方式没有授权码这个中间步骤，所以称为隐藏式。这种方式把令牌直接传给前端，是很不安全的。因此，只能用于一些安全要求不高的场景，并且令牌的有效期必须非常短，通常就是会话期间（session）有效，浏览器关掉，令牌就失效了。

![image-20231220185958063](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272030752.png)

![image-20231222203218334](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272031230.png)

> https://a.com/callback#token=ACCESS_TOKEN
> 将访问令牌包含在URL锚点中的好处：锚点在HTTP请求中不会发送到服务器，减少了泄漏令牌的风险。

### 密码式

**密码式 (`Resource Owner Password Credentials`)：如果你高度信任某个应用，RFC 6749 也允许用户把用户名和密码，直接告诉该应用。该应用就使用你的密码，申请令牌**

这种方式需要用户给出自己的用户名/密码，显然风险很大，因此只适用于其他授权方式都无法采用的情况，而且必须是用户高度信任的应用。

![image-20231220190152888](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272031424.png)

![image-20231222203240921](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272031895.png)

### 凭证式

**凭证式 (`client credentials`)：也叫客户端模式，适用于没有前端的命令行应用，即在命令行下请求令牌**

这种方式给出的令牌，是针对第三方应用的，而不是针对用户的，即有可能多个用户共享同一个令牌。

![image-20231220185958063](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272032692.png)

![image-20231222203259785](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272032779.png)

## 授权类型的选择

从开始节点进入流程，判断访问令牌拥有人是谁，如果是机器的话就选择凭证式，如果是用户时，看客户应用的类型，如果是Web应用就选择授权码模式，如果是原生App或单页应用如果是第一方应用(都属于一个企业的或同一个系统内部的)选择密码模式，如果是第三方应用，原生选择授权码，单页应用选择隐藏式

![image-20231223020052999](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202402272032885.png)


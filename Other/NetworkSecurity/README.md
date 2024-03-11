# 网络安全基础

网络安全是当今互联网时代非常重要的一个议题。在信息爆炸的今天，网络安全问题频频发生，给个人和组织带来了严重的损失和威胁，随着技术的发展和互联网的普及，我们越来越意识到网络安全已经成为一个全球性的挑战。

网络安全问题非同小可，作为网站的开发者，更应该首当其冲解决这些潜在问题。

## 病毒

![image-20230701174518205](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308241330045.png)

在2017年，一款名叫WannaCry的勒索软件席卷全球，导致全球大量计算机受到影响，得益于全球网络的发达，这款病毒从小规模很快发展至全球范围，勒索病毒是自熊猫烧香以来影响力最大的病毒之一。

> WannaCry（又叫Wanna Decryptor），一种“蠕虫式”的[勒索病毒](https://baike.baidu.com/item/勒索病毒/16623990?fromModule=lemma_inlink)软件，大小3.3MB，由不法分子利用[NSA](https://baike.baidu.com/item/NSA/1128824?fromModule=lemma_inlink)（National Security Agency，[美国国家安全局](https://baike.baidu.com/item/美国国家安全局/300052?fromModule=lemma_inlink)）泄露的危险漏洞“EternalBlue”（永恒之蓝）进行传播 [1] 。勒索病毒肆虐，俨然是一场全球性互联网灾难，给广大电脑用户造成了巨大损失。最新统计数据显示，100多个国家和地区超过10万台电脑遭到了勒索病毒攻击、感染。 [2] 勒索病毒是自熊猫烧香以来影响力最大的[病毒](https://baike.baidu.com/item/病毒/4811584?fromModule=lemma_inlink)之一。WannaCry勒索病毒全球大爆发，至少150个国家、30万名用户中招，造成损失达80亿美元，已经影响到金融，能源，医疗等众多行业，造成严重的危机管理问题。

![image-20230701175101126](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308241331135.png)

## DDOS

除了病毒在网络上传播外，还有非常恶心的DDOS攻击，这种攻击方式成本低，效率高，只要持续一段时间就能导致没有特殊防御的网站无法正常运作：

> 2022年4月，郑州警方接到报警称，某政府网站平台于近日遭受多次DDoS攻击，导致平台访问量瞬间暴增多倍，造成数十家政府网站和政务新媒体账号无法正常访问。接到报警后，郑州警方立即开展侦查调查，第一时间锁定犯罪嫌疑人张某某，仅22小时就将犯罪嫌疑人张某某抓捕到案。
>
> 经审查，张某某为某医院网络安全部技术维护人员，2022年4月2日，其个人网站被DDoS攻击，导致网站无法登陆。为减少攻击对其个人网站的影响，张某某私自将个人网站域名解析地址变更为某政府网站平台地址，“转嫁”攻击流量，致使该政府网站平台被恶意网络流量攻击，多家政府网站、政务新媒体账号不能正常使用。
>
> 目前，张某某因涉嫌破坏计算机信息系统犯罪被公安机关移送起诉，案件正在进一步侦办中。

![image-20230701175235964](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308241331220.png)

## 信息泄露

信息泄露也是非常严重的网络安全问题，我们的个人信息是我们的隐私，而在网络发达的今天，在各大APP上填写的信息很有可能会被第三方获取，从中谋取利益。

> 今年早些时候便已发现相关安全漏洞的网络安全专家称， 一个常见漏洞导致了这场有记录以来最大规模的个人数据泄露，也是中国所遭遇最大规模的网络安全事件，造成数据被公开在互联网上，供人取用。
>
> 据这些网络安全专家称，上海警方的这些记录是被安全存储的，其中包含个人姓名、身份证号码、电话号码以及警情信息，涉及近10亿中国公民的数据。他们说，但一个用于管理和访问该数据库的显示界面(dashboard)被设置在一个公开网址上，没有加设密码，任何有相对基本技术知识的人都可以轻松访问、复制或窃取库中的海量信息。
>
> 暗网情报公司Shadowbyte的创始人Vinny Troia说：“他们把这么多数据暴露在外，这太疯狂了。”这家公司专门在网上扫描搜寻存在安全漏洞的数据库，早在今年1月扫描时发现了上海警方的这个数据库。

个人信息数据一旦泄露，相当于别人可以通过网络直接定位你的住址、电话号码、各种社交账号、身份证信息甚至是机票、火车票、酒店开房信息等，做的再绝一定，如果这个酒店被人偷偷放了针孔摄像头，甚至还能根据记录查到你开房视频，这些事情光是想想都可怕，这等同于在互联网上“裸奔”。

# 测试环境搭建

为了测试网站安全性，使用基于SpringBoot 框架搭建一个采用之前的验证方式的简易网站。

**登录界面**：

> login.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登录jc银行</title>
</head>
<body>
  <form action="login" method="post">
    <label>
      用户名：
      <input name="username" type="text">
    </label>
    <label>
      密码：
      <input name="password" type="password">
    </label>
    <button type="submit">登录</button>
  </form>
  <div th:if="${status}">登录失败，用户名或密码错误！</div>
</body>
</html>
```

接着是登录之后的首页：

> home.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>jc银行 - 首页</title>
</head>
<body>

</body>
</html>
```

接着是Controller部分：

> HelloController

```java
@Controller
public class HelloController {
    //处理登录操作并跳转
    @PostMapping("/login")
    public String login(@RequestParam String username,
                        @RequestParam String password,
                        HttpSession session,
                        Model model){
        if("test".equals(username) && "123456".equals(password)) {
            session.setAttribute("login", true);
            return "redirect:/";
        } else {
            model.addAttribute("status", true);
            return "login";
        }
    }

    //处理首页或是登录界面跳转
    @GetMapping("/")
    public String index(HttpSession session){
        if(session.getAttribute("login") != null) {
            return "home";
        }else {
            return "login";
        }
    }
}
```

这样我们就可以进行简单登录了：

![image-20230825140338162](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308251405914.png)

接着在首页加一个转账操作，要求填写转账人账号名称：

```java
@ResponseBody
@PostMapping("/pay")
public JSONObject pay(@RequestParam String account,
                      HttpSession session){
    JSONObject object = new JSONObject();
    //登录之后才能转账
    if(session.getAttribute("login") != null) {
        System.out.println("转账给"+account+"成功，交易已完成！");
        object.put("success", true);
    } else {
        System.out.println("转账给"+account+"失败，用户未登录！");
        object.put("success", false);
    }
    return object;
}
```

接着我们在页面中添加一个简单的转账操作按键：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>jc银行 - 首页</title>
    <script src="https://unpkg.com/axios@1.1.2/dist/axios.min.js"></script>
</head>
<body>
    <div>
        <label>
            转账账号：
            <input type="text" id="account"/>
        </label>
        <button onclick="pay()">立即转账</button>
    </div>
</body>
</html>

<script>
function pay() {
    const account = document.getElementById("account").value
    axios.post('/pay', { account: account }, {
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
        }
    }).then(({data}) => {
        if(data.success)
            alert("转账成功")
        else
            alert("转账失败")
    })
}
</script>
```

这样就成功搭建好网络安全的测试项目：

![image-20230825140857385](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308251408043.png)

### CSRF跨站请求伪造攻击

我们时常会在QQ上收到别人发送的钓鱼网站链接，只要你在上面登陆了你的QQ账号，那么不出意外，你的号已经在别人手中了。实际上这一类网站都属于恶意网站，专门用于盗取他人信息，执行非法操作，甚至获取他人账户中的财产，非法转账等。

![image-20230701181244150](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403112002637.png)

在一开始的时候，服务端会给浏览器一个名为JSESSIONID的Cookie信息作为会话的唯一凭据，只要用户携带此Cookie访问我们的网站，那么我们就可以认定此会话属于哪个浏览器用户。因此，只要此会话的用户执行了登录操作，那么就可以随意访问个人信息等内容。

模拟这种操作，来编写一个钓鱼网站：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>坤坤炒粉放鸡精视频在线观看</title>
    <script src="https://unpkg.com/axios@1.1.2/dist/axios.min.js"></script>
</head>
<body>
<iframe name="hiddenIframe" hidden></iframe>
<form action="http://localhost:8080/mvc/pay" method="post" target="hiddenIframe">
    <input type="text" name="account" value="黑客" hidden>
    <button type="submit">点击下载全套视频</button>
</form>
</body>
</html>
```

这个页面并不是我们官方提供的页面，而是不法分子搭建的恶意网站。我们发现此页面中有一个表单，但是表单中的输入框被隐藏了，而我们看到的只有一个按钮，我们不知道这是一个表单，也不知道表单会提交给那个地址，这时整个页面就非常有迷惑性了。如果我们点击此按钮，那么整个表单的数据会以POST的形式发送给我们的服务端（会携带之前登陆我们网站的Cookie信息），但是这里很明显是另一个网站跳转，通过这样的方式，恶意网站就成功地在我们毫不知情的情况下引导我们执行了转账操作，当你发现上当受骗时，钱已经被转走了。

首先作为用户，先在正常的网站进行登录：

![image-20230825141425804](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308251414100.png)

接着假装自己上当，进入到钓鱼网站：

![image-20230825141508073](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308251415750.png)

现在我们毫不知情，如果是正常人思维的话，就会直接点击下载全套视频，恭喜，此时后台已经转账成功了，留下一脸懵逼的你：

![image-20230825141535583](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308251415130.png)

而这种构建恶意页面，引导用户访问对应网站执行操作的方式称为：**跨站请求伪造**（CSRF，Cross Site Request Forgery）

随着现在的浏览器不断发展，安全性越来越受到重视，很多浏览器都有SameSite保护机制，当用户在两个不同域名的站点操作时，默认情况下Cookie就会被自动屏蔽：

![a3421f74c3cee6b67d214809adb743a5](https://cdn.jsdelivr.net/gh/letengzz/tc2/img202403112002633.png)

SameSite是一种安全机制，旨在防止跨站点请求伪造（CSRF）攻击，它通过限制第三方Cookie的使用来实现这一目的。在Chrome浏览器中，SameSite默认为Lax，这意味着第三方Cookie只能在用户导航到与原始站点相同的站点时发送。这同样大大提升了用户的安全性，让黑客少了许多可乘之机。

### SFA会话固定攻击

这同样是利用Cookie中相同的JSESSIONID进行的攻击，会话固定攻击（Session fixation attack）是一种针对Web应用程序的安全漏洞攻击，攻击者利用这种漏洞，将一个有效的会话ID分配给用户，并诱使用户在该会话中进行操作，然后攻击者可以利用该会话ID获取用户的权限，或者通过此会话继续进行其他攻击。

简单来说，就是黑客把他的JSESSIONID直接给你，你一旦使用这个ID登录，那么在后端这个ID就被认定为已登录状态，那么也就等同于他直接进入了已登录状态，从而直接访问你账号的任意内容，执行任意操作。

攻击者通常使用以下几种方式进行会话固定攻击：

1. 会话传递：攻击者通过URL参数、表单隐藏字段、cookie等方式将会话ID传递给用户。当用户使用该会话ID登录时，攻击者就能利用该会话ID获取用户的权限。
2. 会话劫持：攻击者利用劫持用户与服务器之间的通信流量，获取到用户的会话ID，然后利用该会话ID冒充用户进行操作。
3. 会话劫持：攻击者事先获取到会话ID，并将其分配给用户，之后通过其他方式欺骗用户登录该会话。这样，攻击者就可以利用会话ID获取用户的权限。

这里我们来尝试一下第一种方案，这里我们首先用另一个浏览器访问目标网站，此时需要登录，开始之前记得先清理一下两个浏览器的缓存，否则可能无法生效：

![image-20230825142023020](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308251420616.png)

这里直接记录下这个JSESSIONID，然后将其编写到诈骗网站中，这里有一个恶意脚本，会自动将对应用户的Cookie进行替换，变成JSESSIONID值：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>冠希哥全套视频</title>
    <script src="https://unpkg.com/axios@1.1.2/dist/axios.min.js"></script>
</head>
<body>
<script>
  	//第三方网站恶意脚本，自动修改Cookie信息
    document.cookie = "JSESSIONID=A087D51C1A35DF04EA89310009A28E7C; path=/; domain=localhost"
  	//然后给你弄到原来的网站
  	location.href = 'http://localhost:8080/'
</script>
</body>
</html>
```

接着访问这个恶意网站，然后再作为用户，去正常访问目标网站进行登录操作：

![image-20230825142343165](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308251424793.png)

可以看到此时用户的浏览器JSESSIONID值为刚刚恶意网站伪造的值，现在来进行登录操作：

![image-20230825142425456](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308251424291.png)

此时我们回到一开始的浏览器，刷新之后，我们发现这个浏览器同样已经登录成功了，原理其实很简单，相当于让用户直接帮我们登录了。

当然，现在的浏览器同样有着对应的保护机制，Tomcat发送的SESSIONID默认是勾选了HttpOnly选项的，一旦被设定是无法被随意修改的，当然前提是先得正常访问一次网站才行，否则仍然存在安全隐患。

> HttpOnly是Cookie中一个属性，用于防止客户端脚本通过document.cookie属性访问Cookie，有助于保护Cookie不被跨站脚本攻击窃取或篡改。但是，HttpOnly的应用仍存在局限性，一些浏览器可以阻止客户端脚本对Cookie的读操作，但允许写操作；此外大多数浏览器仍允许通过XMLHTTP对象读取HTTP响应中的Set-Cookie头。

![image-20230702002624508](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308251425919.png)

为了彻底杜绝这个问题，登录成功之后应该重新给用户分配一个新的JSESSIONID才行。

### XSS跨站脚本攻击

CSRF跨站请求伪造攻击和SFA会话固定攻击不过都是从外部干涉，在外部无法干涉的情况下，也可以从内部击溃网站，也就是XSS跨站脚本攻击方式。

XSS（跨站脚本攻击）是一种常见的网络安全漏洞，攻击者通过在合法网站中注入恶意脚本代码来攻击用户。当用户访问受到注入攻击的页面时，恶意代码会在用户的浏览器中执行，从而导致攻击者能够窃取用户的敏感信息、诱导用户操作、甚至控制用户的账号。

XSS攻击常见的方式有三种：

1. 存储型XSS攻击：攻击者将恶意代码存储到目标网站的数据库中，当其他用户访问包含恶意代码的页面时，恶意代码会被执行。
2. 反射型XSS攻击：攻击者将恶意代码嵌入到URL中，当用户点击包含恶意代码的URL时，恶意代码会被执行。
3. DOM-based XSS攻击：攻击者利用前端JavaScript代码的漏洞，通过修改页面的DOM结构来执行恶意代码。

在一些社交网站上，用户可以自由发帖，而帖子是以富文本形式进行编辑和上传的，发送给后台的帖子往往是直接以HTML代码的形式，这个时候就会给黑客可乘之机了。

![image-20230702003831742](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308251427429.png)

正常情况下，用户发帖会向后端上传以下内容，这些是经过转换得到的正常HTML代码，方便后续直接展示：

```html
<div class="content ql-editor">
  <p>
    <strong>萨达睡觉了大数据</strong>
  </p>
  <p>撒大大撒大声地</p>
</div>
```

而现在，黑客不走常规的方式发帖，而是发送以下内容给服务端：

```html
<div class="content ql-editor">
  <p οnlοad="alert('xss')">
    <strong>萨达睡觉了大数据</strong>
  </p>
  <p>撒大大撒大声地</p>
</div>
```

可以看到`p`标签上添加了一段JS恶意脚本，黑客可以利用这种特性，获取用户的各种信息，甚至直接发送到他的后台，这样，我们的个人信息就从网站内部被泄露了。

XSS漏洞最早被发现是在1996年，由于JavaScript的出现，导致在Web应用程序中存在了一些安全问题。在1997年，高智文(Gareth Owen)也就是“XSS之父”，在他的博客中描述了一种称为“脚本注入”(script injection)的攻击技术，这就是XSS漏洞的前身。从那时起，XSS漏洞便成为了Web应用程序中的一种常见安全漏洞。

![image-20230702004827194](https://cdn.jsdelivr.net/gh/letengzz/Two-C@main/img/Java/202308251412899.png)

这种攻击一般需要前端配合后端进行防御，或者后端对前端发送的内容进行安全扫描并处理。
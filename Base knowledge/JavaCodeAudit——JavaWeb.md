# Java Code Audit JavaWeb篇

### Sql Injection for Mybatis

Sql注入的话主要是在和数据库交互时发生的，由于未对用户输入数据或者从前端接受的数据做合法性判断，或者没有使用预编译导致恶意sql语句完成执行，从而导致数据泄露

> ${	如果在select，update，delete，insert子句中基本存在sql injection 针对mybatis xml格式
>
> execute*	主要是针对使用JDBC驱动使用java代码拼接完成数据查询
>
> ​					 execute如果使用预编译prepareStatement基本很少存在注入

### Any File upload

任意文件上传主要是对文件后缀没有做严格的限定，或者限定不完全导致，文件上传在JavaWeb中的影响基本上时远小于php asp这写语言的

> org.apache.commons.fileupload
>
> java.io.File
>
> MutipartFile
>
> ​			其实以上最终都会运行到File和FileOutPutStream方法，所以基本上只要在前端请求发现了File相关的方法的调用，基本可以作为重点关注对象

解决：

使用白名单或者直接重命名文件，这样使得文件上传能得到很大的解决

### XSS

对于XSS漏洞的主要集中在前端，但是如果存在后端直接使用`out.print(用户输入数据)` 那么也是存在的，同时在JavaWeb层面还存在其他的一些标签语言，比如jsp可以直接使用Java 函数如`<%out.print(msg)%>` 还有就是EL表达式语言同样也可以的，比如和`<%= %>` 相似的`<c:out value="${user.msg}">` 也是一样的，当然还有一些其他的模板型的框架使用不当存在的Xss漏洞。

上面主要是在html页面的上的一些Xss发生，在java端也是存在的比如`ModelAndView` `ModeMap` `Model` 这些都是可以将内容传递到 前端页面的，所以需要格外注意，如果没有过滤的话基本上就是存在漏洞了。

对于存储型Xss主要是在 留言、个人信息、文章发布位置存在，如果后端没有进行过滤，那么就会很容易造成Xss，反射型Xss能够造成蠕虫攻击，所以过滤必须得严格

解决：

转义`<` `>` 符号，使得不能构成完整的js语句



### 目录穿越

目录穿越也叫目录遍历漏洞，主要原因是未对用户输入的路径中的`../` `./` 登特殊字符做严格的筛查导致，使得用户可以根据自定义的路径查看目标服务器的目录文件

一般目录穿越都伴随着任意文件下载漏洞

目录穿越主要是查找以下关键字：

>FileInputStream 	文件流
>
>File
>
>filePath
>
>path
>
>resource

解决：

检查用输入路径是否存在`../` `./` 等特殊字符，存在使用replace()替换掉，如果是业务需要，必须后端拼接硬编码值



### Url redirect

Url跳转主要是因为在接收到跳转URL时未作任何合法性检测，主要是因为目前Web服务的扩展会和很多第三方服务接触，导致了Url跳转的存在，个人觉得这和SSRF漏洞很像，但其实两者啥都不一样，然后URL重定向的话最多用在钓鱼方面，因为域名的安全性导致用户通过重定向跑到了攻击者可控网站上去，从而被钓鱼

Url跳转关键字主要有如下：

> sendRedirect()
>
> redirect
>
> ModelAndView
>
> Location

解决：

验证输入url的合法性



### 命令执行

命令执行的话主要是对用户输入的数据未作任何合法性检查，导致命令执行，在JavaWeb中存在命令执行的主要是Runtime.getRuntime.exec()和ProcessBuilder()

对于关键字搜索的话基本上就是这两个了

> exec()
>
> ProcessBuilder()

解决：

首先是使用已存在的API代替敏感操作，如果不能满足，则需屏蔽任何敏感字符



### XXE Injection

XXE就是XML的外部实体注入，主要是因为程序在解析XML输入时，并未对数据做应有的合规检查导致

主要关键字如下：

> XMLReader
>
> SAXBuilder
>
> SAXReader
>
> SAXParserFactory
>
> Disgester
>
> DocumentBuilderFactory

解决：

禁用DTD或者禁止使用外部实体



### SSRF

SSRF主要是服务器去发起请求，然后从请求地址获取相应的内容，如果为对目标或者发起用户做任何合法性验证，则会导致SSRF漏洞，对于Java中的SSRF漏洞仅支持sun.net.www.protocol中的所有协议

```
http:// https:// file:// ftp:// mailto:// netdoc://   协议
```

综上可得，Java并不像php可以支持gopher协议，从而导致攻击面也比php更少

SSRF常用接口如下：

> 1，URL 非接口
>
> 2，URLConnection.getInputStream 	
>
> 3，HttpURLConnection.getInputStream
>
> 4，HttpClient.execute
>
> 5，OKHttpClient.newCall.execute
>
> 6，Request.Get.execute
>
> 7，Request.Post.execute
>
> 8，ImageIO.read

1，2支持protocol中的所有协议，3，4，5仅支持HTTP、HTTPS协议

其实对于SSRF来说，他的所有远程请求结果都会返回到当前网站上面，所以很好判断

解决：

增加token验证

正确处理302跳转

限制请求协议为http/https

设置内网ip黑名单



### SpEL表达式注入

SpEL表达式注入主要是在spring框架上使用的和前面XSS那儿遇到的EL表达式其实都差不多。主要是因为Spring解析SpEL时如果不指定默认采用StandardEvaluationContext来解析，而StandardEvaluationContext中包含了SpEL的所有功能，从而能导致RCE

查找SpEL漏洞的主要关键字：

> org.springframework.expression.spel.standard
>
> SpelExpressionParser
>
> parseExpression
>
> expression.getValue
>
> expression.setValue



解决：

使用SimpleEvaluationContext替换默认的SpEL解析模式



### Java反序列化

Java反序列化漏洞主要时是应用程序在解析前端输入的经过序列化的代码时，会讲序列化的数据流反序列化成应用程序能够解析的数据，而在反序列化的时候程序就会解析用户构造的恶意类，从而导致RCE，而反序列化的话主要时在`readObject` 这里触发的，但也有列外比如fastjson的反序列化就并非时readObject触发

反序列化漏洞的主要关键字：

> readObject
>
> readUnshared
>
> Yaml.load
>
> XStream.fromXML
>
> ObjectMapper.readValue
>
> JSON.parseObject
>
> and so on

并非上面有所有的只要出现了就会存在反序列化漏洞，如果经过各种有限操作使得用户输入的数据仍然不可控的话，漏洞就不存在

解决：

重写resolveClass方法，自定义readObject的实现，然后再配合白名单，当然并不是所有的都有效



### SSTI模板注入

SSTI是服务端模板注入，模板引擎简单点来说就是讲用户输入的数据渲染到HTML静态页面中。就拿简单的Velocity模板引擎来说，其中#标识了脚本语句，比如`#if` 就是判断语句，然后在Velocity中，`$` 标识的是一个对象，所以基本上就可以构造一个RCE，如果存在的话

针对Velocity的话就看哪里调用了Velocity

> org.apache.Velocity

Velocity解决：

用户输入过滤或者使用无逻辑的模板引擎



### 硬编码

这种漏洞现在基本不存在了，其实就是把一些敏感值写固定罢了，比如shiro的key这种
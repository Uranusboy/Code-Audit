# Java Code Audit Framework篇（框架）

## Spring

Spring框架是结合控制反制（IOC）和面向切面（AOP）的容器框架，也是在web开发应用最广的容器之一。使用Spring开发时一般还有SpringMVC或者其他的MVC一起使用

M：模型

V：视图

C：控制层

##### CVE-2018-1260 Spring Security OAuth2 RCE

此漏洞的话主要是由于SpEL引发的，从而导致RCE



解决：

在configure配置.scopes("指定类型")



##### CVE-2018-1273 Spring Data Commons RCE

Spring data 主要是用于简化数据库访问的，并且支持云服务的开源框架。其中就包含有Commons jar包

这一个漏洞同样是由于SpEL引起的，只需要稍微对payload进行变形即可

解决：





##### CVE-2017-8046 Spring Data Rest RCE

Spring Data Rest主要是为了消除curd，所以就引入的模板，作为自己产品那不得用自己有的啊，不然别人说自己都不用自己家产品多打脸，所以SpEL又来了，害



## Struts2

Struts其实有俩产品一个是1哥哥，一个是2弟弟，弟弟就是哥哥的升级版，感觉好别扭。Struts2其本质就是一个Servlet，且是一个具有MVC概念的开源框架，但是由于后期过多的RCE爆出，渐渐落寞了

其实Struts2的漏洞基本上都是和OGNL相关的，只是触发位置不同罢了


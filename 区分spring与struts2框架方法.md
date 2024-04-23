# 区分spring与struts2框架方法

https://cloud.tencent.com/developer/article/2037499

java站点主流框架是Spring系列和struts2系列

### 方法一

在URL的反斜杠部分添加网站不存在的路径，最好是随机字符串组成的较长路径，如果返回同样的页面，则大概率是Struts2框架，如果返回404或者是报错，则大概率是Spring框架。

比如说：http://192.168.237.128:8080/struts2_032/example/aaaaaa1/bbbbbbbb2/ccccccccc3/HelloWorld.action

在Struts2框架下，完全可以正常返回页面

而Spring框架会出错

具体判断过程应参考如下步骤，一会儿讲讲具体原因：

对于如下URL：http://127.0.0.1:9999/S2_016_war/barspace/login.do

**第1步：**

在最后右边反斜杠处添加一个不存在的路径/xxxxxxxxxx/，如下所示：

http://127.0.0.1:9999/S2_016_war/barspace/xxxxxxxxx/login.do 返回与原URL相同页面，则是Struts2框架

http://127.0.0.1:9999/S2_016_war/barspace/xxxxxxxxx/login.do 返回与原URL异同页面，则是Spring框架

**第2步：**

如果两个URL均报错、或者均正常，无法区分，那么继续在前一个反斜杠处添加一个不存在的路径，如下所示：

http://127.0.0.1:9999/S2_016_war/xxxxxxxxx/barspace/login.do 返回与原URL相同页面，则是Struts2框架

http://127.0.0.1:9999/S2_016_war/xxxxxxxxx/barspace/login.do 返回与原URL异同页面，则是Spring框架

**第3步：**

如果还是没法区分，继续在前一个反斜杠处添加一个不存在的路径，如下所示：

http://127.0.0.1:9999/xxxxxxxxx/S2_016_war/barspace/login.do 返回与原URL相同页面，则是Struts2框架

http://127.0.0.1:9999/xxxxxxxxx/S2_016_war/barspace/login.do 返回与原URL异同页面，则是Spring框架

按照前面的步骤，依次添加不存在的路径，直到URL根目录为止。

#### **原理1：Struts2的URL构造**

如果了解Struts2框架的读者，一眼就能看明白这种方法的原理，对于其他读者，要想弄明白这个方法的原理，首先需要看一下Struts2的URL构造：

Struts2站点的URL路径包括四部分组成：工程名+namespace命名空间+action名+Struts2扩展名，举个例子，对于如下URL：http://127.0.0.1:9999/S2_016_war/barspace/login.action

如果在Struts2框架中，大致应该这样去分析这个URL：

/S2-016-war/部分是war包部署的工程名，也可以说是项目名、上下文等等，说法不一。

/barspace/部分是Struts2的命名空间namespace。

/login部分是Struts2的action名，指向具体处理请求的Java类。

.action部分是Struts2的扩展名，也可以定义为.do、.dw等等。

同样对于URL：http://127.0.0.1:9999/S2_016_war/barspace/login.action

如果网站应用直接部署在Tomcat根目录下，则工程名可以为空，此时URL如下所示：

http://127.0.0.1:9999/barspace/login.action

如果namespace设置为空，则URL如下：

http://127.0.0.1:9999/login.action

如果Struts2请求的扩展名为空，则URL如下：

http://127.0.0.1:9999/login（由此可见，这种URL路径也可能是Struts2框架的）

其中namespace可以配置为/barspace/spac1、action也可以配置为/login/user.action。这也是为什么有时候，我们需要在URL的每一个反斜杠前都添加一次不存在URL路径的原因，因为很难直接从URL中判断出哪一部分是namespace、哪一部分是action名。

#### **原理2：Struts2向上查找action名**

在了解了Struts2的URL构造之后，接下来看一下如下URL，在Struts2下是可以返回正常页面的：http://127.0.0.1:9999/S2_016_war/barspace/aaaaaaaaaa1/bbbbbbbb2/ccccccccc3/ddddddddddddd4/login.action

因为按照Struts2框架规则，首先会在当前路径下找action名login，如果没有找到去上一层找，还没有找到会去上上层找，一直找到应用程序的根路径为止。层层向上查找，直接找到应用程序为止。（真实的流程比这个复杂）

注：在网站的前端如果有nginx时，这种方法可能会无效，因为nginx可能会配置一些特殊URL转发，这时候就是nginx转发优先了。

### **方法2、URL添加/struts/domTT.css**

在URL的Web应用根目录下添加/struts/domTT.css，如果返回css代码，那么99%是Struts2。

注：这个domTT.css文件在网站源码文件中是找不到的，用磁盘搜索的方式搜索不到的，那为什么能访问到呢，因为这个文件在Struts2的jar包中。

原理：凡是以/struts开头的URL，Struts2的过滤器都会到struts2-core-2.0.x.jar：/org/apache/struts2/static/下面去找资源，然后读取此文件内容。

在网站根目录下添加/struts/domTT.css后访问，返回css代码。

http://192.168.237.128:8080/struts2_032/struts/domTT.css

### **方法3、404、500响应码返回信息**

输入一个不存在的路径，返回404页面，或者传入一些乱码字符，造成当前页面500响应码报错，抛出异常信息。

Struts2常用的关键字有这些：例如no action mapped、struts2、namespace、defined for action等。

Spring的报错信息如下：含有Whitelabel Error Page 关键字

### **方法4：看网站图标favicon.ico**

注意看下图的左上角，有一片小绿叶的图标，那基本上就是Spring了，而Struts2框架没有常用的favicon.ico图标。






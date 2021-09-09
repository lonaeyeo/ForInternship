##### *Servlet工作原理*

<img src="https://img-blog.csdnimg.cn/2018120522281643.gif" alt="img" style="zoom:50%;" />   

Servlet接口定义了servlet与servlet容器之间的契约。

这个契约是：servlet容器将servlet类载入内存，并产生servlet实例和调用它具体的方法。**BUT**，在一个应用程序中，**每种Servlet类型只能有一个实例**。

**用户请求**致使Servlet容器**调用Servlet的Service()方法**，并传入一个ServletRequest对象和一个ServletResponse对象。ServletRequest对象和ServletResponse对象都是由**Servlet容器（例如TomCat）**封装好的，并不需要程序员去实现，程序员可以直接使用这两个对象。

**ServletRequest中封装了当前的Http请求**，因此，开发人员不必解析和操作原始的Http数据。ServletResponse表示当前用户的Http响应，程序员只需直接操作ServletResponse对象就能把响应轻松的发回给用户。

对于每一个应用程序，Servlet容器还会创建一个ServletContext对象。这个对象中**封装了上下文（应用程序）的环境详情**。每个应用程序只有一个ServletContext。每个Servlet对象也都有一个**封装Servlet配置的ServletConfig对象**。



##### *Servlet生命周期*

```java
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;
 	
 	// 返回由Servlet容器传给init方法的ServletConfig对象。
    ServletConfig getServletConfig();
 
    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;
 
 	// 返回Servlet的一段描述，可以返回一段字符串。
    String getServletInfo();
 
    void destroy();
}
```

其中，**init**( )，**service**( )，**destroy**( )是Servlet生命周期的方法。

1. init( )，当Servlet**第一次被请求**时，Servlet容器就会init( )来**初始化出一个Servlet对象**，once and for all。我们可以利用init( )方法来执行相应的初始化工作。调用这个方法时，Servlet容器会传入**一个ServletConfig对象**进来从而对Servlet对象进行初始化。

2. service( )，**每当请求Servlet时**，Servlet容器就**会调用这个方法**。第一次请求时，Servlet容器会先调用init( )方法初始化一个Servlet对象出来，然后调用service( )方法进行工作，但在**后续的请求**中，**Servlet容器只会调用service方法**了。

3. destory( )，当**要销毁Servlet时**，Servlet容器就会调用这个方法。在卸载应用程序或者关闭Servlet容器时，就会发生这种情况，一般在这个方法中会写一些清除代码。



##### *ServletRequset接口*

Servlet容器对于接受到的**每一个Http请求**，都会**创建一个ServletRequest对象**，并**把这个对象传递给Servlet的Sevice( )方法**。



##### *ServletResponse接口*

ServletResponse接口表示**一个Servlet响应**，在**调用Servlet的Service( )方法前**，Servlet容器会**先创建一个ServletResponse对象**，并把它作为**第二个参数传给Service( )方法**。ServletResponse隐藏了向浏览器发送响应的复杂过程。

```java
void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;
```



##### *ServletContext对象*

 ServletContext对象表示Servlet应用程序。**每个Web应用程序**都**只有一个ServletContext对象**。在将一个应用程序同时部署到多个容器的分布式环境中，每台Java虚拟机上的Web应用都会有一个ServletContext对象。

通过在ServletConfig中调用getServletContext方法，也可以获得ServletContext对象。

<u>*为什么要存在一个ServletContext对象呢？*</u>

因为有了ServletContext对象，就可以**共享从应用程序中的所有资料处访问到的信息**，并且可以动态注册Web对象。前者将对象保存在ServletContext中的一个**内部Map**中。保存在ServletContext中的对象被称作属性。



##### *GenericServlet抽象类* 

前面我们编写Servlet一直是通过实现Servlet接口来编写的，但是，使用这种方法，则必须要实现Servlet接口中定义的所有的方法，即使有一些方法中没有任何东西也要去实现，并且还需要自己手动的维护ServletConfig这个对象的引用。因此，这样去实现Servlet是比较麻烦的。



##### *HttpServlet抽象类*

<img src="https://img-blog.csdn.net/20180513104757248?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" alt="img" style="zoom:50%;" /> 

HttpServlet抽象类覆盖了GenericServlet抽象类中的Service( )方法，并且添加了一个自己独有的Service(HttpServletRequest request，HttpServletResponse) 方法。

```java
public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
    HttpServletRequest request;
    HttpServletResponse response;
    try {
        request = (HttpServletRequest)req;
        response = (HttpServletResponse)res;
    } catch (ClassCastException var6) {
        throw new ServletException("non-HTTP request or response");
    }
 
    this.service(request, response);
}
```

在解析HttpServletRequest中的方法参数，并调用以下方法之一：doGet, doPost, doHead, doPut, doTrace, doOptions和doDelete。

这7种方法中，每一种方法都表示一个Http方法。doGet和doPost是最常用的。所以，如果我们需要实现具体的服务逻辑，不再需要覆盖service方法了，只需要覆盖doGet或者doPost就好了。

    总之，HttpServlet有两个特性是GenericServlet所不具备的：
    
    1.不用覆盖service方法，而是覆盖doGet或者doPost方法。在少数情况，还会覆盖其他的5个方法。
    
    2.使用的是HttpServletRequest和HttpServletResponse对象。


##### *HttpServletRequest*

<img src="https://img-blog.csdn.net/20180513130638615?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" alt="img" style="zoom: 50%;" />  

###### 通过request获得请求行

假设查询字符串为：username=zhangsan&password=123

获得客户端的请求方式：String getMethod()

获得请求的资源：

```java
String getRequestURI()

StringBuffer getRequestURL()

String getContextPath() ---web应用的名称

String getQueryString() ---- get提交url地址后的参数字符串
```



###### 通过request获得请求头

```java
long getDateHeader(String name)

String getHeader(String name)

Enumeration getHeaderNames()

Enumeration getHeaders(String name)

int getIntHeader(String name)

referer头的作用：执行该此访问的的来源，做防盗链
```

 

###### 通过request获得请求体

请求体中的内容是通过post提交的请求参数，格式是：

username=zhangsan&password=123&hobby=football&hobby=basketball

|   key    | value                |
| :------: | -------------------- |
| username | zhangsan             |
| password | 123                  |
|  hobby   | football，basketball |

以上面参数为例，通过一下方法获得请求参数：

```java
String getParameter(String name)

String[] getParameterValues(String name)

Enumeration getParameterNames( )

Map<String,String[]> getParameterMap()
```



##### *Request乱码问题的解决方法*

***在service中使用的编码解码方式默认为：ISO-8859-1编码***，但此编码并不支持中文，因此会出现乱码问题.

<img src="https://img-blog.csdn.net/20180513195355316?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" alt="img" style="zoom:50%;" />  

```java
解决post提交方式的乱码：request.setCharacterEncoding("UTF-8");

解决get提交的方式的乱码：parameter = newString(parameter.getbytes("iso8859-1"),"utf-8");
```



##### *HttpServletResponse接口*

###### HttpServletResponse内封装的响应

<img src="https://img-blog.csdn.net/20180513203525684?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" alt="img" style="zoom:50%;" /> 

######  通过Response设置响应

```java
void addCookie(Cookie var1);//给这个响应添加一个cookie
void addHeader(String var1, String var2);//给这个请求添加一个响应头
void sendRedirect(String var1) throws IOException;//发送一条响应码，讲浏览器跳转到指定的位置
void setStatus(int var1);//设置响应行的状态码
```

```java
PrintWriter getWriter()
```

获得字符流，通过字符流的write(String s)方法可以将字符串设置到**response缓冲区**中，随后Tomcat会将response缓冲区中的内容组装成Http响应返回给浏览器端。

```
ServletOutputStream getOutputStream()
```

获得字节流，通过该字节流的write(byte[] bytes)可以向response缓冲区中写入字节，再由Tomcat服务器将字节内容组成Http响应返回给浏览器。

<img src="https://img-blog.csdn.net/2018051320493643?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" alt="img" style="zoom:50%;" /> 

 ***注意：虽然response对象的getOutSream（）和getWriter（）方法都可以发送响应消息体，但是他们之间相互排斥，不可以同时使用，否则会发生异常。***

<img src="https://img-blog.csdn.net/2018051320525170?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" alt="img" style="zoom:33%;" /> 

 

##### *Response乱码*

**response缓冲区的默认编码是iso8859-1。**

<img src="https://img-blog.csdn.net/20180513205620262?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" alt="img" style="zoom:50%;" /> 

 通过更改response的编码方式为UTF-8，任然无法解决乱码问题，因为发送端服务端虽然改变了编码方式为UTF-8，**但是接收端浏览器端仍然使用GB2312编码方式解码**，还是无法还原正常的中文，因此**还需要告知浏览器端使用UTF-8编码去解码**。

<img src="https://img-blog.csdn.net/20180513210927639?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" alt="img" style="zoom:50%;" /> 

<img src="https://img-blog.csdn.net/20180513204459256?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" alt="img" style="zoom:50%;" />  

<img src="https://img-blog.csdn.net/20180513204718428?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE5NzgyMDE5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" alt="img" style="zoom:50%;" /> 

一行一行的把HTML语句给用Writer输出，早期简单的网页还能应付得住，但是随着互联网的不断发展，网站的内容和功能越来越强大，一个普通的HTML文件可能就达到好几百行，如果在采用使用Servlet去一行一行的输出HTML代码的话，将会非常的繁琐并且浪费大量的时间，且在当时，出现了PHP这种可以内嵌到HTML文件的动态语言，使得制作动态网页变得异常的简单和轻松，因此大量的程序员转上了PHP语言的道路，JAVA的份额急剧减小，当时JAVA的开发者Sun公司为了解决这个问题，也开发出了自己的动态网页生成技术，使得同样可以在HTML文件里内嵌JAVA代码，这就是现在的JSP技术。



##### *ServletContextListener*
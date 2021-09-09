[[Java\] SpringMVC工作原理之一：DispatcherServlet](https://www.cnblogs.com/tengyunhao/p/7518481.html)

## 认识

### SpringMVC和Struts的区别

#### 拦截机制不同

Struts2是**<u>类级别的拦截</u>**，每次请求就会**创建一个Action**，和Spring整合时Struts2的**ActionBean**注入作用域是原型模式prototype，然后通过setter，getter把request数据注入到属性。

Struts2中，**一个Action对应一个request，response上下文**，在接收参数时，可以通过属性接收，这说明**属性参数是让多个方法共享的**。Struts2中Action的一个方法可以对应一个url，而其类属性却被所有方法共享，这也就无法用注解或其他方式标识其所属方法了，只能设计为多例。

SpringMVC是**<u>方法级别的拦截</u>**，**一个方法对应一个Request上下文**，所以**方法**直接基本上是**独立**的，独享request，response数据。而每个方法同时又和一个url对应，**参数的传递是直接注入到方法中**的，是方法所独有的。处理结果通过ModeMap返回给框架。

在Spring整合时，SpringMVC的**Controller Bean默认单例模式Singleton**，所以默认对所有的请求，只会**创建一个Controller**，有应为没有共享的属性，所以是**线程安全的**，如果要改变默认的作用域，需要添加@Scope注解修改。

**Struts2**有自己的拦截**Interceptor机制**，**SpringMVC**这是用的是**独立的Aop方式**，这样导致Struts2的配置文件量还是比SpringMVC大。

#### 底层框架的不同

Struts2采用Filter（StrutsPrepareAndExecuteFilter）实现，SpringMVC（**DispatcherServlet**）则采用Servlet实现。Filter在容器启动之后即初始化；服务停止以后坠毁，晚于Servlet。Servlet在是在调用时初始化，先于Filter调用，服务停止后销毁。

#### 性能方面

Struts2是类级别的拦截，每次请求对应实例一个新的Action，需要加载所有的属性值注入，SpringMVC实现了零配置，由于**SpringMVC基于方法的拦截，有加载一次单例模式bean注入**。所以，SpringMVC开发效率和性能高于Struts2。

#### 配置方面

spring MVC和Spring是无缝的。从这个项目的管理和安全上也比Struts2高。



## DispatcherServlet

### 大致流程

在整个 Spring MVC 框架中，DispatcherServlet 处于核心位置，它**负责协调和组织不同组件完成请求处理**并返回响应工作。请求的大致流程：

1. Tomcat 启动，对 DispatcherServlet 进行实例化，然后调用它的 init() 方法进行初始化，在这个初始化过程中完成了：**对 web.xml 中初始化参数的加载**；**建立 WebApplicationContext (SpringMVC的IOC容器)**；进行组件的初始化；
2. 客户端发出请求，由 Tomcat 接收到这个请求，如果**匹配 DispatcherServlet 在 web.xml 中配置的映射路径**，Tomcat 就**将请求转交给 DispatcherServlet 处理**；
3. DispatcherServlet 从容器中**取出所有 HandlerMapping 实例**（每个实例对应一个 HandlerMapping 接口的实现类）**并遍历**，每个 HandlerMapping 会根据请求信息，通过自己实现类中的方式去**找到处理该请求的 Handler** (执行程序，如***Controller***中的方法)，并且**将**这个 **Handler** 与**一堆 HandlerInterceptor** (拦截器) **封装成一个 HandlerExecutionChain 对象**，一旦找到一个 HandlerMapping，Handler 就退出循环；（详情可以看[[Java\] SpringMVC工作原理之二：HandlerMapping和HandlerAdapter](https://www.cnblogs.com/tengyunhao/p/7658952.html)这篇文章）
4. DispatcherServlet 取出 HandlerAdapter 组件，根据已经找到的 Handler，再**从所有 HandlerAdapter 中找到可以处理该 Handler 的 HandlerAdapter 对象**；
5. 执行 HandlerExecutionChain 中所有拦截器的 **preHandler**() 方法，然后再**利用 HandlerAdapter 执行 Handler** ，执行完成**得到 ModelAndView**，再依次调用拦截器的 **postHandler**() 方法；
6. 利用 ViewResolver 将 ModelAndView 或是 Exception（可解析成 ModelAndView）解析成 View，然后 View 会调用 render() 方法再根据 ModelAndView 中的数据渲染出页面；
7. 最后再依次调用拦截器的 afterCompletion() 方法，这一次请求就结束了。



### DispatcherServlet 源码分析

#### HttpServletBean的init( )

DispatcherServlet 的 init() 方法在其爷爷类的 **HttpServletBean** 中实现的，它覆盖了 GenericServlet 的 init() 方法，主要作用是加载 web.xml 中 DispatcherServlet 的 <init-param> 配置，并调用子类的初始化。

```java
@Override
public final void init() throws ServletException {
    try {
        // ServletConfigPropertyValues 是静态内部类，使用 ServletConfig 获取 web.xml 中配置的参数
        PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
        // 使用 BeanWrapper 来构造 DispatcherServlet
        BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
        ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
        bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
        initBeanWrapper(bw);
        bw.setPropertyValues(pvs, true);
    } catch (BeansException ex) {}
    // 让子类实现的方法，这种在父类定义在子类实现的方式叫做模版方法模式
    initServletBean();
}
```

#### FrameworkServlet的initServletBean( ) 

在 HttpServletBean 的 init() 方法中调用了 initServletBean() 这个方法，它是在 **FrameworkServlet** 类中实现的，主要作用是建立 WebApplicationContext 容器（有时也称上下文），并加**载 SpringMVC 配置文件中定义的 Bean 到该容器**中，最后将该容器添加到 ServletContext 中。

```java
@Override
protected final void initServletBean() throws ServletException {
    try {
        // 初始化 WebApplicationContext (即SpringMVC的IOC容器)
        this.webApplicationContext = initWebApplicationContext();
        initFrameworkServlet();
    } catch (ServletException ex) {
    } catch (RuntimeException ex) {
    }
}
```

**WebApplicationContext** 继承于 **ApplicationContext** 接口，从容器中可以获取当前应用程序环境信息，**它也是 SpringMVC 的 IOC 容器**。

```java
protected WebApplicationContext initWebApplicationContext() {
    // 获取 ContextLoaderListener 初始化并注册在 ServletContext 中的根容器，即 Spring 的容器
    WebApplicationContext rootContext =
    　　　　WebApplicationContextUtils.getWebApplicationContext(getServletContext());
    WebApplicationContext wac = null;
    if (this.webApplicationContext != null) {
        // 因为 WebApplicationContext 不为空，说明该类在构造时已经将其注入
        wac = this.webApplicationContext;
        if (wac instanceof ConfigurableWebApplicationContext) {
            ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
            if (!cwac.isActive()) {
                if (cwac.getParent() == null) {
                    // 将 Spring 的容器设为 SpringMVC 容器的父容器
                    cwac.setParent(rootContext);
                }
                configureAndRefreshWebApplicationContext(cwac);
            }
        }
    }
    if (wac == null) {
    　　// 如果 WebApplicationContext 为空，则进行查找，能找到说明上下文已经在别处初始化。
    　　wac = findWebApplicationContext();
    }
    if (wac == null) {
        // 如果 WebApplicationContext 仍为空，则以 Spring 的容器为父上下文建立一个新的。
        wac = createWebApplicationContext(rootContext);
    }
    if (!this.refreshEventReceived) {
        // 模版方法，由 DispatcherServlet 实现
        onRefresh(wac);
    }
    if (this.publishContext) {
        // 发布这个 WebApplicationContext 容器到 ServletContext 中
        String attrName = getServletContextAttributeName();
        getServletContext().setAttribute(attrName, wac);
    }
    return wac;
}
```

下面是查找 WebApplicationContext 的 findWebApplicationContext() 方法代码：

```java
protected WebApplicationContext findWebApplicationContext() {
    String attrName = getContextAttribute();
    if (attrName == null) {
        return null;
    }
    // 从 ServletContext 中查找已经发布的 WebApplicationContext 容器
    WebApplicationContext wac =
    WebApplicationContextUtils.getWebApplicationContext(getServletContext(), attrName);
    if (wac == null) {
        throw new IllegalStateException("No WebApplicationContext found: initializer not registered?");
    }
    return wac;
}
```

#### DispatcherServlet 的 onRefresh() 方法

建立好 WebApplicationContext(上下文) 后，通过 onRefresh(ApplicationContext context) 方法回调，进入 DispatcherServlet 类中。onRefresh() 方法，提供 **SpringMVC 的初始化**，具体代码如下：

```java
@Override
protected void onRefresh(ApplicationContext context) {
    initStrategies(context);
}
protected void initStrategies(ApplicationContext context) {
    initMultipartResolver(context);
    initLocaleResolver(context);
    initThemeResolver(context);
    initHandlerMappings(context);
    initHandlerAdapters(context);
    initHandlerExceptionResolvers(context);
    initRequestToViewNameTranslator(context);
    initViewResolvers(context);
    initFlashMapManager(context);
}
```

在 initStrategies() 方法中进行了各个组件的初始化。

#### initHandlerMappings( ) 方法

initHandlerMappings() 方法从 **SpringMVC 的容器**及 **Spring 的容器**中**查找**所有的 HandlerMapping 实例，并把它们**放入到 handlerMappings 这个 list** 中。这个方法**并不是**对 HandlerMapping 实例的**创建**，HandlerMapping 实例是在上面 WebApplicationContext 容器初始化，即 SpringMVC 容器初始化的时候创建的。

```java
private void initHandlerMappings(ApplicationContext context) {
    this.handlerMappings = null;
    if (this.detectAllHandlerMappings) {
        // 从 SpringMVC 的 IOC 容器及 Spring 的 IOC 容器中查找 HandlerMapping 实例
        Map<String, HandlerMapping> matchingBeans =
        　　　　BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
        if (!matchingBeans.isEmpty()) {
            this.handlerMappings = new ArrayList<HandlerMapping>(matchingBeans.values());
            // 按一定顺序放置 HandlerMapping 对象
            OrderComparator.sort(this.handlerMappings);
        }
    } else {
        try {
            HandlerMapping hm = context.getBean(HANDLER_MAPPING_BEAN_NAME, HandlerMapping.class);
            this.handlerMappings = Collections.singletonList(hm);
        } catch (NoSuchBeanDefinitionException ex) {
            // Ignore, we'll add a default HandlerMapping later.
        }
    }
    // 如果没有 HandlerMapping，则加载默认的
    if (this.handlerMappings == null) {
        this.handlerMappings = getDefaultStrategies(context, HandlerMapping.class);
    }
}
```



### 处理请求

HttpServlet **提供了 doGet()、doPost()** 等方法，DispatcherServlet 中这些方法是在其父类 FrameworkServlet 中实现的，代码如下：

```java
@Override
protected final void doGet(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
    processRequest(request, response);
}
@Override
protected final void doPost(HttpServletRequest request, HttpServletResponse response)
        throws ServletException, IOException {
    processRequest(request, response);
}
```

这些方法又都调用了 processRequest() 方法，我们来看一下代码：

```java
protected final void processRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    long startTime = System.currentTimeMillis();
    Throwable failureCause = null;
    // 返回与当前线程相关联的 LocaleContext
    LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
    // 根据请求构建 LocaleContext，公开请求的语言环境为当前语言环境
    LocaleContext localeContext = buildLocaleContext(request);
    
    // 返回当前绑定到线程的 RequestAttributes
    RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
    // 根据请求构建ServletRequestAttributes
    ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);
    
    // 获取当前请求的 WebAsyncManager，如果没有找到则创建
    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);    
    asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());
    // 使 LocaleContext 和 requestAttributes 关联
    initContextHolders(request, localeContext, requestAttributes);
    try {
        // 由 DispatcherServlet 实现
        doService(request, response);
    } catch (ServletException ex) {
    } catch (IOException ex) {
    } catch (Throwable ex) {
    } finally {
        // 重置 LocaleContext 和 requestAttributes，解除关联
        resetContextHolders(request, previousLocaleContext, previousAttributes);
        if (requestAttributes != null) {
            requestAttributes.requestCompleted();
        }// 发布 ServletRequestHandlerEvent 事件
        publishRequestHandledEvent(request, startTime, failureCause);
    }
}
```

DispatcherServlet 的 doService() 方法主要是设置一些 request 属性，并调用 doDispatch() 方法进行请求分发处理，doDispatch() 方法的主要过程是通过 HandlerMapping 获取 Handler，再找到用于执行它的 HandlerAdapter，执行 Handler 后得到 ModelAndView ，ModelAndView 是连接“业务逻辑层”与“视图展示层”的桥梁，接下来就要通过 ModelAndView 获得 View，再通过它的 Model 对 View 进行渲染。doDispatch() 方法如下：

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;
    // 获取当前请求的WebAsyncManager，如果没找到则创建并与请求关联
    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    try {
        ModelAndView mv = null;
        Exception dispatchException = null;
        try {
            // 检查是否有 Multipart，有则将请求转换为 Multipart 请求
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);
            // 遍历所有的 HandlerMapping 找到与请求对应的 Handler，并将其与一堆拦截器封装到 HandlerExecution 对象中。
            mappedHandler = getHandler(processedRequest);
            if (mappedHandler == null || mappedHandler.getHandler() == null) {
                noHandlerFound(processedRequest, response);
                return;
            }
            // 遍历所有的 HandlerAdapter，找到可以处理该 Handler 的 HandlerAdapter
            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
            // 处理 last-modified 请求头 
            String method = request.getMethod();
            boolean isGet = "GET".equals(method);
            if (isGet || "HEAD".equals(method)) {
                long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                    return;
                }
            }
            // 遍历拦截器，执行它们的 preHandle() 方法
            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                return;
            }
            try {
                // 执行实际的处理程序
                mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
            } finally {
                if (asyncManager.isConcurrentHandlingStarted()) {
                    return;
                }
            }
            applyDefaultViewName(request, mv);
            // 遍历拦截器，执行它们的 postHandle() 方法
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        } catch (Exception ex) {
            dispatchException = ex;
        }
        // 处理执行结果，是一个 ModelAndView 或 Exception，然后进行渲染
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    } catch (Exception ex) {
    } catch (Error err) {
    } finally {
        if (asyncManager.isConcurrentHandlingStarted()) {
            // 遍历拦截器，执行它们的 afterCompletion() 方法  
            mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
            return;
        }
        // Clean up any resources used by a multipart request.
        if (multipartRequestParsed) {
            cleanupMultipart(processedRequest);
        }
    }
}
```



## HandlerMapping

作用是根据当前请求的找到对应的 Handler，并将 Handler（执行程序）与一堆 HandlerInterceptor（拦截器）封装到 HandlerExecutionChain 对象中。在 HandlerMapping 接口的内部只有一个方法，如下：

```java
HandlerExecutionChain getHandler(HttpServletRequest request);
```

HandlerMapping是由DispatcherServlet


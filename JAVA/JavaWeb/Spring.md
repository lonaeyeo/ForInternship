##### *为什么要使用 spring？*

###### 简介

- 目的：解决企业应用开发的复杂性；
- 功能：使用基本的JavaBean代替EJB，并提供了更多的企业应用功能；
- 范围：任何Java应用；

简单来说，Spring是一个轻量级的控制反转(IoC)和面向切面(AOP)的容器框架。

###### 轻量

从大小与开销两方面而言Spring都是轻量的。**完整的Spring框架**可以在一个大小只有**1MB多的JAR文件**里发布。并且Spring所需的处理开销也是微不足道的。此外，**Spring是非侵入式的**：典型地，**Spring应用中的对象不依赖于Spring的特定类**。

###### 控制反转

Spring通过一种称作控制反转（IoC）的技术促进了松耦合。当应用了IoC，**一个对象依赖的其它对象会通过被动的方式传递进来**，而不是这个对象自己创建或者查找依赖对象。你可以认为IoC与JNDI相反——不是对象从容器中查找依赖，而是**容器在对象初始化时**<u>不等对象请求</u>就**主动将依赖传递给它**。

###### 面向切面　　

Spring提供了面向切面编程的丰富支持，允许**通过分离应用的业务逻辑与系统级服务**（例如审计（auditing）和事务（transaction）管理）进行内聚性的开发。应用对象只实现它们应该做的——完成业务逻辑——仅此而已。它们并不负责（甚至是意识）其它的系统级关注点，例如**日志或事务支持**。

###### 容器

Spring包含并**管理应用对象的配置和生命周期**，在这个意义上它是一种容器，你可以配置你的每个bean如何被创建——基于一个可配置原型（prototype），你的bean可以创建一个单独的实例或者每次需要时都生成一个新的实例——以及它们是如何相互关联的。然而，Spring不应该被混同于传统的重量级的EJB容器，它们经常是庞大与笨重的，难以使用。

###### 框架

Spring可以将简单的组件配置、组合成为复杂的应用。在Spring中，应用对象被声明式地组合，典型地是在一个XML文件里。Spring也提供了很多基础功能（**事务管理**、**持久化框架集成**等等），将应用逻辑的开发留给了你。

所有Spring的这些特征使你能够编写更干净、更可管理、并且更易于测试的代码。它们也为Spring中的各种模块提供了基础支持



##### *Spring运行流程描述*

1. 用户向服务器发送请求，请求被Spring 前端控制DispatcherServlet捕获；

2. DispatcherServlet对**请求URL进行解析**，得到请求资源标识符（URI）。然后根据该URI，**调用HandlerMapping获得该Handler配置的所有相关的对象**（包括Handler对象以及Handler对象对应的拦截器），最后以HandlerExecutionChain对象的形式返回； 

3. DispatcherServlet **根据**获得的**Handler**，选择一个**合适的HandlerAdapter**（附注：如果成功获得HandlerAdapter后，此时将开始执行拦截器的preHandler(...)方法）；

4. 提取Request中的模型数据，**填充Handler入参**，开始**执行Handler（Controller)**。 在填充Handler的入参过程中，根据你的配置，Spring将帮你做一些额外的工作：
   - HttpMessageConveter： 将请求消息（如Json、xml等数据）转换成一个对象，将对象转换为指定的响应信息；
   - 数据转换：对请求消息进行数据转换。如String转换成Integer、Double等；
   - 数据根式化：对请求消息进行数据格式化。 如将字符串转换成格式化数字或格式化日期等；
   - 数据验证： 验证数据的有效性（长度、格式等），验证结果存储到BindingResult或Error中

5. Handler执行完成后，**向DispatcherServlet 返回一个ModelAndView对象**；

6. 根据返回的ModelAndView，选择一个适合的**ViewResolver**（必须是已经注册到Spring容器中的ViewResolver)**返回给DispatcherServlet** ；

7. ViewResolver 结合ModelAndView，来渲染视图；

8. 将渲染结果返回给客户端。



##### *Spring MVC有组件*

Spring MVC的核心组件：

1. DispatcherServlet：中央控制器，把请求给转发到具体的控制类；
2. Controller：具体处理请求的控制器；
3. HandlerMapping：映射处理器，负责**映射中央处理器转发给controller时的映射策略**；
4. ModelAndView：服务层返回的**数据和视图层的封装类**；
5. ViewResolver：视图解析器，**解析**具体的视图；
6. Interceptors ：拦截器，负责拦截我们定义的请求然后做处理工作。



##### *RequestMapping的作用*

RequestMapping是一个**用来处理请求地址映射的注解**，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。

RequestMapping注解有六个属性，下面我们把她分成三类进行说明。

###### value， method：

- value：指定请求的实际地址，指定的地址可以是URI Template 模式（后面将会说明）；
- method：指定请求的method类型， GET、POST、PUT、DELETE等；

###### consumes，produces：

- consumes：指定处理请求的提交内容类型（Content-Type），例如application/json, text/html；
- produces：指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回；

###### params，headers：

- params： 指定request中必须包含某些参数值是，才让该方法处理。
- headers：指定request中必须包含某些指定的header值，才能让该方法处理请求。


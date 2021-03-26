# SpringWeb

## 第五章 SpringWeb应用程序

### Spring MVC起步

#### 跟踪Spring MVC的请求

​		当用户在浏览器中点击链接或提交表单时，请求开始工作。

​		请求离开浏览器（1）时，会带有用户所请求的信息，至少会包含请求的URL，还可能有表单信息等。

​		请求的第一站是Spring的DispatcherServlet。Spring MVC所有的请求会通过一个前端控制器（front controller）Servlet。该Servlet会将请求委托给应用程序的其他组件来执行实际处理。在Spring MVC中，DispatcherServlet就是前端控制器。

​		DispatcherServlet任务是将请求发送给Spring MVC控制器（controller）。DispatcherServlet需要知道应该将请求发送给哪个控制器，所以DispatcherServlet会查询一个或多个处理器映射（handler mapping）（2）来确定请求的下一站在哪里处理。处理器映射器会根据请求的URL来进行决策。

​		选择合适的控制器后，DispatcherServlet会将请求发送给选中的控制器（3）。到了控制器，请求会卸下信息并等待控制器处理信息。

​		控制器在完成逻辑处理后，通常会产生一些信息，这些信息需要返回给用户并在浏览器上显示。这些信息称为模型（model）。不过这些原始信息是对用户不够友好的，因此会将消息发送给视图（view）进行格式化，通常是JSP。

​		控制器所做的最后一件事是将模型数据打包，并且标出用于渲染输出的视图名。它接下来会将请求连同模型和视图名发送回DispatcherServlet（4）。

​		这样控制器不会与特定的视图耦合，传递给DispatcherServlet的视图并不直接代表某个特定的JSP。实际上，他传递了一个逻辑名称，这个名字用来查找产生结果的真正视图。DispatcherServlet将会使用视图解析器（view resolver）（5）来将逻辑视图名匹配为一个特定的视图实现，它可能是也可能不是JSP。

​		当DispatcherServlet知道由哪个视图渲染结果，那请求的任务基本上也就完成了，最后就是视图的实现（6），在这里它交付模型数据。请求的任务也就完成了。视图使用模型数据渲染输出，这个输出会通过响应对象传递给客户端（7）。




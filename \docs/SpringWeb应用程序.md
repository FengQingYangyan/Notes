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

#### 搭建Spring MVC

##### 配置DispatcherServlet

​		继承AbstractAnnotationCongfigDIspatcherServletInitializer的任意类都会自动配置DispatcherServlet和应用上下文，Spring的应用上下文会位于应用程序的Servlet上下文之中，同时将该方案作为传统的web.xml方式的代替方案。

重写getServletMapping()，将路径映射到DispatcherServler上。

##### 两个应用上下文之间的故事

​		当DispatcherServlet启动的时候，会创建Spring应用上下文，并加载配置文件或配置类中声明的bean，重写getServletConfigClasses()，当DispatcherServlet加载应用上下文时，使用该方法中返回的配置类；

​		但是在Spring Web中，通常还会有另一个应用上下文，是由ContextLoaderListener创建的，我们希望DispatcherServlet加载Web组件的bean，如控制器、视图解析器以及处理器映射，而ContextLoaderListener加载应用中的其他bean。这些bean通常是驱动应用后端的中间层和数据层组件。

##### 启用Spring MVC

采用@EnableWebMvc注解来启用Spring MVC，但是需要解决以下问题：

1、没有配置视图解析器。这种情况下，Spring默认使用BeanNameViewResolver，这个视图解析器会根据查找ID与视图名称匹配的bean，并且查找的bean要实现View接口。

2、没有启动组件扫描，Spring只能找到显式声明在配置类中的控制器。

3、DispatcherServlet会映射到默认的Servlet，因此会处理所有的请求，包括对静态数据的请求。

为解决以上问题，需要在WebConfig中加上一些配置

1、添加@ComponentScan注解；

2、添加ViewResolver bean。更具体的讲，是InternalResourceViewResolver。

3、继承WebMvcConfigureAdapter并重写configureDefaultServlerHandling()方法。通过调用DefaultServletHandlerConfigurer的enable()方法，我们要求DispatcherServlet将对静态资源的请求转发到Servlet容器默认的Servlet上，而不是使用DispatcherServlet本身来处理此类请求。

### 编写基本的控制器

@Controller注解用来声明控制器，但是该注解只是为了表意上好一些，其实用@Component注解是相同的；

@RequestMapping(value = "/", method = GET)注解：该注解定义在方法上，其中value属性指定了这个方法所要处理的请求路径，method属性表示它所处理的请求为HTTP GET请求；

#### 测试控制器

MockMvc mock = standaloneSetup(controller).build()

传递一个controller到setup中，再调用build()方法来构建MockMvc实例，然后使用该实例来执行请求。

#### 定义类级别的请求处理

@RequestMapping("/")：定义在类上，将控制器映射在控制器上，表示会应用到该控制器的所有方法上。

@RequestMapping(method = GET)：定义在方法上，表明该方法会处理对“/”路径的请求。


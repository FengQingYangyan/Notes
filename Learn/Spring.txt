# Spring核心

## 第一章 Spring概览

### Spring容器

分为两种类型：bean工厂；应用上下文(基于BeanFactory构建)；

#### 常见应用上下文

AnnotationConfigApplicationContext：从一个或多个基于Java的配置类中加载Spring应用上下文；

AnnotationConfigWebApplicationContext：从一个或多个基于Java的配置类中加载Spring Web应用上下文；

ClassPathXmlApplicationContext：从类路径下的一个或多个XML配置文件中加载上下文定义，把应用上下文的定义文件作为类资源；

FileSystemXmlApplicationContext：从文件系统下的一个或多个XML配置文件中加载上下文定义；

XmlWebApplicationContext：从Web应用下的一个或多个XML配置文件中加载上下文定义。

### bean的生命周期

1、Spring对bean进行实例化；

2、Spring将值和bean的引用注入到bean对应的属性中；

3、如果bean实现了BeanNameAware接口，Spring将bean的ID传递给setBeanName()方法；

4、如果bean实现了BeanFactoryAware接口，Spring将调用setBeanFactory()方法，将BeanFactory容器实例传入；

5、如果bean实现了ApplicationContextAware接口，Spring将调用setApplicationContext方法，将bean所在的应用上下文的引用传入进来；

6、如果bean实现了BeanPostProcessor接口，Spring将调用它们的postProcessBeforeInitialization()方法；

7、如果bean实现了InitializingBean接口，Spring将调用它们的afterPropertiesSet()方法。类似地，如果bean使用init-method声明了初始化方法，发方法也会被调用；

8、如果bean实现了BeanPostProcessor接口，Spring将调用它们的postProcessAftrerInitialization()方法；

9、此时，bean已经准备就绪，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到该应用上下文被销毁；

10、如果bean实现了DisposableBean接口，Spring将调用它的destroy()接口方法。同样，如果bean使用destroy-method声明了销毁方法，该方法也会被调用。

### Spring的6大功能

Spring的核心容器；Spring的AOP模块；数据访问与集成；Web与远程调试；Instrumentation；测试

## 第二章 装配Bean

三种装配机制：在XML中进行显式配置；在Java中进行显式配置；隐式的bean发现机制和自动装配；

### 自动化装配

@Component注解：标明该类会作为组件类，默认采用类名首字母小写为bean的ID；

@ComponentScan注解：默认扫描与配置类相同的包，扫描包以及子包，查找带有@Component注解的类；

@ContextConfiguration注解：表明需要在相应类中加载配置；

@AutoWired注解：自动注入；

### 通过Java代码装配bean

@Configuration注解：表明这是一个配置类，该类应该包含在Spring应用上下文中如何创建bean的细节；

@Bean注解：加在方法前，告诉Spring这个方法将会返回一个对象，该对象要注册为Spring应用上下文中的bean；通过该注解实现依赖注入；

### 通过XML装配bean；

首先需要创建一个XML文件，并且以<beans>为根，装配bean的XML元素包含在spring-beans模式之中；

<bean>元素：<bean id = "xxx" class = "类的全限定名">，如果没有明确ID属性，默认ID为“类全限定名#0”；

#### 构造器注入

构造器注入bean使用<constructor-arg ref = "xxx">元素作为bean元素的子元素；c-命名空间注入<c:参数名/_参数索引-ref = "xxx">，作为bean元素一个属性；

注：使用命名空间需要在XML中进行声明；

将字面量注入到构造器中：<constructor-arg value = "">元素；

装配集合：<constructor-arg><null/></constructor-arg>采用<null>元素设为空值，在注入期可以正常使用，但是在调用时会报空指针异常；<list>作为<constructor-arg>的子元素，<value>指定集合中的每个元素，可以使用<ref>元素代替<value>；同样可以使用<set>元素；在这种情况下<constructor-arg>比c命名空间更有优势； 

#### 设置属性

<property name = "属性名" ref = "依赖类">元素为属性提供的功能与<constructor-arg>元素是一样，类似的也有p-命名空间，所遵循规则与c-命名空间相同；

将字面量注入属性中：<property name = "属性名" value = "赋值">，可以采用p-命名空间装配，遵循规则与c-命名空间相同；

### 导入和混合配置

#### 在JavaConfig中引入XML配置

通过@Import({配置类，配置类})注解导入另一个JavaConfig；

通过@ImportResourece(xml文件)导入XML配置；

#### 在XML中引入JavaConfig

<import resource = "xml文件"/>标签引入其他的xml文件；

采用<bean class = "配置类"/>标签引入JavaConfig类；

## 第四章 什么是面向切面编程

### AOP术语

通知：前置通知，后置通知，返回通知，异常通知，环绕通知；连接点；切点；切面；引入；织入

### Spring提供的4类AOP支持

基于代理的经典Spring AOP；

纯POJO切面；

@AspectJ注解驱动的切面；

注入式AspectJ切面（使用于Spring各版本）；

### 通过切点来选择连接点

切点表达式(指示器)：execution(返回类型 方法所属类.方法(任意参数)) && within(限定类)：表示在方法执行时触发，并且限定在某类下边才触发；

execution(返回类型 方法所属类.方法(任意参数)) and bean ('类名')：在制定的bean下才应用通知；

### 使用注解创建切面

为AspectJ5引入的新特性

@Aspect：表明该类是一个切面类

@Before(指示器)：在执行指示器中的方法之前要做的事；通知方法会在目标方法调用之前执行；

@After(指示器)：通知方法会在目标方法返回或抛出异常后调用；

@AfterReturn(指示器)：通知方法会在目标方法返回后调用；

@AfterThrowing(指示器)：通知方法会在目标方法泡池异常后调用；

@PointCut("切点表示式")：定义可重用的切点；

在使用Aspect注解后，并不会被解析，也不会创建将其转化为切面的代理；

#### 自动代理的两种方式

1、在JavaConfig中，使用@EnableAspectJAutoProxy注解启动自动代理功能；

2、在XML中，使用Spring aop命名空间中的<aop:aspectj-autoproxy />元素；

之后自动代理会为使用@Aspect注解的bean创建一个代理；

#### 创建环绕通知

@Around注解：表明该方法为环绕通知；但是使用该注解时，需要在注解方法上传入ProceedingJoinPoint的proceed()方法，同时需要的点去调用它；

#### 处理通知中的参数

在指示器中添加 ”&& args(指定参数)“，表明传递给切点的参数也会传递到通知中去，并且切点定义中参数与切点方法中的参数名称要保持一致，只有这样才能确保参数从切点到通知方法的转移；

#### 通过注解引入新功能

@DeclareParents注解：由三部分组成

@DeclareParents(value = "要引入的bean(哪个地方在使用该注解标注的属性)"，defaultImpl = "为引入功能提供实现的类(实现类)")

value属性：要引入的bean(哪个地方在使用该注解标注的属性)；

defaultImpl属性：为引入接口提供实现的类(实现类)；

注解标注的属性：表明要引入哪个接口；

### 在XML中声明切面

<aop:config>

​	<aop:aspect ref = "audience">

​		<aop:before pointcut = "切点表达式" method = "通知方法" />

​		<aop:after pointcut = "切点表达式" method = "通知方法" />

​	</aop:aspect>

</aop:config>

在XML中配置切点表达式：

<aop: pointcut  id = "***" expression = "切点表达式" />

配置后使用：

<aop:before point-ref = "id">

声明环绕通知：

<aop: around pointcut-ref = "id" method = "环绕通知">

为通知传递参数：与采用注解的方式仅仅区别在切点表达式使用and关键字而不是&&；

通过切面引入新功能：
<aop:aspect>

​	<aop: declare-parents

​		type-matching = "要引入的bean(哪个地方在使用该注解标注的属性)"

​		implement-interface = "表明要引入哪个接口"

​		default-impl = "实现类"

​	/>

</aop:aspect>


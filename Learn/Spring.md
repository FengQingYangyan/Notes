# Spring核心

## Spring概览

### Spring容器

分为两种类型：bean工厂；应用上下文(基于BeanFactory构建)；

#### 常见应用上下文

AnnotaionConfigApplicationContext：从一个或多个基于Java的配置类中加载Spring应用上下文；

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

6、如果bean实现了BeanPOstProcessor接口，Spring将调用它们的postProcessBeforeInitialization()方法；

7、如果bean实现了InitalizingBean接口，Spring将调用它们的afterPropertiesSer()方法。类似地，如果bean使用init-method声明了初始化方法，发方法也会被调用；

8、如果bean实现了BeanPostProcessor接口，Spring将调用它们的postProcessAtrerInitialization()方法；

9、此时，bean已经准备就绪，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到该应用上下文被销毁；

10、如果bean实现了DisposanleBean接口，Spring将调用它的destroy()接口方法。同样，如果bean使用destroy-method声明了销毁方法，该方法也会被调用。

### Spring的6大功能

Spring的核心容器；Spring的AOP模块；数据访问与集成；Web与远程调试；Instrumentation；测试

## 装配Bean

三种装配机制：在XML中进行显式配置；在Java中进行显式配置；隐士的bean发现机制和自动装配；

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






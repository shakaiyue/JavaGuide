##Spring3.x企业应用开发实站学习笔记

###url是如何映射到Servlet的？
当映射到Servlet时，URL匹配的一部分是上下文。
Web 容器接下来必须用下面描述的路径匹配步骤找出servlet来处理请求。用于映射到Servlet的路径是请求对象的请求URL减去上下文和路径参数部分。下面的URL路径映射规则按顺序使用。使用第一个匹配成功的且不会进一步尝试匹配：

- 容器将尝试找到一个请求路径到servlet路径的精确匹配。成功匹配则选择该servlet。
- 容器将递归地尝试匹配最长路径前缀。这是通过一次一个目录的遍历路径树完成的，使用‘/’字符作为
路径分隔符。最长匹配确定选择的servlet。
- 如果URL最后一部分包含一个扩展名（如.do），servlet容器将视图匹配为扩展名处理请求的Servlet。扩展名定义在最后一部分的最后一个‘.’字符之后。
- 如果前三个规则都没有产生一个servlet匹配，容器将试图为请求资源提供相关的内容。如果应用中定义了一个“default”servlet，它将被使用。许多容器提供了一种隐式的default servlet用于提供内容。容器必须使用区分大小写字符串比较匹配。

下面详细描述容器的匹配过程：

- 精确路径匹配。例子：比如servletA 的url-pattern为 /test，servletB的url-pattern为 /* ，这个时候，如果我访问的url为http://localhost/test ，这个时候容器就会先进行精确路径匹配，发现/test正好被servletA精确匹配，那么就去调用servletA，也不会去理会其他的 servlet了。 
- 最长路径匹配。例子：servletA的url-pattern为/test/*，而servletB的url-pattern为/test/a/*，此时访问http://localhost/test/a时，容器会选择路径最长的servlet来匹配，也就是这里的servletB。 
- 扩展匹配，如果url最后一段包含扩展，容器将会根据扩展选择合适的servlet。例子：servletA的url-pattern：*.action 
- 如果前面三条规则都没有找到一个servlet，容器会根据url选择对应的请求资源。如果应用定义了一个default servlet，则容器会将请求丢给default servlet（什么是default servlet？后面会讲）。 

                       |-- Context Path --|-- Servlet Path -|--Path Info--|
://www.myserver.com     /mywebapp        /helloServlet      /hello

                       |-------- Request URI  ----------------------------|
###Servlet 知识背景
Servlet的核心技术是Servlet接口。接口定义了五个方法：

- init：第一次请求servlet时会调用该方法。初始化Servlet容器，开启servlet生命周期；
- service：每次请求servlet时都会调用该方法。在这编写需要servlet完成的代码。
- destroy：销毁servlet时会调用该方法。在卸载应用程序或者是关闭servlet容器时使用，在这个部分可添加一些资源清理的代码。


**Spring:**Spring是分层的java SE/EE应用一站式的轻量式开源框架，以IOC和AOP为内核，提供了展示层SpringMVC和持久层Spring JDBC以及业务层事务管理等众多的企业级应用技术，此外，Spring还整合了众多第三方框架和类库。


>  Spring一直贯彻的理念：好的设计优于具体实现，代码应易于测试。

###Spring带来的好处：

- **方便解耦，易于开发：**通过Ioc容器，我们可以将对象之间的依赖关系交由Spring控制，避免硬编码造成的过度程序耦合。
- **AOP编程的支持：**面向切面编程。
- **声明式事务的支持：** 通过声明式方式灵活的进行事务的管理，提高开发效率和质量。
- **方便程序的测试：**
- **方便集成各种优秀的框架：**


Spring包含1400多个类；按照所属功能分类，可分为五大模块，分别是**IOC，AOP，测试框架，数据访问和集成（JDBC，ORM，事务管理），web及远程控制（MVC，portlet，service）**

- **IOC：**Spring的核心控制模块实现了IOC功能，它将类与类之间的依赖从代码中脱离出来，用配置的方式进行依赖关系描述，由IOC容器负责依赖类之间的创建、拼接、管理、获取等工作。BeanFactory实现了容器许多核心的功能。
- **AOP：**AOP是继OOP之后对编程设计思想影响最大的技术之一。AOP是进行横切逻辑编程的思想，它开拓了解决问题的思路。
- **数据访问和集成：**任何应用程序，其核心就是对数据的访问和操作。数据有很多种表现形式，如数据表，XML，消息等。每种数据形式又有不同的数据访问技术。Spring站在DAO的抽象层面，建立了一套DAO层统一的异常体系，同时将各种访问数据的检查型异常，为整合各种持久层框架提供基础，其次，Spring通过模板化技术对各种数据访问技术进行了薄层的封装，将模式化的代码隐藏起来，使数据访问的程序得到大幅简化，这样Spring就建立齐了和数据形式及访问技术无关的统一DAO层，借助AOP技术，Spring提供了声明式事务的功能。
- **web及远程控制：**Spring自己提供了一个完整的类似于Struts的MVC框架，成为Spring MVC。


dao bean注解：Repository  
service 注解：Service  
自动注入 注解：Autowired  
控制器   注解：Controller

    <!-- 配置Hibernate的局部事务管理器，使用HibernateTransactionManager类 -->
	<!-- 该类实现PlatformTransactionManager接口，是针对Hibernate的特定实现-->
	<!-- 并注入SessionFactory的引用 -->
	<bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager"
		p:sessionFactory-ref="sessionFactory"/>
		
	<tx:advice id="txAdvice" transaction-manager="transactionManager" >
		<!-- 用于配置详细的事务语义 -->
		<tx:attributes>
			<!-- 调度任务以存单为单位开启新事物 -->
			<tx:method name="reqNewPro*" propagation="REQUIRES_NEW" read-only="false" rollback-for="com.cfets.framework.exceptions.ServiceException" />
			<tx:method name="processIssueResultTimeout" read-only="true"/>
			<tx:method name="processIssueMature" read-only="true"/>
			<tx:method name="processIssueSetTimeout" read-only="true"/>
			<tx:method name="dealIssueSetTimeout" read-only="true"/>
			
			<!-- 记录日志的事务设置 -->
			<tx:method name="saveSysLog" propagation="REQUIRES_NEW" read-only="false" rollback-for="com.cfets.framework.exceptions.ServiceException" />
			<!-- 所有以'find'开头的方法是read-only的  -->
			<tx:method name="find*" read-only="true"/>
			<!-- 所有以'load'开头的方法是read-only的  -->
			<tx:method name="load*" read-only="true"/>
			<!-- 所有以'query'开头的方法是read-only的  -->
			<tx:method name="query*" read-only="true"/>
			<!-- 其他方法使用默认的事务设置 -->
			<tx:method name="*" propagation="REQUIRED" read-only="false" rollback-for="com.cfets.framework.exceptions.ServiceException" />
		</tx:attributes>
	</tx:advice>
	<tx:annotation-driven transaction-manager="transactionManager" order="200"/>
	<!-- 事务控制 拦截在service层 -->
	<aop:config>
		<!-- 配置一个切入点，匹配*Service Bean的所有方法的执行 -->
		<aop:pointcut id="myPointcut" expression="bean(*Service)" />
		
		<!-- 指定在myPointcut切入点应用txAdvice事务增强处理 -->
		<aop:advisor advice-ref="txAdvice" pointcut-ref="myPointcut"/>
	</aop:config>

###IoC容器
> Java反射技术是Spring实现依赖注入的Java底层技术。

**核心概念：**

- Inverse of Control 控制反转：
  某一接口的具体实现类的选择控制权从调用类中移除，转交第三方决定（Spring容器）；
- Dependency Injection 依赖注入。
  让调用类对某一接口的依赖关系由第三方（容器或协助类）注入，以移除调用类对某一接口实现类的依赖。

**IoC类型：**构造函数注入，属性注入，接口注入

- 构造函数注入：通过调用类的构造函数，将接口实现类以构造函数变量的形式注入到构造函数中，代码如下

 **MoAttack :构造函数注入革离的扮演者**

	public class MoAttack{
	private Geli geli;  
	//1.构造函数注入革离的扮演者
	public MoAttack(Geli geli){
		this.geli = geli;
	}
	public void cityGateAsk(){
		geli.responseAsk("墨者革离");
	}} 


- 接口注入：将调用类所有依赖注入的方法抽取到一个接口中，调用类通过实现该接口，提供相应的注入方法。先写一个接口ActorArrangeAble

`
	public interface ActorArrangeAble {
		void init_object(Geli geli);
	}

	public class MoAttack inplements ActorArrangeAble{
		private Geli geli；
		public void init_object(Geli geli){
			this.geli = geli;
		}
		public void cityGateAsk(){
		geli.responseAsk("墨者革离");
		}
	}
###类加载器ClassLoader
类加载器的工作就是寻找类的字节码文件并构造出类在JVM内部表示对象的组件。
步骤：
1.装载：查找和导入Class文件；
2.链接：执行校验，准备和解析步骤；
3.初始化：对类的静态变量，静态代码块执行初始化工作。

JVM在运行时会产生三个类加载器，分别是根装载器，EXTClassLoader（扩展类加载器），AppClassLoader（系统类装载器）。其中根装载器是用C++编写的，主要负责装载JRE的核心类库，ExtClassLoader是负责装载JRE中的JAR类包的，只有APPClassLoader是负责装载ClassPath路径下的类包的、

####资源访问利器Resource接口
假设有一个文件位于web应用类路径下，用户可以通过以下方式对这个文件资源进行访问：

- 通过FileSystemResource以文件系统绝对路径的方式进行访问；
- 通过ClassPathResource以类路径的方式进行访问；
- 通过ServletContextResource以相对于web应用根目录进行访问；

#####3.4BenFactory和ApplicationContext
Spring通过一个配置文件描述了Bean及Bean之间的依赖关系，利用Java语言的反射功能实例化Bean并建立Bean之间的依赖关系。Spring的IOC容器还提供了Bean实例缓存，生命周期管理，Bean实例代理，事件发布，资源装载等高级服务。

Bean工厂是Spring框架的最核心接口，它提供了高级IOC的配置机制。应用上下文(ApplicationContext)建立在BenFactory基础之上，提供了更多面向应用的功能，它提供了国际化支持和框架事件体系。

通常使用时，在bean.xml中配置类的信息，通过Resource加载配置xml启动IOC容器，建立一个BenFactory调用getBean即可。
若使用注解则先在bean类上  使用@Configuration  在bean方法上 使用@Bean(name = "car")
最后在使用时只需要使用 ApplicationContext ctx = new AnnotationConfigApplicationContext（Beans.class）; Car car = ctx.getBean("car",Car.class);


####在IOC容器中装配bean
####4.1Spring配置概述：
- 4.1.1：Spring容器高层视图：  
要是Spring容器启动成功，需要满足以下三个条件：
	- 1.Spring框架的的类包都放到应用程序的类路径下；
	- 2.应用程序为Spring提供了完备的Bean配置信息；
	- 3.Bean类都已经放到应用程序的类路径下；  

	Spring在启动时读取应用程序的Bean配置信息，并在Spring容器中生成一份相应的Bean配置注册表，然后根据注册表实例化Bean，装配好Bean之间的依赖关系，以供应用程序使用。Bean配置信息是Bean的元数据信息，它由以下4个方面组成：
 - 1.Bean的实现类；
 - 2.Bean的属性信息，如数据源的连接数，用户名，密码等；
 - 3.Bean的依赖关系；
 - 4.Bean的行为配置，如生命周期氛围及生命周期各过程的回调函数等。
 
	配置信息定义了Bean的实现和依赖关系，Spring容器根据各种形式的Bean配置信息在容器内部建立Bean定义注册表，然后根据注册表加载，实例化Bean，并建立Bean的依赖关系，最后将这些准备就绪的Bean放到缓存池内，供应用程序调用。

- 4.2：Bean基本配置
	- 1.id和name均可使用，Id唯一，name可重复，以最后的为准，若两者都未指定，系统自动将类名作为Bean的名称，

- 4.3：依赖注入：
	- 1.属性注入：通过setXXX()方法注入Bean的属性值或者依赖对象，由于属性注入方式具有可选择性和灵活性高的特点，因此属性注入是实际应用中最常采用的注入方式；  
	eg:具体bean类中提供set方法，在配置信息中设置<property>标签，并赋值，理论上配置信息中配置的<property>在具体实现类中需要有对应set方法。变量名前两个字母要么全大写要么全小写。
	- 2.构造函数注入：
		- 按照类型匹配入参。  
		eg：具体类中提供构造方法，配置信息中使用<constructor-arg type="java.lang.String"><value>红旗</value></constructor-arg> 给构造参数赋值，
		- 按照索引匹配入参。  
		eg：当构造函数数据类型一致时需要使用索引匹配入参。<constructor-arg index="0" value="红旗"/><constructor-arg index="1" value="中国一汽"/>	
		- 联合类型和索引入参  
		eg：<constructor-arg index="0" type="java.lang.String"><value>红旗</value></constructor-arg> 
		- 通过自身类型反射匹配入参  
	- 3.工厂方法注入:
		- 1.非静态工厂方法：即必须实例化工厂后才能调用的方法。  
		eg：有个具体factory类，里面有具体的方法构造对象，通过配置信息调用，配置信息中
		<bean id="carFactory" class="com.baobaotao.ditype.CarFactory"/>
		<bean id="car5" factory-bean="carFactory" factory-method="createHongQiCar"/>
		- 2.静态工厂方法：  
		eg：<bean id="car6" factory-bean="com.baobaotao.ditype.CarFactory" factory-method="getIntance"/>

- 4.4:注入参数详解：
- 4.5方法注入：
- 4.6bean之间的关系：
- 4.7factoryBean:Spring重要接口、可快捷建立bean，类似工厂类的作用。
- 4.8基于注解的配置：
	- 4.8.1 使用注解定义Bean：采用基于注解的配置文件时，Bean定义信息即通过在Bean实现类上标注注解实现。以下是四种注解
		- @Component("")通用注解
		- @Repository:用于对DAO实现类进行标注；
		- @Service:用于对Service实现类进行标注；
		- @Controller:用于对Controller实现类进行标注； 
	- 4.8.2 自动装配Bean
	  - 使用@Autowired进行自动注入：使用这个模式时，系统会默认按照类型匹配模式，在容器中查找匹配的Bean，当有且只有一个匹配的Bean时，Spring将其注入到@Autowired标注的变量中。
	  - 使用@Autowired的required属性：当容器中找不到对应的Bean时，这是会抛出异常，若想不抛出异常，可以使用@Autowired(required=false)进行标注
	  - 使用@Qualifier指定注入Bean的名称：当容器中有一个以上匹配的Bean时，可以通过这个限定Bean的名称  
	  eg：@Autowired  
		 @qualifer("userDao")
	  - 对类方法进行标注：可以对类的方法成员变量及方法入参注解，
	  - 对集合类进行标注：如果对类中集合类的变量或方法入参进行@Autowired标注，Spring会将容器中类型匹配的的所有Bean都自动注入进来。
	- 4.8.3 Bean的作用范围及生命过程方法  
	@Scope("prototype") 使用注解限定Bean范围

- 4.9基于Java类的配置：
	- 4.9.1 使用Java提供Bean定义信息：普通的POJO只要标注@Configuration注解，就可以为Spring容器提供Bean定义的信息了，每一个标注了@Bean的类方法都相当于提供一个Bean的定义信息。
	- 4.9.2直接通过@Configuration类启动Spring容器:Spring提供了一个AnnotationConfigApplicationContext类，它能够直接通过标注@Configuration的Java类启动Spring容器

- 4.10不同配置的比较  P：148页表
	  

####5.Spring容器高级主题
主要分析Spring容器内部结构；Spring属性编辑器；使用外部属性文件；容器事件体系；

- 5.1 Spring技术内幕：
	- 5.1.1内部工作机制： 
	
- 5.2 属性编辑器：

####6.Spring AOP基础
- 6.1 AOP概述
	- 1.1 AOP到底是什么：Aspect Oriented Programing 面向切面编程；
	- 1.2 AOP术语：
		- 连接点（JoinPoint）：程序执行的某个特定位置，类开始初始化前，类初始化后，类某个方法调用前，调用后，方法异常抛出前后，这种具有边界性质的特定点，成为“连接点”。Spring仅支持方法的连接点。连接点由两个信息确认：第一是方法表示的程序执行点；第二是相对点表示的方位，方位为方法执行前的位置。
		- 执行点：个人理解，方法本身；连接点，方法前后；切点，查询条件；
		- 切点（Pointcut）：方法就是连接点，连接点相当于数据库的记录，切点相当于查询条件，切点和连接点可以是一对多的关系。 
		- 增强（advice也叫通知）：包含一段添加到目标连接点的一段执行逻辑，还包含了用于定位连接点的方位信息。	
		- 目标对象（target）：增强逻辑的植入类。正常方法只实现正常功能，AOP增强逻辑可实现性能监测，事务管理等功能。
		- 引介（Introduction）：一种特殊的增强，为类添加一些属性或方法，包括实现他原本未实现的接口。
		- 织入（Weaving）：指将增强添加到对目标类具体连接点上的过程。Spring实现动态代理植入
		- 代理（Proxy）：一个类被AOP织入增强后，就会产生一个结果类，也叫代理类。
		- 切面（Aspect）：切面由切点和增强组成，既包括了横切逻辑的定义，又包括了连接点的定义。AOP的工作重心在于：第一，如何通过切点和增强定位到连接点上；第二，如何在增强中编写切面的代码。
	- 1.3 AOP的具体实现者：AOP工具的核心目的是将横切问题模块化，核心是连接点模型。
		- AspectJ：语言级的AOP实现，
		- AspectWerkz:基于JAVA的简单的，动态，轻量级AOP框架。他支持在运行期和类装载期织入类代码。
		- Spring AOP：使用纯JAVA实现，他不需要专门的编译过程，不需要特殊的类装载器，它是运行期通过代理方式向目标类织入增强代码。

- 6.2 AOP基础知识
	- 2.1 JDK动态代理：主要涉及java.lang.reflect中的两个类：Proxy和InvocationHandler其中InvocationHandler是一个接口，通过实现该接口定义横切逻辑，并通过反射，调用目标类的代码，动态将横切逻辑和业务逻辑编辑在一起。而Proxy利用InvocationHandler动态创建一个符合某一接口的实例，生成目标类的代理对象。 P195
	- 2.2 CGLib动态代理：使用JDK创建代理有一个限制，它只能为接口创建代理实例，这点我们可以从Proxyde的接口newProxyInstance(ClassLoader Loader, Class[] interfaces, InvocationHandler h)的方法签名中第二个参数就是需要代理实例实现的接口列表。  
	CGLib采用非常底层的字节码技术，可以为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，并顺势织入横切逻辑。 P201
	- 2.3 AOP联盟：众多开源AOP项目的联合组织，目的是为了制定一套规范描述AOP标准，定义标准的AOP接口，以便各种遵守标准的具体实现可以使用。
	- 2.4 代理小结：Spring AOP的底层就是通过使用JDK代理或者是CGLib动态代理技术为目标Bean织入横切逻辑。aop主要工作分为三点展开：
		- 1.通过切点指定在哪些类的那些方法上织入横切逻辑，
		- 2.通过advice描述横切逻辑和方法的具体植入点，
		- 3.通过advisor（切面）将Pointcut和Advice两者组装起来。
	
- 6.3 创建增强类：既包含一段添加到目标连接点的一段横切逻辑，还包含了用于定位连接点的方位信息。	
	- 3.1 增强类型：Spring支持五种类型的增强分别是一下五种，ProxyFactory是spring的代理工厂，封装了jdk代理和CGLib代理，代理接口时使用jdk，代理方法类时使用CGLib。
	- 3.2 前置增强：BeforeAdvice代表前置增强，因SPring只支持方法级增强，因此MethodBeforeAdvice是目前可用的前置增强，在目标方法之前前执行增强逻辑  
	eg:public void before(Method method, Object[] args, Object obj) throw Throwable
	- 3.3 后置增强：在目标方法执行之后执行增强逻辑：org.springframework.aop.AfterReturningAdvice；
	- 3.4 环绕增强：在目标方法执行前后执行增强逻辑：org.aopalliance.intercept.MethodInterceptor;
	- 3.5 异常增强：在目标方法抛出异常后执行增强逻辑：org.springframework.aop.ThrowsAdvice;
	最适用的场景是事务管理。  
	eg：public void afterThrowing(Method method, Object[] target, Object target, Exception ex) throws Throwable
	- 3.6 引介增强：在目标类中添加新的属性和方法：org.springframework.aop.IntroductionInterceptor；

- 6.4 创建切面：增强有一个很明显的缺陷，那就是横切逻辑会被添加到目标对象的所有方法中，这时候就需要切面编程了。切面和增强相比，多了切点的概念；
	- 4.1 切点类型：Pointcut由ClassFilter和MethodMatcher构成，前者定位特定类，后者定位特定方法。spring提供6中切点类型。
		- 静态方法切点；
		- 动态方法切点；
		- 注解切点；
		- 表达式切点；
		- 流程切点；
		- 复合切点；  		
	- 4.2 切面类型： 切面包含增强类及切点，大致可分为三类
		- Advisor(一般切面)：代表一般切面，仅包含一个advice。增强本身就是一个简单的切面，但是由于范围太大，一般不会直接使用。
		- PointcutAdvisor（切点切面）：代表具有切点的切面，它包含advice和pointcut两个类，这样就可以通过advice定位类，通过pointcut定位具体方法。
		- IntroductionAdvisor(引介切面)：代表引介切面。
		
	- 4.3 静态普通方法名匹配切面：StaticMethodMatcherPointcutAdvisor代表静态方法匹配切面(我依然认为其内部只有一个方法静态匹配器，isRuntime()返回值为false)，内部通过StaticMethodMatcherPointcut来定义切点。并通过类过滤和方法名来匹配所定义的切点。使用步骤如下：
		- ① 定义一个类继承StaticMethodMatcherPointcutAdvisor抽象类，复写matches()方法和getClassFilter()方法。
		- ② 定义一个增强类。
		- ③ 将增强类设置到切面中。 
	注意：第一步也可以通过Spring配置的方式获取ClassFilter，将其作为切面的属性，但是matches还是得自己编码。

	- 4.4 静态正则表达式方法匹配切面；
	- 4.5 流程切面：流程切面由DefaultPointcutAdvisor和ControlFlowPointcut实现，其使用和前面的静态普通方法名匹配切面不太一样，其使用步骤总结如下(按照Spring配置的方式)：
		- ① 配置一个ControlFlowPointcut的Bean，指定流程切点的类(调用其他方法的类)和方法(调用其他方法的类的方法)；
		- ② 配置一个DefaultPointcutAdvisor的Bean，指定其pointcut属性为①的Bean以及一个增强的Bean；
		- ③ 配置一个被调用类的代理类的Bean，指定其InterceptorNames属性为②中的Bean；
		- ④ 将③的Bean装配到一个调用类中，调用①中配置的方法，即可发现被调用类对应的方法添加了横切逻辑。
	- 4.6 复合切点切面：复合切点用来组合两个或两个以上的单独切点，将这些切点以交集或并集的方式组合起来。 ComposablePointcut本身就是一个起点，可以通过如下的构造器静定定义：
		- ① ComposablePointcut():构造一个匹配所有类所有方法的复合切点。 
		- ② ComposablePointcut(ClassFilter classFilter):构造一个匹配特定类所有方法的复合切点。
		- ③ ComposablePointcut(MethodMatcher methodMatcher):构造一个匹配所有类特定方法的复合切点。
		- ④ ComposablePointcut(ClassFilter classFilter, MethodMatcher methodMatcher):构造一个匹配特定类特定方法的复合切点。    
		复合切点包含三个切点交集运算的方法
		- ① ComposablePointcut intersection(ClassFilter filter)：将复合切点和一个 ClassFilter对象进行交集运算，得到一个复合切点。
		- ② ComposablePointcut intersection(MethodMatcher mm):将复合切点和一个MethodMatcher对象进行交集运算，得到一个复合切点。
		- ③ ComposablePointcut intersection(Pointcut other):将复合切点和一个切点对象进行交集运算，得到一个复合切点。  
至于并集运算方面，方法名为union，仅提供了ClassFilter和MethodMatcher两种参数形式的重载。如果需要直接对两个Pointcut进行交并集的运算，可以使用org.springframework.aop.support.Pointcuts工具类进行：
Pointcut intersection(Pointcut a, Pointcut b);
Pointcut union(Pointcut a, Pointcut b);
复合切面的编程步骤总结：
		- ① 定义一个复合切点（通过类或方法）。
		- ② 在Spring配置中配置切面，只要把复合切点和增强赋给DefaultPointcutAdvisor即可代表。
 		- ③ 创建代理

- 6.5 自动创建代理:Spring在内部使用BeanPostProcessor自动完成代理的创建工作，BeanPostProcessor只是一个接口，其实现类根据一些规则自动在容器实例化Bean时为匹配的Bean生成代理实例。这些代理分为三类：
	- ① 基于Bean配置名称规则的自动代理创建器：允许为一组特定配置名的Bean自动创建代理实例。实现类为BeanNameAutoCreator。
	- ② 基于Advisor匹配机制的自动代理创建器：它会对容器中所有的Advisor进行扫描，自动将这些切面应用到匹配的Bean中，为期创建代理实例，实现类为DefaultAdvisorAutoProxyCreator。
	- ③ 基于Bean中AspectJ注解标签的自动代理创建器：为包含AspectJ注解的Bean自动创建代理实例，实现类为AnnotationAwareAspectJAutoProxyCreator。  
	- 由于自动代理创建器都实现了BeanPostProcessor，因此可以知道自动创建代理实例发生在容器实例化Bean后期加工的时候，因此自动代理创建器有机会对满足条件的Bean自动创建代理实例。

####7.基于@AspectJ和Schema的AOP
- 7.1 Spring 对 AOP的支持：spring2.0以后对AOP功能进行了重要的增强，主要变现在以下几个方面：
	- 新增了基于Schema的配置支持，为AOP提供了专门的aop命名空间；
	- 新增了对AspectJ切点表达式语言的支持，通过注解在POJO中定义切面；
	- 无缝集成AspectJ  
	Spring AOP包括基于XML配置的AOP和基于@AspectJ注解的AOP，这两种方式虽然变现形式不同但是底层都采用的动态代理技术（JDK or CGLib）

- 7.2 JDK5.0注解知识
	- 2.1 什么是注解
		- 注解是Javadoc标签和Xdoclet标签的延伸和发展，在java5.0以上版本，我们可以自定义这些标签，并通过java语言的反射机制获取类中标注的注解，完成特定的功能。
		- 注解是代码的附属信息，它遵循一个基本原则：注解不能直接干扰程序代码的运行，无论增加或删除注解，代码都能正常运行
	- 2.2 一个简单的注解类： 注解的成员声明有以下几点限制：
		- 成员以无入参、无抛出异常的方式声明，如int value(String aa)，boolean value() throws Exception等都是非法的；
		- 可以通过default为成员指定一个默认值；
		- 成员类型是受限的，合法的类型包括原始类型及其封装类、String、Class、枚举类、注解类型、上述类型的数组类型；
		- 如果注解只有一个成员，则成员名必须取为value（），这样在使用时，可以忽略成员名和赋值等号（=）；
		- 如果注解有多个成员，只对value赋值时可以省略value和赋值等号（=），多个成员赋值用逗号隔开；
		- 所有的注解都隐式继承于Annotation，但注解不允许显式继承其他接口；
		- 如果成员是数组类型，则使用“{xxx,xxx,xxx}”将值赋给成员名。
	- 2.3 使用注解：	

- 7.3 着手使用@AspectJ
	- 3.1 一个简单的例子：
		- @Aspect--------------//①通过该注解将类标识为一个切面  
		public class TestAspectj{  
		 @Before("execution(* greetTo(..))")//②定义切点和增强类型  
			public void beforeGreeting(){  
			------//③增强的横切逻辑..............................  
		}
	}  
注：②处的切点表达式的意思是，在greetTo方法处织入增强，greetTo方法可以有任意入参和返回值。
	
 	- 3.2 通过配置使用@Aspect切面：
	 	- 如同spring使用proxyfactory将切面织入目标对象，AspectJ使用AspectJProxyFactory将切面织入目标对象。
	 	- AspectJ的自动代理创建器是
`
<bean   class="org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator"/>
`
		- 抛开自动代理创建器，使用基于Schema的aop命名空间进行配置更简单：

- 7.4 @Aspect语法基础
	- 4.1 切点表达式函数:切点表达式由关键字和操作参数组成，比如execution(* greetTo(..))，exccution就是关键字，* greetTo(..)就是操作参数。  
	spring支持9种@AspectJ切点表达式函数，根据描述对象的不同，可以分为4种类型：
		- 方法切点函数：通过描述目标类方法信息定义连接点；
		- 方法入参切点函数：通过描述目标类方法入参的信息定义连接点；
		- 目标类切点函数：通过描述目标类的代理类的信息定义连接点；
		- 代理类切点函数：通过描述目标类的代理类的信息定义连接点。
	- 4.2 在函数入参中使用通配符
		- @Aspect支持3种通配符：
			-  *    匹配任意字符，但只能匹配上下文中的一个元素；
			- ..    匹配任意字符，可以匹配上下文中的多个元素，但在表示类时，必须和*联合使用，而在表示入参时，则单独使用；
			- +   表示按类型匹配指定类的所有类，必须跟在类名后面，如com.baobaotao.Car+。表示匹配Car类及其子类；
		- @AspectJ函数按其是否支持通配符及其支持的程度，可以分为以下3类：
			- 支持所有通配符：execution()，within()；
			- 仅支持“+”通配符：args()，this()，target()；
			- 不支持通配符：@args()，@within()，@target()和@annotation()。
	- 4.3 逻辑运算符：切点表达式由切点函数组成，切点函数之间还可以进行逻辑运算，组成复合切点，spring支持以下的切点运算符：&&，||，！  
由于以上在XML中是特殊字符，spring在XML中的切点表达式函数提供and，or，not作为替代。  注：当使用not时，前面得有空格，否则报解析错误
	- 4.4 不同的增强类型：	
		- @Before    相当于BeforeAdvice，它有2个成员：
			- value 该成员用于定义切点；
			- argNames 由于无法通过反射机制获取方法入参名，所以如果在Java编译时未启用调试信息或者需要在运行期解析切点，就必须通过这个成员指定注解所标注增强方法的参数名（注意两者名字必须完全相同），多个参数名用逗号分隔。
		- @AfterReturning 后置增强，相当于AfterReturningAdvice，AfterReturning，它拥有4个成员：
			- value    该成员用于定义切点；
			- pointcut    表示切点的信息，如果显示指定point值，它将覆盖value的设置值，可以将pointcut成员看成是value的同义词；
			- returning    将目标对象方法的返回值绑定给增强的方法；
			- argNames   如前所述。
		- @Around    环绕增强，相当于MethodInterceptor，它有2个成员：
			- value    该成员用于定义切点；
			- argNames   如前所述。
		- @AfterThrowing   抛出增强，相当于ThrowsAdvice，AfterThrowing，它有4个成员：
			- value    该成员用于定义切点；
			- pointcut    表示切点的信息，如果显示指定point值，它将覆盖value的设置值，可以将pointcut成员看成是value的同义词；
			- throwing    将抛出的异常绑定到增强方法中；
			- argNames   如前所述。
		- @After： Final增强，不管是抛出异常还是正常退出，该增强都会得到执行，该增强没有对应的增强接口，可以把它看成ThrowsAdvice和AfterReturningAdvice的混合物，一般用于释放资源，它有2个成员：
			- value    该成员用于定义切点；
			- argNames   如前所述。
		- @DeclareParents    引介增强，相当于IntroductionInterceptor，它有2个成员：
			- value    该成员用于定义切点，它表示在哪个目标类上添加引介增强，即不需要使用函数表达式，直接指定全类名；
			- defaultImpl    默认的接口实现类。

- 7.5 切点函数详解：
	- @annotation(com.baobaotao.anno.NeedTest),则目标类上注解了@NeedTest的方法会调用改关键字函数切点。 
	- execution()是最常用的切点函数，其语法如下所示：  
	execution（<访问修饰符>？<返回类型><方法名>（<方法参数>）<异常>？）  
	？：表示前面的内容可选，即除了返回类型，方法名和方法参数，其他都是可选的。eg
		- 通过方法签名定义切点：
			- execution(public**(..)) ：匹配所有目标类的public方法，第一个*代表返回类型，第二个*代表方法名，..表示任意入参。
		    - execution(**To(..)) :匹配目标类以To为结尾的方法，第一个*代表返回类型，第二个代表以To为后缀的方法。
		- 通过类定义切点：
			-  execution(*com.baobaotao.Waiter.*(..))
			-  execution(*com.baobaotao.Waiter+.*(..))匹配Waiter接口及其所有实现类的方法。
		- 通过类包定义切点：
			- 在类包模式中，“*”表示包下所有类，而“..*”表示包，子孙包下的所有类。
		- 通过方法入参定义切点：
			- execution(*joke(String,int)) 匹配joke(String,int)方法，入参若不是java.lang下的包可以直接使用类名，否则必须使用全类名。如joke(java.util.List,int);
			- execution(*joke(String,*))
			- execution(*joke(String,..))
	- arg()和@args():arg()函数的入参是类名，而@argue()的入参是注解类的类名；
		- arg(com.baobaotao.Waiter)表示运行时入参是Waiter类型的方法比如addWaiter(Waiter waiter);
		- @arg()  入参是注解类的类名；
	- within():within匹配的连接点是针对目标类而言，eg:within(com.baobaotao.NativeWaiter),匹配目标类NativeWaiter的所有方法。

- 7.6 @aspectJ进阶
	- 6.1 切点复合计算：使用了切点复合运算符
	- 6.2命名切点：切点声明在增强方法处，切点声明方式为匿名切点，匿名切点只能在声明处使用。若希望在其他地方重用一个切点，我们可以通过@Pointcut注解以及切面类方法以及对切点进行命名。
	- 6.3 增强织入顺序：  
	一个连接点可以同时匹配多个切点，切点对应的增强在连接点上的织入顺序是如何安排的呢？
	这个问题要分3种情况讨论：
		- 如果增强在同一个切面类中声明，则依照增强在切面类中定义的顺序进行织入；
		- 如果增强位于不同的切面类中，且这些切面类都实现了org.springframework.core.Ordered接口，则顺序号小的先织入；
		- 如果增强位于不同的切面类中，且这些切面没有实现Ordered接口，则织入的顺序是不确定的。
		
- 7.7 基于Schema的配置
	- 用xml的方式去配置，具体看书

- 7.8 混合切面类型


###数据访问
####8.Spring对dao的支持
spring对多个持久化提供了支持，还通过Spring JDBC对JDBC API框架进行了简化，Spring定制了一个异常体系，使业务层和具体持久化解耦，借此spring实现了对多种持久化的整合，体现了“开-闭原则”的经典应用。

- 8.1 Spring的dao理念：
DAO(Data Access Object)是用于访问数据的对象。  
在UserDao中定义访问User数据对象的接口方法，业务层通过UserDao操作数据，并使用具体的持久化技术实现UserDao接口方法，这样业务层和具体的持久化技术就实现了解耦。

- 8.2 统一的异常体系:统一的异常体系是整合不同持久化框架的关键，Spring提供了一套与实现技术无关，面向DAO层次语义的异常体系并通过转换器将不同持久化技术异常转换成Spring异常。
	- 2.1 Spring的异常体系：Spring的异常体系建立在运行期异常的基础上，开发者可以根据需要捕捉感兴趣的异常。通过getCause()方法获取原始的异常信息
	- JDBC的异常转换器：  
	在org.springframework.jdbc.support包中定义了SQLExceptionTranslator接口，该接口的两个实现类SQLErrorCodeSQLExceptionTranslator和SQLStateSQLExceptionTranslator分别处理SQLException中错误码和SQL状态码的翻译工作。
	
- 8.3 统一数据访问模板

	- spring 将流程固化至模板中，并将数据访问中固定和变化的部分分开，同时保存模板类是否安全，以便多个数据访问线程共享同一模板实例。
 <img src="https://img-blog.csdnimg.cn/20181117110524350.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2E2MTQ1MjgxOTU=,size_16,color_FFFFFF,t_70" height="300" width="400">
	- 模板类：在回调中编写具体的数据操作逻辑，使用模板执行数据操作，在Spring中，这是典型的数据操作模式。

 <img src="https://img-blog.csdnimg.cn/20181117111042952.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2E2MTQ1MjgxOTU=,size_16,color_FFFFFF,t_70" height="200" width="600">

	- 支持类： 不但包含数据访问模板，还包含数据源或会话等内容。通过扩展支持类定义自己的数据访问类是最简单的数据访问方式。

- 8.4 数据源
	- 不管通过何种持久化技术，都必须拥有数据连接，在Spring中 数据连接是通过数据源获取的，
	- 数据源种类有c3p0，DBCP，JNDI数据源


#####9.Spring的事务
主要内容：  
1.事务属性值的实际意义；  
2.ThreadLocal的工作机制；  
3.Spring事务管理的体系结构；  
4.基于XML的事务配置；  
5.基于注解的事务配置；

- 9.1 数据库事务基本知识：见mysql学习文档；
- 9.2 ThreadLocal的工作机制
	- 2.1 ThreadLocal是什么：不是一个线程，而是一个线程的本地对化象，线程局部变量就是说当调用他时，ThreadLocal为每个使用该变量的线程分配一个独立的变量副本。每个多线程都可以独立改变副本，且不会影响其他线程。
	- 2.2 ThreadLocal的接口方法：
		- void set(object value)：设置当前线程的线程局部变量的值
		- public Object get():返回当前线程所对应的线程局部变量
		- public void remove():删除当前线程对应的线程局部变量
		- protected Object initialValue():返回该线程局部变量的初始值
		
	- 2.3 与Thread同步比较
		- 线程同步靠的是锁实现的 保证同一时间只有一个线程访问变量，此时变量是多个线程共享的。
		- 而ThreadLocal是为每一个线程提供独立副本，从而隔离了	多个线程对访问数据的冲突。
	- 2 .4 Spring使用ThreadLocal解决线程安全问题
- 9.3 Spring对事务管理的支持：Spring为事务管理提供了一致的编程模板，在高层次建立了统一的事务抽象，Spring提供了事务模板类TranasactionTemplate,通过这个配合使用事务回调TranasactionCallback指定具体的持久化操作就可以通过编程方式实现事务管理，而无须关注资源获取、复用、释放、事务同步与异常处理的操作。  
spring事务管理的亮点在于可以使用声明式事务管理。Spring允许通过声明方式，在IOC配置中指定事务的边界和事务属性，Spring自动在指定的事务边界上应用事务属性，
	- 3.1 事务管理关键抽象
	- 3.2 Spring的事务管理器实现类
	- 3.3 事务同步管理器
	- 3.4 事务传播行为

- 9.4 编程式的事务管理
- 9.5 使用XML配置声明式事务：  
spring的声明式事务管理是通过spring AOP实现的，通过事务的声明性信息，Spring负责将事务管理增强逻辑动态织入到业务方法相应连接点中，这些逻辑包括获取线程绑定资源、开始事务、提交/回滚事务、进行异常转换和处理等工作。
	- 具体举例说明 P328
- 9.5 使用注解配置声明式事务：  
	除了基于xml配置之外还可以通过注解的形式配置声明式事务，即@	Transactional对需要事务增强的bean接口，实现类或方法进行标注，在容器中配置基于注解的事务增强驱动，即可启用基于注解的声明式事务。
	- 使用@	Transactional注解：

#####10.Spring的事务管理难点剖析
- 10.1 DAO和事务管理的关系  
没有事务管理，DAO也可以访问数据的；
- 10.2 应用分层的迷惑
- 10.3 多线程的困惑
	- spring通过单实例化Bean简化多线程问题；
	- Spring的事务管理是通过线程相关的ThreadLocal来保存Connection实例，再结合通过IOC和AOP实现高级声明式事务功能。
- 10.4 多种联合时的事务管理：一般ORM技术的事务(session)和一个JDBC技术的连接(Connection)的封装。因此只需要采用前者的事务管理，即ORM的事务覆盖JDBC的事务管理即可。

#####11.使用Spring JDBC访问数据库：
- 11.1 基本的数据操作：
	- 更改数据；
	- 返回数据库的自增值；
	- 批量更改数据：2种
		- public int[] batchUpdate(String[] sql):多条语句组成的sql，批量执行完成。
		- int[] batchUpdate(String sql, BacthPreparedStatementSetter pss):同条sql，绑定多组参数。

	- 查询数据；
	- 查询单值数据：JDBCTemplate提供三组方法，分别用于获取int,long,Object类型的数据返回。
	- 调用存储过程；
- 11.2 blob/clob类型数据的操作：lob代表大对象数据，包括blob和clob。前者用于存储大块的二进制数据，如图片，视频等。而后者用于存储长文本数据，如论坛的帖子内容，产品的详细描述等。
	- oracle对应的是Blob和Clob。mysql对应的是Blob和LongText，mysql的长数据可直接访问，oracle是长数据需要用流的形式去访问，
	- LobCreator：统一操作lob类型数据的接口，里有set方法
	- LobHandler：为操作大二进制字段和大文本字段提供了统一的访问接口，另外可充当LobCreator的工厂类
	- 插入lob型的数据：调用LobCretor的方法
	- 以块数据方式读取Lob数据：
	- 以流数据方式读取Lob数据： 
	
- 11.3 其他类型的JDBCTemplate
- 11.4 以oo的方式访问数据库：
	- 使用MappingSqlQuery查询数据：SQLQuery是一个可重用，线程安全的类，它封装了一个SQL查询。
	- 使用sqlupate更新数据；
	- 使用storedprocedure执行存储过程；
	- sqlfunction类

#####12.Spring整合其他ORM
- 12.1 使用spring所提供的ORM整合方案，我们可以获得许多好处，
	- 方便基础设施搭建：Spring对于不同框架,首先，始终可以采用相同的方式配置数据源；其次，Spring为不同的ORM框架提供相应的FactoryBean，以初始化ORM框架的基础设施。
	- 异常封装：能够转换各种ORM异常，并将ORM专有的检查型异常转换为Spring DAO异常体系中的普通异常
	- 统一的事务管理：为不同的ORM框架提供不同的事务处理器，可用声明的方式进行事务管理；
	- 允许混合使用多种ORM框架：由于Spring在DAO，事务，资源等高级层次建立了抽象，可以让业务层对DAO具体实现的技术不敏感。因此开发者可混合多种ORM框架；
	- 方便单元测试；
- 12.2 在SPring中使用Hibernate
	- 2.1 配置sessionfactory:
		- 回忆hibernate配置方式，首先编写好对象关系映射的xxx.hbm.xml,然后通过一个Hibernate的配置文件hibernate.cfg.xml将所有的xxx.hbm.xml映射文件组装起来，最后通过new Configuration().configure("hibernate.cfg.xml")得到配置对象，然后 cfg.buildSessionFactory();得到sessionFactory。
		- spring中配置sessionFactory：首先指定数据源，然后指定Hibernate实体类映射文件，随后配置Hibernate配置属性
	- 2.2 使用HibernateTemplate
		- 基于模板类使用hibernate是最简单的方式。在spring配置文件中，首先要配置一个HIbernateTemplate Bean，她基于SessionFactory工作，随后配置事务管理器。
	- 2.3 处理LOB类型数据
		- Spirng在org.springframework.orm.hibernate3.support包中为BLOB和CLOB类型提供了几个UserTYpe的实现类。
			- BlobByteArrayType:将BLOB格式的数据映射为byte[]类型的属性；
			- BlobStringType：将BLOB格式的数据映射为String类型的数据；
			- BlobSerializableType
			- ClobStringType：将CLOB格式的数据映射为String类型的数据；
			
	- 2.4 添加Hibernate事件监听器
	- 2.5 可直接使用Hibernate原生API
	- 2.6 使用注解配置
		- spring专门提供了一个配套的annotationSessionFactoryBean，用于创建基于注解的SessionFactory
	- 2.7 事务处理
	
- 12.3 在Spring中使用myBatis
	- 对于具体的数据操作，Hibernate会自动生成SQL语句，而Ibatis则要求开发者编写具体的sql语句，在sql的开发的工作量和数据库移植性上做出了让步，为数据持久化操作提供了更大的空间。
	- 3.1 配置SqlMapClient：每一个Mybatis的应用程序都以一个SqlSessionFactory对象实例为核心。
	- 3.2 在spring中mybatis
		

####业务层及web技术
#####13.任务调度和异步调度器
- 13.1 任务调度概述
	- 通过Spring提供的一系列FactoryBean，可以很容易的创建任务调度框架的实例，此外Spring提供的几个工具类，可以将某个具体的Bean的方法直接作为被调度的任务，简化了任务的定义。
- 13.2 Quartz快速进阶
	- 2.1 Quartz是任务资源调度框架中的翘楚，它提供了强大的任务调度机制，它允许开发人员灵活的定义触发器的调度时间表，并可对触发器和任务进行关联映射，此外，Quartz提供了调度运行环境的持久化机制，可以保存并恢复调度现场，保证数据不会丢失。
	- 2.2 Quartz基础：Quartz提出了调度器，任务和触发器这三个概念。
		- job:实现改接口定义需要执行的任务。
		- Trigger：描述触发JOB执行的时间触发规则。
		- Calendar：日历特定点集合，用于一些节假日的判断
		- Scheduler：代表一个Quartz容器，Trigger和JobDetail可以注册到Scheduler中。当Trigger被触发时，对应的job就会被执行、一个job可以对应多个trigger但一个trigger只能对应一个job，
		- ThreadPool：Scheduler使用一个线程池作为任务运行的基础设施，任务通过共享线程池中的线程提高工作效率。
- 13.3 在Spring中使用Quartz：spring提供了一系列的factoryBean类。
		- 创建JobDetail
		- 创建Trigger
		- 创建Scheduler
		
##### 14 使用oxm进行对象xml映射
- 14.1 认识xml解析技术

# Spring

## 为什么Spring?

1. IOC进行松耦合与容器化管理对象，AOP进行业务和系统逻辑分开。
2. 轻量，几兆而已。
3. 事务处理能力：本地全局都可以进行统一事物管理。
4. 异常处理：可以全局捕获运行时异常。
5. MVC进行web编程提高生产效率。

## AOP的基本概念

1. 切⾯（Aspect）：官⽅的抽象定义为“⼀个关注点的模块化，这个关注点可能会横切多个对象”。
2. 连接点（Joinpoint）：程序执⾏过程中的某⼀⾏为。 
3. 通知（Advice）：“切⾯”对于某个“连接点”所产⽣的动作。 
4. 切⼊点（Pointcut）：匹配连接点的断⾔，在 AOP 中通知和⼀个切⼊点表达式关联。
5. ⽬标对象（Target Object）：被⼀个或者多个切⾯所通知的对象。 
6. AOP 代理（AOP Proxy）：在 Spring AOP 中有两种代理⽅式，JDK 动态代理和 CGLIB 代理。

## 通知类型有哪些？

1. 前置通知（Before advice）：在某连接点（JoinPoint）之前执⾏的通知，但这个通知不能阻⽌连接点前的执 ⾏。ApplicationContext 中在 aop:aspect ⾥⾯使⽤ aop:before 元素进⾏声明； 
2. 后置通知（After advice）：当某连接点退出的时候执⾏的通知（不论是正常返回还是异常退出）。 ApplicationContext 中在 aop:aspect ⾥⾯使⽤ aop:after 元素进⾏声明。 
3. 返回后通知（After return advice ：在某连接点正常完成后执⾏的通知，不包括抛出异常的情况。 ApplicationContext 中在 aop:aspect ⾥⾯使⽤ <> 元素进⾏声明。
4. 环绕通知（Around advice）：包围⼀个连接点的通知，类似 Web 中 Servlet规范中的 Filter 的 doFilter ⽅ 法。可以在⽅法的调⽤前后完成⾃定义的⾏为，也可以选择不执⾏。ApplicationContext 中在 aop:aspect ⾥ ⾯使⽤ aop:around 元素进⾏声明。

5. 抛出异常后通知（After throwing advice）：在⽅法抛出异常退出时执⾏的通知。ApplicationContext 中在 a op:aspect ⾥⾯使⽤ aop:after-throwing 元素进⾏声明

## Bean的生命周期

1. Spring 启动，查找并加载需要被 Spring 管理的 Bean，进⾏ Bean 的实例化； 
2. Bean 实例化后，对 Bean 的引⼊和值注⼊到 Bean 的属性中； 
3. 如果 Bean 实现了 BeanNameAware 接⼝的话，Spring 将 Bean 的 Id 传递给 setBeanName() ⽅法； 
4. 如果 Bean 实现了 BeanFactoryAware 接⼝的话，Spring 将调⽤ setBeanFactory() ⽅法，将 BeanFactory 容器实例传⼊； 
5. 如果 Bean 实现了 ApplicationContextAware 接⼝的话，Spring 将调⽤ Bean 的 setApplicationContext() ⽅ 法，将 Bean 所在应⽤上下⽂引⽤传⼊进来；
6. 如果 Bean 实现了 BeanPostProcessor 接⼝，Spring 就将调⽤它们的 postProcessBeforeInitialization() ⽅ 法； 
7. 如果 Bean 实现了 InitializingBean 接⼝，Spring 将调⽤它们的 afterPropertiesSet() ⽅法。类似地，如果 Bean 使⽤ init-method 声明了初始化⽅法，该⽅法也会被调⽤； 
8. 如果 Bean 实现了 BeanPostProcessor 接⼝，Spring 就将调⽤它们的 postProcessAfterInitialization() ⽅ 法； 
9. 此时，Bean 已经准备就绪，可以被应⽤程序使⽤了。它们将⼀直驻留在应⽤上下⽂中，直到应⽤上下⽂被销 毁； 
10. 如果 Bean 实现了 DisposableBean 接⼝，Spring 将调⽤它的 destory() 接⼝⽅法，同样，如果 Bean 使⽤了 destory-method 声明销毁⽅法，该⽅法也会被调⽤。

## Bean的作用域

1. singleton : 唯⼀ bean 实例，Spring 中的 bean 默认都是单例的； 
2. prototype : 每次请求都会创建⼀个新的 bean 实例； 
3. request：每⼀次 HTTP 请求都会产⽣⼀个新的 bean，该 bean 仅在当前 HTTP request 内有效； 
4. session : 每⼀次 HTTP 请求都会产⽣⼀个新的 bean，该 bean 仅在当前 HTTP session 内有效； 
5. global-session：全局 session 作⽤域，仅仅在基于 portlet 的 web 应⽤中才有意义，Spring5 已经没有了。 Portlet 是能够⽣成语义代码(例如：HTML)⽚段的⼩型 Java Web 插件。它们基于 portlet 容器，可以像 servlet ⼀样处理 HTTP 请求。但是，与 servlet 不同，每个 portlet 都有不同的会话。

## Spring的事物隔离级别

TransactionDefinition 接⼝中定义了五个表示隔离级别的常量： **TransactionDefinition.ISOLATION_DEFAULT**：使⽤后端数据库默认的隔离级别，MySQL 默认采⽤的 REPEATABLE_READ 隔离级别 Oracle 默认采⽤的 READ_COMMITTED 隔离级别； TransactionDefinition.ISOLATION_READ_UNCOMMITTED：最低的隔离级别，允许读取尚未提交的数据变更，可 能会导致脏读、幻读或不可᯿复读； **TransactionDefinition.ISOLATION_READ_COMMITTED**：允许读取并发事务已经提交的数据，可以阻⽌脏读，但 是幻读或不可᯿复读仍有可能发⽣； **TransactionDefinition.ISOLATION_REPEATABLE_READ**：对同⼀字段的多次读取结果都是⼀致的，除⾮数据是被 本身事务⾃⼰所修改，可以阻⽌脏读和不可᯿复读，但幻读仍有可能发⽣； 

**TransactionDefinition.ISOLATION_SERIALIZABLE**：最⾼的隔离级别，完全服从 ACID 的隔离级别。所有的事务依 次逐个执⾏，这样事务之间就完全不可能产⽣⼲扰，也就是说，该级别可以防⽌脏读、不可᯿复读以及幻读。但是 这将严᯿影响程序的性能。通常情况下也不会⽤到该级别.

## Spring的事物传播行为

事务传播⾏为是为了解决业务层⽅法之间互相调⽤的事务问题。当事务⽅法被另⼀个事务⽅法调⽤时，必须指定事 务应该如何传播。例如：⽅法可能继续在现有事务中运⾏，也可能开启⼀个新事务，并在⾃⼰的事务中运⾏。在 TransactionDefinition 定义中包括了如下⼏个表示传播⾏为的常量： ⽀持当前事务的情况： **TransactionDefinition.PROPAGATION_REQUIRED**：如果当前存在事务，则加⼊该事务；如果当前没有事务，则 创建⼀个新的事务； **TransactionDefinition.PROPAGATION_SUPPORTS**：如果当前存在事务，则加⼊该事务；如果当前没有事务，则 以⾮事务的⽅式继续运⾏； **TransactionDefinition.PROPAGATION_MANDATORY**：如果当前存在事务，则加⼊该事务；如果当前没有事务， 则抛出异常。 不⽀持当前事务的情况： **TransactionDefinition.PROPAGATION_REQUIRES_NEW**：创建⼀个新的事务，如果当前存在事务，则把当前事务 挂起； **TransactionDefinition.PROPAGATION_NOT_SUPPORTED**：以⾮事务⽅式运⾏，如果当前存在事务，则把当前事 务挂起。 TransactionDefinition.PROPAGATION_NEVER：以⾮事务⽅式运⾏，如果当前存在事务，则抛出异常。 其他情况： **TransactionDefinition.PROPAGATION_NESTED**：如果当前存在事务，则创建⼀个事务作为当前事务的嵌套事务 来运⾏；如果当前没有事务，则该取值等价于 TransactionDefinition.PROPAGATION_REQUIRED

## 循环依赖如何解决？

## 	MVC执行流程

1. ⽤户向服务器发送请求，请求被 Spring 前端控制Servelt DispatcherServlet 捕获； 
2. DispatcherServlet 对请求 URL 进⾏解析，得到请求资源标识符（URI）。然后根据该 URI，调⽤ HandlerMapping 获得该 Handler 配置的所有相关的对象（包括 Handler 对象以及 Handler 对象对应的拦截 器），最后以 HandlerExecutionChain 对象的形式返回； 
3. DispatcherServlet 根据获得的 Handler，选择⼀个合适的HandlerAdapter；（附注：如果成功获得 HandlerAdapter 后，此时将开始执⾏拦截器的 preHandler(...)⽅法） 
4. 提取 Request 中的模型数据，填充 Handler ⼊参，开始执⾏Handler（Controller)。在填充 Handler 的⼊参 过程中，根据你的配置，Spring 将帮你做⼀些额外的⼯作： （1）HttpMessageConveter：将请求消息（如：Json、xml 等数据）转换成⼀个对象，将对象转换为指定的响应 信息； （2）数据转换：对请求消息进⾏数据转换。如：String 转换成 Integer、Double 等； （3）数据格式化：对请求消息进⾏数据格式化。如：将字符串转换成格式化数字或格式化⽇期等； （4）数据验证：验证数据的有效性（⻓度、格式等），验证结果存储到 BindingResult 或 Error 中; 
5. Handler 执⾏完成后，向 DispatcherServlet 返回⼀个 ModelAndView 对象； 
6. 根据返回的 ModelAndView，选择⼀个适合的 ViewResolver（必须是已经注册到 Spring 容器中的 ViewResolver)返回给DispatcherServlet； 
7. ViewResolver 结合 Model 和 View，来渲染视图； 
8. 将渲染结果返回给客户端。

## MVC核心组件

1. 前端控制器 DispatcherServlet 作⽤：Spring MVC 的⼊⼝函数。接收请求，响应结果，相当于转发器，中央处理器。有了 DispatcherServlet 减 少了其它组件之间的耦合度。⽤户请求到达前端控制器，它就相当于 MVC 模式中的 C，DispatcherServlet 是整个 流程控制的中⼼，由它调⽤其它组件处理⽤户的请求，DispatcherServlet 的存在降低了组件之间的耦合性。 
2. 处理器映射器 HandlerMapping 作⽤：根据请求的 url 查找 Handler。HandlerMapping 负责根据⽤户请求找到 Handler 即处理器 （Controller），SpringMVC 提供了不同的映射器实现不同的映射⽅式，例如：配置⽂件⽅式，实现接⼝⽅式，注 解⽅式等。 
3. 处理器适配器 HandlerAdapter 作⽤：按照特定规则（HandlerAdapter 要求的规则）去执⾏ Handler。通过 HandlerAdapter 对处理器进⾏执 ⾏，这是适配器模式的应⽤，通过扩展适配器可以对更多类型的处理器进⾏执⾏。
4. 处理器 Handler 注意：编写 Handler 时按照 HandlerAdapter 的要求去做，这样适配器才可以去正确执⾏ Handler。Handler 是继 DispatcherServlet 前端控制器的后端控制器，在 DispatcherServlet 的控制下 Handler 对具体的⽤户请求进⾏处 理。由于 Handler 涉及到具体的⽤户业务请求，所以⼀般情况需要⼯程师根据业务需求开发 Handler。 
5. 视图解析器 View resolver 作⽤：进⾏视图解析，根据逻辑视图名解析成真正的视图（View ）。View Resolver 负责将处理结果⽣成 View 视 图，View Resolver ⾸先根据逻辑视图名解析成物理视图名即具体的⻚⾯地址，再⽣成 View 视图对象，最后对 View 进⾏渲染将处理结果通过⻚⾯展示给⽤户。SpringMVC 框架提供了很多的 View 视图类型，包括：jstlView、 freemarkerView、pdfView 等。⼀般情况下需要通过⻚⾯标签或⻚⾯模版技术将模型数据通过⻚⾯展示给⽤户， 需要由⼯程师根据业务需求开发具体的⻚⾯。 
6. 视图 View View 是⼀个接⼝，实现类⽀持不同的 View 类型（jsp、freemarker...）。 注意：处理器 Handler（也就是我们平常说的 Controller 控制器）以及视图层 View 都是需要我们⾃⼰⼿动 开发的。其他的⼀些组件⽐如：前端控制器 DispatcherServlet、处理器映射器 HandlerMapping、处理器适 配器 HandlerAdapter 等等都是框架提供给我们的，不需要⾃⼰⼿动开发。

## SpringBoot

> 优点：
>
> 1. 简化spring配置，自动配置。
> 2. 内嵌各种容器，可以直接jar包启动
> 3. 通过starter封装各种配置
> 4. 避免了maven冲突
> 5. 提供了监控服务
>
> 缺点：
>
> 1. 封装太多，排查问题麻烦点。

## SpringBoot自动配置原理

1. 启动加载大量自动配置类
2. 检查需要的功能有没有在自动配置中
3. 可以通过properties文件添加组件

## SpringBoot启动流程

1. SpringBoot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的 值 
2. 将这些值作为⾃动配置类导⼊容器 ， ⾃动配置类就⽣效 ， 帮我们进⾏⾃动配置⼯作； 
3. 整个J2EE的整体解决⽅案和⾃动配置都在springboot-autoconfigure的jar包中； 
4. 它会给容器中导⼊⾮常多的⾃动配置类 （xxxAutoConfiguration）, 就是给容器中导⼊这个场景需要的所有组 件 ， 并配置好这些组件 ； 5. 有了⾃动配置类 ， 免去了我们⼿动编写配置注⼊功能组件等的⼯作；

## SpringCloud



## 集成定时任务

1. 配置Quartz
2. 定义job和trigger即可
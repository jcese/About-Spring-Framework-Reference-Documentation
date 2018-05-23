# About-Spring-Framework-Reference-Documentation
第七章部分章节

 7.3 Bean overvie

1.srping loC容器至少管理一个bean，并且通过提供给容器的配置元素来创建

2.bean属性：class、name、name、scope、constructor、properties、autowiring mode、lazy-initialization mode、initialization method、destruction method

3.bean的命名：使用id或者name属性指定bean的标识符。

4.bean的实例化：

	①.容器直接通过反射调用bean的构造方法来创建一个bean，等价于java中使用new；

	②.容器通过调用static（静态）工厂方法创建一个bean，返回的对象类型可能是同一个类，也可能是不同的另一个类。

7.4 Dependencies

1.依赖注入的优势：代码更干净，并且当对象提供了它们的依赖时解耦更有效。对象不查找它的依赖，且不知道依赖的位置或类。 

2.依赖注入的两种主要方式：

	基于构造方法的依赖注入  

    public class SimpleMovieLister {
        // the SimpleMovieLister has a dependency on a MovieFinder
        private MovieFinder movieFinder;
        // a constructor so that the Spring container can inject a MovieFinder
        public SimpleMovieLister(MovieFinder movieFinder) {
            this.movieFinder = movieFinder;
        }
        // business logic that actually uses the injected MovieFinder is omitted...
    }



	基于setter方法的依赖注入 

    public class SimpleMovieLister {
    
        // the SimpleMovieLister has a dependency on the MovieFinder
        private MovieFinder movieFinder;
    
        // a setter method so that the Spring container can inject a MovieFinder
        public void setMovieFinder(MovieFinder movieFinder) {
            this.movieFinder = movieFinder;
        }
    
        // business logic that actually uses the injected MovieFinder is omitted...
    
    }

3.依赖注入的过程：

- ApplicationContext被创建并初始化描述所有bean的配置元数据。配置元数据可以是XML、Java代码或注解。
- 对于每一个bean，它的依赖以属性、构造方法参数或静态工厂方法参数的形式表示。这些依赖在bean实际创建时被提供给它。
- 每一个属性或构造方法参数都是将被设置的值的实际定义，或容器中另一个bean的引用。
- 每一个值类型的属性或构造方法参数都会从特定的形式转化为它的实际类型。默认地，Spring可以把字符串形式的值转化为所有的内置类型，比如int, long, String, boolean，等等。

4.依赖与配置：可以定义bean的属性和构造方法的参数引用其它的bean（合作者），或设置值。在XML配置中，可以使用<property/>或<constructor-arg/>的属性达到这个目的。 

定义值： <property/> 的value的属性可以为属性或构造方法可以指定为人们可读的字符串形式的值，spring的转换服务可以把这些值转换成属性或参数的实际类型。

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <!-- results in a setDriverClassName(String) call -->
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
        <property name="username" value="root"/>
        <property name="password" value="masterkaoli"/>
    </bean>
    

idref元素：是一种简单的查错方式，可以把容器中的另一个ID（字符串值-不是引用）给<constructor-arg/>或<property/>元素传递 。

    <bean id="theTargetBean" class="..."/>
    
    <bean id="theClientBean" class="...">
        <property name="targetName">
            <idref bean="theTargetBean" />
        </property>
    </bean>

    <bean id="theTargetBean" class="..." />
    
    <bean id="client" class="...">
        <property name="targetName" value="theTargetBean" />
    </bean>

上面两种形式的代码块，虽然运行结果相同，但是使用了idref标签的时候会在部署的时候去验证其引用的bean是否真实的存在；而第二个代码块并不会进行任何的验证，错误只能在bean实例化的时候才被发现。所以，若bean是一个prototype类型的bean，那么就很有可能错误在容器启动以后很久才会被发现。

5.引用其他bean

ref可以使一个bean的指定属性引用另一个bean。被引用的bean就是这个bean的一个依赖，并且会在此属性设置前被初始化。所有引用最终都是对另一个对象的引用。

    <ref bean="someBean"/>

通过parent属性可以指定对当前容器的父容器中的bean的引用。parent属性的值可能是目标bean的id属性或name属性中的一个，而且目标bean必须在当前容器的父容器中。这种引用形式主要出现在有容器继承并且想把父容器中存在的bean包装成同样名字的代理用于子容器的时候。 

    <!-- in the parent context -->
    <bean id="accountService" class="com.foo.SimpleAccountService">
        <!-- insert dependencies as required as here -->
    </bean>

6.内部bean：在<property/>或<constructor-arg/>元素内部定义的bean就是所谓的内部bean。 

内部bean不需要定义id或name，即使指定了，容器也不会使用它作为标识符。容器在创建内部bean时也会忽略其作用域标志，内部bean总是匿名的且总是随着外部bean一起创建。不可能把内部bean注入到除封闭bean以外的合作bean，也不可能单独访问它们。

有一种边界情况，可以从自定义作用域中接收销毁方法的回调，例如，对于包含在一个单例bean中的request作用域的内部bean，它的创建依赖于包含它的bean，但是销毁方法的回调允许它参与到request作用域的生命周期中。这并不是一种常见的情况，内部bean一般共享包含它的bean的作用域。

7.集合：<list/>, <set/>, <map/>和<props/> 

    <bean id="moreComplexObject" class="example.ComplexObject">
        <!-- results in a setAdminEmails(java.util.Properties) call -->
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.org</prop>
                <prop key="support">support@example.org</prop>
                <prop key="development">development@example.org</prop>
            </props>
        </property>
        <!-- results in a setSomeList(java.util.List) call -->
        <property name="someList">
            <list>
                <value>a list element followed by a reference</value>
                <ref bean="myDataSource" />
            </list>
        </property>
        <!-- results in a setSomeMap(java.util.Map) call -->
        <property name="someMap">
            <map>
                <entry key="an entry" value="just some string"/>
                <entry key ="a ref" value-ref="myDataSource"/>
            </map>
        </property>
        <!-- results in a setSomeSet(java.util.Set) call -->
        <property name="someSet">
            <set>
                <value>just some string</value>
                <ref bean="myDataSource" />
            </set>
        </property>
    </bean>

- map的键值或set的值也可以是 :bean | ref | idref | list | set | map | props | value | null 

8.集合的合并：定义一个父类型的<list/>, <set/>, <map/>或<props/>元素，然后让子类型继承它，也可以重写父集合里的值。 

    <beans>
        <bean id="parent" abstract="true" class="example.ComplexObject">
            <property name="adminEmails">
                <props>
                    <prop key="administrator">administrator@example.com</prop>
                    <prop key="support">support@example.com</prop>
                </props>
            </property>
        </bean>
        <bean id="child" parent="parent">
            <property name="adminEmails">
                <!-- the merge is specified on the child collection definition -->
                <props merge="true">
                    <prop key="sales">sales@example.com</prop>
                    <prop key="support">support@example.co.uk</prop>
                </props>
            </property>
        </bean>
    <beans>

- 子bean的adminEmails属性上的props元素的merge=true。当child被实例化时，它拥有一个adminEmails Properties集合，这个集合包含了父子两个集合adminEmails合并的结果。 
- 不能合并不同的集合类型

9.null和空字符串值：Spring把对属性的空参数作为空字符串 。

10.使用p命名空间简写XML：p命名空间使你可以使用bean元素的属性，而不是内置的<property/>元素，来描述属性值和合作的bean。 

11.使用c命名空间简写XML：与使用p命名空间简写XML类似，c命名空间是Spring3.1新引入的，允许使用内联属性配置构造方法的参数，而不用嵌套constructor-arg元素。 

12.合成属性名：可以使用合成或嵌套的属性名设置bean的属性，只要路径中除了最后的属性值的所有的组件都不为null。

13.使用depends-on：如果一个bean是另一个bean的依赖，那通常意味着这个bean被设置为另一个bean的属性。典型地，在XML配置中使用<ref/>元素来完成这件事。 

- 要表示对多个bean的依赖，可以为depends-on属性的值提供多个名字，使用逗号，空格或分号分割： 
      <bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
          <property name="manager" ref="manager" />
      </bean>
      
      <bean id="manager" class="ManagerBean" />
      <bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
- depends-on属性能够同时指定初始化时依赖和通信销毁时依赖，这只能运用在单例bean的情况中。依赖的bean在给定的bean销毁之前被销毁。因此，depends-on也能够控制关闭的顺序。 （初始化的时候依赖的对象先初始化才能注入，销毁时需要依赖的对象先销毁才能解绑） 

14.延迟初始化的bean：如果不需要预初始化，可以把bean定义为延迟初始化来阻止预初始化。延迟初始化的bean会告诉IoC容器在第一次请求到这个bean时才初始化，而不是启动的时候。 

在XML中，这种行为是通过<bean/>元素的lazy-init属性控制的，例如： 

    <bean id="lazy" class="com.foo.ExpensiveToCreateBean" lazy-init="true"/>
    <bean name="not.lazy" class="com.foo.AnotherBean"/>

15.自动装配合作者：Spring容器可以相互合作的bean间自动装配其关系。你可让让Spring通过检查ApplicationContext的内容自动为你解决bean之间的依赖。 

16.自动装配的优点：

- 自动装配将极大地减少指定属性或构造方法参数的需要（在这点上，其它机制比如本章其它小节讲解的bean模板也是有价值的）。
- 自动装配可以更新配置当你的对象进化时。例如，如果你需要为一个类添加一个依赖，那么不需要修改配置就可以自动满足。因此，自动装配在开发期间非常有用，但不否定在代码库变得更稳定的时候切换到显式的装配。

  模式         	解释                                      
  no         	默认地没有自动自动装配。bean的引用必须通过ref元素定义。对于大型部署，不推荐更改默认设置，因为显式地指定合作者能够更好地控制且更清晰。在一定程度上，这也记录了系统的结构。
  byName     	按属性名称自动装配。Spring为待装配的属性寻找同名的bean。例如，如果一个bean被设置为按属性名称自动装配，且它包含一个属性叫master（亦即，它有setMaster(…)方法），Spring会找到一个名叫master的bean并把它设置到这个属性中。
  byType     	按属性的类型自动装配，如果这个属性的类型在容器中只存在一个bean。如果多于一个，则会抛出异常，这意味着不能为那个bean使用按类型装配。如果一个都没有，则什么事都不会发生，这个属性不会被装配。
  constructor	与按类型装配类似，只不过用于构造方法的参数。如果这个构造方法的参数类型在容器中不存在明确的一个bean，将会抛出异常。

17.自动装配的局限性和缺点：

- 在property和constructor-arg上显式设置的依赖总是覆盖自动装配。而且，不能自动装配所谓的简单属性，如基本类型、String和Classes（也包括简单属性的数组）。这是设计层面的局限性。
- 自动装配没有显式装配精确。如上表所述，尽管Spring很小心地避免了模棱两可的猜测，但其管理的对象之间的关系并不会被明确地记录。
- 从Spring容器生成文档的工具可能找不到装配信息。
- 容器中的多个bean定义可能会匹配setter方法或构造方法参数指定的类型。对于数组、集合或Map，这未必是个问题。但是，对于那些需要单个值的依赖，这种歧义并不能随意解决。如果没有各自不同的bean定义，将会抛出异常。

18.避免bean自动装配

在单个bean上，可以避免自动装配。在xml形式中，设置<bean>元素的autowire-candidate属性为false，容器就不会让这个bean进入自动装配的架构中（包括注解形式的配置，比如@Autowired）。

也可以通过定义bean名称的匹配模式避免自动装配。顶级元素<beans>的属性default-autowire-candidates可以接受一个或多个模式。例如，为了限制以Repository结尾的bean自动装配，提供一个值Repository*即可。如果需要提供多个模式，请以逗号分隔。bean定义上的autowire-candidate**属性优先级要更高一点，对于这类bean，匹配模式将不起作用。

19.方法注入与查找方法注入

查找方法注入：

- 为了能使动态的子类有效，被继承的类不能是final，且被重写的方法也不能是final。
- 单元测试一个具有抽象方法的类时，需要手动继承此类并重写其抽象方法。
- 组件扫描的具体方法也需要具体类。
- 一项关键限制是查找方法不能使用工厂方法和配置类中的@Bean方法，因为容器不会在运行时创建一个子类及其实例。
- 最后，方法注入的目标对象不能被序列化。

7.9 基于注解的容器配置

1.@Required：从Spring 4.3开始，如果目标bean只有一个构造方法，则@Autowired的构造方法不再是必要的。如果有多个构造方法，那么至少一个必须被注解以便告诉容器使用哪个 。

其可使用的情况分为：①、可以在setter方法上使用 。

				       ②、可以应用在具有任意名字和多个参数的方法上 

				       ③、可以应用在字段上，甚至可以与构造方法上混用 

                                       ④、可以从ApplicationContext中提供特定类型的所有bean，只要添加这个注解在一个那种类型的数组字段或方法上 

                                       ⑤、适用于集合类型 

				       ⑥、甚至Map也可以被自动装配，只要key的类型是String就可以。Map的value将包含所有的特定类型的bean，并且key会包含这些bean的名字

				       ⑦、可以把@Autowired用在那些著名的可解析的依赖的接口上：BeanFactory, ApplicationContext, Environment, ResourceLoader, ApplicationEventPublisher, 以及MessageSource。这些接口和它们扩展的接口，比如ConfigurableApplicationContext或ResourcePatternResolver，会自动解析，不需要特殊设置。 



** @Autowired， @Inject， @Resource和@Value注解都是被Spring的BeanPostProcessor处理的，这反过来意味着我们不能使用自己的BeanPostProcessor或BeanFactoryPostProcessor类型来处理这些注解。这些类型必须通过XML或使用Spring的@Bean方法显式地装配。 

2.使用@Primary微调基于注解的自动装配

因为基于类型的自动装配可能会导致多个候选者，所以对这种过程通常需要更多的控制。一种方式是使用Spring的@Primary注解。它表示如果存在多个候选者且另一个bean只需要一个特定类型的bean依赖时，就使用标记了@Primary注解的那个依赖。如果只有一个候选者那就直接使用那个候选者即可。 

3.使用限定符微调基于注解的自动装配

当一个类型有几个实例时使用@Primary是一种有效的方式。当需要对选择过程做更多的控制时，那就需要用到Spring的@Qualifier注解了。为指定的参数绑定一个限定的值，可以缩小类型匹配的范围，使用这种方式，这个值可以是一个很普通的描述性的值。

3.使用泛型作为自动装配限定符：除了@Qualifier注解，也可以使用Java的泛型类型作为一种显式的限定 。

4.CustomAutowireConfigurer：是一个BeanFactoryPostProcessor，它可以注册自定义的限定符注解类型，甚至它们没有被Spring的@Qualifier注解所注解。 

AutowireCandidateResolver通过以下方式决定了自动装配的候选者：

- 每个bean定义的auto-candidate值
- <beans/>元素上定义的default-autowire-candidates模式
- 使用了@Qualifier注解或任何在CustomAutowireConfigurer中注册的自定义注解

当多个bean被限定为候选者时，主要决定因素如下：如果这些候选者中有一个bean定义上明确地设置了primay属性为true，那么它将被选择。

5.使用限定符微调基于注解的自动装配

当一个类型有几个实例时使用@Primary是一种有效的方式。当需要对选择过程做更多的控制时，那就需要用到Spring的@Qualifier注解了。为指定的参数绑定一个限定的值，可以缩小类型匹配的范围，使用这种方式，这个值可以是一个很普通的描述性的值 。

6.CustomAutowireConfigurer：CustomAutowireConfigurer是一个BeanFactoryPostProcessor，它可以注册自定义的限定符注解类型，甚至它们没有被Spring的@Qualifier注解所注解。 

7.@Resource：Spring也支持使用JSR-250的@Resource注解在字段和setter方法上进行注入。这在Java EE 5和6中是一种通用的模式，例如，在JSF 1.2管理的bean或JAX-WS 2.0终端。Spring也同样支持这种模式来管理对象。@Resource拥有一个name属性，默认地，Spring会把这个name属性的值解释为将要注入的bean的名字。换句话说，它按照名字的语法进行注入。

8.@PostConstruct和@PreDestroy

CommonAnnotationBeanPostProcessor不仅能够识别到@Resource注解，还能识别到JSR-250的生命周期注解。这是在Spring 2.5引入的，这项支持为初始化回调及销毁回调又提供了一种替代方案。CommonAnnotationBeanPostProcessor是在ApplicationContext中注册的，因此带有这些注解的方法会与Spring自身的生命周期接口方法或显式声明的回调方法在同样调用。 

7.11 使用JSR 330标准注解

1.使用@Inject和@Named依赖注入

与@Autowired一样，可以在字段级别、方法级别或构造参数级别使用@Inject。另外，也可以定义注入点为Provider，以便按需访问短作用域的bean或通过调用Provider.get()延迟访问其它的bean。 

2.@Named：与@Component注解等价

- 与@Component不同的是，JSR-330的@Named注解不能组合成其它的注解，因此，如果需要构建自定义的注解，请使用Spring的注解。

3.JSR-330标准注解的局限性

  Spring             	javax.inject.*   	javax.inject的局限性                        
  @Autowired         	@Inject          	@Inject没有require属性，可以使用Java 8的Optional代替。
  @Component         	@Named           	JSR-330没有提供组合模型，仅仅只是一种标识组件的方式           
  @Scope(“singleton”)	@Singleton       	JSR-330默认的作用域类似于Spring的prototype。然而，为了与Spring一般的配置的默认值保持一致，JSR-330配置的bean在Spring中默认为singleton。为了使用singleton以外的作用域，必须使用Spring的@Scope注解。javax.inject也提供了一个@Scope注解，不过这仅仅被用于创建自己的注解。
  @Qualifier         	@Qualifier/@Named	javax.inject.Qualifier仅使用创建自定义的限定符。可以通过javax.inject.Named创建与Spring中@Qualifier一样的限定符
  @Value             	-                	无                                       
  @Required          	-                	无                                       
  @Lazy              	-                	无                                       
  ObjectFactory      	Provider         	javax.inject.Provider是对Spring的ObjectFactory的直接替代，仅仅使用简短的get()方法即可。它也可以与Spring的@Autowired或无注解的构造方法和setter方法一起使用。

7.12 基于Java的容器配置

1.@Bean和@Configuration

@Bean注解表示一个方法将会实例化、配置并初始化一个对象，且这个对象会被Spring容器管理。这就像在XML中<beans/>元素中<bean/>元素一样。@Bean注解可用于任何Spring的@Component注解的类中，但大部分都只用于@Configuration注解的类中。 

2.使用AnnotationConfigApplicationContext实例化Spring容器

简单的构造方法：与使用ClassPathXmlApplicationContext注入XML文件一样，可以使用AnnotationConfigApplicationContext注入@Configuration类。 

 使用scan(String…)扫描组件、使用AnnotationConfigWebApplicationContext支持web应用、使用register(Class）

3.使用@Bean注解：@Bean是方法级别的注解，它与XML中的<bean/>类似，同样地，也支持<bean/>的一些属性，比如，init-method, destroy-method, autowring和name。 

 声明bean：只要在方法上简单的加上@Bean注解就可以定义一个bean了，这样就在ApplicationContext中注册了一个类型为方法返回值的bean。 

Bean之间的依赖：@Bean注解的方法可以有任意个参数用于描述这个bean的依赖关系。比如，如果TransferService需要一个AccountRepository，我们可以通过方法参数实现这种依赖注入。 

接收生命周期回调

指定bean的作用域：使用@Scope注解 、@Scope和scoped-proxy 

4.组合的Java配置：使用@Import注解、在导入的@Bean定义上注入依赖、有条件地包含@Configuration类或@Bean方法

5.绑定Java与XML配置：XML为主使用@Configuration类 、以@Configuration类为主使用@ImportResource引入XML 

第十二章

12.1 Introduction



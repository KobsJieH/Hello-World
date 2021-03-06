* [目录](#--)
  * [Spring](##Spring)
    * [IoC和DI](#IoC和DI) 
    * [元注解](#元注解)
    * [Lombok](#插件Lombok简化代码的部分注解)
    * [Bean](#Spring-Bean)
  * [Spring Boot](#SpringBoot) 
    * [CLI](#CLI)
    * [Devtools热部署](#devtools热部署)
    * [Spring Boot Annotation](#spring-boot-annotation)
  * [Maven](#maven)
    * [配置maven本地资源库](#配置maven本地资源库)
    * [pom文件](#pom文件)
  * [Swagger API](#swagger-api)
  * [Spring Security](#Spring-Security)
  * [Spring Data JPA](#Spring-Data-JPA)
    * [JpaRepostory接口](#jparepository接口)
   
--- 
    
## Spring

### IoC和DI

#### IoC(Inversion of Control, 控制反转)
> 控制：IoC容器控制了对象——外部资源获取（包括文件等）<br>
> 反转：容器帮忙创建及注入依赖对象，对象只是被动的接受依赖对象

#### DI(Dependency Injection)
> 即“依赖注入”：由容器动态的将某个依赖关系注入到组件之中。

需要弄清楚以下问题
1. 谁依赖于谁：应用程序依赖于IoC容器；

2. 为什么需要依赖：应用程序需要IoC容器来提供对象需要的外部资源；

3. 谁注入谁：IoC容器注入应用程序某个对象，应用程序依赖的对象；

4. 注入了什么：就是注入某个对象所需要的外部资源（包括对象、资源、常量数据）。

### 面向切面编程(Aspect Oriented Programming, AOP)

面向切面编程，指扩展功能不修改源代码，直接从业务逻辑代码中分离出来。

> 纵向为去重复，横向为提取

* 主要功能：日志记录、性能统计、安全控制、事务以及异常处理等；  

将上述功能代码从主要业务逻辑代码中分离出来，独立到非业务逻辑代码的方法中，进而实现在改变这些行为的时候不影响主要业务逻辑功能。

基本术语  
1. execute point(执行点)——类初始化，方法调用  
2. join point(连接点)——是程序执行中的一个精确执行点；如类中的一个方法，是抽象概念，实现时不需要定义具体的join point
3. point cut(切入点)——本质上是一个捕获连接点的结构；在类中可以直接定义，用来捕获相关方法的调用
4. advice(通知)——是point cut的执行代码，“执行”的具体逻辑
5. aspect(切面)——point cut和advice结合起来就是aspect，类似于OOP中定义的一个类，但它代表的更多是对象间横向的关系。
6. proxy——一个类被AOP织入后产生一个结果类，它便是融合了源类和增强逻辑的代理类

#### @Pointcut(...)注解

确定Advice织入点，通过一下几种方法，具体表达式的写法省略

关键字 | 功能
:---: | :---:
execution | 匹配连接点，一般是针对方法进行匹配
within | 某个类里面
this | 指定AOP代理类的类型
target |指定目标对象的类型
args | 指定参数的类型
bean |指定特定的bean名称，可以使用通配符（Spring自带的）
@target| 带有指定注解的类型
@args | 指定运行时传的参数带有指定的注解
@within | 匹配使用指定注解的类
@annotation |指定方法所应用的注解

#### @Before、@After和@Around

这些注解用于Advice的具体实现方法，@Before是前置通知，@After是后置通知以及@Around是环绕通知。

#### JoinPoint类

一般作为BeforeAdvice、AfterAdvice的传入参数，用于访问目标方法

方法名 | 功能  
:---: | :---
Signature getSignature() | 获取封装了署名信息的对象,在该对象中可以获取到目标方法名,所属类的Class等信息
Object getTarget() | 获取被代理对象
Object getThis() | 获取一个对象生成的代理对象
Object[] getArgs() | 获取传入目标方法的参数列表


最重要的作用：写代码时实现只需要考虑主流程，而不需要考虑那些不重要的流程

### 元注解
1. @Document，说明该注解被包含在javadoc中；
2. @Target，定义注解的作用目标，包括类、字段和方法等等  
`@Target(ElementType.TYPE)`//接口、类、枚举、注解  
`@Target(ElementType.FIELD)`//字段、枚举的常量  
`@Target(ElementType.METHOD)`//方法  
`@Target(ElementType.PARAMETER)`//方法参数  
`@Target(ElementType.CONSTRUCTOR)`//构造函数  
`@Target(ElementType.LOCAL_VARIABLE)`//局部变量  
`@Target(ElementType.ANNOTATION_TYPE)`//注解  
`@Target(ElementType.PACKAGE)`//包
3. @Retention，定义注解的保留策略  
`@Retention(RetentionPolicy.SOURCE)`//注解仅存在于源码中，在class字节码文件中不包含  
`@Retention(RetentionPolicy.CLASS)`// 默认的保留策略，注解会在class字节码文件中存在，但运行时无法获得  
`@Retention(RetentionPolicy.RUNTIME)`// 注解会在class字节码文件中存在，在运行时可以通过反射获取到    
4. @Inherited, 说明子类可以继承父类中的该注解  


### 注解@Component、@Repository、@Service、@Controller

+ @Controller---控制层组件
+ @Service------业务层组件
+ @Repository---持久层组件，用于标记数据访问，即DAO组件
+ @Component----泛指组件，当组件不好归类时，可以使用这个注解进行标注  

如果 Web 应用程序采用了经典的三层分层结构的话，最好在持久层、业务层和控制层分别采用 @Repository、@Service 和 @Controller 对分层中的类进行注释，而用 @Component 对那些比较中立的类进行注释

### 插件[Lombok](https://projectlombok.org/features/all)简化代码的部分注解

+ `@NonNull` 可以标注在方法、字段、参数之上，表示对应的值不可以为空，这样就不会出现NullPointerException
+ `@Nullable` 可以标注在方法、字段、参数之上，表示对应的值可以为空
+ `@CleanUp` 自动资源管理：不用再在finally中添加资源的close方法
+ `@Setter/@Getter` 自动生成set和get方法
+ `@ToString` 自动生成toString方法
+ `@EqualsAndHashcode` 从对象的字段中生成hashCode和equals的实现
+ `@NoArgsConstructor`,`@RequiredArgsConstructor` 和 `@AllArgsConstructor`自动生成构造方法
+ `@Data` 自动生成set/get方法，toString方法，equals方法，hashCode方法，不带参数的构造方法
+ `@Value` 用于注解final类（不可变且不允许被继承），把特殊的值注入到成员变量中
+ `@Builder`  产生复杂的构建器api类
+ `@SneakyThrows` 异常处理（谨慎使用）
+ `@Synchronized` 同步方法安全的转化
+ `@Getter(lazy=true)`在用到cached的时候自动生成，同时，Lombok会自动管理线程安全的问题，不会重复赋值
+ `@Log` 支持各种logger对象，使用时用对应的注解放在类上，如@Log4j（@Log的变体），会得到一个静态的最终日志字段

附：[某博客](http://greeks.studio/tags/Lombok)关于Lombok几个注解的翻译

### Validation验证机制

#### @Valid和@Validated
+ @Validated: 可用在类型、方法和方法参数上，但是不能用在成员属性（字段）上
+ @Valid: 可用在方法、构造函数、方法参数和成员属性（字段）上  

嵌套验证：@Validated和@Valid加在方法参数前，都不会自动对参数进行嵌套验证  
##### 两者区别：
+ @Validated：用在方法入参上无法单独提供嵌套验证功能。不能用在成员属性（字段）上，也无法提示框架进行嵌套验证。能配合嵌套验证注解@Valid进行嵌套验证。
- @Valid：用在方法入参上无法单独提供嵌套验证功能。能够用在成员属性（字段）上，提示验证框架进行嵌套验证。能配合嵌套验证注解@Valid进行嵌套验证。

### @NotNull和@NotEmpty和@NotBlank区别  
+ @NotNull 不能为null，但可以是empty
  > `("", " ", "  ")`
- @NotEmpty 不能为null，而且长度必须大于0
  > `(" ", "  ")`
+ @NotBlank 只能作用在String上，不能为null，而且调用trim()后长度须大于0
  > `("test")`

### Spring Bean

#### Bean的作用域

在面向对象程序设计中作用域一般指对象或变量之间的可见范围，而在Spring容器中是指起创建的Bean对象相对于其他的Bean对象的请求可见范围。

Spring Framework支持五种作用域，可以是基于xml或使用注解@Scope声明  
![Bean的五种作用域](https://github.com/Kjobs/Hello-World/blob/master/image/Spring_Bean_5scope.png)

1） 单例(Singleton)：只要对Bean的请求id和Bean的定义域相匹配，则会返回bean的同一个实例；单一实例会被存储在单例缓存中  
2） 原型(Prototype)：一个Bean的作用域为prototype时，表示定义对应了多个对象实例；Prototype是原型类型，它在创建容器的时候并没有实例化，而是当获取Bean的时候才会创建一个对象，而且每次获取到的对象都不是同一个对象  
+ 对于有状态的Bean应该使用prototype，对于无状态的Bean则使用singleton  

3） 请求(Request)：为Http请求创建一个Bean实例，对应http的生命周期，请求处理完后遍会销毁这个bean    
4） 会话(Session): 针对http session起作用，Spring容器会根据该Bean的定义来创建一个全新的Bean的实例  
5） 全局会话(Global Session)：类似标准的http session作用域，不过仅仅在基于portlet的web应用当中才有意义。Portlet规范定义了全局Session的概念，它被所有构成某个portlet web应用的各种不同的portlet所共享

---

## SpringBoot

### CLI
> CLI(Spring Boot Command Line)是一个命令行工具，可以用它来快速构建Spring原型应用。
  
windows下安装cmd命令  
```bash
set PATH=D:\bin路径;%PATH%
set PATH=D:\release目录\spring-boot-cli-2.1.4.RELEASE\bin;%PATH%
```  

### mysql连接time zone错误

在配置连接url中加入参数`serverTimezone=GMT%2B8`

### devtools热部署

1. 添加maven依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```
2. 在application.properties或application.yml中添加相关配置，例：
```
spring.devtools.restart.enabled=true    ——表示允许重启项目
spring.devtools.restart.additional-paths=src/main/java/    ——该路径下修改文件会自动重启项目
```


### Spring Boot Annotation

#### @RestController
>@RestController注解继承自@Controller<br>

1. 使用@RestController注解Controller，则Controller中的方法无法返回jsp页面；
2. 如果需要返回指定页面，则需用@Controller配合视图解析器InternalResourceViewResolver才行；
3. 如果需要返回JSON、XML或自定义Type到页面，则需在对应的方法上加@ResponseBody注解。


#### @GetMapping 
> @GetMapping是一个组合注解，是@RequestMapping(method = RequestMethod.GET)的缩写<br>

自Spring4.3引入{@GetMapping、PostMapping、PutMapping、DeleteMapping}简化常用的HTTP映射

#### @AutoWired 
> 可以对类成员变量、方法及构造函数进行标注，完成自动分配  


#### @GeneratedValue
> 在JPA中，@GeneratedValue注解是为了一个实体生成一个唯一标识的主键

有四种主键生成策略，被定义在枚举类GenerationType中<br/>
1. `@Id @GeneratedValue(strategy=GenerationType.AUTO) long id`  
这种方式会把主键生成策略交给持久化引擎，持久化引擎会选择下面其中一种，
由于JPA默认是AUTO方式，所以不用特意指定策略，例：`@Id @GeneratedValue long id`
2. `@Id @GeneratedValue(strategy=GenerationType.IDENTITY) long id`  
即数据库中的主键自增长，包括MySQL在内的大部分数据库支持  
3. `@Id @GeneratedValue(strategy=GenerationType.SEQUENCE) long id`  
部分数据库(Oracle, PostgreSQL, DB2等)生成“序列(sequence)”对象，与注解@SequenceGenerator一起用于主键生成策略  
4. `@Id @GeneratedValue(strategy=GenerationType.TABLE) long id`  
使用一个特定的数据表格来保存主键，不依赖与外部环境和数据库具体实现，容易移植。
一般与另一个注解@TableGenerator搭配使用  

更多详解请参略:[Auto Generated Values](https://www.objectdb.com/java/jpa/entity/generated)

#### @Transient

+ hibernate JPA注解，该属性不与数据库字段做映射，ORM框架将忽略该属性  
- 若实现Serializable接口，该属性也不会被序列化  

---

## maven  

### 配置Maven本地资源库

> maven默认的LocalRepository在Windows下是放在目录`${user.home}/.m2/repository`下的

考虑到重装、恢复系统，因此建议修改默认LocalRepository地址
1. 在maven目录下新建repository文件夹

2. 在maven目录下`/conf/setting.xml`文件中添加  
```xml
<localRepository>D:/maven/repository</localRepository>
```


### POM文件  
1. 每个POM文件需要project元素和三个必须的字段：GroupId(eg, "org.\*", "com.\*")，ArtifactId(项目名称，如hello-world)和version
2. POM.xml的根元素是project，他有三个主要的子节点
 ```xml
<project>
...
    <parent>
         <groupId>工程组的标识</groupId>
         <artifactId>工程的标识</artifactId>
         <version>工程的版本号</version>
    </parent>
 ...
</project> 
```

### mvn常见命令

`mvn --help`——帮助  
`mvn --version/-v`——显示版本信息  
`mvn package`——生成target目录，编译、测试代码，生成测试报告、jar/war文件  
`mvn clean`——清空生成文件  
`mvn compile`——编译  
`mvn test`——编译并测试  

可接受参数
+ -D 指定参数，如 -Dmaven.test.skip=true 跳过单元测试；
+ -P 指定 Profile 配置，可以用于区分环境；
+ -e 显示maven运行出错的信息；
+ -o 离线执行命令,即不去远程仓库更新包； 
+ -X 显示maven允许的debug信息；
+ -U 强制去远程更新snapshot的插件或依赖，默认每天只更新一次

更多可见:[Maven Apache Project](https://maven.apache.org/index.html)

<br>

---

## swagger API

### 配置依赖
1. 添加Maven依赖
2. 创建Swagger2配置类  
通过@Configuration注解，让Spring来加载该类配置。再通过@EnableSwagger2注解来启用Swagger2;
主要包括两个：创建Docket的Bean，实现函数apiInfo()用来创建Api的基本信息。
3. 为接口函数添加Api注解

若要在Swagger2 Api中实现权限控制，需要在创建Docket时加上securitySchemes、securityContexts的相关配置

### 常用Api注解  
1）@Api用于类  
> 参数说明：url{路径信息}  

2）@ApiOperation用于方法  
> 参数说明：value{接口说明},httpMethod{接口请求方式}，response{接口返回参数类型}，note{接口发布说明}   

3）@ApiParam用于方法、参数、字段说明  
> 参数说明：required{是否必须参数},name{参数名称},value{参数参数具体描述}
  	
描述 | 名称 | 作用
:---:|:----|:---:
协议集描述|@Api|用于类上，说明类的作用
协议描述|@ApiOperation|用在方法上，说明方法的作用
响应方式|@ApiResponse|用在方法上，描述一个操作可能的返回结果(状态码及其描述)
响应方式集|@ApiResponses|为@ApiResponse集合，响应集配置
非对象参数描述|@ApiImplicitParam|用在方法上,需要使用Servlet或non-JAX-RS环境时定义参数
非对象参数集|@ApiImplicitParams|为@ApiImplicitParam集合
返回对象的意义|@ApiModel|用在返回对象类上，用于描述model信息
对象属性|@ApiModelProperty|用于方法、字段上，描述model属性或者数据操作更改

附：[swagger annotation docs](http://docs.swagger.io/swagger-core/v1.3.12/apidocs/index.html?com/wordnik/swagger/annotations)

---

## Spring Security

+ 认证(Authentication)：确认用户可以访问系统  
+ 授权(Authorization)：确认用户在当前系统中是否能够执行某个操作，即用户所拥有的功能权限

---

## Spring Data JPA

### JpaRepository接口
```java
public interface JpaRepository<T,ID> extends PagingAndSortingRepository<T,ID>, QueryByExampleExecutor<T> {}
```
> JpaRepository继承了接口PagingAndSortingRepository和QueryByExampleExecutor；
PagingAndSortingRepository又继承CrudRepository，因此，JpaRepository接口同时拥有了基本CRUD功能以及分页功能。

Repository继承两个接口
```java
public interface EntityRepository<TKey extends Serializable, TEntity extends Entity<TKey>>
        extends JpaRepository<TEntity, TKey>, JpaSpecificationExecutor<TEntity> {
}
```
若查询是不带查询条件(例只做列表展示和分页)的，则只需继承JpaRepository接口即可；若查询带查询条件则需要继承JpaSpecificationExecutor接口。

### @Temporal
+ @Temporal(TemporalType.DATE)实体类会封装成日期“yyyy-MM-dd”的Date类型
+ @Temporal(TemporalType.TIME)实体类会封装成时间“hh-MM-ss”的Date类型
+ @Temporal(TemporalType.TIMESTAMP)实体类会封装成完整的时间“yyyy-MM-dd hh:MM:ss”的Date类型  

### Jpa实现多条件查询

要实现Jpa的Criteria查询，需要集成Jpa的相应接口JpaSpecificationExecutor

#### 重写Specification的toPredicate方法

1. CriteriaQuery<?> criteriaQuery ：这是一个面向对象查询，代表的是Specific的顶层查询对象，它包含查询的各个部分：select，from，where，group by, order by等，不过它是一个面向对象的查询方式，只对实体类型，嵌入式类型的Criteria查询起作用。
2. Root<T> root ：代表要查询的对象，也就是实体类型，实体类型好比sql语句中的from后的表。传入实体类型后，会被CriteriaQuery的父类AbstractQuery.from将实体类型传入。
3. CriteriaBuilder criteriaBuilder ：用来构建CriteriaQuery的构建器对象Predicate（谓语），即：一个简单或者复杂的谓语类型，相当于条件或者多条件集合。
4. Predicate ：就是多条件查询中的条件，可以通过List<Predicate> 实现多个条件操作。

---








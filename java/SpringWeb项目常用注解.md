注解分为两类:

一类是使用Bean，即是把已经在xml文件中配置好的Bean拿来用，完成属性、方法的组装；比如@Autowired , @Resource，可以通过byTYPE（@Autowired）、byNAME（@Resource）的方式获取Bean；

一类是注册Bean,@Component , @Repository , @ Controller , @Service , @Configration这些注解都是把你要实例化的对象转化成一个Bean，放在IoC容器中，等你要用的时候，它会和上面的@Autowired , @Resource配合到一起，把对象、属性、方法完美组装。

### 1、**@SpringBootApplication**

　　 SpringBoot项目核心注解，主要目的是开启自动配置

### 2、**@ResponseBody**

作用： 

​      该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区。

使用时机：

​      返回的数据不是html标签的页面，而是其他某种格式的数据时（如json、xml等）使用

### 3、**@RequestBody**

作用： 

​      i) 该注解用于读取Request请求的body部分数据，使用系统默认配置的HttpMessageConverter进行解析，然后把相应的数据绑定到要返回的对象上；

​      ii) 再把HttpMessageConverter返回的对象数据绑定到 controller中方法的参数上。

使用时机：

A) GET、POST方式提时， 根据request header Content-Type的值来判断:

- ​    application/x-www-form-urlencoded， 可选（即非必须，因为这种情况的数据@RequestParam, @ModelAttribute也可以处理，当然@RequestBody也能处理）；
- ​    multipart/form-data, 不能处理（即使用@RequestBody不能处理这种格式的数据）；
- ​    其他格式， 必须（其他格式包括application/json, application/xml等。这些格式的数据，必须使用@RequestBody来处理）；

B) PUT方式提交时， 根据request header Content-Type的值来判断:

- ​    application/x-www-form-urlencoded， 必须；
- ​    multipart/form-data, 不能处理；
- ​    其他格式， 必须；

说明：request的body部分的数据编码格式由header部分的Content-Type指定；

### 4、**@RequestMapping（“/”）**

　　 在Spring MVC 中使用 @RequestMapping 来映射请求，也就是通过它来指定控制器可以处理哪些URL请求，相当于Servlet中在web.xml中配置。

### 5、**@Service**

 　　下面的类是Service，业务层功能组件

### 6、**@Autowired**

 　　注入组件（Bean），通过Spring注入的Bean一般都被定义成private，如：

```java
@Autowired
private ClasDAO clasDAO;
```

　　Spring启动时，AutowiredAnnotationBeanPostProcessor会扫描所有的Bean，当发现其中有@Autowired注解时，就会找相应类型的Bean，并且实现注入，省去实例化的过程，直接可用。

### 7、**@RestController**

　　@Controller和@RestController的区别？
官方文档：
　　@RestController is a stereotype annotation that combines @ResponseBody and @Controller.
意思是：
　　@RestController注解相当于@ResponseBody ＋ @Controller合在一起的作用。



　　1)如果只是使用@RestController注解Controller，则Controller中的方法无法返回jsp页面，配置的视图解析器InternalResourceViewResolver不起作用，返回的内容就是Return 里的内容。

　　例如：本来应该到success.jsp页面的，则其显示success.

　　2)如果需要返回到指定页面，则需要用 @Controller配合视图解析器InternalResourceViewResolver才行。
　　3)如果需要返回JSON，XML或自定义mediaType内容到页面，则需要在对应的方法上加上@ResponseBody注解。

### 8、**@Controller**

　　表示下面的类是controller类，控制层业务组件（用于实现前后台交互）

### 9、**@Resource**

　　@Resource的作用相当于@Autowired，只不过@Autowired按byType自动注入，而@Resource默认按 byName自动注入罢了。@Resource有两个属性是比较重要的，分是name和type，Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。所以如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。如果既不指定name也不指定type属性，这时将通过反射机制使用byName自动注入策略。
　　@Resource装配顺序
　　1. 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常
　　2. 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常
　　3. 如果指定了type，则从上下文中找到类型匹配的唯一bean进行装配，找不到或者找到多个，都会抛出异常
　　4. 如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配； 　　

　　@Resource（这个注解属于J2EE的），默认安照名称进行装配，名称可以通过name属性进行指定， 
如果没有指定name属性，当注解写在字段上时，默认取字段名进行按照名称查找，如果注解写在setter方法上默认取属性名进行装配。 当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。如：

```java
@Resource(name="baseDao")     
private BaseDao baseDao;  
```

　　@Autowired与@Resource都可以用来装配bean. 都可以写在字段上,或写在setter方法上。　　

　　@Autowired默认按类型装配（这个注解是属业spring的），默认情况下必须要求依赖对象必须存在，如果要允许null 值，可以设置它的required属性为false，如：@Autowired(required=false) ，如果我们想使用名称装配可以结合@Qualifier注解进行使用，如下： 

```java
@Autowired() @Qualifier("baseDao")     
private BaseDao baseDao; 
```

### 10、**@Configuration**

　　类是配置类，来对bean进行配置。>

### 11、**@Bean**

 　  @Bean明确地指示了一种方法，什么方法呢——产生一个bean的方法，并且交给Spring容器管理；从这我们就明白了为啥@Bean是放在方法的注释上了，因为它很明确地告诉被注释的方法，你给我产生一个Bean，然后交给Spring容器，剩下的你就别管了　

   　记住，@Bean就放在方法上，就是产生一个Bean。
# 使用 Keycloak 轻松保护 Spring Boot 应用程序


尽管安全性是应用程序的一个关键点，但是在开发中实施起来确实比较麻烦。更加麻烦的是，这个关键点通常不怎么受重视，实现的效果普遍的 low，而且受到诸多方面的掣肘。而最近安全服务器的出现，就可以将认证和授权方面的业务逻辑外包和分派出去。 在这些服务器中，最有希望的是 Keycloak，因为它开放源代码，灵活，而且未来能适合任何技术亦未可知，它也可以轻松地部署并适应于其自身的基础设施之中。

另外，Keycloak 也不仅仅是一个身份验证服务器，它还提供了完整的身份管理系统，可以实现诸如 LDAP 那样的第三方的用户联盟。可以看看[这里](https://link.juejin.im/?target=http%3A%2F%2Fwww.keycloak.org%2F )。

该项目可以在[GitHub](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fsebastienblanc%2Fspring-boot-keycloak-tutorial )上找到。

## Spring Boot 和 Keycloak 

Keycloak 为需要与 Keycloak 实例交互的应用程序提供适配器。 有用于 WildFly / EAP，NodeJS，JavaScript 的适配器，当然也有用于 Spring Boot 的适配器。

### 设置 Keycloak 服务 

你有多个选项来设置一个 Keycloak 服务器，但最简单的一个可能是获取一个独立的发布。解压缩，瞧着！ 打开一个终端，然后切换到你解压缩的 Keycloak 服务，并从 bin 目录下运行：

```
./standalone.sh(bat)
```

然后打开浏览器，然后转到进入 *http://localhost:8080/auth*。

由于这是服务的第一次运行，你必须创建一个管理员账户才行，所以让来我们创建一个管理员用户，用户名为 admin，密码为 admin：

![img](https://static.oschina.net/uploads/space/2017/0627/151225_PyIy_2903254.png)

现在您可以登录到管理控制台并开始配置 Keycloak。

### 创建一个新的 Realm 

Keycloak 定义了一个 realm 的概念，并且你将在 realm 中定义客户端，在 Keycloak 中的术语是指由 Keycloak 保护的应用程序。 它可以是 Web App，Java EE 后端，Spring Boot 等等。

所以让我们创建一个新的 realm，只需点击“添加领域”按钮：

![img](https://static.oschina.net/uploads/space/2017/0627/151248_lmTW_2903254.png)

让我们称之为“SpringBoot.”

### 创建客户端，角色以及用户 

现在我们需要定义一个客户端，这就会是我们的 Spring Boot 应用程序。转到“Clients”部分，然后单击“Create”按钮。我们把这个客户端叫做“product-app”：

![img](https://static.oschina.net/uploads/space/2017/0627/153133_3ky7_2903254.png)

在接下来的界面上，我们可以让大部分东西都保留默认设置，只需要输入一个有效的重定向 URL，让 Keycloak 将其用于用户进行了身份验证之后。这里我们输入这个 URL：“http：// localhost：8081 / *”。

![img](https://static.oschina.net/uploads/space/2017/0627/153150_gA6P_2903254.png)

不要忘了把这些配置保存下来哦！

现在，我们要把需要分配给用户的角色定义好。创建一个名为“user”的简单角色：

![img](https://static.oschina.net/uploads/space/2017/0627/153203_k5Nh_2903254.png)

最后，我们得创建一个用户。这里只需要用户名属性就可以了，这里我们就叫他“testuser”吧：

![img](https://static.oschina.net/uploads/space/2017/0627/153217_Bz1v_2903254.png)

最后，我们需要设置一下凭据，所以要转到用户的“Credencials（凭据）”选项卡界面，并选择一个密码。我会在本文的余下部分使用“password”，还得确保把“Temporary（临时）”这个标志关闭了，除非你是想要用户可以在首次认证时更改密码。

现在转到“Role Mappings（角色映射）”选项卡界面来分配“user”这个角色：

![img](https://static.oschina.net/uploads/space/2017/0627/153230_2RAO_2903254.png)

现在我们就完成了 Keycloak 服务器的配置，可以开始构建 Spring Boot App 了！

## 创建一个简单的 App 

让我们来创建一个简单的 Spring Boot App。你也许会想要使用 [Spring Initializrr](https://link.juejin.im/?target=https%3A%2F%2Fstart.spring.io%2F )，这时候要把如下选项给选上：

- Web
- Freemarker
- Keycloak

将你的 App 命名为“product-app”，然后把生成的工程下载下来：

![img](https://static.oschina.net/uploads/space/2017/0627/153244_MGLp_2903254.png)

将应用程序导入你喜欢使用的 IDE 里面，这里我会使用 IntelliJ。

我们的应用程序会比较简单，只包含两个页面：

- 一个 index.html，它将是登录页面，里面只包含产品页面的链接。
- Products.ftl，它将是我们的产品页面的模板，只能被通过了身份验证的用户访问到。

首先我们在“/src/resources/static”目录中创建一个简单的 index.html 文件：

```html
<html>
 
 <head>
   <title>My awesome landing page</title>
 </head>
 
 <body>
   <h1>Landing page</h1> <a href="/products ">My products</a>
 </body>
 
</html>
```

现在，我们需要一个控制器：

```java
@Controller
class ProductController {
 
  @Autowired ProductService productService;
 
  @GetMapping(path = "/products ")
  public String getProducts(Model model){
     model.addAttribute("products ", productService.getProducts());
     return "products ";
  }
 
  @GetMapping(path = "/logout ")
  public String logout(HttpServletRequest request) throws ServletException {
     request.logout();
     return "/ ";
  }
}
```

你会发现这很简单，就是定义了产品页面的映射，然后再为注销操作定义一个映射。你还会注意到，我们调用了一个“ProductService”，它会返回一个字符串列表，我们把这个列表放到 Spring MVC Model 对象里面去，所以我们要创建这个服务：

```java
@Component
class ProductService {
  public List<String> getProducts() {
     return Arrays.asList("iPad ","iPod ","iPhone ");
  }
}
```

我们还需要创建 product.ftl 模板。要在“src/resources/templates”中创建此文件：

```html
<#import "/spring.ftl " as spring>
   <html>
   <h1>My products</h1>
   <ul>
       <#list products as product>
           <li>${product}</li>
       </#list>
   </ul>
   <p> <a href="/logout ">Logout</a> </p>
 
   </html>
```

在这里，我们简单地遍历了 Spring MVC Model 对象中的产品列表，并添加一个从我们的应用程序中注销的链接。

我们要做的就是向 application.properties 中添加一些 keycloak 相关的属性。

### 定义 Keycloak 的配置 

一些属性是必须要有的：

```properties
keycloak.auth-server-url=http://localhost:8080/auth keycloak.realm=springboot keycloak.public-client=true keycloak.resource=product-app
```

我们需要定义一些安全方面的约束，就像你在 web.xml 中使用 Java EE 应用的时候要进行的配置一样：

```properties
keycloak.security-constraints[0].authRoles[0]=user keycloak.security-constraints[0].securityCollections[0].patterns[0]=/products/*
```

在这里，我们简单地定义每个向 /products/* 发起的请求都应该通过用户验证，而且该用户得有“user”这个角色。

现在，我们只需要配置最后一个属性来确保我们的应用程序将会在端口8081上运行：

```properties
server.port=8081
```

这样我们就都设置好了，可以把应用程序运行起来了！

要运行这个 Spring Boot 应用程序，有很多方式可以选择。使用 Maven 的话，你可以简单地像下面这样做就行了：

```
mvn clean spring-boot:run
```

现在导航到“http//localhost8080”，你应该就可以看到登录页面你了。点击“products”链接，会被重定向到 Keycloak 登录页面：

![img](https://static.oschina.net/uploads/space/2017/0627/153303_LcLe_2903254.png)

使用我们的用户“testuser/password”进行登录，应该会重定向到产品页面：

![img](https://static.oschina.net/uploads/space/2017/0627/155614_X0qN_2903254.png)

恭喜哦！现在你已经使用 Keycloak 为你的第一个 Spring Boot 应用程序加上了防护措施。现在注销并返回到 Keycloak 管理员控制台，你就会知道如何去“调整”登录页面。例如，您可以启用“Remember Me（记住我）”和“User Registration（用户注册）”功能。为此，请点击保存按钮并返回到登录界面。在那里你会这些功能已经添加上了。

## 引入 Spring Security 支持 

如果你是 Spring 用户而且一直在玩安全方面的东西的话，那么很有可能就会要用到 Spring Security。而我这里有一个好消息：我们还有一个 Keycloak Spring Security Adapter，而且它已经被包含在我们的 Spring Boot Keycloak Starter 中了。

我们来看看如何将 Spring Security 和 Keycloak 放到一起使用。

### 添加 Spring Security Starter 

首先，我们需要 Spring Security 的库。最容易的方法就是将 spring-boot-starter-security  的 artifact 添加到你的 pom.xml 中:

```java
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### 创建一个 SecurityConfig 类

同其它的得到 Spring Security 防护的应用程序一样, 这里也需要一个扩展自 WebSecurityConfigurerAdapter 的配置类。Keycloak 提供了它自己的一个子类来给你进行再次继承：

```java
@Configuration
@EnableWebSecurity
@ComponentScan(basePackageClasses = KeycloakSecurityComponents.class)
class SecurityConfig extends KeycloakWebSecurityConfigurerAdapter
{
  /**
   * Registers the KeycloakAuthenticationProvider with the authentication manager.
   */
  @Autowired
  public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
     KeycloakAuthenticationProvider keycloakAuthenticationProvider = keycloakAuthenticationProvider();
     keycloakAuthenticationProvider.setGrantedAuthoritiesMapper(new SimpleAuthorityMapper());
     auth.authenticationProvider(keycloakAuthenticationProvider);
  }
 
  @Bean
  public KeycloakConfigResolver KeycloakConfigResolver() {
     return new KeycloakSpringBootConfigResolver();
  }
 
  /**
   * Defines the session authentication strategy.
   */
  @Bean
  @Override
  protected SessionAuthenticationStrategy sessionAuthenticationStrategy() {
     return new RegisterSessionAuthenticationStrategy(new SessionRegistryImpl());
  }
 
  @Override
  protected void configure(HttpSecurity http) throws Exception
  {
     super.configure(http);
     http
           .authorizeRequests()
           .antMatchers("/products* ").hasRole("user ")
           .anyRequest().permitAll();
  }
}
```

让我们来仔细看看最重要的几个方法：

- configureGlobal: 这里我们修改 Granted Authority Mapper。在 Spring Security 中角色都默认带上了前缀 ROLE_。我们可以在我们的 Realm 配置中把这个改掉，不过这样做会让其它不知道这个约定的应用程序感到困惑, 所以这里我们分配了一个 SimpleAuthorityMapper 来确保不会有前缀被加上去。
- keycloakConfigResolver: Keycloak Spring Security Adapter 默认会从你的 classpath 中找一个叫做 keycloak.json 的文件。不过这里我们并不像要利用上 Spring Boot 的属性文件支持。
- configure: 这就是我们定义安全限制的地方。这个相当容易理解。我们要做的就是把带有“user” 角色的路径 “/products” 给保护起来。

现在我们可以在我们的 application.properties 文件中将之前已经定义的安全限制给去掉了，然后添加另外一个属性来将我们的 KeyCloak 用户名映射到 Principal 名称上去：

```properties
keycloak.principal-attribute=preferred_username
```

现在我们设置可以将规则诸如到我们的控制器方法中去，并且将用户名放到 Spring 的 MVC model 中:

```java
@GetMapping(path = "/products ")
public String getProducts(Principal principal, Model model){
  model.addAttribute("principal ",principal);
  model.addAttribute("products ", productService.getProducts());
  return "products ";
}
```

最后，我们可以更新 product.ftl 模板，来把用户名打印出来：

```html
<#import "/spring.ftl " as spring>
   <html>
   <h1>Hello ${principal.getName()}</h1>
   <ul>
       <#list products as product>
           <li>${product}</li>
       </#list>
   </ul>
   <p> <a href="/logout
                            ">Logout</a> </p>
 
   </html>
```

再一次重启你的 App，可以看到它仍然可以运行起来，你也应该能看到你的用户打印在了产品页面之上：

![img](https://static.oschina.net/uploads/space/2017/0627/155630_Br0T_2903254.png)
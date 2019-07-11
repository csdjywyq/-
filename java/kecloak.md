# 使用Keycloak实现安全的SpringBoot微服务

Keycloak是RedHat的开源身份和访问管理解决方案，本文介绍如何在我们的微服务安全模块中使用keycloak，特别是基于SpringBoot的微服务。

## Keycloak

它提供了身份和访问管理的有用功能：

- 单点登录（SSO），身份代理和社交登录
- 用户联合
- 客户端适配器
- 管理控制台和帐户管理控制台。

虽然安全性是任何应用程序的一个重要方面，但安全性的实现部分是复杂和困难的。通常，它在代码中经常被忽略或执行不当和干扰。

开发人员需要安全服务器，允许外包和授权认证和授权方面。他们想要一种能够自动开发应用程序安全功能的工具，这通常是一项复杂的任务。

Keycloak是最有前途的开源IDAM（身份和访问管理）服务器之一，它与任何技术无关，可以在自己的基础架构中轻松部署/适应。

Keycloak尝试解决基于REST的Web应用程序和Web服务的单点登录。Keycloak的最终目标是使安全性足够简单，以便作为服务和应用程序中的安全模块插入。安全功能很混乱，当开发人员手动为自己编写时，它会变得更危险，更容易出错。Keycloak通过提供开箱即用的安全功能帮助我们，并且可以根据任何组织的个性化需求轻松定制。

Keycloak可以帮助我们在应用程序中引入这些功能：

- 用于登录，注册，管理和帐户管理的可自定义用户界面
- 集成到现有LDAP和活动目录服务器
- 将身份验证委托给Twitter和Github等第三方身份提供商

该项目是开源的，可以在Github上找到。

## 安装：

有不同的方法来安装keycloak。最简单的是只需下载Keycloak，这是一种独立的安装模式，只需将其解压缩即可。现在打开一个终端并转到解压缩的Keycloak服务器并导航到bin目录 - 然后只需运行以下命令：

./standalone.sh(bat）

安装完成后，打开浏览器并转到[http：// localhost：8080 / auth]()。

默认情况下，Keycloak附带一个H2数据库，但如果选择RDMS，可以使用带有RDMS的Keycloak。

由于是第一次运行服务器，因此您必须创建管理员用户。创建一个管理员用户，其中“admin”作为用户名，“admin”作为密码。

## 创造一个新领域

在Keycloak中，领域是您定义客户端的地方。这意味着这些客户端是将由Keycloak保护的应用程序 - 可能是Web应用程序或Spring Boot。

注意：'Master'是Keycloak的默认领域。让我们只需点击“添加领域”按钮即可创建新领域

## 创建客户端

Spring Boot应用程序是您的客户端。就这么简单。在Keycloak中，客户端是使用Keycloak保护的应用程序。

如何在Keycloak中创建客户端：

1. 转到门户网站中的“Client”菜单

2. 单击“创建”按钮

3. 为Client-Id提供名称。我们称之为Rbi-Service。

在下一个屏幕上将所有内容保持为默认值，您需要的只是输入一个有效的重定向URL，Keycloak将在该应用程序中对用户进行身份验证时使用该URL。我们将选择在有效的重定向URL部分中保留http:// localhost:8081/*。

## Keycloak API：

您现在可能正在使用当前项目中的REST API。别担心，Keycloak提供对REST API的完全支持。实际上，它还提供了一个完整的管理控制台，可以通过REST API使用。

这是可能的Keycloak管理REST API列表。

对于管理REST API，有一个Java客户端库，可以很容易地与Java一起使用。要从您的应用程序中使用它 - 就像您对任何第三方库所做的那样 - 只需在项目中添加keycloak-admin-client库的依赖项。而已。

```XML
<dependency>
  <groupId>org.keycloak</groupId>
  <artifactId>keycloak-admin-client</artifactId>
  <version>3.2.1.Final</version>
</dependency>
```

application.yml配置：

```yaml
server:
  port: 9999

keycloak:
  url: http://localhost:8080/auth
  realm: master
  username: admin
  password: admin
  clientId: admin-cli
```

注意：我们需要`clientId作为admin-cli`，这由keycloak提供的所有默认客户端ID，用于实现keycloak为admin提供的REST端点，

## REST API实现

使用REST端点在域中创建用户有几个步骤：

### 创建用户：

第1步：通过getInstance（）方法使用主管理员的详细信息创建实例

```java
Keycloak kcMaster = Keycloak.getInstance(serverUrl, masterRealm, masterUsername, masterPassword, masterClientId);
```


第2步：在UserRepresentation中设置用户数据

```java
private String userName;
private String firstName;
private String lastName;
private String email;
private String password;
private String companyName;
```



密码映射到CredentialRepresentation。您必须强制用户在首次登录时通过以下代码更改密码：

```java
credential.setTemporary(isTempPassword);
and user.setRequiredActions(Arrays.asList(ACTION_UPDATE_PASSWORD));
```


最后，您所要做的就是在给定领域中调用createUser：

```java
kcMaster.realm(request.getCompanyName()).users().create(user);
```

由于Keycloak拥有自己的数据库，因此新创建的用户信息存储在表中：user_entity

恭喜！您刚刚使用REST端点在Keyclaok中创建了一个用户。

### 用户登录访问和刷新令牌

现在遇到一个重大挑战：将用户登录到您的应用程序中

Keycloak根据您在Keycloak中配置客户端的方式提供了几种获取访问令牌的方法。Keycloak为您提供AccessTokenResponse，它是一个基于JWT的令牌，包含访问令牌，刷新令牌和这些属性的相关信息。

```java
@PostMapping(path = "/login", consumes = APPLICATION_JSON_VALUE, produces = APPLICATION_JSON_VALUE)
    public AccessTokenResponse getToken(@RequestBody Map<String, Object> credentials) {   
        Keycloak kc = Keycloak.getInstance(serverUrl,
                               (String  credentials.get("company"), 
                               (String) credentials.get("username"), 
                               (String) credentials.get("password"),
                               (String) credentials.get("clientId"));               
               return kc.tokenManager().getAccessToken();

}
```
您可以重新检查详细数据，该数据嵌入了jwt.io上的访问令牌

## Keycloak with Spring

Keycloak知道自己的API与应用程序的交互，并为愿意与Keycloak通信的应用程序提供适配器。它已经为Javascript，NodeJs应用程序，WildFly / EAP和Spring Boot提供了适配器。Spring-boot的keycloak依赖（[源代码](http://start.spring.io/)）

关于安全配置：

如果您是后端开发人员而且您正在使用Spring，并且您必须处理与安全相关的任务，那么您肯定使用Spring Security。好吧，有一个好消息：有一个Keycloak Spring安全适配器，它包含在Spring Boot keycloak starter中。

如果您使用过Spring Security，那么您可能知道SecurityConfig类扩展了WebSecurityConfigurerAdapter。这是为创建WebSecurityConfigurer实例提供了方便的基类，并且任何使用Spring Security的应用程序都需要它。Keycloak在WebSecurityConfigurerAdapter上提供了一个包装类，名为KeycloakWebSecurityConfigurerAdapter。

现在让我们看看如何将Spring Security和Keycloak结合在一起。

### 添加Spring Security Starter

首先，我们需要在pom.xml中添加spring-boot-starter-security工件来获取Spring Security库：

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### 创建SecurityConfig类

```java
@Configuration
@EnableWebSecurity
@ComponentScan(basePackageClasses = KeycloakSecurityComponents.class)
public class SecurityConfig extends KeycloakWebSecurityConfigurerAdapter {

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        KeycloakAuthenticationProvider keycloakAuthenticationProvider = keycloakAuthenticationProvider();
        keycloakAuthenticationProvider.setGrantedAuthoritiesMapper(new SimpleAuthorityMapper());
        auth.authenticationProvider(keycloakAuthenticationProvider);
    }

    @Bean
    public KeycloakConfigResolver KeycloakConfigResolver() {
        return new KeycloakSpringBootConfigResolver();
    }

    @Bean
    @Override
    protected SessionAuthenticationStrategy sessionAuthenticationStrategy() {
        return new RegisterSessionAuthenticationStrategy(new SessionRegistryImpl());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception
    {
        super.configure(http);
        http
            .authorizeRequests()
            .antMatchers("/features*").hasRole("user")
            .anyRequest().permitAll();

    }
}
```

让我们仔细看看最重要的方法：

- configureGlobal：如果你还记得Spring Security，那么角色的前缀是ROLE_。这也可以在keycloak中完成，但这可能会导致其他应用程序不知道此约定的混淆。通过更改授权机构映射器，我们分配一个SimpleAuthorityMapper，确保不添加前缀。

- keycloakConfigResolver：默认情况下，Keycloak Spring Security Adapter将扫描类路径中存在的名为keycloak.json的文件。在这里，我们要提供Spring Boot属性文件支持。

- configure：这是我们定义安全验证的地方。很容易理解我们正在使用角色“user”保护路径“/ features”。

## 结论

在您的应用程序中构建安全组件总是很困难，特别是当它是基于微服务的架构时。当然有多种选择可用于构建安全服务，但Keycloak承担这一责任并帮助开发人员专注于产品或应用程序所需的内容。

由于Keycloak中的每个组件都经过了很好的尝试和测试，因此在安全模块方面遇到任何麻烦的可能性都很小。

Keycloak确实为Spring开发人员提供了处理安全性的巨大潜力，特别是在开发正朝着构建微服务战略的方向发展时。
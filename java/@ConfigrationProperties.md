# Spring Boot 2.0 @ConfigurationProperties 使用

### 引言

Spring Boot的一个便捷功能是外部化配置，可以轻松访问属性文件中定义的属性。本文将详细介绍@ConfigurationProperties的使用。

### 配置项目POM

- 在pom.xml中定义Spring-Boot 为parent

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.4.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>
```

- 添加依赖
  1. 添加*web*,因为我们需要使用到JSR-303规范的*Validator*,如果不想使用web依赖,也可以直接依赖*hibernate-validator*
  2. 添加*spring-boot-configuration-processor*,可以在编译时生成属性元数据(spring-configuration-metadata.json).
  3. 添加*lombok*,可以方便使用注释处理器的功能省去Pojo定义中get set这些麻烦工作.

```xml
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--<dependency>-->
      <!--<groupId>org.hibernate.validator</groupId>-->
      <!--<artifactId>hibernate-validator</artifactId>-->
      <!--<version>6.0.11.Final</version>-->
      <!--<scope>compile</scope>-->
    <!--</dependency>-->
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-configuration-processor</artifactId>
      <optional>true</optional>
    </dependency>
```

### 例子编写

首先定义一个*DocumentServerProperties*对象,下面这个文档服务器配置是我假设的,主要是为了演示属性配置的大部分情况

```java
@Getter
@Setter
public class DocumentServerProperties {

    private String remoteAddress;
    private boolean preferIpAddress;
    private int maxConnections=0;
    private int port;
    private AuthInfo authInfo;
    private List<String> whitelist;
    private Map<String,String> converter;
    private List<Person> defaultShareUsers;

    @Getter
    @Setter
    public static class AuthInfo {

        private String username;
        private String password;
    }
}
```

#### 绑定属性配置

注意*@ConfigurationProperties*并没有把当前类注册成为一个Spring的Bean,下面介绍*@ConfigurationProperties*配置注入的三种方式.

- 配合*@Component*注解直接进行注入

```java
@ConfigurationProperties(prefix = "doc")
@Component
public class DocumentServerProperties {
    //代码...
}
```

- 使用*@EnableConfigurationProperties*,通常配置在标有*@Configuration*的类上,当然其他*@Component*注解的派生类也可以，不过不推荐.

```java
@ConfigurationProperties(prefix = "doc")
public class DocumentServerProperties {
    //代码...
}
@EnableConfigurationProperties
@Configuration
public class SomeConfiguration {
    private DocumentServerProperties documentServerProperties
       
    public SomeConfiguration(DocumentServerProperties documentServerProperties) {
        this.documentServerProperties = documentServerProperties;
    }

}
```

- 使用*@Bean*方式在标有*@Configuration*的类进行注入,这种方式通常可以用在对第三方类进行配置属性注册

```java
@Configuration
public class SomeConfiguration {
    
    @Bean
    public DocumentServerProperties documentServerProperties(){
        return new DocumentServerProperties();
    }
    
    @ConfigurationProperties("demo.third")
    @Bean
    public ThirdComponent thirdComponent(){
        return new ThirdComponent();
    }

}
```

#### 编写配置文件

Spring-Boot中配置文件的格式有properties和yaml两种格式,针对上面的配置对象分别写了两种格式的配置文件例子.

- Properties

```
doc.remote-address=127.0.0.1
doc.port=8080
doc.max-connections=30
doc.prefer-ip-address=true
#doc.whitelist=192.168.0.1,192.168.0.2
# 这种等同于下面的doc.whitelist[0] doc.whitelist[1]
doc.whitelist[0]=192.168.0.1
doc.whitelist[1]=192.168.0.2
doc.default-share-users[0].name=jack
doc.default-share-users[0].age=18
doc.converter.a=xxConverter
doc.converter.b=xxConverter
doc.auth-info.username=user
doc.auth-info.password=password
```

- Yaml

```yaml
doc:
  remote-address: 127.0.0.1
  port: 8080
  max-connections: 30
  prefer-ip-address: true
  whitelist: 
    - 192.168.0.1
    - 192.168.0.2
  default-share-users: 
    - name: jack
      age: 18
  converter: 
    a: aConverter
    b: bConverter
  auth-info:
    username: user
    password: password
```

在上面的两个配置文件中,其实已经把我们平常大部分能使用到的属性配置场景都覆盖了,可能还有一些特殊的未介绍到,比如Duration、InetAddress等。

#### 增加属性验证

下面我们利用JSR303规范的实现对DocumentServerProperties属性配置类,添加一些常规验证,比如Null检查、数字校验等操作,

需要注意在Spring-Boot 2.0版本以后,如果使用JSR303对属性配置进行验证必须添加*@Validated*注解,使用方式如下片段：

```java
@ConfigurationProperties(prefix = "doc")
@Validated
public class DocumentServerProperties {
    @NotNull // 判断不为空的情况
    private String remoteAddress;
    
    //限制端口只能是80-65536之间
    @Min(80)
    @Max(65536)
    private int port;
    //其他代码
}
```

在有些数情况下,我们希望自定义验证器,有两种方式可以进行实现

1. 实现*org.springframework.validation.Validator*接口,并且在配置一个Bean名称必须叫**configurationPropertiesValidator**，代码如下：

```java
public class UserLoginValidator implements Validator {

    private static final int MINIMUM_PASSWORD_LENGTH = 6;

    public boolean supports(Class clazz) {
       return UserLogin.class.isAssignableFrom(clazz);
    }

    public void validate(Object target, Errors errors) {
       ValidationUtils.rejectIfEmptyOrWhitespace(errors, "userName", "field.required");
       ValidationUtils.rejectIfEmptyOrWhitespace(errors, "password", "field.required");
       UserLogin login = (UserLogin) target;
       if (login.getPassword() != null
             && login.getPassword().trim().length() < MINIMUM_PASSWORD_LENGTH) {
          errors.rejectValue("password", "field.min.length",
                new Object[]{Integer.valueOf(MINIMUM_PASSWORD_LENGTH)},
                "The password must be at least [" + MINIMUM_PASSWORD_LENGTH + "] characters in );
       }
    }
}
```

1. 和上面一样也是实现org.springframework.validation.Validator接口,不过是需要验证的属性配置类本身去实现这个接口

```java
@ConfigurationProperties(prefix = "doc")
public class DocumentServerProperties implements Validator{
    @NotNull
    private String remoteAddress;
    private boolean preferIpAddress;
       //其他属性 
   
    @Override
    public boolean supports(Class<?> clazz) {
        return true;
    }

    @Override
    public void validate(Object target, Errors errors) {
        //判断逻辑其实可以参照上面的代码片段
    }
}
```

**特别注意**:

- 只有在需要使用JSR303规范实现的验证器时,才需要对对象配置@Validated,刚刚上面两种方式并不需要。
- 第一种实现和第二种实现都是实现org.springframework.validation.Validator接口,但是前者是针对全局的,后者只针对实现这个接口的配置对象

关于上述两点，我为啥确定? 来自*ConfigurationPropertiesBinder*的源码片段

```java
private List<Validator> getValidators(Bindable<?> target) {
    List<Validator> validators = new ArrayList<>(3);
    if (this.configurationPropertiesValidator != null) {
        validators.add(this.configurationPropertiesValidator);
    }
    if (this.jsr303Present && target.getAnnotation(Validated.class) != null) {
            validators.add(getJsr303Validator());
    }
    if (target.getValue() != null && target.getValue().get() instanceof Validator) {
        validators.add((Validator) target.getValue().get());
    }
    return validators;
}
```

### 总结

通过上面的例子,我们了解了@ConfigurationProperties的使用以及如何进行验证,包括属性验证器的几种实现方式.下个章节我会从源码的角度分析属性的加载,以及如何解析到Bean里面去的。
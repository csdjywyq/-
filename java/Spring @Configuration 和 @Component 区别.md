## Spring @Configuration 和 @Component 区别(精简)

### 汇总版本

@Configuration 中所有带 @Bean 注解的方法都会被动态代理，因此调用该方法返回的都是同一个实例。


@Configuration 注解本质上还是 @Component，因此\<context:component-scan/> 或者 @ComponentScan 都能处理@Configuration 注解的类。

 @Configuration 注解的 bean 都已经变成了增强的类。

### 案例描述

@Bean 注解方法执行策略

```java
@Configuration
public class MyBeanConfig {
 
    @Bean
    public Country country(){
        return new Country();
    }
 
    @Bean
    public UserInfo userInfo(){
        return new UserInfo(country());
    }
 
}
```

特别重要！！特别重要！！--》直接调用 country() 方法返回的是同一个实例，因为注解是@Configuration增强版本类

但是如果是变成@Component之后，此时返回的就不是一个实例了，每次都会创建一个实例

@Component 注解并没有通过 cglib 来代理@Bean 方法的调用

```java
@Component
public class MyBeanConfig {

    @Bean
    public Country country(){
        return new Country();
    }
     
    @Bean
    public UserInfo userInfo(){
        return new UserInfo(country());
    }
}
```

- 修正版本：此时也是一个实例（用@Autowired 注入）

这种方式可以保证使用的同一个 Country 实例。

```java
@Component
public class MyBeanConfig {

    @Autowired
    private Country country;
     
    @Bean
    public Country country(){
        return new Country();
    }
     
    @Bean
    public UserInfo userInfo(){
        return new UserInfo(country);
    }

}
```


@Configuration 标记的类必须符合下面的要求：

- 配置类必须以类的形式提供（不能是工厂方法返回的实例），允许通过生成子类在运行时增强（cglib 动态代理）。
- 配置类不能是 final 类（没法动态代理）。
- 配置注解通常为了通过` @Bean `注解生成 Spring 容器管理的类，
- 配置类必须是非本地的（即不能在方法中声明，不能是 private）。
- 任何嵌套配置类都必须声明为static。
- `@Bean` 方法可能不会反过来创建进一步的配置类（也就是返回的 bean 如果带有 `@Configuration`，也不会被特殊处理，只会作为普通的 bean）。
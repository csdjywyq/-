### @Required 注解

Spring 依赖检查 bean 配置文件用于确定的特定类型(基本，集合或对象)的所有属性被设置。在大多数情况下，你只需要确保特定属性已经设置但不是所有属性。对于这种情况，你需要 @Required 注解。

### @Autowired 注解

@Autowired 可以用来装配 bean，可以写在字段上，或者方法上。@Autowired 默认按类型装配，默认情况下要求依赖对象必须存在，如果要允许 null 值，可以设置它的 required 属性为 false。

### @Qualifier 注解

这个注解通常和@Autowired 一起使用，当你想对注入的过程做更多的控制，@Qualifier 可以帮助你指定做更详细配置。一般在两个或者多个 bean 是相同的类型，spring 在注入的时候会出现混乱。

### @Configuration和@bean注解

Spring 中为了减少 XML 配置，可以声明一个配置类类对 bean 进行配置，主要用到两个注解@Configuration 和@bean，定义一个配置类，用@Configuration 注解该类，等价于 XML 里的 beans，用@Bean 注解方法，等价于 XML 配置的 bean，方法名等于 bean Id。
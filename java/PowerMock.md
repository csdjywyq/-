## 1、为什么要用mock

一本书的解释：

　　（1）创建所需的DB数据可能需要很长时间，如：调用别的接口，模拟很多数据

　　（2）调用第三方API接口，测试很慢，

　　（3）编写满足所有外部依赖的测试可能很复杂，复杂到不值得编写，Mock模拟内部或外部依赖可以帮助我们解决这些问题

另一本TDD书的解释：

　　（1）对象的结果不确定，如每获取当前时间，得到的结果都不一样，无法符合我们的预期；

　　（2）实现这个接口的对象不存在；

　　（3）对象速度缓慢

　　对于TDD还有一个更重要原因:通过模拟可以隔离当前方法使用的的所有依赖，让我们更加专注于单个单元，忽略其调用的代码的内部工作原理

一本博客的干货：

　　（1）Mock可以用来解除测试对象对外部服务的依赖（比如数据库，第三方接口等），使得测试用例可以独立运行。不管是传统的单体应用，还是现在流行的微服务，这点都特别重要，因为任何外部依赖的存在都会极大的限制测试用例的可迁移性和稳定性。

　　（2）Mock的第二个好处是替换外部服务调用，提升测试用例的运行速度。任何外部服务调用至少是跨进程级别的消耗，甚至是跨系统、跨网络的消耗，而Mock可以把消耗降低到进程内。比如原来一次秒级的网络请求，通过Mock可以降至毫秒级，整整3个数量级的差别。

　　（3）Mock的第三个好处是提升测试效率。这里说的测试效率有两层含义。第一层含义是单位时间运行的测试用例数，这是运行速度提升带来的直接好处。而第二层含义是一个测试人员单位时间创建的测试用例数。

　　以单体应用为例，随着业务复杂度的上升，为了运行一个测试用例可能需要准备很多测试数据，与此同时还要尽量保证多个测试用例之间的测试数据互不干扰。为了做到这一点，测试人员往往需要花费大量的时间来维护一套可运行的测试数据。有了Mock之后，由于去除了测试用例之间共享的数据库依赖，测试人员就可以针对每一个或者每一组测试用例设计一套独立的测试数据，从而很容易的做到不同测试用例之间的数据隔离性。而对于微服务，由于一个微服务可能级联依赖很多其他的微服务，运行一个测试用例甚至需要跨系统准备一套测试数据，如果没有Mock，基本上可以说是不可能的。因此，不管是单体应用还是微服务，有了Mock之后，QE就可以省去大量的准备测试数据的时间，专注于测试用例本身，自然也就提升了单人的测试效率。

现如今比较流行的Mock工具如jMock 、EasyMock 、Mockito等都有一个共同的缺点：不能mock静态、final、私有方法等。而PowerMock能够完美的弥补以上三个Mock工具的不足

## 2、实战：

用PoweMockito框架，直接上代码：如何mock私有方法，静态方法，测试私有方法，final类

依赖：


```xml
　　　　　　<dependency>
            <groupId>org.powermock</groupId>
            <artifactId>powermock-api-mockito2</artifactId>
            <version>1.7.4</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.powermock</groupId>
            <artifactId>powermock-module-junit4</artifactId>
            <version>1.7.4</version>
            <scope>test</scope>
        </dependency>
```


powermock可能与mockito有版本冲突，我们可以讲mockito版本改成2.8.47  ：

```xml
　　<properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <mockito.version>2.8.47</mockito.version>
    </properties>
```

service层代码：一个简单的保存，返回user，这里就不调用持久层了

```java
  　@Override
    public User save(User user) {
        return user;
    }
```

controller层代码：一些简单的调用，接下来写个test 调用controller方法，mock service里面的保存，mock controller的私有方法和静态方法，测试controller的private的私有方法


```java
package com.example.demo.controller;

import com.example.demo.entity.User;
import com.example.demo.service.UserService;

public final class UserController {

    private final UserService service;

    public UserController(UserService service) {
        this.service = service;
    }

    public User saveUser(User user) {
        User save = service.save(user);
        return save;
    }

    public String returnName(){
       return getStaticName("ljw1");
    }

    public static String getStaticName(String name) {
        return "A_" + name;
    }
    public  String getPrivateName(String name) {

        if (publicCheck()){
            return "public 被mock 了";
        }
        if (check(name)){
            return "private 被mock 了";
        }
        return "A_" + name;
    }

    public boolean publicCheck() {
        return false;
    }

    private boolean check(String name) {
        return false;
    }

    private String say(String content) {
        return "ljw say " + content;
    }

}
```


test:测试
注意：测试private方法和mock private方法的区别


```java
package com.example.demo.demo1;

import com.example.demo.controller.UserController;
import com.example.demo.entity.User;
import com.example.demo.service.impl.UserServiceImpl;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.powermock.api.mockito.PowerMockito;
import org.powermock.core.classloader.annotations.PrepareForTest;
import org.powermock.modules.junit4.PowerMockRunner;
import org.powermock.reflect.Whitebox;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

import static org.junit.Assert.assertEquals;
import static org.mockito.ArgumentMatchers.any;
import static org.powermock.api.mockito.PowerMockito.when;
@RunWith(PowerMockRunner.class)
@PrepareForTest(UserController.class)
public class LastMock {
    @Mock
    private UserServiceImpl serviceImpl;

    @InjectMocks
    private UserController controller;

    /**
     * mock service的保存方法
     */
    @Test
    public void mockSave() {
        User user1 = new User();
        User user2 = new User();
        user1.setId("1");
        user2.setId("2");
        Mockito.when(serviceImpl.save(user1)).thenReturn(user2); //当调用service的save()时，mock让他返回user2
        User saveUser = controller.saveUser(user1); //调用
        Mockito.verify(serviceImpl,Mockito.times(1)).save(user1);//verify验证mock次数
        assertEquals(user2, saveUser);//断言是否mock返回的是user2
    }

    /**
     * mock spy public方法
     * @throws Exception xx
     */
    @Test
    public void spy_public_method() throws Exception {
        UserController spy = PowerMockito.spy(controller); //监视controller的publicCheck方法，让他返回true
        Mockito.when(spy.publicCheck()).thenReturn(true);
        String name = spy.getPrivateName("ljw");//执行该方法
        assertEquals("public 被mock 了", name);//验证
    }

    /**
     * mock私有方法
     * @throws Exception xx
     */
    @Test
    public void spy_private_method() throws Exception {
        UserController spy = PowerMockito.spy(controller);
        PowerMockito.when(spy, "check", any()).thenReturn(true);//私有方法mockito不行了，需要用无所不能的PowerMock监视spy
        String name = spy.getPrivateName("ljw");
        assertEquals("private 被mock 了", name);
    }

    /**
     * mock 静态方法
     */
    @Test
    public void mockStaticMethod() {
        PowerMockito.mockStatic(UserController.class);//mock静态方法
        when(UserController.getStaticName(any())).thenReturn("hi");
        String staticName = UserController.getStaticName("ljw");//执行
        assertEquals("hi", staticName);//验证
    }

    @Test
    public void mockStaticMethod_2() {
        PowerMockito.mockStatic(UserController.class);
        when(UserController.getStaticName(any())).thenReturn("hi");
        String staticName = controller.returnName();//通过returnName()调用，看能否被mock
        assertEquals("hi", staticName);
    }

    /**
     * 测试私有方法一
     * @throws InvocationTargetException xx
     * @throws IllegalAccessException xx
     */
    @Test
    public void testPrivateMethod() throws InvocationTargetException, IllegalAccessException {
        Method method = PowerMockito.method(UserController.class, "say", String.class);
        Object say = method.invoke(controller, "hi");
        assertEquals("ljw say hi", say);
    }

    /**
     * 测试私有方法二
     * @throws Exception xx
     */
    @Test
    public void testPrivateMethod_2() throws Exception {
        Object say = Whitebox.invokeMethod(controller, "say", "hi");
        assertEquals("ljw say hi", say);
    }

}
```
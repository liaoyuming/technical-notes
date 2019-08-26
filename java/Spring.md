## Spring 框架

![spring 框架总体结构](https://i.screenshot.net/lvd0qtm)

组成整个 Spring 框架的各种服务实现被划分到多个互相独立却又互相依赖的模块当中。

整个 Spring 框架是构建在 Core 核心模块之上的，它为 Spring 提供了 IoC 容器实现，用于帮助我们以依赖注入的方式管理对象之间的依赖关系。

![Spring 构成图](https://i.screenshot.net/rzd48tz)

## Spring IoC 容器

IoC 容器是 Spring 框架的核心。容器将创建对象，把它们连接在一起，配置它们，并管理他们的整个生命周期从创建到销毁。Spring 容器使用依赖注入（DI）来管理组成一个应用程序的组件。这些对象被称为 Spring Beans


### BeanFactory 接口
org.springframework.beans.factory.BeanFactory 是一个简单接口，所以可以针对各种底层存储方法实现。最常用的 BeanFactory 定义是 XmlBeanFactory，它根据 XML 文件中的定义装入 bean
```
XmlBeanFactory factory = new XmlBeanFactory
                             (new ClassPathResource("Beans.xml"));
HelloWorld obj = (HelloWorld) factory.getBean("helloWorld");

```
HelloWorld 实现：
```
package com.tutorialspoint;

public class HelloWorld {
   private String message;
   public void setMessage(String message){
      this.message  = message;
   }
   public void getMessage(){
      System.out.println("Your Message : " + message);
   }
}
```

Beans.xml 实现：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id="helloWorld" class="com.tutorialspoint.HelloWorld">
       <property name="message" value="Hello World!"/>
   </bean>

</beans>
```
### ApplicationContext 接口

最常被使用的 ApplicationContext 接口实现：
- FileSystemXmlApplicationContext： 该容器从 XML 文件中加载已被定义的 bean。在这里，你需要提供给构造器 XML 文件的完整路径。
    ```
    ApplicationContext context = new FileSystemXmlApplicationContext
                ("C:/Users/ZARA/workspace/HelloSpring/src/Beans.xml");
    HelloWorld obj = (HelloWorld) context.getBean("helloWorld");
    ```
- ClassPathXmlApplicationContext：该容器从 XML 文件中加载已被定义的 bean。在这里，你不需要提供 XML 文件的完整路径，只需正确配置 CLASSPATH 环境变量即可，因为，容器会从 CLASSPATH 中搜索 bean 配置文件。
    ```
    ApplicationContext context = 
                 new ClassPathXmlApplicationContext("Beans.xml");
    HelloWorld obj = (HelloWorld) context.getBean("helloWorld");
    ```

- WebXmlApplicationContext：该容器会在一个 web 应用程序的范围内加载在 XML 文件中已被定义的 bean。

### Spring Bean

Bean 与 Spring 容器之间的关系：

![spring bean](https://i.screenshot.net/v94q0cv)

#### Bean 配置
- 基于 XML 的配置文件
- 基于注解的配置
    - [@Required](https://www.w3cschool.cn/wkspring/9sle1mmh.html)
    - [@Autowired](https://www.w3cschool.cn/wkspring/rw2h1mmj.html)
    - [@Qualifier](https://www.w3cschool.cn/wkspring/knqr1mm2.html)
- 基于 Java 类的配置
    ```
    package com.tutorialspoint;
    import org.springframework.context.annotation.*;
    @Configuration
    public class HelloWorldConfig {
       @Bean 
       public HelloWorld helloWorld(){
          return new HelloWorld();
       }
    }
    ```
    ```
    ApplicationContext ctx = new AnnotationConfigApplicationContext(HelloWorldConfig.class); 
   HelloWorld helloWorld = ctx.getBean(HelloWorld.class);
    ```

## Spring MVC

Spring MVC从接收请求到返回响应的处理流程:

![spring mvc request life cycle](https://i.screenshot.net/pnkd5cn)

1. DispatcherServlet 接收请求。
2. DispatcherServlet 调度选择适合的 Controller 到 HandlerMapping。HandlerMapping 选择映射到传入请求URL的 Controller，并将（选择的 Handler）和 Controller 返回给 DispatcherServlet。
3. DispatcherServlet 将执行 Controller 的业务逻辑的任务分派给 HandlerAdapter。
4. HandlerAdapter 调用 Controller 的业务逻辑过程。
5. Controller 执行业务逻辑，在 Model 中设置处理结果，并将视图的逻辑名称返回给 HandlerAdapter。
6. DispatcherServlet 调度 ViewResolver。ViewResolver 返回了对应视图名的 View。
7. DispatcherServlet 执行返回的 View。
8. View 呈现 Model 数据并返回响应。


### 配置
- web.xml 中配置 Servlet
    ```
    <web-app id="WebApp_ID" version="2.4"
       xmlns="http://java.sun.com/xml/ns/j2ee" 
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee 
       http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
    
       <display-name>Spring MVC Application</display-name>
    
       <servlet>
          <servlet-name>SpringMVC</servlet-name>
          <servlet-class>
             org.springframework.web.servlet.DispatcherServlet
          </servlet-class>
          <load-on-startup>1</load-on-startup>
       </servlet>
    
       <servlet-mapping>
          <servlet-name>SpringMVC</servlet-name>
          <url-pattern>/</url-pattern>
       </servlet-mapping>
       
    </web-app>

    ```
- 创建 Spring MVC 的 xml 配置文件
    ```
    <!-- WEB-INF/SpringMVC-servlet.xml -->
    <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans     
       http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
       http://www.springframework.org/schema/context 
       http://www.springframework.org/schema/context/spring-context-3.0.xsd">
    
       <context:component-scan base-package="com.springmvc" />
    
    </beans>
    ```
    - component-scan标签默认情况下自动扫描指定路径下的包（含所有子包），将带有 @Component、@Repository、@Service、@Controller标签的类自动注册到 spring 容器。对标记了 @Required、@Autowired、@Resourcet 等注解的类进行对应的操作使注解生效
        - getBean的默认名称是类名（头字母小写）

### 定义控制器
```
@Controller
@RequestMapping("/hello")
public class HelloController{

   @RequestMapping(value = "/world" method = RequestMethod.GET)
   public String world(@RequestParam(required = false) String test) {
      return "hello world";
   }

}
```
### 注解
- @Controller 定义该类是作为控制器的角色。
- @RequestMapping 用于将 URL 映射到整个类或特定处理程序方法。
- @RequestBody 将 HTTP 请求正文转换为适合的 HttpMessageConverter 对象。
- @ResponseBody 将内容或对象作为 HTTP 响应正文返回，并调用适合 HttpMessageConverter 的 Adapter 转换对象，写入输出流。
- @RequestParam 用于在控制层获取参数
- @PathVariable 用于将请求 URL 中的模板变量映射到处理方法的参数上。
    ```
    @RequestMapping("/user/{id}")  
    public void find(@PathVariable String id) {}
    ```
- @Repository 用于注解 dao 层，在 daoImpl 类上面注解。
- @Service 用于注解业务逻辑层，就是 service 或者 manager 层
- @Component 字面意思就是组件，它在你确定不了事哪一个层的时候使用

## Spring AOP 
待写
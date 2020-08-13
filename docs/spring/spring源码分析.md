# spring常见的面试题

## 1.什么是spring框架？

Spring框架是一个为Java应用程序开发提供综合、广泛的基础性支持的Java平台。Spring帮助开发者解决了开发中基础性的问题，使得开发人员可以专注于应用程序的开发。Spring框架本身也是按照设计模式精心打造，这使得我们可以在开发环境中安心地集成Spring框架，不必担心Spring是如何在后台工作的。

## 2.使用spring框架能带来哪些好处？

Spring并没有闭门造车，Spring利用了已有的技术，比如ORM框架、logging框架、J2EE、Quartz和JDK Timer，以及其他视图技术。

Spring的Web框架也是一个精心设计的Web MVC框架，为开发者们在Web框架的选择上提供了一个除主流框架比如Struts、过度设计的、不流行Web框架以外的选择。

## 3.什么是控制反转，什么是依赖注入？

控制反转：自己的理解：就是将对象的创建交给框架来做，这是一种思想。

依赖注入：编译阶段尚未知所需的功能是来自哪个的类的情况下，将其他对象所依赖的功能对象实例化的模式。

## 4.在java中依赖注入有哪些方式？

1. 构造器注入
2. setter注入
3. 接口注入

## 5.BeanFactory和ApplicationContext有什么区别？



## 6.spring提供几种配置方式来设置原数据



（1）基于XML的配置。
（2）基于注解的配置。
（3）基于Java的配置。

## 7.如何使用XML配置的方式配置spring



```java
<beans>
   <!-- JSON Support -->
   <bean name="viewResolver"
        class="org.springframework.web.servlet.view.BeanNameViewResolver"/>
   <bean name="jsonTemplate"
        class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
   <bean id="restTemplate" class="org.springframework.web.client.RestTemplate"/>
</beans>
```



## 8.Spring提供哪些配置形式？

spring对java配置的支持是由@Configuration注解和@Bean注解实现的。由@Bean注解的方法进行实例化、配置和初始化一个新的对象。这个对象就由spring的IOC容器来管理。



```java
@Configuration
public class AppConfig{
   @Bean
   public MyService myService() {
      return new MyServiceImpl();
   }
}
```

```java
<beans>
   <bean id="myService" class="com.gupaoedu.services.MyServiceImpl"/>
</beans>
```

## 9.怎么用注解的方式配置Spring

```java
<beans>
   <context:annotation-config/>
</beans>
```

## 10.请解释spring bean的生命周期？

Spring

Spring Bean的生命周期简单易懂。在一个Bean实例被初始化时，需要执行一系列初始化操作以达到可用的状态。同样，当一个Bean不再被调用时需要进行相关的析构操作，并从Bean容器中移除。 
Spring Bean Factory 负责管理在Spring容器中被创建的Bean的生命周期。Bean的生命周期由两组回调方法组成。 
（1）初始化之后调用的回调方法。 
（2）销毁之前调用的回调方法。 
Spring框架提供了以下四种方式来管理Bean的生命周期事件：
（1）InitializingBean和DisposableBean回调接口。
（2）针对特殊行为的其他Aware接口。
（3）Bean配置文件中的Custom init()方法和destroy()方法。
（4）@PostConstruct和@PreDestroy注解方式。
使用customInit()和 customDestroy()方法管理Bean生命周期的代码样例如下：

## 11.spring bean作用域的区别是什么？

1. 1.singleton:这种bean范围是默认的，这种范围确保不管接收到多少请求，每一个容器中只有一个Bean的实例，单例的模式是由Bean Factory自身来维护。
2. prototype：与单例模式相反，为每一个bean请求提供一个实例。
3. Session:与请求范围类似，确保每一个Session中有一个Bean的实例，在session过后，Bean也会随之失效。
4. request：在请求范围内为每一个来自客户端的网络请求创建一个实例，在请求完成之后，bean会失效并被垃圾回收。

## 12.什么是Spring Inner Bean?



## 13.spring框架中的单例bean是线程安全的吗？

​        spring框架并没有对单例bean进行任何多线程的封装处理。关于单例Bean的线程安全和并发问题需要开发者自行搞定。

​			但实际上，大部分Spring Bean并没有可变的状态（比如Serview类和DAO类），所以在某种程度上说，Spring的单例Bean是线程安全的。如果你的Bean有多种状态（比如View Model对象），就需要自行保证线程安全。最浅显的解决办法就是将多态Bean的作用域由“singleton”变更为“prototype”。

**Spring容器中的Bean本身不具备线程安全的特性**

原型bean ：对于单例bean，都会创建一个新的对象，也就是说线程之间不存在bean共享，自然不会有线程安全性问题。



单例bean：对于单例bean，所有的线程都共享一个单例实例bean，因此会存在资源的竞争。

如果单例bean是一个没有状态的bean，就只有查询操作，不会有其他的操作，那么这个bean就是线程安全的，比如spring MVC的controller service dao,这些bean大多数都是无状态的，值关注方法本身。

对于有状态的bean，Spring官方提供的bean，一般提供了通过ThreadLocal去解决线程安全的方法，比如RequestContextHolder、TransactionSynchronizationManager、LocaleContextHolder等。

## 14.使用@Autowired和@Resource的区别

相同点：

@Resource的作用相当于@Autowired,都可以标注在字段或者属性的setter的方法上。

主要的区别就是：@Auto  是安装属性进行装配的，@Resource是按照名称进行装配的

不同点：

@Autowired默认是按照**类型**装配的（这个注解是属于spring的），默认情况下必须要求**依赖的对象必须存在**，如果允许为null，可以设置它的required的属性为false，如@Autowired(required=false)



@Resource 是jdk1.6支持的注解，默认按照名称进行装配，名称可以通过name属性进行指定。



## 15.请解释spring bean的自动装配？

- 通过xml文件进行注入。

```java
<bean id="employeeDAO" class="com.gupaoedu.EmployeeDAOImpl" autowire="byName" />
    
```

- 通过注解的方式进行自动装配

## 16.请举例解释@Required Annotation？

**@Required** 注释应用于 bean 属性的 setter 方法，**是用于检查一个Bean的属性在配置期间是否被赋值。**
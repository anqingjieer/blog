springboot的循环依赖怎么解决

## 1.循环依赖是什么？

Bean A 依赖   Bean B, Bean B  依赖  Bean A这种情况出现的循环依赖。

Bean A → Bean B → Bean A
更复杂的间接依赖造成的循环依赖如下。
Bean A → Bean B → Bean C → Bean D → Bean E → Bean A

## 2.循环依赖会产生什么后果

当spring正在加载所有的Bean时，Spring尝试以能正常创建Bean的循序去创建Bean。

如果存在以下依赖：

Bean A → Bean B → Bean A

存在循环依赖时，spring将无法决定先创建那个bean，这时会抛异常 BeanCurrentlyInCreationException

## 3.普通注入之间的循环依赖

```java
@Component
public class A {
 
  private B b;
 
  public void setB(B b) {
    this.b = b;
  }
}

@Component
public class B {
 
  private A a;
 
  public void setA(A a) {
    this.a = a;
  }
}
```

使用延迟加载的方法去解决

注入的时候  加上  @Lazy

## 4.构造器注入循环依赖实例

通过   Setter进行注入

Spring文档建议的一种方式是使用setter注入。当依赖最终被使用时才进行注入。对前文的样例代码少做修改，来观察测试效果。

```java

@Component
public class CircularDependencyA {
 
    private CircularDependencyB circB;
 
    @Autowired
    public void setCircB(CircularDependencyB circB) {
        this.circB = circB;
    }
 
    public CircularDependencyB getCircB() {
        return circB;
    }
}
@Component
public class CircularDependencyB {
 
    private CircularDependencyA circA;
 
    private String message = "Hi!";
 
    @Autowired
    public void setCircA(CircularDependencyA circA) {
        this.circA = circA;
    }
 
    public String getMessage() {
        return message;
    }
}
```

## 5.spring bean的创建

​          对象的创建一般分为二部分一个是当前对象实例化和当前对象属性的实例化，在spring中，对象的实例化一般是通过反射区实现的，而对象的属性是在对象实例化之后通过一定的方式设置的，这个过程可以按照如下方式进行理解：

![image-20200724102509427](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200724102523.png)

过程演示：

A和B相互依赖

```java
@Component
public class A {

  private B b;

  public void setB(B b) {
    this.b = b;
  }
}
```



```java
@Component
public class B {

  private A a;

  public void setA(A a) {
    this.a = a;
  }
}
```

![image-20200724160955772](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200724160955.png)



​          首先Spring尝试通过ApplicationContext.getBean()方法获取A对象的实例，由于Spring容器中还没有A对象实例，因而其会创建一个A对象，然后发现其依赖了B对象，因而会尝试递归的通过ApplicationContext.getBean()方法获取B对象的实例，但是Spring容器中此时也没有B对象的实例，因而其还是会先创建一个B对象的实例。

​         读者需要注意这个时间点，此时A对象和B对象都已经创建了，并且保存在Spring容器中了，只不过A对象的属性b和B对象的属性a都还没有设置进去。

​         在前面Spring创建B对象之后，Spring发现B对象依赖了属性A，因而此时还是会尝试递归的调用ApplicationContext.getBean()方法获取A对象的实例，因为Spring中已经有一个A对象的实例，虽然只是半成品（其属性b还未初始化），但其也还是目标bean，因而会将该A对象的实例返回。

​       此时，B对象的属性a就设置进去了，然后还是ApplicationContext.getBean()方法递归的返回，也就是将B对象的实例返回，此时就会将该实例设置到A对象的属性b中。

​     这个时候，注意A对象的属性b和B对象的属性a都已经设置了目标对象的实例了。读者朋友可能会比较疑惑的是，前面在为对象B设置属性a的时候，这个A类型属性还是个半成品。

​		但是需要注意的是，这个A是一个引用，其本质上还是最开始就实例化的A对象。

​		而在上面这个递归过程的最后，Spring将获取到的B对象实例设置到了A对象的属性b中了，这里的A对象其实和前面设置到实例B中的半成品A对象是同一个对象，其引用地址是同一个，这里为A对象的b属性设置了值，其实也就是为那个半成品的a属性设置了值。





- **Spring是通过递归的方式获取目标bean及其所依赖的bean的；**
- **Spring实例化一个bean的时候，是分两步进行的，首先实例化目标bean，然后为其注入属性。**


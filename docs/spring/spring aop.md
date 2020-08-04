# spring aop

## 一、spring aop的应用场景

AOP 是OOP 的延续，是Aspect Oriented Programming 的缩写，意思是面向切面编程。可以通过**预编译方式**和**运行期动态代理实现在不修改源代码**的情况下给程序动态统一添加功能的一种技术。

AOP实现了把业务需求和系统需求分开来做。

### aop中重要的概念

#### 1.切面（Aspect）

一个关注点的模块化，这个关注点可能会横切多个对象.切面”在ApplicationContext 中<aop:aspect>来配置。

#### 2.通知（Adcie）

“切面”对于某个“连接点”所产生的动作。其中，一个“切面”可以包含多个“Advice”。

#### 3.切入点

#### 4.目标对象

#### 5.aop代理

在Spring AOP 中有两种代理方式，JDK 动态代理和CGLib 代理。默认情况下，TargetObject 实现了接口时，则采用JDK 动态代理，例如，AServiceImpl；反之，采用CGLib 代理，例如，BServiceImpl。强制使用CGLib 代理需要将<aop:config>的proxy-target-class 属性设为true。通知（Advice）类型：

#### 6.前置通知

#### 7.后置通知

#### 8.返回时通知

#### 9.环绕通知

#### 10异常通知

## 二、采用代理模式

Spring提供了两种方式来生成代理对象: JDKProxy和Cglib，具体使用哪种方式生成由AopProxyFactory根据AdvisedSupport对象的配置来决定。默认的策略是如果目标类是接口，则使用JDK动态代理技术，否则使用Cglib来生成代理。
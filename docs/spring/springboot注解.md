注解的使用

@ComponentScan

扫描包下面所有的类，并其加入到spring容器中。

这个注解是大家接触得最多的了，相当于 xml 配置文件中的 context:component scan> 。

扫描：@Component 、@Repository 、 @Service 、 @Controller 这类的注解标识的。默认会扫描当前包下面所有的类。

@Configuration

是jdk1.5出现的，主要是使用注解。

通过使用注解的方式代替配置文件。

```java
@Configuration
public class TestConfig {

    public TestConfig(){
        System.out.println("testconfig collection  init success");
    }
}

public class Main {

    public static void main(String[] args) {
        // @Configuration注解的spring容器加载方式，用AnnotationConfigApplicationContext替换ClassPathXmlApplicationContext
        ApplicationContext context = new AnnotationConfigApplicationContext(TestConfig.class);

        // 如果加载spring-context.xml文件：
        // ApplicationContext context = new
        // ClassPathXmlApplicationContext("spring-context.xml");
    }
}
```



@**`EnableAutoConfiguration`**

目的就是动态的注入bean

```java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!this.isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    } else {
        AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = 			this.getAutoConfigurationEntry(annotationMetadata);
        return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }
}
```

```java
protected AutoConfigurationImportSelector.AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
    if (!this.isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    } else {
        //获取配置文件
        AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
        List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
        configurations = this.removeDuplicates(configurations);
        Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
        this.checkExcludedClasses(configurations, exclusions);
        configurations.removeAll(exclusions);
        configurations = this.getConfigurationClassFilter().filter(configurations);
        this.fireAutoConfigurationImportEvents(configurations, exclusions);
        return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
    }
}
```



 条件注入。

1. 此时如果没有禁用自动装配则进入else分枝，第一步操作首先会去加载所有Spring预先定义的配置条件信息，这些配置信息在`org.springframework.boot.autoconfigure`包下的`META-INF/spring-autoconfigure-metadata.properties`文件中
2. 这些配置条件主要含义大致是这样的：如果你要自动装配某个类的话，你觉得先存在哪些类或者哪些配置文件等等条件，这些条件的判断主要是利用了`@ConditionalXXX`注解，关于`@ConditionalXXX`系列注解可以参考这篇文章：[SpringBoot条件注解@Conditional](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s/RXYIh_g5iU1e3liK-8n5zA)
3. 这个文件里的内容格式是这样的：
4. 
5. ![image-20200618103622404](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200618103622.png)
6. 接下来就是扫描spring。factories文件。可以进行重写faxtories,



1. 基于你对springboot的理解描述一下什么是springboot？

   基于spring封装的框架是一种框架上的框架，简化了配置。

2. 约定优于配置指的是什么？

   springboot的优势就是：在传统所需要配置的地方，springboot都进行了约定（配置好了），开发人员配置的越少，更能直接的开发项目，写业务逻辑

3. @SpringBootApplication由哪几个注解组成，这几个注解分别表示什么作用

   @ComponentScan  扫描本包下面所有的包，将增加有注解@Crotller @Service @Comment的类增加到IOC容器中。

   @Configuration 基于注解进行注入

   **`@EnableAutoConfiguration`**实现自动注入功能，首先 扫描 spring boot目录下面的spring-autoconfigure-metadata.properties`，将符合条件的bean加载到容器中。（可以进行筛选）

   其次 扫描 spring.factories里面相关的bean进行注入。

   

4. springboot自动装配的实现原理

5. spring中的spi机制的原理是什么？

   对spring.factorties里面的类，进行扩展。
   
   

## 二、什么是starter

Starter是springboot中一个非常重要的概念，starter相当于模块，它能将模块所依赖正好起来并对模块内的Bean环境进行自动装配，。

### 1.自定义一个starter

重写一个spring.factories

```xml
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.gupaoedu.starter.autoconfiguration.HelloAutoConfiguration
```

 定义一个 HelloAutoConfiguration



```java
@Import(FormatAutoConfiguration.class)
@EnableConfigurationProperties(HelloProperties.class)
@Configuration
public class HelloAutoConfiguration {

    @Bean
    public HelloFormatTemplate helloFormatTemplate(HelloProperties helloProperties,FormatProcessor formatProcessor){
        return new HelloFormatTemplate(helloProperties,formatProcessor);
    }
}
```

```java
// 定义公司前缀
@ConfigurationProperties(prefix=HelloProperties.HELLO_FORMAT_PREFIX)
public class HelloProperties {

    public static final String HELLO_FORMAT_PREFIX="gupao.hello.format";
    private Map<String,Object> info;

    public Map<String, Object> getInfo() {
        return info;
    }

    public void setInfo(Map<String, Object> info) {
        this.info = info;
    }
}

```

```java
//将其注入
@Configuration
public class FormatAutoConfiguration {

    /**
     *
     * @return
     *
     */

    //metadata-auto....当Spring Context中不存在该Bean时
    //进行条件注入 当classpath下发现该类的情况下进行自动配置
    @ConditionalOnMissingClass("com.alibaba.fastjson.JSON")
    @Bean
    @Primary
    public FormatProcessor stringFormat(){
        return new StringFormatProcessor();
    }

    @ConditionalOnClass(name = "com.alibaba.fastjson.JSON")
    @Bean
    public FormatProcessor jsonFormat(){
        return new JsonFormatProcessor();
    }

}
```


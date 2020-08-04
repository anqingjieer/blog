spring MVC的请求处理过程

![image-20200701094904819](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200701094904.png)

过程一：、DispatcherServlet 是SpringMVC 中的前端控制器(Front Controller),负责接收Request 并将Request 转发给对应的处理组件。

过程二：  HandlerMapping是spring MVC中完成url到controller映射的组件。

过程三：Controller处理Request,并返回ModelAndView对象，controller是负责处理request的组件，modelandview是封装结果的视图。

过程四五六视图解析器ModelAndView对象并返回对应的视图给客户端。

![image-20200701143201743](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200701143201.png)

![image-20200701143109101](https://gitee.com/anqingjieer/pengbo/raw/master/img/20200701143109.png)







容器初始化的时候，会将url和controller中的method的对应关系保存到HandleMapping中。用户请求根据request请求的url快速定位到Controller中的某个方法中。

在Spring 中先将url 和Controller 的对应关系,保存到Map<url,Controller>中

## 一、初始化阶段



spring MVC的九大组件。

### 1.handlermapping

保存映射关系

### 2.handlerAdapters

参数转换

### 3.HandlerExceptionResolvers

处理异常机制 404 500

### 4.ViewResolver

视图解析器。 

### 5.requestToViewNameTranslator

把一个请求转换为为一个可以展示

### 6.LocalResolver

### 7.ThemeResolver

​	主题解析

### 8.MultipartResolver

​	上传文件的一个

### 9.FlashMapManager

flashmap




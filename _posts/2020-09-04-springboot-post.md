---
layout: post
title: SpringBoot note 01
date: 2020-09-04 15:00:00 +0800
category: springboot
thumbnail: /style/image/note.png
icon: note
---

* content
{:toc}

springboot的学习记录过程，更详细的[官方文档](https://snailclimb.gitee.io/springboot-guide/)

# SrpingBoot启动类
通过**@SpringBootApplication**标注该类是一个SpringBoot的启动类


```java
//@SpringBootApplication注解的实现
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
    ......
}
```
其中重要的注解有
* @SpringBootConfiguration
* @EnableAutoConfiguration 
* @ComponentScan

## @SpringBootConfiguration
它的代码为
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {

}
```
它允许在上下文中注册额外的bean或导入其他配置类


## @EnableAutoConfiguration  
用于启用SpringBoot的自动配置机制


## @ComponentScan
扫描被@Component (@Service,@Controller)注解的bean，注解默认会扫描该类所在的包下所有的类。




# @Controller 
通过@Controller和@RequestMapping(@GetMapping/@PostMapping)来指定请求的路径

> 可以指定RequestMethod的具体类型
> @RequestMapping(method=RequestMethod.GET,value="hello") 表示这是一个get请求,等同于@GetMapping("hello")


响应前端请求时，一般返回内容有 视图 和 json 两种方式

开发前后端分离的项目时，一般通过restful接口实现,后端将数据直接返回
**@ResponseBody** 会将return的内容以json的形式写入到response中

在Spring 4 之后提供了@RestController (@Controller+@ResponseBody)
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
public @interface RestController {
    ...
}
```
`

## 使用@RestController
```java
@RestController
@RequestMapping("hello")
public class HelloController {

    @GetMapping("user")
    public String helloDefault(String name){
        if(name == null){
            return "Hello, Stranger:<br> welcome to use Spring boot";
        }
        return  "Hello, " + name +":<br> welcome to use Spring boot";
    }
}
```
项目启动后可以通过访问

* *localhost:port/hello/user* (不传name值)
* *localhost:port/hello/user?name=gaoooyh* (get方式传入name值)

得到对应的相应内容

RESTful Web 服务与传统的 MVC 开发一个关键区别是返回给客户端的内容的创建方式：传统的 MVC 模式开发会直接返回给客户端一个视图，但是 RESTful Web 服务一般会将返回的数据以 JSON 的形式返回，这也就是现在所推崇的前后端分离开发

目前开发模式更倾向于前后端分离，各司其职后端将数据以json的形式返回给前端，前端解析json并进行展示


## 使用@Controller

```java
//Controller部分
@Controller
@RequestMapping("stu")
public class StudentController {
    @RequestMapping("list")
    public String list(Model model){
        model.addAttribute("name","gaoooyh");
        model.addAttribute("gender","♂");
        List<Student> list = new ArrayList<>();
        list.add(new Student(20,"gaoooyh","16130140336"));
        list.add(new Student(20,"xduzy","16130140372"));
        model.addAttribute("stuList",list);
        return "stu/list";
    }
}
```
对应的 stu/list.ftl文件
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
Hello : ${gender}${name}
<br>
<table border = "1">
    <tr>
        <td>StuName</td>
        <td>Age</td>
        <td>StuId</td>
    </tr>
    <#list stuList?sort_by("age")? reverse as stu>
        <tr>
            <td>${stu.name}</td>
            <td>${stu.age}</td>
            <td>${stu.stuId}</td>
        </tr>
    </#list>
</table>
</body>
</html>
```
不使用@ResponseBody时 MVC会匹配 resources/templates路径下的 stu/list.ftl模板，将模板内容返回给用户，在模板中读取后端传过来的model中的数据  

此处使用的前端模板引擎是 **freemarker**,对应的依赖为：
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```
目前SpringBoot更推荐使用 **thymeleaf** 模板引擎
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```




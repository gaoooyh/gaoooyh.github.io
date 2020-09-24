# 几种读取文件方式的区别

* Class.getResource()
* Class.getResourceAsStream()
* FileInputStream()

## FileInputStream 

从当前工作目录的相对路径中加载文件

用于从本地磁盘文件系统读取文件

首选提供绝对路径，因为您无法控制当前的工作路径。

从给定的相对路径加载文件可能会在Web应用程序中抛出FileNotFoundException



## getResource
根路径为target/classes

getResource的实现代码为
```
    public java.net.URL getResource(String name) {
        name = resolveName(name);
        ClassLoader cl = getClassLoader0();
        if (cl==null) {
            // A system class.
            return ClassLoader.getSystemResource(name);
        }
        return cl.getResource(name);
    }
```


假设我们有如下目录
```
|--target 
    |--classes
        |--com.demo
            |--Test.class 
            |--file1.txt 
        |--file2.txt
```
填写路径时以/开始,如 /... 即为 target/classes/... 

    eg: class.getResource("/com/demo/file1.txt");
    eg: class.getResource("file1.txt");

* this.getClass().getResource("");
不以'/'开头时是.class类所在的包路径(Test.class 和 file1.txt在同一个包下)

* this.getClass().getResource("/...");
 以'/'开头时是从项目的ClassPath根下获取资源
从classpath的根路径获取 (target/classes/...)

* this.getClass().getClassLoader().getResource("...")
  从项目的ClassPath根下获取资源
  
    >  **class.getResource("/...") == class.getClassLoader().getResource("...")**



## getResourceAsStream
getResourceAsStream() 方法仅仅是获取对应路径文件的输入流，在路径的用法上与getResource()一致

getResourceAsStream()方法的参数与getResouce()方法是一样的，它相当于你用getResource()取得File文件后，再new InputStream(file)一样的结果 

******

# 总结 
* getResource()和getResourceAsStream必须使用相对路径, 从相对于应用程序的类路径加载文件,通常在网络应用中使用.  它使用类名的类加载器查找和加载文件/资源​​,并且可以加载JAR文件中的资源

* class.getResource() 不带"/"时候是从当前类所在包路径去获取资源, 带"/"时候是从classpath的根路径获取
* class.getResource() 本质上也是调用了getClassLoader，只是封装了一层方便了我们使用而已
* class.getClassLoader().getResource("") 不带"/"时候是从classpath的根路径获取, 带有"/"会报错
* getResourceAsStream() 方法仅仅是获取对应路径文件的输入流，在路径的用法上与getResource()一致


* 当具有文件的绝对路径时使用FileInputStream加载文件

*****

使用栗子: 
```
//当前线程的类加载器,不依赖于调用它的类
InputStream inputStream = Thread.currentThread().getContextClassLoader().getResourceAsStream("files/file.txt");
//当前类的类加载器
InputStream inputStream = getClass().getClassLoader().getResourceAsStream("files/file.txt");


InputStream inputStream = new FileInputStream("E:\workspace\springboot-demo\src\main\resources\files\file.txt");

InputStream inputStream = getClass().getResourceAsStream("/files/file.txt");  
InputStream inputStream = getClass().getResource("/files/file.txt").openStream();

//使用ResourceUtil和classpath获取resource路径
InputStream inputStream = new FileInputStream(ResourceUtils.getFile("classpath:files/file.txt"));
InputStream inputStream =new ClassPathResource("files/file.txt").getInputStream();
```


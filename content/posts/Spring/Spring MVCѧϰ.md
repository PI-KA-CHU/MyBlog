+++
author = "pikachu"
title = "Spring MVC学习"
date = "2019-06-28"
description = " "
tags = [
	"",
    "spring"
]

categories = [
    "it", "spring"

]

+++



> 最近在学习Spring的相关模块及源码，在学习Bean的相关创建方法时，发现spring的bean一般都是单例模式，在xml中可以通过`scope=prototype`、注解中可以通过`@Scope("prototypr")`设置为多例模式（`singleton`为单例模式），不禁想到自己项目组中的Spring MVC中使用的是单例还是多例？下面是对Controller层的代码测试：

&nbsp;

## 1、Spring MVC的单例和多例测试

- 下面代码中在controller层中加入成员变量（此处添加只为测试，为非线程安全），现在通过调用此url接口，发现输出的test分别为：`1,2,3...`，说明conreoller层为单例模式，只生成了一个实例，加入`@Scope("prototype")`的注解后，输出结果为：`1,1,1...`，说明为多例模式(原型模式)，每次请求都会生成对象。

```
@Controller
@RequestMapping("/api/login")
public class LoginController {

    private Logger logger = LoggerFactory.getLogger(this.getClass());
    
    private int test = 1; //非线程安全
    
    @Autowired
    private LoginService loginService;

    @Autowired
    private LoginDao loginDao;

    @RequestMapping(value = "/admin", method = RequestMethod.GET)
    public void adminLogin() {
    	System.out.println(test ++);
    }
```

&nbsp;

## 2、单例和多例的应用场景
- 如果对象是`无状态的`（如上面的LoginService），或者对象的成员变量（共享）是线程安全的，如静态变量，常量，Threadlocal变量等，则可以使用单例模式，这样在性能上能够大大提升，减少了实例对象的开销。
- 如果对象存在线程不安全的变量，则需要采用`多例模式`，否则会出现线程安全问题。（一般情况下都是将变量修改为线程安全模式），所以在controller的编程中尽量不要出现成员变量。

&nbsp;

## 3、单例模式的扩展
- 单例模式有懒汉模式和饿汉模式
    - **饿汉模式**：在启动Bean容器时，为xml中配置的bean都实例化一个对象（`Spring的默认形式`）
    - **懒汉模式**：第一个请求时才创建实例，后续的请求都使用此实例。
    

&nbsp;

#### 参考
- https://blog.csdn.net/qianyiyiding/article/details/77104736
- https://blog.csdn.net/qq_35661171/article/details/83180546
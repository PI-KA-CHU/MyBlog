+++
author = "pikachu"
title = "SpringBoot2.0静态变量注入及过滤器的使用"
date = "2019-01-29"
description = " "
tags = [
	"",
	"spring-boot"
]

categories = [
    "it", "spring"

]

+++


## 一、实现静态变量的注入

- **参考：**

    - SpringBoot的常见注解：https://blog.csdn.net/fxbin123/article/details/80387668
    - https://blog.csdn.net/RogueFist/article/details/79575665
    - https://my.oschina.net/u/2617082/blog/1924530
    
- **问题：**

    最近使用封装的工具类时遇到了无法正常自动注入的问题，在工具类中对静态成员变量使用`@Autowired`进入自动注入，虽然编译正常，但是在运行的时候会报`java.lang.NullPointerException: null`异常。

- **原因：**

    在Springframework里，我们是不能@Autowired一个静态变量，使之成为一个Spring bean的。因为当类加载器加载静态变量时，Spring上下文尚未加载。所以类加载器不会在bean中正确注入静态类，并且会失败。

- **解决：**( @Component用于将类注册到Spring中,注册完记得重新打包一下项目war包)

    - **方法一：**
    
        通过@Autowired**注解构造函数**的方式进行注入：Spring扫描到AutowiredTypeComponent的bean，然后赋给静态变量component。
        
        ```
        @Component
        public class TestClass {
        
            private static AutowiredTypeComponent component;
        
            @Autowired
            public TestClass(AutowiredTypeComponent component) {
                TestClass.component = component;
            }
        
            // 调用静态组件的方法
            public static void testMethod() {
                component.callTestMethod()；
            }
        
        }
        ```
    
    - **方法二：**
    
        **给静态组件加setter方法，并在这个方法上加上@Autowired**：Spring能扫描到AutowiredTypeComponent的bean，然后通过setter方法注入
        
        ```
        @Component
        public class TestClass {
        
            private static AutowiredTypeComponent component;
        
            @Autowired
            public void setComponent(AutowiredTypeComponent component){
                TestClass.component = component;
            }
        
            // 调用静态组件的方法
            public static void testMethod() {
                component.callTestMethod()；
            }
        
        }
        ```

&nbsp;
&nbsp;

## 二、过滤器的使用

- **参考：**

    - Servlet3.0中@WebFilter的新特性：
      https://blog.csdn.net/u012334071/article/details/42131943

- **问题：**
  
    - 没有使用web.xml文件，不知如何进行监听器的注册
    - 过滤器对`/*`进行过滤时登陆会出现死循环过滤的情况

- **解决：**
  
    - 对监听用的类使用@WebFilter注解进行注册
    - 将静态资源及登陆界面和登陆方法进行过滤

- **代码示例：**
    ```
    @WebFilter(filterName = "loginFilter",urlPatterns = {"/*"})
    public class LoginFilter implements Filter {
    
        private Logger log = LoggerFactory.getLogger(this.getClass());
        private static final String COOKIE_NAME = "LB_USERID";
        private static final Long COOKIE_TIME = 60 * 30L;
    
        @Override
        public void init(FilterConfig filterConfig) throws ServletException {
    
        }
    
        @Override
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
            HttpServletRequest req = (HttpServletRequest)request;
            HttpServletResponse resp = (HttpServletResponse)response;
    
            StringBuffer URL = req.getRequestURL();
            log.debug("Request URL : {}",URL);
    
            //项目的根URL地址
            String path = req.getContextPath();
            String basePath = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort() +  path + "/";
    
            //登陆界面及静态资源不进行过滤
            if(URL.indexOf("/index") > -1 || URL.indexOf("/static") > -1 || URL.indexOf("/login/check") > -1){
                chain.doFilter(request,response);
                return;
            }
    
            Cookie[] cookies = req.getCookies();
    
            if (cookies != null && cookies.length != 0){
    
                try {
                    String loginToken = CookieUtil.readLoginToken(req);
                    if(StringUtils.isNotEmpty(loginToken)){
    
                        //刷新redis中缓存的有效时间
                        RedisPoolUtil.expire(loginToken,COOKIE_TIME);
                        //刷新客户端Cookie的有效时间
                        CookieUtil.refreshLoginToken(req,resp);
    
                        //递交给下一个过滤器（若没有则结束）
                        chain.doFilter(request,response);
                        return;
                    }
                } catch (NullPointerException e) {
                    log.error("redis 中的缓存过期",e);
                }
    
            }
    
            log.info("用户未登录，跳转至登陆页面");
            resp.sendRedirect(basePath + "index");
    
        }
    
        @Override
        public void destroy() {
    
        }
    }
    ```
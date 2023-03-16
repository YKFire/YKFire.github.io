---
title: 过滤器与拦截器
date: 2023-03-16 20:21:00 +0800
categories: [技术科普]
tags: [学习]
pin: true
author: YKFire

toc: true
comments: true

math: false
mermaid: true

typora-root-url: ../../YKFire.github.io

---

# 过滤器与拦截器 

## 一、过滤器

### 1、定义

过滤器 Filter 是 Sun 公司在 Servlet 2.3 规范中添加的新功能，其作用是对客户端发送给 Servlet 的请求以及对 Servlet 返回给客户端的响应做一些定制化的处理，例如校验请求的参数、设置请求/响应的 Header、修改请求/响应的内容等。

Filter 引入了过滤链（Filter Chain）的概念，**一个 Web 应用可以部署多个 Filter**，这些 Filter 会组成一种链式结构**，客户端的请求在到达 Servlet 之前会一直在这个链上传递**，不同的 Filter 负责对请求/响应做不同的处理。 Filter 的处理流程如下图所示：

![image-20230316202712048](/assets/blog_res/2023-03-16-Filtersandinterceptors.assets/image-20230316202712048.png)



Filter 的作用可认为是对 Servlet 功能的增强，因为 Filter 可以对用户的请求做预处理，也可以对返回的响应做后处理，且这些处理逻辑与 Servlet 的处理逻辑是分隔开的，这使得程序中各部分业务逻辑之间的耦合度降低，从而提高了程序的可维护性和可扩展性。



### 2、创建方式

创建 Filter 需要实现 javax.servlet.Filter 接口，或者继承实现了 Filter 接口的父类。Filter 接口中定义了三个方法：

- init：在 Web 程序启动时被调用，用于初始化 Filter。
- doFilter：在客户端的请求到达时被调用，doFilter 方法中定义了 Filter 的主要处理逻辑，同时该方法还负责将请求传递给下一个 Filter 或 Servlet。
- destroy：在 Web 程序关闭时被调用，用于销毁一些资源。

下面我们通过实现 Filter 接口来创建一个自定义的 Filter：

```java
public class TestFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println(filterConfig.getFilterName() + " 被初始化");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        System.out.println("Filter 拦截到了请求: " + request.getRequestURL());
        System.out.println("Filter 对请求做预处理...");
        filterChain.doFilter(servletRequest, servletResponse);
        System.out.println("Filter 修改响应的内容...");
    }

    @Override
    public void destroy() {
        System.out.println("Filter 被回收");
    }
}
```

---

**1. @WebFilter 注解 + 包扫描**

除了 FilterRegistrationBean 外，Servlet 3.0 引入的注解 @WebFilter 也可用于配置 Filter。我们只需要在自定义的 Filter 类上添加该注解，就可以设置 Filter 的名称和拦截规则：

```java
@WebFilter(urlPatterns = "/*", filterName = "TestFilter")
public class TestFilter implements Filter {
    // 省略部分代码
}
复制代码
```

由于@WebFilter 并非 Spring 提供，因此若要使自定义的 Filter 生效，还需在配置类上添加 @ServletComponetScan 注解，并指定扫描的包：

```java
@SpringBootApplication
@ServletComponentScan("com.example.filter")
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
复制代码
```

需要注意的是，**@WebFilter 注解并不允许我们设置 Filter 的执行顺序，且在 Filter 类上添加 @Order 注解也是无效的。如果项目中有多个被 @WebFilter 修饰的 Filter，那么这些 Filter 的执行顺序由其 "类名的字典序" 决定**，例如类名为 "Axx" 的 Filter 的执行顺序要先于类名为 "Bxx" 的 Filter。

> 添加了 @WebFilter 注解后就不要再添加 @Component 注解了，如果都添加，那么系统会创建两个 Filter。

**2. @Component 注解**

Spring 项目中，我们可以通过添加 @Component 注解将自定义的 Bean 交给 Spring 容器管理。同样的，对于自定义的 Filter，我们也可以直接添加 @Component 注解使其生效，而且还可以添加 @Order 注解来设置不同 Filter 的执行顺序。

```java
@Component
@Order(1)
public class TestFilter implements Filter {
    // 省略部分代码
}
复制代码
```

此种配置方式一般不常使用，因为其无法设置 Filter 的拦截规则，默认的拦截路径为 `/*`。虽然不能配置拦截规则，但我们可以在 doFilter 方法中定义请求的放行规则，例如当请求的 URL 匹配我们设置的规则时，直接将该请求放行，也就是立即执行 `filterChain.doFilter(servletRequest, servletResponse);`。



**3. 继承 OncePerRequestFilter**

OncePerRequestFilter 是一个由 Spring 提供的抽象类，在项目中，我们可以采用继承 OncePerRequestFilter 的方式创建 Filter，然后重写 doFilterInternal 方法定义 Filter 的处理逻辑，重写 shouldNotFilter 方法设置 Filter 的放行规则。对于多个 Filter 的执行顺序，我们也可以通过添加 @Order 注解进行设置。当然，若要使 Filter 生效，还需添加 @Component 注解将其注册到 Spring 容器。

```java
@Component
@Order(1)
public class CSpringFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        // 处理逻辑
    }

    @Override
    protected boolean shouldNotFilter(HttpServletRequest request) throws ServletException {
        // 放行规则
    }
}
复制代码
```

实际上，方式 2 和方式 3 本质上并没有什么区别，因为 OncePerRequestFilter 底层也是通过实现 Filter 接口来达到过滤请求/响应的目的，只不过 Spring 在 OncePerRequestFilter 中帮我们封装了许多功能，因此更推荐采用此种方式创建 Filter。



### 3、配置

Spring 项目中，我们可以使用 `@Configuration + @Bean + FilterRegistrationBean` 对 Filter 进行配置：

```java
@Configuration
public class FilterConfig {
    @Bean
    public FilterRegistrationBean<TestFilter> registryFilter() {
        FilterRegistrationBean<TestFilter> registration = new FilterRegistrationBean<>();
        registration.setFilter(new TestFilter());
        registration.addUrlPatterns("/*");
        registration.setName("TestFilter");
        registration.setOrder(0);
        return registration;
    }
}
复制代码
```

上述代码中，setFilter 方法用于设置 Filter 的类型；addUrlPatterns 方法用于设置拦截的规则；setName 方法用于设置 Filter 的名称；setOrder 方法用于设置 Filter 的优先级，数字越小优先级越高。



### 4、优先级

上文中提到，使用配置类或添加 @Order 注解可以显式的设置 Filter 的执行顺序，修改类名可以隐式的设置 Filter 的执行顺序。如果项目中存在多个 Filter，且这些 Filter 由不同的方式创建，那么它们的执行顺序是怎样的呢？

能够确定的是，Spring 根据 Filter 的 order 决定其优先级，如果我们通过配置类或者通过 @Order 注解设置了 Filter 的 order，那么 **order 值越小的 Filter 的优先级越高，无论 Filter 由何种方式创建**。如果多个 Filter 的优先级相同，那么执行顺序为：

1. 配置类中配置的 Filter 优先执行，如果配置类中存在多个 Filter，那么 Spring 按照其在配置类中配置的顺序依次执行。
2. @WebFilter 注解修饰的 Filter 之后执行，如果存在多个 Filter，那么 Spring 按照其类名的字典序依次执行。
3. @Component 注解修饰的 Filter 最后执行，如果存在多个 Filter，那么 Spring 按照其类名的字典序依次执行。

注意，以上优先级顺序仅适用于 order 相同的特殊情况。如果我们不配置 Filter 的 order，那么 Spring 默认将其 order 设置为 `LOWEST_PRECEDENCE = Integer.MAX_VALUE`，也就是最低优先级。由于被 @WebFilter 注解修饰的 Filter 无法显式配置优先级，因此其 order 为 Integer.MAX_VALUE。本文所说的 Filter 的优先级指的是 Filter 对请求做预处理的优先级，对响应做后处理的优先级与之相反。



### 5、应用场景

1. 解决跨域访问：前后端分离的项目往往存在跨域访问的问题，Filter 允许我们在 response 的 Header 中设置 "Access-Control-Allow-Origin"、"Access-Control-Allow-Methods" 等头域，以此解决跨域失败问题。
2. 设置字符编码：字符编码 Filter 可以在 request 提交到 Servlet 之前或者在 response 返回给客户端之前为请求/响应设置特定的编码格式，以解决请求/响应内容乱码的问题。
3. 记录日志：日志记录 Filter 可以在拦截到请求后，记录请求的 IP、访问的 URL，拦截到响应后记录请求的处理时间。当不需要记录日志时，也可以直接将 Filter 的配置注释掉。
4. 校验权限：Web 服务中，客户端在发送请求时会携带 cookie 或者 token 进行身份认证，权限校验 Filter 可以在 request 提交到 Servlet 之前对 cookie 或 token 进行校验，如果用户未登录或者权限不够，那么 Filter 可以对请求做重定向或返回错误信息。
5. 替换内容：内容替换 Filter 可以对网站的内容进行控制，防止输入/输出非法内容和敏感信息。例如在请求到达 Servlet 之前对请求的内容进行转义，防止 XSS 攻击；在 Servlet 将内容输出到 response 时，使用 response 将内容缓存起来，然后在 Filter 中进行替换，最后再输出到客户浏览器（由于默认的 response 并不能严格的缓存输出内容，因此需要自定义一个具备缓存功能的 response）。



### 6、实现原理

了解了 Filter 的使用方法之后，我们来分析一下 Filter 的实现原理。上文中提到，客户端的请求会在过滤链上依次传递，链上的每个 Filter 都会调用 doFilter 方法对请求进行处理。这个过程能够有条不紊的进行，底层依靠的是函数的回调机制。

**1. 回调函数**

在介绍 Filter 的回调机制之前，我们先了解一下回调函数的概念。如果将函数（C++ 中的函数指针，Java 中的匿名函数、方法引用等）作为参数传递给主方法，那么这个函数就称为回调函数，主方法会在某一时刻调用回调函数。

> 为了便于区分，我们使用 "主方法" 和 "函数" 来分辨主函数和回调函数。

使用回调函数的好处是能够实现函数逻辑的解耦，主方法内可以定义通用的处理逻辑，部分特定的操作则交给回调函数来完成。例如 Java 中 Arrays 类的 `sort(T[] a, Comparator<? super T> c)` 方法允许我们传入一个比较器来自定义排序规则，这个比较器的 compare 方法就属于回调函数，sort 方法会在排序时调用 compare 方法。

**2. Filter 的回调机制**

接下来介绍 Filter 的回调机制，上文中提到，我们自定义的 xxFilter 类需要实现 Filter 接口，且需要重写 doFilter 方法：

```java
public class TestFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        // ...
        filterChain.doFilter(servletRequest, servletResponse);
    }
}
复制代码
```

Filter 接口的 doFilter 方法接收一个 FilterChain 类型的参数，这个 FilterChain 对象可认为是传递给 doFilter 方法的回调函数，严格来说应该是这个 FilterChain 对象的 doFilter 方法，注意这里提到了两个 doFilter 方法。Filter 接口的 doFilter 方法在执行结束或执行完某些步骤后会调用 FilterChain 对象的 doFilter 方法，即调用回调函数。

FilterChain 对象的实际类型为 ApplicationFilterChain，其 doFilter() 方法的处理逻辑如下（省略部分代码）：

```java
public final class ApplicationFilterChain implements FilterChain {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response) throws IOException, ServletException {
        // ...
        internalDoFilter(request,response);
    }
    
    private void internalDoFilter(ServletRequest request, ServletResponse response) throws IOException, ServletException {
        if (pos < n) {
            // 获取第 pos 个 filter, 即 xxFilter  
            ApplicationFilterConfig filterConfig = filters[pos++];       
            Filter filter = filterConfig.getFilter();
            // ...
            // 调用 xxFilter 的 doFilter 方法
            filter.doFilter(request, response, this);
        }
    }
}
复制代码
```

可见，ApplicationFilterChain 的 doFilter 方法首先根据索引查询到我们定义的 xxFilter，然后调用 xxFilter 的 doFilter 方法，在调用时，ApplicationFilterChain 会将自己作为参数传递进去。xxFilter 的 doFilter 方法执行完某些步骤后，会调用回调函数，即 ApplicationFilterChain 的 doFilter 方法，这样 ApplicationFilterChain 就可以获取到下一个 xxFilter，并调用下一个 xxFilter 的 doFilter 方法，如此循环下去，直到所有的 xxFilter 全部被调用。整个流程如下图所示：

![image-20230316205001755](/assets/blog_res/2023-03-16-Filtersandinterceptors.assets/image-20230316205001755.png)



> xxFilter 执行回调函数的过程就像是给了 ApplicationFilterChain 一个通知，即通知 ApplicationFilterChain 可以执行下一个 xxFilter 的处理逻辑了。

综上，如果我们在重写自定义 Filter 的 doFilter 方法时没有添加 `filterChain.doFilter(servletRequest, servletResponse);`，那么该 Filter 之后的其它 Filter 和 Servlet 就不会被调用。



## 二、拦截器

### 1、定义

> 本文所说的拦截器指的是 Spring MVC 中的拦截器。

拦截器 Interceptor 是 Spring MVC 中的高级组件之一，其作用是拦截用户的请求，并在请求处理前后做一些自定义的处理，如校验权限、记录日志等。这一点和 Filter 非常相似，但不同的是，F**ilter 在请求到达 Servlet 之前对请求进行拦截**，**而 Interceptor 则是在请求到达 Controller 之前对请求进行拦截**，响应也同理。

与 Filter 一样，Interceptor 也具备链式结构，我们在项目中可以配置多个 Interceptor，当请求到达时，每个 Interceptor 根据其声明的顺序依次执行。



### 2、创建方式

创建 Interceptor 需要实现 org.springframework.web.servlet.HandlerInterceptor 接口，HandlerInterceptor 接口中定义了三个方法：

- preHandle：在 Controller 方法执行前被调用，可以对请求做预处理。该方法的返回值是一个 boolean 变量，只有当返回值为 true 时，程序才会继续向下执行。
- postHandle：在 Controller 方法执行结束，DispatcherServlet 进行视图渲染之前被调用，该方法内可以操作 Controller 处理后的 ModelAndView 对象。
- afterCompletion：在整个请求处理完成（包括视图渲染）后被调用，通常用来清理资源。

注意，postHandle 方法和 afterCompletion 方法执行的前提条件是 preHandle 方法的返回值为 true。

下面我们创建一个 Interceptor：

```java
@Component
public class TestInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("Interceptor 拦截到了请求: " + request.getRequestURL());
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

        System.out.println("Interceptor 操作 modelAndView...");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("Interceptor 清理资源...");
    }
}
```



### 3、配置

Interceptor 需要注册到 Spring 容器才能够生效，注册的方法是在配置类中实现 WebMvcConfigurer 接口，并重写 addInterceptors 方法:

```java
@Configuration
public class TestInterceptorConfig implements WebMvcConfigurer {

    @Autowired
    private TestInterceptor testInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(testInterceptor)
                .addPathPatterns("/*")
                .excludePathPatterns("/**/*.css", "/**/*.js", "/**/*.png", "/**/*.jpg", "/**/*.jpeg")
                .order(1);
    }
}
复制代码
```

上述代码中，addInterceptor 方法用于注册 Interceptor；addPathPatterns 方法用于设置拦截规则；excludePathPatterns 方法用于设置放行规则，order 方法用于设置 Interceptor 的优先级，数字越小优先级越高。



### 4、应用场景

Interceptor 的应用场可以参考上文中介绍的 Filter 的应用场景，可以说 Filter 能做到的事 Interceptor 都能做。由于 Filter 在 Servlet 前后起作用，而 Interceptor 可以在 Controller 方法前后起作用，例如操作 Controller 处理后的 ModelAndView，因此 Interceptor 更加灵活，在 Spring 项目中，如果能使用 Interceptor 的话尽量使用 Interceptor。



### 5、实现原理

了解了 Interceptor 的使用方法之后，我们也来分析一下 Interceptor 的实现原理。Interceptor 的调用发生在 DispatcherServlet 的 doDispatch 方法中，DispatcherServlet 是 Spring MVC 中请求的统一入口，当客户端的请求到达时，DispatcherServlet 会调用 doDispatch 方法处理当前请求：

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    // 省略部分代码
    try {
        ModelAndView mv = null;
        Exception dispatchException = null;

        try {
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);
            // 根据请求查找处理链 HandlerExecutionChain
            mappedHandler = getHandler(processedRequest);
            if (mappedHandler == null) {
                noHandlerFound(processedRequest, response);
                return;
            }

            // 获取适配器 HandlerAdapter, HandlerAdapter 用于处理请求
            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

            // 省略部分代码

            // 依次执行所有拦截器的 preHandle 方法
            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                return;
            }

            // 执行 Controller 方法, 返回 ModelAndView
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

            if (asyncManager.isConcurrentHandlingStarted()) {
                return;
            }

            applyDefaultViewName(processedRequest, mv);
            // 依次执行所有拦截器的 postHandle 方法, 顺序和 preHandle 方法相反
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        }
        catch (Exception ex) {
            dispatchException = ex;
        }
        catch (Throwable err) {
            dispatchException = new NestedServletException("Handler dispatch failed", err);
        }
        // 进行视图渲染, 结束后执行拦截器的 afterCompletion 方法
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    // 如果发生异常, 那么执行拦截器的 afterCompletion 方法并抛出异常
    catch (Exception ex) {
        triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
    }
    catch (Throwable err) {
        triggerAfterCompletion(processedRequest, response, mappedHandler,
                new NestedServletException("Handler processing failed", err));
    }
    // 省略部分代码
}
复制代码
```

1. 首先获取当前请求对应的处理链 HandlerExecutionChain，**HandlerExecutionChain 中包含了当前请求匹配的 Interceptor 列表**。
2. 然后根据 HandlerExecutionChain 获取适配器 HandlerAdapter，HandlerAdapter 用于处理当前请求。
3. 依次执行所有 Interceptor 的 preHandle 方法，如果某个 preHandle 方法返回 false，那么程序将不再向下执行。
4. 调用 HandlerAdapter 处理请求，返回 ModelAndView。
5. 请求处理完成后，依次执行所有 Interceptor 的 postHandle 方法，执行的顺序与 preHandle 方法相反。
6. 进行视图渲染，渲染结束后执行所有 Interceptor 的 afterCompletion 方法，执行顺序与 postHandle 方法相同。
7. 如果以上操作发生异常，那么执行所有 Interceptor 的 afterCompletion 方法并抛出异常。

> 由上述过程我们得知，如果 Controller 抛出异常，那么 postHandle 方法将不会执行，afterCompletion 方法则一定执行。



## 三、两者的区别

### **1. 规范不同**

Filter 在 Servlet 规范中定义，依赖于 Servlet 容器（如 Tomcat）；Interceptor 由 Spring 定义，依赖于 Spring 容器（IoC 容器）。

### **2. 适用范围不同**

Filter 仅可用于 Web 程序，因为其依赖于 Servlet 容器；Interceptor 不仅可以用于 Web 程序，还可以用于 Application、Swing 等程序。

### **3. 触发时机不同**

Filter 在请求进入 Servlet 容器，且到达 Servlet 之前对请求做预处理；在 Servlet 处理完请求后对响应做后处理。

Interceptor 在请求进入 Servlet，且到达 Controller 之前对请求做预处理；在 Controller 处理完请求后对 ModelAndView 做后处理，在视图渲染完成后再做一些收尾工作。

下图展示了二者的触发时机：

![image-20230316210359912](/assets/blog_res/2023-03-16-Filtersandinterceptors.assets/image-20230316210359912.png)

当 Filter 和 Interceptor 同时存在时，Filter 对请求的预处理要先于 Interceptor 的 preHandle 方法；Filter 对响应的后处理要后于 Interceptor 的 postHandle 方法和 afterCompletion 方法。



## 四、补充说明

### **1. 在 Filter 和 Interceptor 注入 Bean 的注意事项**

有些文章在介绍 Filter 和 Interceptor 的区别时强调 Filter 不能通过 IoC 注入 Bean，如果我们采用本文中的第一种创建 Filter，那么确实不能注入成功：

```java
// 自定义的 Filter, 未添加 @Component 注解
public class TestFilter implements Filter {

    @Autowired
    private UserService userService;

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        System.out.println(userService);
        filterChain.doFilter(servletRequest, servletResponse);
    }
    // ...
}

// 配置类
@Configuration
public class FilterConfig {
    @Bean
    public FilterRegistrationBean<TestFilter> registryFilter() {
        FilterRegistrationBean<TestFilter> registration = new FilterRegistrationBean<>();
        registration.setFilter(new TestFilter());
        registration.addUrlPatterns("/*");
        registration.setName("TestFilter");
        registration.setOrder(0);
        return registration;
    }
}
复制代码
```

上述代码执行后，userService 输出为 null，因为注册到 IoC 容器中的是 new 出来的一个 TestFilter 对象（`registration.setFilter(new TestFilter());`），并不是 Spring 自动装配的。若要使 userService 注入成功，可改为如下写法：

```java
// 自定义的 Filter, 未添加 @Component 注解
@Component
public class TestFilter implements Filter {

    @Autowired
    private UserService userService;

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        System.out.println(userService);
        filterChain.doFilter(servletRequest, servletResponse);
    }
    // ...
}

// 配置类
@Configuration
public class FilterConfig {

    @Autowired
    private TestFilter testFilter;

    @Bean
    public FilterRegistrationBean<TestFilter> registryFilter() {
        FilterRegistrationBean<TestFilter> registration = new FilterRegistrationBean<>();
        registration.setFilter(testFilter);
        registration.addUrlPatterns("/*");
        registration.setName("TestFilter");
        registration.setOrder(0);
        return registration;
    }
}
复制代码
```

与第一种写法的区别在于，TestFilter 类上添加了 @Component 注解，且配置类中通过 @Autowired 注入 TestFilter 对象。除了使用配置类外，本文介绍的其它几种方式（添加 @Component 注解或 @WebFilter 注解）都可以直接注入 Bean。

> 所以还是采用继承 OncePerRequestFilter 的方式创建 Filter 比较方便。

另外，使用本文介绍的创建 Interceptor 的写法是可以直接注入 Bean 的，该写法也是先在自定义的 Interceptor 上添加 @Component 注解，然后在配置类中使用 @Autowired 注入自定义的 Interceptor。

### **2. Interceptor 可以拦截静态请求**

有文章提到 Interceptor 不能拦截静态请求，其实在 Spring 1.x 的版本中确实是这样的，但 Spring 2.x 对静态资源也进行了拦截，例如上文中我们在测试 TestInterceptor 是否生效时，发现其拦截到了 `/favicon.ico` 请求，该请求是一个由浏览器自动发送的静态请求。
### 请求处理

#### 请求映射@RequestMapping

#### HiddenHttpMethodFilter

- 如果不启用这个Filter，那么REST的DELETE和PUT方法就没有办法启用

- 需要开启配置

```yaml
spring:
  mvc:
    hiddenmethod:
      filter:
        enabled: true   #开启页面表单的Rest功能
```

- 在html中：表单method=post，隐藏域 _method=put

``` html
<form action="/user" method="post">
    <input name="_method" type="hidden" value="DELET"/>
</form>
```

Rest原理（html表单提交要使用REST的时候）

- 表单提交会带上**_method=PUT**
- **请求过来被**HiddenHttpMethodFilter拦截

- - 请求是否正常，并且是POST

- - - 获取到**_method**的值。
    - 判断是否是以下请求；**PUT**.**DELETE**.**PATCH**
    - 如果是上述判断的请求，那么将HttpServletRequest包装在HttpMethodRequestWrapper中，创建HttpMethodRequestWrapper对象时传入的method参数是处理过的**PUT**.**DELETE**.**PATCH**method，HttpMethodRequestWrapper重写了getMethod方法，该方法返回的是传入的值。
    - **过滤器链（filterChain）放行（doFilter方法）的时候用wrapper。以后的方法调用getMethod是调用HttpMethodRequestWrapper的。**这样就可以获取hidden的方法（**PUT**.**DELETE**.**PATCH**）



HiddenHttpMethodFilter处理进来的请求，将其转为HttpMethodRequestWrapper，之后过滤器链（filterChain）放行（doFilter方法）的时候使用HttpMethodRequestWrapper

```java
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        HttpServletRequest requestToUse = request;
        if ("POST".equals(request.getMethod()) && request.getAttribute("javax.servlet.error.exception") == null) {
            String paramValue = request.getParameter(this.methodParam);
            if (StringUtils.hasLength(paramValue)) {
                String method = paramValue.toUpperCase(Locale.ENGLISH);
                if (ALLOWED_METHODS.contains(method)) {
                    requestToUse = new HiddenHttpMethodFilter.HttpMethodRequestWrapper(request, method);
                }
            }
        }

        filterChain.doFilter((ServletRequest)requestToUse, response);
    }

```



HttpMethodRequestWrapper

```java
    private static class HttpMethodRequestWrapper extends HttpServletRequestWrapper {
        private final String method;

        public HttpMethodRequestWrapper(HttpServletRequest request, String method) {
            super(request);
            this.method = method;
        }

        public String getMethod() {
            return this.method;
        }
    }
```





**Rest使用客户端工具，**

- 如PostMan直接发送Put、delete等方式请求，无需Filter。

用@PutMapping、@GetMapping、@PostMapping、@DeleteMapping简化@RequestMapping



#### 请求映射的原理

DispatcherServlet是请求处理的开始

- 调用doGet、doPost方法
- doGet、doPost方法的调用processRequest方法
- processRequest方法调用doService方法
- doService方法调用doDispatch
- doDispatch方法
  - 将原生请求包装为HttpServletRequest类型
  - 找到、确定哪个handler来解决我们的请求（mappedHandler = getHandler(processedRequest);）
  - getHandler方法中的handlerMapping保存了不同请求映射的规则，共有5个映射。其中有一个是欢迎页请求映射，现在我们要讨论的是**RequestMappingHandlerMapping**，它保存了所有@RequestMapping注解指示的路径 和handler的映射规则，在程序一启动时就扫描了@RequestMapping注解下的映射规则并将其注册储存起来。



responseBody与正常的springMVC流程不同

- SpringMVC：

DispatchServelet：接受请求，将请求分发，通过HandlerMapping找到处理这个请求的具体的handler。

Hanlder执行完成后返回Modelandview

HandlerAdapter将Handler执行结果ModelAndView返回到DispatcherServlet

再通过视图解析器（ViewReslover）处理返回具体的View，然后DispatchServelet渲染响应用户

- responseBody：

handler处理后将方法的返回直接写入Http response body中返回json数据。




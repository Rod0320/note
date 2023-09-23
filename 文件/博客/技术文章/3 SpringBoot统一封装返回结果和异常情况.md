# SpringBoot统一封装返回结果和异常情况

> 如果文章有帮助到你，还请点个赞或留下评论😊

## 原因

在springboot项目里我们希望接口返回的数据包含至少三个属性：

- code：请求接口的返回码，成功或者异常等返回编码，例如定义请求成功。
- message：请求接口的描述，也就是对返回编码的描述。
- data：请求接口成功，返回的结果。

```json
{
  "code":20000,
  "message":"成功",
  "data":{
    "info":"测试成功"
  }
}
```

## 开发环境[#](https://www.cnblogs.com/sw-code/p/13956522.html#开发环境)

- 工具：IDEA
- SpringBoot版本：`2.2.2.RELEASE`
- 依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
</dependency>
<!-- fastjson -->
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>fastjson</artifactId>
  <version>1.2.62</version>
</dependency>
<!-- Spring web -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!-- lombok -->
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
</dependency>
<!-- swagger3 -->
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-boot-starter</artifactId>
  <version>3.0.0</version>
</dependency>
```

## 创建 SpringBoot 工程[#](https://www.cnblogs.com/sw-code/p/13956522.html#创建-springboot-工程)

新建 SpringBoot 项目`common_utils`，包名`com.spring.utils`

## 返回结果统一[#](https://www.cnblogs.com/sw-code/p/13956522.html#返回结果统一)

### 创建code枚举[#](https://www.cnblogs.com/sw-code/p/13956522.html#创建code枚举)

在`com.spring.utils`中创建 `pojo` 包，并添加枚举`ResultCode`

```java
public enum ResultCode {

  /* 成功状态码 */
  SUCCESS(20000, "成功"),
  /* 参数错误 */
  PARAM_IS_INVALID(1001, "参数无效"),
  PARAM_IS_BLANK(1002, "参数为空"),
  PARAM_TYPE_BIND_ERROR(1003, "参数类型错误"),
  PARAM_NOT_COMPLETE(1004, "参数缺失"),
  /* 用户错误 2001-2999*/
  USER_NOTLOGGED_IN(2001, "用户未登录"),
  USER_LOGIN_ERROR(2002, "账号不存在或密码错误"),
  SYSTEM_ERROR(10000, "系统异常，请稍后重试");

  private Integer code;
  private String message;

  private ResultCode(Integer code, String message) {
    this.code = code;
    this.message = message;
  }

  public Integer code() {
    return this.code;
  }
  public String message() {
    return this.message;
  }
}
```

- 可根据项目自定义，结果返回码

### 创建返回结果实体[#](https://www.cnblogs.com/sw-code/p/13956522.html#创建返回结果实体)

在 `pojo` 包中添加返回结果类`R`

```java
@Data
@ApiModel(value = "返回结果实体类", description = "结果实体类")
public class R implements Serializable {

  private static final long serialVersionUID = 1L;

  @ApiModelProperty(value = "返回码")
  private Integer code;

  @ApiModelProperty(value = "返回消息")
  private String message;

  @ApiModelProperty(value = "返回数据")
  private Object data;

  private R() {

  }

  public R(ResultCode resultCode, Object data) {
    this.code = resultCode.code();
    this.message = resultCode.message();
    this.data = data;
  }

  private void setResultCode(ResultCode resultCode) {
    this.code = resultCode.code();
    this.message = resultCode.message();
  }

  // 返回成功
  public static R success() {
    R result = new R();
    result.setResultCode(ResultCode.SUCCESS);
    return result;
  }
  // 返回成功
  public static R success(Object data) {
    R result = new R();
    result.setResultCode(ResultCode.SUCCESS);
    result.setData(data);
    return result;
  }

  // 返回失败
  public static R fail(Integer code, String message) {
    R result = new R();
    result.setCode(code);
    result.setMessage(message);
    return result;
  }
  // 返回失败
  public static R fail(ResultCode resultCode) {
    R result = new R();
    result.setResultCode(resultCode);
    return result;
  }
}
```

### 自定义一个注解[#](https://www.cnblogs.com/sw-code/p/13956522.html#自定义一个注解)

新建包`annotation`，并添加`ResponseResult`注解类

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ ElementType.TYPE, ElementType.METHOD})
@Documented
public @interface ResponseResult {

}
```

### 定义拦截器[#](https://www.cnblogs.com/sw-code/p/13956522.html#定义拦截器)

新建包`interceptor`，并添加`ResponseResultInterceptor`Java类

```java
@Component
public class ResponseResultInterceptor implements HandlerInterceptor {
  //标记名称
  public static final String RESPONSE_RESULT_ANN = "RESPONSE-RESULT-ANN";

  @Override
  public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
    throws Exception {
    // TODO Auto-generated method stub
    if (handler instanceof HandlerMethod) {
      final HandlerMethod handlerMethod = (HandlerMethod) handler;
      final Class<?> clazz = handlerMethod.getBeanType();
      final Method method = handlerMethod.getMethod();
      // 判断是否在类对象上添加了注解
      if (clazz.isAnnotationPresent(ResponseResult.class)) {
        // 设置此请求返回体，需要包装，往下传递，在ResponseBodyAdvice接口进行判断
        request.setAttribute(RESPONSE_RESULT_ANN, clazz.getAnnotation(ResponseResult.class));
      } else if (method.isAnnotationPresent(ResponseResult.class)) {
        request.setAttribute(RESPONSE_RESULT_ANN, method.getAnnotation(ResponseResult.class));
      }
    }
    return true;
  }
}
```

- 用于拦截请求，判断 Controller 是否添加了`@ResponseResult`注解

### 注册拦截器[#](https://www.cnblogs.com/sw-code/p/13956522.html#注册拦截器)

新建包`config`，并添加`WebAppConfig`配置类

```java
@Configuration
public class WebAppConfig implements WebMvcConfigurer {

    // SpringMVC 需要手动添加拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        ResponseResultInterceptor interceptor = new ResponseResultInterceptor();
        registry.addInterceptor(interceptor);
        WebMvcConfigurer.super.addInterceptors(registry);
    }
}
```

### 方法返回值拦截处理器[#](https://www.cnblogs.com/sw-code/p/13956522.html#方法返回值拦截处理器)

新建包`handler`，并添加`ResponseResultHandler`配置类，实现`ResponseBodyAdvice`重写两个方法

```java
import org.springframework.web.method.HandlerMethod;

/**
 * 使用 @ControllerAdvice & ResponseBodyAdvice 
 * 拦截Controller方法默认返回参数，统一处理返回值/响应体
 */
@ControllerAdvice
public class ResponseResultHandler implements ResponseBodyAdvice<Object> {
   
   // 标记名称
   public static final String RESPONSE_RESULT_ANN = "RESPONSE-RESULT-ANN";

   
   // 判断是否要执行 beforeBodyWrite 方法，true为执行，false不执行，有注解标记的时候处理返回值
   @Override
   public boolean supports(MethodParameter arg0, Class<? extends HttpMessageConverter<?>> arg1) {
      ServletRequestAttributes sra = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
      HttpServletRequest request = sra.getRequest();
      // 判断请求是否有包装标记
      ResponseResult responseResultAnn = (ResponseResult) request.getAttribute(RESPONSE_RESULT_ANN);
      return responseResultAnn == null ? false : true;
   }
   
   // 对返回值做包装处理
   @Override
   public Object beforeBodyWrite(Object body, MethodParameter arg1, MediaType arg2,
                          Class<? extends HttpMessageConverter<?>> arg3, ServerHttpRequest arg4, ServerHttpResponse arg5) {
      if (body instanceof R) {
         return (R) body;
      } else if (body instanceof String) {
          return body;
      }
      return R.success(body);
   }
}
```

- 实现`ResponseBodyAdvice`重写两个方法
- 添加`@ControllerAdvice`注解

### 测试

新建包`controller`，并添加`TestController`测试类

```java
@RestController
@ResponseResult
public class TestController {

  @GetMapping("/test")
  public Map<String, Object> test() {
    HashMap<String, Object> data = new HashMap<>();
    data.put("info", "测试成功");
    return data;
  }
}
```

- 添加`@ResponseResult`注解

启动项目，在默认端口: `8080`

浏览器访问地址：`localhost:8080/test`

```json
{"code":20000,"message":"成功","data":{"info":"测试成功"}}
```

### 总结

1、创建code枚举和返回结果实体类

2、自定义一个注解`@ResponseResult`

3、定义拦截器，拦截请求，判断Controller上是否添加了`@ResponseResult`注解。如果添加了注解在request中添加注解标记，往下传递

4、添加`@ControllerAdvice`注解 ，实现`ResponseBodyAdvice`接口，并重写两个方法，通过判断request中是否有注解标记，如果有就往下执行，进一步包装。没有就直接返回，不需包装。

### 问题

1、如果要返回错误结果，这种方法显然不方便

```java
@GetMapping("/fail")
public R error() {
  int res = 0; // 查询结果数
  if(res == 0) {
    return R.fail(10001, "没有数据");
  }
  return R.success(res);
}
```

2、我们需要对错误和异常进行进一步的封装

## 封装错误和异常结果

### 创建错误结果实体

在包`pojo`中添加`ErrorResult`实体类

```java
/**
 * 异常结果包装类
 * @author sw-code
 *
 */
public class ErrorResult {

  private Integer code;

  private String message;

  private String exception;

  public Integer getCode() {
    return code;
  }

  public void setCode(Integer code) {
    this.code = code;
  }

  public String getMessage() {
    return message;
  }

  public void setMessage(String message) {
    this.message = message;
  }

  public String getException() {
    return exception;
  }

  public void setException(String exception) {
    this.exception = exception;
  }

  public static ErrorResult fail(ResultCode resultCode, Throwable e, String message) {
    ErrorResult errorResult = ErrorResult.fail(resultCode, e);
    errorResult.setMessage(message);
    return errorResult;
  }

  public static ErrorResult fail(ResultCode resultCode, Throwable e) {
    ErrorResult errorResult = new ErrorResult();
    errorResult.setCode(resultCode.code());
    errorResult.setMessage(resultCode.message());
    errorResult.setException(e.getClass().getName());
    return errorResult;
  }
  public static ErrorResult fail(Integer code, String message) {
    ErrorResult errorResult = new ErrorResult();
    errorResult.setCode(code);
    errorResult.setMessage(message);
    return errorResult;
  }	
}
```

### 自定义异常类

在包`pojo`中添加`BizException`实体类，继承`RuntimeException`

```java
@Data
public class BizException extends RuntimeException {

  /**
     * 错误码
     */
  private Integer code;

  /**
     * 错误信息
     */
  private String message;

  public BizException() {
    super();
  }

  public BizException(ResultCode resultCode) {
    super(resultCode.message());
    this.code = resultCode.code();
    this.message = resultCode.message();
  }

  public BizException(ResultCode resultCode, Throwable cause) {
    super(resultCode.message(), cause);
    this.code = resultCode.code();
    this.message = resultCode.message();
  }

  public BizException(String message) {
    super(message);
    this.code = -1;
    this.message = message;
  }

  public BizException(Integer code, String message) {
    super(message);
    this.code = code;
    this.message = message;
  }

  public BizException(Integer code, String message, Throwable cause) {
    super(message, cause);
    this.code = code;
    this.message = message;
  }

  @Override
  public synchronized Throwable fillInStackTrace() {
    return this;
  }
}
```

### 全局异常处理类

在包`handler`中添加`GlobalExceptionHandler`，添加`@RestControllerAdvice`注解

```java
/**
 * 全局异常处理类
 * @RestControllerAdvice(@ControllerAdvice)，拦截异常并统一处理
 * @author sw-code
 *
 */
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

  /**
	 * 处理自定义的业务异常
	 * @param e	异常对象
	 * @param request	request
	 * @return	错误结果
	 */
  @ExceptionHandler(BizException.class)
  public ErrorResult bizExceptionHandler(BizException e, HttpServletRequest request) {
    log.error("发生业务异常！原因是: {}", e.getMessage());
    return ErrorResult.fail(e.getCode(), e.getMessage());
  }

  // 拦截抛出的异常，@ResponseStatus：用来改变响应状态码
  @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
  @ExceptionHandler(Throwable.class)
  public ErrorResult handlerThrowable(Throwable e, HttpServletRequest request) {
    log.error("发生未知异常！原因是: ", e);
    ErrorResult error = ErrorResult.fail(ResultCode.SYSTEM_ERROR, e);
    return error;
  }

  // 参数校验异常
  @ExceptionHandler(BindException.class)
  public ErrorResult handleBindExcpetion(BindException e, HttpServletRequest request) {
    log.error("发生参数校验异常！原因是：",e);
    ErrorResult error = ErrorResult.fail(ResultCode.PARAM_IS_INVALID, e, e.getAllErrors().get(0).getDefaultMessage());
    return error;
  }

  @ExceptionHandler(MethodArgumentNotValidException.class)
  public ErrorResult handleMethodArgumentNotValidException(MethodArgumentNotValidException e, HttpServletRequest request) {
    log.error("发生参数校验异常！原因是：",e);
    ErrorResult error = ErrorResult.fail(ResultCode.PARAM_IS_INVALID,e,e.getBindingResult().getAllErrors().get(0).getDefaultMessage());
    return error;
  }	
}
```

- 添加注解`@RestControllerAdvice(@ControllerAdvice)`，拦截异常并统一处理

### 修改方法返回值拦截处理器

将错误和异常结果也进行统一封装

```java
/**
 * 使用 @ControllerAdvice & ResponseBodyAdvice 
 * 拦截Controller方法默认返回参数，统一处理返回值/响应体
 */
@ControllerAdvice
public class ResponseResultHandler implements ResponseBodyAdvice<Object> {

  // 标记名称
  public static final String RESPONSE_RESULT_ANN = "RESPONSE-RESULT-ANN";


  // 判断是否要执行 beforeBodyWrite 方法，true为执行，false不执行，有注解标记的时候处理返回值
  @Override
  public boolean supports(MethodParameter arg0, Class<? extends HttpMessageConverter<?>> arg1) {
    ServletRequestAttributes sra = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
    HttpServletRequest request = sra.getRequest();
    // 判断请求是否有包装标记
    ResponseResult responseResultAnn = (ResponseResult) request.getAttribute(RESPONSE_RESULT_ANN);
    return responseResultAnn == null ? false : true;
  }


  // 对返回值做包装处理，如果属于异常结果，则需要再包装
  @Override
  public Object beforeBodyWrite(Object body, MethodParameter arg1, MediaType arg2,
                                Class<? extends HttpMessageConverter<?>> arg3, ServerHttpRequest arg4, ServerHttpResponse arg5) {
    if (body instanceof ErrorResult) {
      ErrorResult error = (ErrorResult) body;
      return R.fail(error.getCode(), error.getMessage());
    } else if (body instanceof R) {
      return (R) body;
    } else if  (body instanceof String) {
      return body;
    }
    return R.success(body);
  }
}
```

### 测试

```java
@GetMapping("/fail")
public Integer error() {
  int res = 0; // 查询结果数
  if( res == 0 ) {
    throw new BizException("没有数据");
  }
  return res;
}
```

返回结果

```json
{"code":-1,"message":"没有数据","data":null}
```

我们无需担心返回类型，如果需要返回错误提示信息，可以直接抛出自定义异常(`BizException`)，并添加自定义错误信息。
---
title: 스프링의 예외 처리 방법
date: 2023-05-06 00:10:00 +09:00
categories: [Spring, MVC]
tags: [BasicErrorController, HandlerExceptionResolver, ResponseStatus, ExceptionHandler, ControllerAdvice, RestControllerAdvice]
---

# 스프링의 기본 예외 처리 방법

![spring-exception-handling](/assets/img/spring/mvc/exception-handling/spring-exception-handling.png)  

자바 프로그램은 예외가 발생하면 예외 정보를 남기고 쓰레드가 종료되는 반면에 스프링은 예외가 발생했을 때 웹 애플리케이션이 종료되지 않고 HTTP 상태 코드가 노출이 됩니다.
스프링 부트는 컨트롤러 이하에서 발생한 예외를 캐치하여 쓰레드를 종료 시키는 것이 아닌 예외 내용을 디스패처서블릿에서 에러컨트롤러로 다시 요청을 보냄으로써 마치 정상 요청인 것처럼 예외 내용을 처리하도록 동작하게 됩니다.
하지만 최초 요청과 더불어 예외 요청을 위해 컨트롤러를 2번 호출한다는 것은 꽤나 복잡하다고 느껴지며 또 이는 필터나 인터셉터를 2번 호출하는 것과 같은 다른 문제를 야기할 수 있습니다.  

이런 중복 호출을 방지하기 위해 필터는 등록할 때 DispatcherType을 설정 할 수 있으며 별다른 설정이 없다면 DispatcherType이 ERROR일 경우에는 호출되지 않도록 설정 되어 있습니다.   

![do-filter](/assets/img/spring/mvc/exception-handling/do-filter.png)  

인터셉터는 최초 인터셉터 등록 시 URI 패턴으로 에러 URI나 다양한 URI를 URI 패턴으로 제외할 수 있습니다.  

```java 
@Configuration
public class SampleConfig implements WebMvcConfigurer {

    @Value(value="${server.error.path}")
    private final String errorPath;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new SampleInterceptor())
                .excludePathPatterns(errorPath);
    }
}
```   

## BasicErrorController

```java
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {

    private final ErrorProperties errorProperties;
    
    @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
    public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
        ...
        return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
    }
	
    @RequestMapping
    public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
        ...
        return new ResponseEntity<>(body, status);
    }
	
}
```

스프링 부트는 기본적으로 예외를 캐치하여 호출할 에러 컨트롤러로  ``org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController`` 를 구현해 사용하고 있습니다.
베이직에러컨트롤러는 ``/error`` 경로일 때 실행되는 컨트롤러로 이는 properties에서 ``server.error.path``의 값으로 변경 가능합니다. 
주요 메소드로는 ``errorHtml(HttpServletRequest, HttpServletResponse), error(HttpServletRequest)``가 있는데, 해당 메소드는 에러컨트롤러를 호출할 때 들어오는 리퀘스트의 유형에 따라 반환하는 타입이 다릅니다.  
메소드 네임에서 유추할 수 있듯이, ``errorHtml()``같은 경우는 ``@RequestMapping(produces = MediaType.TEXT_HTML_VALUE)`` 어노테이션을 적용해, HTTP 요청 헤더의 Accept 값이 ``text/html``일 경우 랜더링하기 위한  ``ModelAndView`` 객체를 반환합니다. 
따로 에러 페이지를 설정을 하지 않았다면, 기본 에러 페이지인 ``Whitelabel Error Page`` 화면이 노출됩니다.   

![whitelabel-error-page](/assets/img/spring/mvc/exception-handling/whitelabel-error-page.png)  

그 외로 경우라면, ``error()`` 메소드를 통해 리스폰즈 객체가 반환이 됩니다. 

![postman](/assets/img/spring/mvc/exception-handling/postman.png)  

추가로 베이직에러컨트롤러는 예외 처리시 필요한 에러 속성 값을 처리하는데, ``getErrorAttributes(HttpServletRequest, ErrorAttributeOptions)`` 메소드를 통해 웹 화면 또는 json으로 보여질 속성 값들을 가져옵니다. 

![private-get-error-attributes](/assets/img/spring/mvc/exception-handling/private-get-error-attributes.png)

``DefaultErrorAttributes``에서 제공하는 기본 예외 속성으로 가능한 경우 아래와 같은 속성 값을 제공합니다.  

- timestamp: 예외 발생 시간
- status: Http 상태
- error: 에러 코드
- exception: 최상위 예외 클래스 이름(설정한 경우)
- message: 예외 메세지(설정한 경우)
- errors: BindingExecption에 의해 생긴 예외 목록(설정한 경우)
- trace: 에러 스택 트레이스(설정한 경우)
- path: 예외가 발생한 URL 경로

![get-error-attributes](/assets/img/spring/mvc/exception-handling/get-error-attributes.png)  

그 후 ``DefaultErrorAttributes``에서는 전체 에서 속성값들 중에서 에러 속성 옵션에 맞게 속성을 설정합니다.  
``exception, message``에 대한 속성 설정 후 결과적으로 아래와 같은 에러 속성을 클라이언트에게 반환합니다. 

```json
{
    "timestamp": "2023-05-06T14:35:44.799+00:00",
    "status": 500,
    "error": "Internal Server Error",
    "path": "/sample"
}
```

![error-attribute](/assets/img/spring/mvc/exception-handling/error-attribute.png)   

결과적으로 json 객체와 웹 화면에 가각 ``timestamp , status , error , path``에 대한 에러 속성 값들이 바인딩되어 나타납니다.  

하지만 위와 같은 응답은 클라이언트 입장에서는 다소 불친절하게 느껴질 수 있습니다. 
클라이언트는 ``Internal Server Error``라는 다소 모호한 메시지보다는 명시적으로 해당 오류에 대한 유의미한 응답 메세지를 받고 싶을 것입니다. 
그러므로 별도의 다양한 에러 처리를 통해 상황에 맞는 적절한 에러 응답을 제공해야 합니다. 

# 스프링이 제공하는 예외 처리 방법

## HandlerExceptionResolver

``HandlerExceptionResolver``는 컨트롤러 이하에서 발생한 예외를 서블릿 컨테이너까지 전달하지 않고, 스프링 MVC 내부에서 예외 처리를 하기 위해 만들어진 객체입니다. 

```java
/**
 * Interface to be implemented by objects that can resolve exceptions thrown during
 * handler mapping or execution, in the typical case to error views. Implementors are
 * typically registered as beans in the application context.
 *
 * <p>Error views are analogous to JSP error pages but can be used with any kind of
 * exception including any checked exception, with potentially fine-grained mappings for
 * specific handlers.
 *
 * @author Juergen Hoeller
 * @since 22.11.2003
 */
public interface HandlerExceptionResolver {

	/**
	 * Try to resolve the given exception that got thrown during handler execution,
	 * returning a {@link ModelAndView} that represents a specific error page if appropriate.
	 * <p>The returned {@code ModelAndView} may be {@linkplain ModelAndView#isEmpty() empty}
	 * to indicate that the exception has been resolved successfully but that no view
	 * should be rendered, for instance by setting a status code.
	 * @param request current HTTP request
	 * @param response current HTTP response
	 * @param handler the executed handler, or {@code null} if none chosen at the
	 * time of the exception (for example, if multipart resolution failed)
	 * @param ex the exception that got thrown during handler execution
	 * @return a corresponding {@code ModelAndView} to forward to,
	 * or {@code null} for default processing in the resolution chain
	 */
	@Nullable
	ModelAndView resolveException(
			HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex);

}
```   

스프링은 예외 처리 로직을 메인 로직으로부터 분리하여 다양한 전략으로 예외를 처리할 수 있도록 추상화할 수 있는 ``HandlerExceptionResolver`` 인터페이스를 구현했습니다. 
실제로 스프링 부트는 ``HandlerExceptionResolver``를 상속 받아 구현한 다양한 이셉션 처리 로직이 있으며, 해당 객체들은 우선 순위에 따라 처리하게 됩니다. 

![exception-resolvers](/assets/img/spring/mvc/exception-handling/exception-resolvers.png)  

예외가 발생 했을 때 적용 가능한 구현체를 ``HandlerExceptionResolverComposite``가 담고 있는 목록입니다. 
각 구현체들은 아래의 나열된 순서대로 우선 순위를 가집니다.   

- ExceptionHandlerExceptionResolver: ``@ExceptionHandler``를 처리합니다. 
- ResponseStatusExceptionResolver: HTTP 상태 코드를 지정해줍니다. (``@ResponseStatus``, ``ResponseStatusException`` 를 처리합니다.)
- DefaultHandlerExceptionResolver: 스프링 내부의 기본 예외들을 처리합니다.

* DefaultErrorAttributes: 에러 속성을 처리하기 위한 객체로, 직접적인 예외를 처리하지않습니다.

## DefaultHandlerExceptionResolver

스프링 내부의 기본 예외들을 처리하는 ``HandlerExceptionResolver`` 구현체로 발생한 예외들을 적절한 HTTP 상태 코드로 변경해주는 역활을 합니다. 

아래 표는 ``DefaultHandlerExceptionResolver``에서 변경해주는 예외 HTTP 상태 코드입니다.  

|                 Exception                 |            HTTP  상태 코드            |
|:-----------------------------------------:|:---------------------------------:|
|  HttpRequestMethodNotSupportedException   |    405 (SC_METHOD_NOT_ALLOWED)    |
|    HttpMediaTypeNotSupportedException     |  415 (SC_UNSUPPORTED_MEDIA_TYPE)  |
|    HttpMediaTypeNotAcceptableException    |      406 (SC_NOT_ACCEPTABLE)      |
|       MissingPathVariableException        |  500 (SC_INTERNAL_SERVER_ERROR)   |
|  MissingServletRequestParameterException  |       400 (SC_BAD_REQUEST)        |
|      ServletRequestBindingException       |       400 (SC_BAD_REQUEST)        |
|      ConversionNotSupportedException      |  500 (SC_INTERNAL_SERVER_ERROR)   |
|           TypeMismatchException           |       400 (SC_BAD_REQUEST)        |
|      HttpMessageNotReadableException      |       400 (SC_BAD_REQUEST)        |
|      HttpMessageNotWritableException      |  500 (SC_INTERNAL_SERVER_ERROR)   |
|      MethodArgumentNotValidException      |       400 (SC_BAD_REQUEST)        |
|    MissingServletRequestPartException     |       400 (SC_BAD_REQUEST)        |
|               BindException               |       400 (SC_BAD_REQUEST)        |
|          NoHandlerFoundException          |        404 (SC_NOT_FOUND)         |
|       AsyncRequestTimeoutException        |   503 (SC_SERVICE_UNAVAILABLE)    |


## ResponseStatusExceptionResolver

## ExceptionHandlerExceptionResolver

### @ControllerAdvice

### @RestControllerAdvice


오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.  
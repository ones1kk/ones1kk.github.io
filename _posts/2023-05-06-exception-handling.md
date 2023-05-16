---
title: 스프링의 예외 처리 방법
date: 2023-05-06 00:10:00 +09:00
categories: [Spring, MVC]
tags: [BasicErrorController, HandlerExceptionResolver, ResponseStatus, ExceptionHandler, ControllerAdvice, RestControllerAdvice]
---

# 스프링의 기본 예외 처리 방법

![spring-exception-handling](/assets/img/spring/mvc/exception-handling/spring-exception-handling.png)  

자바 프로그램은 예외가 발생하면 예외 정보를 남기고 쓰레드가 종료되는 반면에 스프링은 예외가 발생했을 때 웹 애플리케이션이 종료되지 않고 HTTP 상태 코드가 노출이 됩니다.
즉 스프링 부트는 컨트롤러 이하에서 발생한 예외를 캐치하여 쓰레드를 종료 시키는 것이 아닌 예외 내용을 디스패처서블릿에서 에러컨트롤러로 요청을 보냄으로써 마치 정상 요청인 것처럼 동작하게 됩니다.
하지만 컨트롤러를 2번 호출한다는 것은 꽤나 불필요하다고 느껴지며 또 이는 필터나 인터셉터를 2번 호출하는 것을 의미합니다.

이를 방지하기 위해 필터같은 경우는 등록할 때 DispatcherType을 설정 할 수 있으며 별다른 설정이 없다면 DispatcherType이 ERROR일 경우에는 호출되지 않습니다.  

![do-filter](/assets/img/spring/mvc/exception-handling/do-filter.png)  

인터셉터는 최초 인터셉터 등록 시 URI 패턴으로 처리 에러 URI를 제외해야합니다.  

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

먼저 기본으로 사용될 에러 속성 값을 생성합니다.  
- timestamp: 에러 발생 시간
- status: Http 상태
- error: 에러 코드
- path: 호출 uri
- exception: 최상위 예외 클래스 이름(Optional)
- message: 에러 메세지(Optional)
- errors: BindingExecption에 의해 생긴 에러 목록(Optional)
- trace: 에러 스택 트레이스(Optional)

![get-error-attributes](/assets/img/spring/mvc/exception-handling/get-error-attributes.png)  

그 후 ``DefaultErrorAttributes``에서는 전체 에서 속성값들 중에서 설정에 맞게 불필요한 속성을 제거합니다. 
``exception, message``에 대한 속성을 제거한 후 결과적으로 아래와 같은 에러 속성을 클라이언트에게 반환합니다. 

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
클라이언트는 ``Internal Server Error``라는 다소 모호한 메시지 보다는 명시적으로 해당 오류에 대한 유의미한 응답 메세지를 받고 싶을 것입니다. 
그러므로 별도의 다양한 에러 처리를 통해 상황에 맞는 적절한 에러 응답을 제공해야 합니다. 

# 스프링이 제공하는 예외 처리 방법

## HandlerExceptionResolver




## DefaultHandlerExceptionResolver

## ResponseStatusExceptionResolver

## ExceptionHandlerExceptionResolver

### @ControllerAdvice

### @RestControllerAdvice


오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.  

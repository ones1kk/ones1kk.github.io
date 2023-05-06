---
title: 스프링의 예외 처리 방법
date: 2023-05-06 00:10:00 +09:00
categories: [Spring, MVC]
tags: [BasicErrorController, HandlerExceptionResolver, ResponseStatus, ExceptionHandler, ControllerAdvice, RestControllerAdvice]
---

# 스프링의 기본 예외 처리 방법

![spring-exception-handling](/assets/img/spring/mvc/exception-handling/spring-exception-handling.png)  

스프링은 기본적으로 예외를 받기 위한 ``org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController``를 구현해 놓았고, 이는 ``server.error.path``에 설정된 주소(기본은/error)가 매핑된 컨트롤러 입니다.
컨트롤러 이하에서 발생한 예외를 캐치하여 해당 내용을 디스패처서블릿에서 에러컨트롤러로 요청을 보내게 됩니다.
즉 최초 요청 외에 에러컨트롤러를 한 번 더 요청하여 총 2번의 컨트롤러 호출이 발생합니다. 
2번 컨트롤러를 호출한다는 것은 필터나 인터셉터를 2번 호출하게 되는 문제를 야기함을 의미합니다. 
이를 방지하기 위해 필터를 등록할 때, DispatcherType을 설정 할 수 있으며, 인터셉터같은 경우는 최초 인터셉터 등록 시 URI 패턴으로 처리해야 합니다.

![do-filter](/assets/img/spring/mvc/exception-handling/do-filter.png)

> 필터같은 경우는 별다른 설정이 없다면, ``javax.servlet.DispatcherType`` 값이 ERROR일 경우에는 호출되지 않습니다.

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

베이직에러컨트롤러는 ``/error`` 경로일 때 실행되는 컨트롤러로, 이는 properties에서 ``server.error.path``의 값으로 변경 가능합니다. 
주요 메소드로는 ``errorHtml(HttpServletRequest, HttpServletResponse), error(HttpServletRequest)``가 있는데, 해당 메소드는 에러컨트롤러를 호출할 때 들어오는 리퀘스트의 유형에 따라 반환하는 타입이 다릅니다.  
메소드 네임에서 유추할 수 있듯이, ``errorHtml()``같은 경우는 웹 화면에 에러 페이지를 랜더링해주기 위해 ``ModelAndView`` 객체를 반환합니다. 
따로 에러 페이지를 설정을 하지 않았다면, 기본 에러 페이지인 ``Whitelabel Error Page`` 화면이 노출됩니다.   

![whitelabel-error-page](/assets/img/spring/mvc/exception-handling/whitelabel-error-page.png)  

그 외로 반환할 MediaType이 HTML이 아니라면, ``error()`` 메소드를 통해 리스폰즈 객체가 반환이 됩니다. 

![postman](/assets/img/spring/mvc/exception-handling/postman.png)  

두 메소드는 모두 반환할 에러 속성 값을 ``getErrorAttributes(HttpServletRequest, ErrorAttributeOptions)`` 메소드를 통해 가져옵니다.

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

하지만 위와 같은 응답은 클라이언트 입장에서는 다소 불친절하게 느껴질 수 있습니다. 
클라이언트는 ``Internal Server Error``라는 다소 모호한 메시지 보다는 명시적으로 해당 오류에 대한 유의미한 응답 메세지를 받고 싶을 것입니다. 
그러므로 별도의 다양한 에러 처리를 통해 상황에 맞는 적절한 에러 응답을 제공해야 한다. 

# 스프링이 제공하는 예외 처리 방법

## ExceptionResolver

###  ResponseStatus

### ExceptionHandler

### @ControllerAdvice

### @RestControllerAdvice


오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.  

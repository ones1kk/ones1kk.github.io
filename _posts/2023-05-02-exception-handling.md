---
title: 스프링의 예외 처리 방법
date: 2023-05-03 00:10:00 +09:00
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

필터같은 경우는 별다른 설정이 없다면, ``javax.servlet.DispatcherType`` 값이 ERROR일 경우에는 호출되지 않습니다.

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

# 스프링이 제공하는 예외 처리 방법

## ExceptionResolver

###  ResponseStatus

### ExceptionHandler

### @ControllerAdvice

### @RestControllerAdvice


오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.  

---
title: 필터(Filter) & 인터셉터(Interceptor)
date: 2023-04-21 00:09:00 +09:00
categories: [Spring, MVC]
tags: [Filter, Interceptor]
---

# 들어가기에 앞서

스프링(Spring)은 자바(Java) 기반의 웹 애플리케이션 개발을 위한 프레임워크로, 다양한 기능과 기술을 제공합니다.   
그 중에서도 Filter와 Interceptor는 웹 애플리케이션의 요청과 응답을 처리하고 조작하는 기능을 제공하는데, 이 두 가지의 개념과 차이점에 대해 알아보겠습니다.

# Filter

![spring-request-lifecycle](/assets/img/spring/mvc/request-lifecycle/spring-request-lifecycle.jpg)
> [출처](https://justforchangesake.wordpress.com/2014/05/07/spring-mvc-request-life-cycle/)

필터는 스프링에서 제공하는 독자적인 기술이 아닌 자바 서블릿에서 제공하는 기술입니다.  
``` DispatcherServlet ```이 요청을 처리하기 전, 후의 웹 애플리케이션 요청과 응답을 가로채 필터링하는 역할을 합니다.  
또한 필터는  **웹 애플리케이션 영역(Context)** 내에서 동작하므로, 스프링의 영역를 접근하기 어렵기 때문에 웹 애플리케이션 영역 내에서 필요한 자원들을 활용하여 사용되는 것이 특징입니다.

![filterchain](/assets/img/spring/mvc/request-lifecycle/filterchain.png)
> [출처](https://docs.spring.io/spring-security/reference/servlet/architecture.html)

필터는 여러개가 모여서 FilterChain이라는 하나의 체인으로 형성되어 사슬처럼 연쇄적으로 동작합니다.  
다음 필터나 서블릿에게 요청을 전달하여 동작하며, 여러 개의 필터를 조합하여 사용할 수 있습니다.

일반적으로 필터의 사용 용도는 아래와 같습니다.

1. 보안 관련 공통 작업.
2. 요청에 대한 로깅 또는 감사.
3. 이미지/데이터 압축 및 문자열 인코딩.

추가로 ``` Filter ``` 인터페이스에서 제공해주는 메소드를 확인해보도록 하겠습니다.

```java 
public interface Filter {
	default void init(FilterConfig filterConfig) throws ServletException {};
	void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException;
	default void destroy() {};
}
```  

1. init: 서블릿 컨테이너는 필터를 인스턴스화한 후 정확히 한 번 init 메서드를 호출합니다. 필터가 필터링 작업을 수행하도록 요청받기 전에 init 메서드가 성공적으로 완료되어야 합니다.
2. doFilter: 리소스에 대한 클라이언트 요청으로 요청/응답 쌍이 체인을 통과할때마다 웹 컨테이너에 의해 호출됩니다.
3. destory: 이 메서드는 필터의 doFilter 메서드 내의 모든 스레드가 종료되었거나 시간 초과 기간이 경과한 후에만 호출됩니다. 웹 컨테이너가 필터에 의해 호출되어 필터가 서비스를 중단하고 있음을 나타냅니다.

# Interceptor

![interceptor-work](/assets/img/spring/mvc/request-lifecycle/interceptor-work.png)
> [출처](https://medium.com/geekculture/what-is-handlerinterceptor-in-spring-mvc-110681604bd7)

스프링의 인터셉터는 스프링 MVC에서만 동작하는 개념으로, ``` DispatcherServlet과 Controller ``` 사이에서 요청과 응답을 가로채어 처리하는 기능을 제공합니다.  
인터셉터는 스프링 MVC의 특정 핸들러(컨트롤러)의 메소드 호출 전, 후 또는 완료 후에 작업을 수행할 수 있습니다.   
인터셉터는 컨트롤러의 실행, 뷰의 렌더링 전, 후 등 다양한 시점에서 작업을 수행할 수 있습니다.

또한 인터셉터의 경우 **스프링 영역(context)**내에 관리되고 있어서 스프링에서 생성된 빈들에 자유롭게 접근할 수 있습니다.  
가령 ``` HandlerInterceptor ```는 기분적인 구조를 유지하면서 스프링 빈을 DI(의존성 주입)받아 사용할 수 있기 때문에 스프링 컨택스트 리소스 사용을 수월하게 할 수 있습니다.

일반적으로 인터페이스의 사용 용도는 아래와 같습니다.

1. 인증/인가 관련 공통 작업.
2. 핸들러(Controller)로 넘겨줄 정보의 가공.
3. API 호출에 대한 로깅 또는 감사.

추가로 ``` HandlerInterceptor ``` 인터페이스에서 제공해주는 메소드를 확인해보도록 하겠습니다.

```java 
    public interface HandlerInterceptor {
        default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            return true;
        }
        default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {
        }
        default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {
        }
    }
```  

1. preHandle: ``` HandlerAdapter ```가 핸들러(컨트롤러) 메소드를 호출하기 전에 호출됩니다. 일반적으로 HTTP 오류를 보내거나 사용자 지정 응답을 작성합니다.
2. postHandle: ``` HandlerAdapter ```가 핸들러(컨트롤러) 메소드를 호출한 후 실행되며, ``` DispatcherServlet ```이 화면을 렌더링하기 전에 호출됩니다.
3. afterCompletion: 요청 처리 완료 후, 즉 화면 렌더링 후 호출됩니다. 핸들러(컨트롤러) 메소드 결과에 상관 없이 호출되므로 리스소 정리를 수월하게 할 수있습니다.

## HandlerInterceptorAdapter

```java 
/**
 * Abstract adapter class for the {@link AsyncHandlerInterceptor} interface,
 * for simplified implementation of pre-only/post-only interceptors.
 *
 * @author Juergen Hoeller
 * @since 05.12.2003
 * @deprecated as of 5.3 in favor of implementing {@link HandlerInterceptor}
 * and/or {@link AsyncHandlerInterceptor} directly.
 */
@Deprecated
public abstract class HandlerInterceptorAdapter implements AsyncHandlerInterceptor {

}
```   

``` HandlerInterceptor ``` 인터페이스의 메소드를 미리 구현해두고, 사용자가 필요한 메소드만 오버라이드하여 구현할 수 있도록 도와주는 클래스입니다.

하지만 스프링 v5.3부터는 **Deprecated** 되면서, ``` HandlerInterceptor 또는 AsyncHandlerInterceptor ```를 구현하여 사용하는 것으로 변경되었습니다.

## AsyncHandlerInterceptor

```java 

public interface AsyncHandlerInterceptor extends HandlerInterceptor {

	default void afterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
	}
}
```   

**Servlet 3.0**부터 비동기 처리를 위한 ``` AsyncContext 와 javax.servlet.AsyncListener ```라는 클래스와 인터페이스가 도입되었습니다.  
``` AsyncHandlerInterceptor ``` 스프링 MVC에서 비동기 요청 처리 시 필요한 로직을 구현할 수 있도록 지원해주는 클래스입니다.

비동기 요청을 시작하면 ``` DispatcherServlet```은 일반적인 동기 요청 수행처럼 ```postHandle 및 afterCompletion ```을 호출하지 않고 대신 ``` afterConcurrentHandlingStarted ```를 호출하여 서블릿 컨테이너로 넘기기 전에 작업을 진행합니다.

# 차이

|          -          |           필터(Filter)           |인터셉터(Interceptor)|
|:-------------------:|:------------------------------:|:---:|
|      범위(Scope)      |          웹 애플리케이션 전역           |스프링 컨테이너 내부|
|    위치(Position)     |   ```DispatcherServlet``` 이전   |``` DispatcherServlet ```과 핸들러(컨트롤러) 사이|
| 구현(Implementation)  |```javax.servlet.Filter``` 인터페이스|``` HandlerInterceptor ``` 인터페이스|
| 작업 시점(Time of Work) |    웹 애플리케이션 요청 전처리, 응답 후처리     |핸들러(컨트롤러)의 실행 전, 후 또는 뷰의 렌더링 전, 후 |    

# 결론

필터는 웹 애플리케이션 전역에서 동작하며, ```DispatcherServlet ``` 이전에 요청과 응답을 가로채고 처리하는 기능을 가지고 있습니다.  
반면 인터셉터는 스프링 컨테이너에서만 동작하며, ``` DispatcherServlet ```과 핸들러(Controller) 사이에서의 특정 핸들러에만 적용되는 기능을 가지고 있습니다.  
따라서 필요한 기능과 범위에 따라 적절하게 필터와 인터셉터를 선택하여 사용하면 됩니다.

오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.  

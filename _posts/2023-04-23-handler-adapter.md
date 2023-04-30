---
title: 핸들러어댑터(HandlerAdapter)
date: 2023-04-23 00:01:00 +09:00
categories: [Spring, MVC]
tags: [Handler, HandlerMapping, HandlerAdapter]
---

# 들어가기에 앞서

스프링 MVC는 클라이언트로부터 들어오는 HTTP 요청을 처리하고, 응답을 생성하는 등의 작업을 수행하는데, 이를 가능하게 해주는 중요한 컴포넌트 중 하나가 ``` org.springframework.web.servlet.HandlerAdapter ```입니다. 핸들러어댑터는 스프링 MVC에서 클라이언트의 요청을 처리하는 실제 로직을 구현한 컨트롤러(Controller)를 의미하는 핸들러(Handler)를 실행하고 처리하는 역할을 담당합니다. 핸들러어댑터는 핸들러의 실행을 담당하여, 요청의 처리 결과를 적절한 응답으로 변환하여 클라이언트에게 반환합니다.

# 특징

## 핸들러(Handler)

웹 요청을 처리(Handle)하는 객체를 핸들러(Hanlder)라고 부릅니다. 일반적으로 알고 있는 컨트롤러(Controller)가 웹 요청을 처리 할 수 있다고 알고있지만 ``` org.springframework.web.servlet.DispatcherServlet ```은 컨트롤러 이외에도 여러 종류의 객체(Handler)로 웹 요청을 처리할 수 있습니다. 결국 컨트롤러는 웹 요청을 처리하는 한 핸들러로서 생각하면 되겠습니다.

## 핸들러매핑(HandlerMapping)

웹 요청 URL과 핸들러를 연결해주는 객체입니다. 스프링은 여러 가지 핸들러매핑 전략을 사용하여 웹 요청과 핸들러를 매핑합니다. 각각의 핸들러매핑 객체들이 혼용되어 사용되는 것을 방지하기위해 운선 순위가 높은 핸들러매핑을 사용합니다. 만일 적당한 객체가 없다면 디스패처서블릿은 유명한 ``` HTTP 오류 코드 404 ```를 응답으로 받습니다.

### 핸들러매핑의 종류

아래 메소드는 디스패처서블릿에서 적절한 핸들러매핑에서 핸들러를 가져오는 메소드입니다. 디스패처서블릿에서 관리하는 핸들러매핑을 아래에서 확인할 수 있습니다.

![handler-mappings](/assets/img/spring/mvc/handler-adapter/handler-mappings.png)

핸들러매핑은 아래의 우선순위로 실행이 됩니다.

1. RequestMappingHandlerMapping: 가장 일반적인 핸들러매핑 전략으로, 어노테이션 기반의 컨트롤러와 매핑합니다. ``` @RequestMapping, @GetMapping, @PostMapping ```등의 어노테이션을 사용하여 웹 요청과 핸들러를 매핑합니다.

2. BeanNameUrlHandlerMapping: 빈의 이름을 기반으로 URL 패턴과 매핑합니다.

3. RouterFunctionMapping: 함수형 엔드포인트를 정의하는 방식으로 요청과 핸들러를 매핑합니다.

4. SimpleUrlHandlerMapping: URL 패턴과 매핑되는 핸들러를 정의할 수 있습니다. Ant 스타일의 패턴을 사용하여 요청과 핸들러를 매핑합니다.

5. WelcomePageHandlerMapping: Welcome 페이지(기본 페이지)를 처리하는 핸들러매핑입니다. 루트 경로("/")로 들어오는 요청을 처리할 때 사용됩니다.

위의 핸들러매핑 이용하여 적절한 핸들러를 찾지 못해 null이 반환되면, 디스패처서블릿은 404 에러 코드를 반환합니다.

![no-handler-found](/assets/img/spring/mvc/handler-adapter/no-handler-found.png)

## 핸들러어댑터(HandlerAdapter)

핸들러어댑터는 이름에서 확인할 수 있듯이 어댑터 패턴으로 구현된 클래스입니다. 핸들러어댑터를 통해 스프링에서 지원하는 다양한 핸들러들을 처리할 수 있는 표준화된 인터페이스입니다.

![get-handler-adapter](/assets/img/spring/mvc/handler-adapter/get-handler-adapter.png)

### 핸들러어댑터의 종류

1. RequestMappingHandlerAdapter: ``` @RequestMapping ```을 사용한 핸들러를 처리하는데 사용됩니다. ``` supports() ```메소드에서는 해당 핸들러가가 ``` RequestMappingHandler ``` 인터페이스를 구현하고 있어야 합니다.

2. HandlerFunctionAdapter: 스프링 5에서 추가된 함수형 핸들러를 처리하는데 사용됩니다. ``` supports() ``` 메소드에서는 HandlerFunction 인터페이스를 구현하고 있어야 합니다.

3. HttpRequestHandlerAdapter: ``` javax.servlet.http ``` 패키지의 ``` HttpRequestHandler ``` 인터페이스를 구현한 핸들러를 처리하는데 사용됩니다. ``` supports() ``` 메소드에서는 해당 핸들러가 ``` HttpRequestHandler ``` 인터페이스를 구현하고 있어야 합니다.

4. SimpleControllerHandlerAdapter: 스프링 3 이전에 사용되었던 ``` SimpleController ``` 인터페이스를 구현한 컨트롤러를 처리하는데 사용됩니다. ``` supports() ``` 메소드에서는 해당 컨트롤러가 ``` SimpleController ``` 인터페이스를 구현하고 있어야 합니다.

디스패처서블릿은 웹 요청을 처리할 적절한 핸들러를 찾았다면, 각 핸들러를 처리할 수 있는 핸들러어댑터를 가져옵니다.

![call-hanlde](/assets/img/spring/mvc/handler-adapter/call-handle.png)

이후 핸들러어댑터는 핸들러를 처리할 수 있는 적절한 어댑터로 핸들러의 메소드를 실행시킵니다.

![invoke-handler-method-1](/assets/img/spring/mvc/handler-adapter/invoke-handler-method-1.png)

현재까지는 실행시킬 핸들러 & 메소드의 정보를 구했을(매핑했을)뿐, 사실 핸들러 메소드를 동작시키기 위한 준비는 아직 끝난 상태가 아닙니다. 가령 해당 핸들러에서 처리할 파라미터, 반환 값 등을 어떤 식으로 어떻게 처리하기 위해 내부적으로는 해당 부분 처리 객체 세팅을 진행해야합니다.

![invoke-handler-method-2](/assets/img/spring/mvc/handler-adapter/invoke-handler-method-2.png)

``` asyncManager, argumentResolvers, returnValueHandlers... ``` 들과 같은 사전 작업을 끝낸 후 실제 핸들러 메소드를 **리플렉션(Reflection)**을 통해 실행시킵니다.  
개발자가 작성한 코드를 스프링에서는 아주 정교한 단계별 검증을 통해 런타임 시점에 동적으로 처리 가능하도록 구현해 놓은 것입니다.

# 결론

핸들러어댑터는 스프링 프레임워크에서 핸들러 메소드를 호출하고, 그 결과를 처리하는 역할을 수행하는 인터페이스 중 하나입니다.
핸들러어댑터는 컨트롤러와 뷰를 연결하는 등의 작업을 수행하여 스프링 MVC의 핵심적인 동작을 가능하게 합니다. 따라서 핸들러어댑터의 역할과 작동 원리에 대해 깊이 이해하면 스프링을 이용하는 개발자들에게 큰 도움이 될 것으로 보여집니다.

오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다. 

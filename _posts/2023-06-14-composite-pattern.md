---
title: 컴포지트 패턴(Composite Pattern)
date: 2023-06-14 21:50:00 +09:00
categories: [ Software Architecture, Design Pattern ]
tags: [ GoF, Design-Pattern, Composite-Pattern ]
---

# Composite Pattern

컴포지트 패턴(Composite Pattern)은 Gang of Four(GoF) 디자인 패턴 중 하나로 [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.yes24.com/Product/Goods/17525598)라는 책에서 소개 된 23가지 디자인 패턴 중 하나입니다.
컴포지트 패턴은 구조 패턴으로서 객체들을 트리 구조로 구성하는 방법을 제공하며 개별 객체와 복합 객체를 일관된 방식으로 다룰 수 있게 해주고, 객체 간의 전체-부분 계층을 표현할 수 있도록 합니다.
컴포지트 패턴의 주요 아이디어는 **단일 객체와 복합 객체를 동일한 인터페이스로 다루기**입니다. 
이를 통해 클라이언트는 개별 객체와 복합 객체를 구별하지 않고 일관된 방식으로 즉, 클라이언트는 개별 객체든 복합 객체든 동일한 방식으로 메소드를 호출하여 사용할 수 있습니다.

# 구조

![composite-pattern](/assets/img/software-architecture/design-pattern/composite-pattern/composite-pattern.png)  
> [출처](https://ko.wikipedia.org/wiki/%EC%BB%B4%ED%8F%AC%EC%A7%80%ED%8A%B8_%ED%8C%A8%ED%84%B4)

- Component
  - 복합 객체와 개별 객체를 동일한 인터페이스로 표현하는 역할이며 복합 객체와 단일 객체에 대한 공통 작업을 선언합니다. 

- Leaf
  - 복합 객체의 단일 객체를 나타냅니다.  
  - Component 인터페이스를 구현하며 더 이상의 하위 요소를 가질 수 없는 주로 단일 객체로 동작합니다.

- Composite
  - 복합 객체를 나타내는 역할입니다.
  - Component 인터페이스를 구현하며, 자식 요소를 가질 수 있습니다.
  - 자식 요소를 관리하고, 자식 요소에 대한 작업을 위한 메서드를 구현합니다.
  - 복합 객체의 구조를 재귀적으로 표현할 수 있습니다.

# 스프링의 컴포지트 패턴

## HandlerExceptionResolverComposite

``org.springframework.web.servlet.handler.HandlerExceptionResolverComposite``는 ``org.springframework.web.servlet.HandlerExceptionResolver``를 구현한 복합 객체입니다.
리졸버들을 담는 컨테이너 역할을 수행하면서 자체적으로 예외 처리를 수행하지는 않고 등록된 여러 개의 이셉션 리졸버를 내부적으로 가지고 있으며, 예외가 발생했을 때 등록된 이셉션 리졸버들에게 차례대로 처리를 위임합니다. 
이를 통해 각각의 이셉션 리졸버가 자신이 담당하는 예외 전략에 따른 적절한 처리를 수행할 수 있습니다. 

![process-dispatch-result](/assets/img/software-architecture/design-pattern/composite-pattern/process-dispatch-result.png)  

먼저 애플리케이션에서 예외가 발생하면 ``DispatcherServlet``은 캐치한 예외를 처리하기 위한 ``processDispatchResult()`` 메소드를 실행시킵니다. 
발생한 예외가 ``ModelAndViewDefiningException``가 아니라면 등록된 이셉션 리졸버들을 순차적으로 실행시키기 위한 ``processHandlerException()`` 메소드를 실행시킵니다. 

![process-handler-exception](/assets/img/software-architecture/design-pattern/composite-pattern/process-handler-exception.png)  

``processHandlerException()`` 메소드를 실행시키게 되면 컴포지트 패턴으로 구현된 handlerExceptionResolvers를 볼 수 있습니다.
단일 객체 ``org.springframework.boot.web.servlet.error.DefaultErrorAttributes``와 복합 객체의 ``org.springframework.web.servlet.handler.HandlerExceptionResolverComposite``가 담겨있습니다. 
``DefaultErrorAttributes``는 단일 객체로서 기본 에러 속성과 관련 처리를 하며, 복합 객체인 ``HandlerExceptionResolverComposite``는 각 리졸버가 담당하는 예외 처리 전략에 따라 예외를 순차적으로 처리합니다.  

### ViewResolverComposite

``org.springframework.web.servlet.view.ViewResolverComposite``는 ``org.springframework.web.servlet.ViewResolver``를 구현한 복합 객체입니다. 
``ViewResolverComposite`` 또한 리졸버들을 담는 컨테이너 역활을 수행하면서 등록된 여러 개의 뷰 리졸버를 내부적으로 유지하고, 요청된 뷰를 찾을 때 등록된 뷰 리졸버들을 순차적으로 검사합니다. 
첫 번째로 매칭되는 뷰 리졸버가 발견되면 해당 뷰 리졸버를 사용하여 요청된 뷰를 찾고 랜더링합니다. 

![resolve-view-name](/assets/img/software-architecture/design-pattern/composite-pattern/resolve-view-name.png) 

하지만 ``ViewResolverComposite``가 비어 있는 것을 확인할 수 있는데, 이는  ``org.springframework.web.servlet.config.annotation.WebMvcConfigurer.configureViewResolvers()`` 메소드를 통해 ``ViewResolver`` 등록해야 합니다.  

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

@Override
public void configureViewResolvers(ViewResolverRegistry registry) {
    registry.viewResolver(new MyViewResolver());
}

  static class MyViewResolver implements ViewResolver {
      @Override
      public View resolveViewName(String viewName, Locale locale) throws Exception {
          return null;
      }
  }
}
```  

![add-view-resolver](/assets/img/software-architecture/design-pattern/composite-pattern/add-view-resolver.png)  

``WebMvcConfigurer``를 상속 받아 구현한 설정 파일을 통해 등록한 ``viewResolver``들을 ``ViewResolverComposite``가 관리하며 실행시키게 됩니다.

오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다. 

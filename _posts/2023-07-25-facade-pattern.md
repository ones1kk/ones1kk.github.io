---
title: 파사드 패턴(Facade Pattern)
date: 2023-07-26 00:00:00 +09:00
categories: [ Software Architecture, Design Pattern ]
tags: [ GoF, Design-Pattern, Facade-Pattern ]
---

# Facade Pattern

파사드 패턴(Facade Pattern)은 Gang of Four(GoF) 디자인 패턴 중 하나로 [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.yes24.com/Product/Goods/17525598) 책에서 소개 된 23가지 디자인 패턴 중 하나입니다.
파사드 패턴은 **구조 패턴**으로써 하위 시스템을 보다 쉽게 사용할 수 있게 해주는 고급 인터페이스로 표현됩니다. 

먼저 파사드(Facade)는 프랑스어 Façade 에서 유래된 단어로 건물의 '외관'이라는 뜻입니다.
하위 시스템(내부 구조)에 있는 인터페이스들에 대한 통합된 고급 인터페이스(외벽)를 제공하는 패턴입니다.
이로써 내부 시스템의 복잡도를 감추기 위해 복잡한 기능을 감싸고 상호 작용할 더 단순한 메소드를 제공하는 계층을 생성하고 서브시스템과 상호 작용 복잡도를 낮추는데 의미가 있습니다.

# Structure

![facade-pattern-diagram](/assets/img/software-architecture/design-pattern/facade-pattern/facade-pattern-diagram.png) 

[출처](https://psid23.medium.com/facade-design-pattern-8db51f37cc44)  

- Facade
  - 클라이언트의 요청을 서브시스템 객체에 전달하는 단순하고 일관된 통합 인터페이스입니다.
  - 클라이언트와 서브시스템이 긴밀하게 연결되는 것을 방지하며 서브시스템과의 상호 작용의 복잡도를 낮춥니다. 
- Subsystem
  - Facade에 대한 정보를 가지지 않고 서브시스템의 기능을 구현하는 클래스입니다.
  - 서브시스템은 Facade에 의해서만 사용됩니다.

## 디미터의 법칙 (Law of Demeter) - 최소 지식의 원칙(Principle of least knowledge)

디미터의 법칙은 객체 사이의 결합도를 낮추고 응집도를 높이기 위해 정의한 원칙으로 객체 지향 프로그래밍 설계 원칙 중 하나입니다.
메소드 내에서 호출하는 메소드는 다음과 같은 범위에 속해야 합니다. 

1. 해당 클래스의 메소드
2. 메소드에 매개변수로 전달된 객체의 메소드
3. 메소드에서 생성된 객체의 메소드

즉 객체가 다른 객체와 상호작용할 때 가능한 한 가까운 범위 내에 있는 객체와만 통신하도록 제한하며 객체는 자신이 가지고 있는 메소드를 통해 직접적으로 관련된 객체와만 소통하고 다른 객체의 내부 구조를 불필요하게 알 필요가 없도록 해야 합니다.  

Facade Pattern에서 클라이언트는 단순히 Facade 클래스의 인터페이스를 사용하여 서브시스템과 상호작용하므로 클라이언트는 서브시스템 내부의 복잡한 구조를 알 필요가 없어 디미터의 법칙을 준수하게 됩니다.

# Example of Facade Pattern

## StandardSessionFacade

![standard-session-facade](/assets/img/software-architecture/design-pattern/facade-pattern/standard-session-facade.png)  

`org.apache.catalina.session.StandardSessionFacade` 는 Apache Tomcat 웹 애플리케이션 서버에서 세션 관리를 담당하는 클래스로 더 편리하게 세션을 다룰 수 있도록 해주는 Facade입니다. 
`StandardSessionFacade`는 `StandardSession` 클래스의 복잡한 내부 구조를 감추고 `javax.servlet.http.HttpSession` 인터페이스를 구현하는 역할을 담당합니다.
`StandardSession` 클래스의 복잡한 기능을 단순화된 인터페이스로 제공하여 Tomcat 내부적으로 StandardSession과 같은 세션 구현을 변경하더라도 `javax.servlet.http.HttpSession` 인터페이스와 `StandardSessionFacade` 클래스는 변경하지 않고도 호환성을 유지할 수 있습니다.

![standard-session-facade-method](/assets/img/software-architecture/design-pattern/facade-pattern/standard-session-facade-method.png)  

![standard-session-method](/assets/img/software-architecture/design-pattern/facade-pattern/standard-session-method.png)

위의 사진과 같이 `StandardSessionFacade` 클래스를 사용하여 `HttpSession` 인터페이스의 메소드를 호출하면 실제로는 `StandardSession` 클래스의 내부 기능이 실행됩니다.

## HttpClientFacade

![http-client-facade](/assets/img/software-architecture/design-pattern/facade-pattern/http-client-facade.png)

`jdk.internal.net.http.HttpClientFacade`는 JDK 내부의 구현과 관련된 비공개 API에 속하는 클래스로 직접적으로 접근하거나 사용하지 못하도록 설정되었습니다.
`HttpClientFacade` 클래스는 `jdk.internal.net.http.HttpClientImpl`에 대한 패키징된(facade) 래퍼(wrapper) 역할을 합니다. 
즉 외부에서 `HttpClientFacade` 클래스를 통해 `HttpClientImpl`의 인스턴스를 생성하고 메소드를 호출할 수 있도록 도와줍니다.  

![http-client-facade-method](/assets/img/software-architecture/design-pattern/facade-pattern/http-client-facade-method.png)  

![http-client-impl-method](/assets/img/software-architecture/design-pattern/facade-pattern/http-client-impl-method.png)

외부에서 HTTP 클라이언트를 사용해야 할 때는 ``java.net.http.HttpClient``를 사용하는 것이 권장됩니다. 

오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.

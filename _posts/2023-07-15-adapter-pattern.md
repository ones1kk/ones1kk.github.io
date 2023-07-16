---
title: 어댑터 패턴(Adapter Pattern)
date: 2023-07-15 02:50:00 +09:00
categories: [ Software Architecture, Design Pattern ]
tags: [ GoF, Design-Pattern, Adapter-Pattern ]
---

# Adapter Pattern

어댑터 패턴(Adapter Pattern)은 Gang of Four(GoF) 디자인 패턴 중 하나로 [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.yes24.com/Product/Goods/17525598) 책에서 소개 된 23가지 디자인 패턴 중 하나입니다.
어댑터 패턴은 **구조 패턴**으로써 인터페이스를 사용자가 기대하는 다른 인터페이스로 변환하는 패턴으로 호환되지 않는 인터페이스 때문에 함께 동작할 수 없는 인터페이스들이 함께 작동할 수 있도록 변환해주는 디자인 패턴입니다.
이는 인터페이스가 현재의 시스템과 호환되지 않는다고해서 현재의 시스템을 변경을 해야하는 것이 아닌 새로운 소스 코드 수정 없이 함께 작동하도록 만들어주는 것에 의미가 있습니다. 

어댑터 패턴은 주로 다음과 같은 문제를 해결하는데 사용됩니다. 

- 기존 클래스와 인터페이스를 호환시켜야할 때 
  - 기존 클래스의 인터페이스와 클라이언트의 요구사항이 일치하지 않을 때 어댑터 패턴을 사용하여 기존 클래스의 인터페이스를 타겟 인터페이스에 맞추어 호환성을 제공할 수 있습니다.
- 이미 존재하는 클래스를 재사용하고 싶을 때
  - 기존 클래스를 활용하면서 새로운 인터페이스에 맞추어 사용하고자 할 때 어댑터 패턴을 사용하여 기존 클래스를 재사용할 수 있습니다.
- 인터페이스의 변환 및 통/폐합
  - 서로 다른 인터페이스를 가진 여러 개의 클래스를 통합해야 할 때 어댑터 패턴을 사용하여 인터페이스 변환을 수행하고 클라이언트가 단일 인터페이스를 통해 여러 클래스를 사용할 수 있도합니다.

# Structure

어댑터 패턴은 기존 인터페이스 호환을 위해 상속(Inheritance)과 합성(Composition)이 두 가지 주요한 접근 방식이 있습니다.

## 클래스 어댑터

![adapter-pattern](/assets/img/software-architecture/design-pattern/adapter-pattern/class-adapter-pattern.png)  
[출처](https://www.tutorialspoint.com/design_pattern/adapter_pattern.htm)

상속을 사용한 어댑터 패턴에서는 Adapter 클래스가 Adaptee 클래스와 Target 클래스를 상속합니다.
Adaptee 클래스와 Target 인터페이스를 동시에 사용하며 상속을 통해 Target 인터페이스를 구현하는 유연성을 제공합니다. 
하지만 기존 클래스를 상속받을 수 없거나 다른 클래스를 상속받고 있는 경우에는 어댑터 패턴을 적용하기 어렵습니다.

- Clitent: Adaptee를 사용하려는 주체입니다. 
- Target(Client Interface): Adapter가 구현하는 인터페이스로 Client는 Target을 통해 Adaptee를 사용하게 됩니다.
- Adaptee(Service): Adapter의 대상 객체로 기존 시스템, 외부 시스템, 써드파티 라이브러리 등을 의미합니다. 
- Adapter: 호환되지 않는 Client와 Adaptee의 사이에서 둘을 연결시켜주는 역활을 담당하며 Target을 구현하여 Adaptee가 수행가능한 방법으로 전달합니다.   


## 객체 어댑터

![adapter-pattern](/assets/img/software-architecture/design-pattern/adapter-pattern/object-adapter-pattern.png)  
[출처](https://www.tutorialspoint.com/design_pattern/adapter_pattern.htm)

Adapter 클래스는 Adaptee 클래스의 인스턴스를 멤버 변수로 가지고 있습니다.
Adapter 클래스는 Target 인터페이스를 구현하여 내부에서 Adaptee 클래스의 인스턴스를 활용하여 Target 인터페이스를 구현합니다.
합성을 사용하기 때문에 Adapter 클래스가 여러 개의 Adaptee 클래스를 동시에 사용하거나 다른 클래스와 함께 사용하는 것도 가능합니다.

- Clitent: Adaptee를 사용하려는 주체입니다.
- Target(Client Interface): Adapter가 구현하는 인터페이스로 Client는 Target을 통해 Adaptee를 사용하게 됩니다.
- Adaptee(Service): Adapter의 대상 객체로 기존 시스템, 외부 시스템, 써드파티 라이브러리 등을 의미합니다.
- Adapter: 호환되지 않는 Client와 Adaptee의 사이에서 둘을 연결 시켜주는 역활을 담당하며 Target을 구현하여 Adaptee가 수행가능한 방법으로 전달합니다.

# Example of Adapter Pattern

## InputStreamReader

```java
BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));
```

문자열을 입력 받는 스트림으로 ``System.in``을 사용하고 싶지만 ``BufferedReader``의 생성자에는 ``Reader``가 와야하기 때문에 중간에서 어댑터 역활을 하는 ``InputStreamReader``를 활용해 위와 같은 코드가 작성 가능합니다.  

![buffered-reader-construct](/assets/img/software-architecture/design-pattern/adapter-pattern/buffered-reader-construct.png)

![input-stream-reader](/assets/img/software-architecture/design-pattern/adapter-pattern/input-stream-reader.png) 

``InputStreamReader``은 ``Reader`` 인터페이스를 상속 받아 구현하고 있으며 ``InputStream``을 매개변수로 받는 생성자를 가지고 있습니다.  

![input-stream-reader-construct](/assets/img/software-architecture/design-pattern/adapter-pattern/input-stream-reader-construct.png)  

``System.in`` 을 ``InputStreamReader`` 생성자에 넘겨 인스턴스화하면  ``Reader`` 부모 클래스를 상속하고 있는 ``BufferedReader``의 생성자의 인자로 넘겨줄수 있기 때문에
마치 ``System.in``을 ``BufferedReader``에 넣은 것처럼 ``InputStreamReader``가 어댑터로서 역활을 수행하고 있습니다. 

오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.

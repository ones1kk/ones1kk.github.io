---
title: 템플릿 메소드 패턴(Template Method Pattern)
date: 2023-06-27 00:01:00 +09:00
categories: [ Software Architecture, Design Pattern ]
tags: [ GoF, Design-Pattern, Template-Method-Pattern ]
---

# Template Method Pattern

템플릿 메소드 패턴(Template Method Pattern)은 은 Gang of Four(GoF) 디자인 패턴 중
하나로 [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.yes24.com/Product/Goods/17525598)라는 책에서
소개 된 23가지 디자인 패턴 중 하나입니다.
템플릿 메소드 패턴은 **행동 패턴**으로서 특정 알고리즘을 사용하는 유사한 클래스들의 알고리즘의 구조를 정의하고, 하위 클래스에서는 구조를 변경하지 않고 특정 단계 메소드들을 구체화하여 해당 구조를 표현하기 위해서
사용됩니다.
이를 통해 클라이언트는 특정 알고리즘 구조(템플릿)를 변경하지 않고 구체 메소드를 확장하거나 재정의할 수 있는 유연성을 제공합니다. 또한 정의된 구조를 제외하고 하위의 세부 실행 내용을 다양하게 구현하여 사용할 수
있습니다.

# 구조

![template_pattern_uml_diagram](/assets/img/software-architecture/design-pattern/template-method-pattern/template_pattern_uml_diagram.png)
> [출처](https://www.tutorialspoint.com/design_pattern/template_pattern.htm)

- Abstract Class
  - 알고리즘의 구조를 정의하고 템플릿 메소드를 포함합니다.
  - 추상 메소드와 구체 메소드를 포함하며 구체 메소드들은 템플릿 메소드를 호출하여 알고리즘의 구조를 유지합니다.
  - 순서를 제어용으로 사용되는 메소드로 기본적인 내용만 구현하거나 비워놓는 메서드인 ``hook method``를 가집니다.
- Concrete Class
  - 추상 클래스를 상속받아 템플릿 메소드를 구현하고, 필요한 구체 메소드들을 오버라이딩하여 알고리즘의 특정 단계를 구체화합니다.

# Template Method Pattern of Spring Framework 

## JDBCTemplate

``org.springframework.jdbc.core.JDBCTemplate``는 추상 클래스인 ``org.springframework.jdbc.support.JdbcAccessor``를 상속 받아 구현된 객체입니다. 
``JdbcAccessor``는 JDBC를 사용하여 데이터베이스에 접근하는 DAO(Database Access Object) Helper 클래스들을 위한 기본 클래스입니다.
일반적으로 JDBC 접근 작업에 대한 데이터베이스 연결, 트랜잭션 관리, 예외 처리 등과 같은 공통된 동작을 처리하는 메소드들이 정의되어 있습니다.
추가로 템플릿 메소드인 ``execute()`` 메소드는 데이터베이스 작업의 흐름을 제어하고, 구체적인 작업은 하위 클래스에서 구현할 수 있도록 합니다.

오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다. 


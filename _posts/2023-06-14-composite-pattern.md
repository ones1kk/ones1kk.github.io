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
  - 복합 객체와 개별 객체를 동일한 인터페이스로 표현하는 역할이며 복합 객체와 개별 객체에 대한 공통 작업을 선언합니다. 

- Leaf
  - 복합 객체의 단일 요소를 나타내는 역할입니다. 
  - Component 인터페이스를 구현하며 더 이상의 하위 요소를 가질 수 없는 주로 단일 객체로 동작합니다.

- Composite
  - 복합 객체를 나타내는 역할입니다.
  - Component 인터페이스를 구현하며, 자식 요소를 가질 수 있습니다.
  - 자식 요소를 관리하고, 자식 요소에 대한 작업을 위한 메서드를 구현합니다.
  - 복합 객체의 구조를 재귀적으로 표현할 수 있습니다.

# 스프링의 컴포지트 패턴

## HandlerExceptionResolverComposite

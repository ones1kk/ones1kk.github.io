---
title: Spring AOP
date: 2023-08-16 00:00:00 +09:00
categories: [ Spring Core ]
tags: [ AOP, Aspect, Pointcut, Advice ]
---

# AOP

AOP는 Aspect Oriented Programming의 약자로 관점 지향 프로그래밍을 뜻합니다. 
AOP는 시스템을 구성하고 모듈화하는 새로운 방법을 제공하는 프로그래밍 방법론으로 OOP(Object Oriented Programming)의 보완적인 개념으로 등장했습니다.
OOP는 주로 주요 비즈니스 로직을 객체로 추상화하여 모델링하는데 중점을 두기 때문에 중복되는 관심사(Concerns) 문제를 해결하기에는 제한적입니다.  

AOP는 이러한 OOP의 한계를 극복하기 위해 등장한 개념으로 Cross-cutting Concerns와 핵심 비즈니스 로직을 분리하여 모듈화하는 것을 목표로 합니다.  
관심사를 Aspect 라는 모듈로 모델링하여 주요 비즈니스 로직과는 별개로 존재하면서도 여러 모듈에 걸쳐 적용되는 보조 기능을 담당합니다.

## Cross-cutting Concerns

![cross-cutting-concerns](/assets/img/spring/core/aop/cross-cutting-concerns.png)  

[출처](https://www.researchgate.net/figure/Banking-system-cross-cutting-concerns_fig1_332881268)

AOP는 객체를 핵심 관심사(Core Concerns)와 횡단 관심사(Cross-cutting Concerns)로 구분하고 분리합니다. 
핵심 관심사는 각 객체가 가지고 있는 주요 비즈니스 로직이나 주된 기능을 의미합니다. 
반면에 횡단 관심사는 주요 비즈니스 로직과는 독립적으로 여러 부분에 걸쳐서 반복적으로 발생하는 부가적인 기능을 의미합니다.
예를 들어 로깅, 보안, 트랜잭션 관리 등은 횡단 관심사에 해당합니다.

## 용어

- Aspect: 횡단 관심사를 모듈화한 것으로 로깅, 보안, 트랜잭션 관리와 같은 보조 기능을 담고 있는 모듈입니다. Aspect는 Advice와 Pointcut으로 구성됩니다.
- Joinpoint: 프로그램 실행 중에 Aspect가 적용될 수 있는 특정 시점을 가리킵니다. 메소드 호출, 객체 생성, 필드 접근 등이 결합점에 해당합니다.
- Advice: Aspect의 동작을 정의한 부분으로 횡단 관심사를 어떻게 실행할지를 정의합니다. 
- Pointcut: 어떤 Joinpoint가 Aspect의 Advice에 적용될지를 결정합니다. 또한 Pointcut 표현식을 통해 특정 메서드 호출 패턴이나 클래스 등 좀 더 자세한 적용 대상을 지정할 수 있습니다. 
- Weaving: 컴파일 시점, 클래스 로딩 시점, 실행 시점 등에서 Aspect의 코드를 원래 코드에 '직접' 또는 '간접'으로 삽입하여 횡단 관심사를 적용합니다.

# Spring AOP

Spring AOP는 프로시 기반의 AOP 프레임워크입니다. 
이는 대상 객체에 대한 Aspect를 구현하기 위해 해당 객체의 프록시를 생성함을 의미합니다.

[//]: # (Spring AOP는 실제 객체에 대한 접근을 제어하고 중간에서 작업을 수행할 수 있는 중간 계층을 제공하기 위해 Proxy Pattern을 이용하여 AOP를 구현했습니다. )






오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.

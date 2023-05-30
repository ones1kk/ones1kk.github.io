---
title: DL(Dependency Lookup)
date: 2023-05-30 21:50:00 +09:00
categories: [ Spring, Core ]
tags: [ DL, Dependency-Lookup, ObjectFactory, ObjectProvider, JSR-330 Provider ]
---

# 의존성 조회(Dependency Lookup: DL)이란?

의존성 조회(Dependency Lookup: DL)은 제어의 역전(Inversion of Control: IoC) 원칙을 구현한 디자인 패턴 중 하나입니다.
IoC 컨테이너가 객체의 생성과 의존성 주입을 담당하는 의존성 주입(Dependency Injection: DI)와 다르게 DL은 개발자가 필요한 시점에 의존성을 요청하여 컨테이너로부터 해당 의존성을 가져옵니다.
DL은 문맥별 의존성 조회(Contextualized Dependency Lookup: CDL)를 사용하여 의존성을 조회하는데, 이는 결국 스프링에서 관리하는 컨택스트인 ``ApplicationContext``를
기반으로 빈과 빈들의 의존성을 조회한다는 뜻입니다.
즉, 특정 설정 파일에서 의존성에 사용할 인스턴스를 가져오는 DI와 달리 DL은 자원을 관리하는 ``Context(Container)``에서 의존성을 가져오며, 항상 수행되는 것이 아닌 개발자가 필요한 시점에
가져오는 것이 가장 큰 특징입니다.

# DL의 동작 코드

# 의존 관계를 조회하는 방법

## ObjectFactory

## ObjectProvider

## JSR-330 Provider

오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.

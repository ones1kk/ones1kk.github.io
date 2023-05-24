---
title: 스프링 컨테이너(Spring Container)
date: 2023-05-22 20:10:00 +09:00
categories: [Spring, Core]
tags: [Spring-Container, Singleton-Container, Configuration, Component, ComponentScan, BeanDefinition]
---

# 스프링 컨테이너

스프링 컨테이너는 스프링에서 객체의 생성, 관리, 조립 등을 담당하는 컨테이너입니다. 
``org.springframework.context.ApplicationContext`` 인터페이스를 스프링 컨테이라 부르며, XML 또는 어노테이션 기반의 자바 설정 클래스로 생성이 가능합니다. 

![application-context](/assets/img/spring/core/spring-container/application-context.png)  

``ApplicationContext``는 다양한 인터페이스를 상속받으며 빈 관리 기능 & 어플리케이션 부가 기능을 제공합니다.
``ApplicationContext``가 상속 받는 인터페이스의 제공 기능은 아래와 같습니다. 

- BeanFactory: 빈 객체를 생성하고 빈의 인스턴스화, 의존성 주입, 라이프사이클 관리하는 역할을 담당하는 인터페이스입니다. 
- HierarchicalBeanFactory: 부모-자식 관계를 가지는 ``BeanFactory``를 표현하고 다룰 수 있게 해줍니다. 전역적으로 사용되는 설정 정보나 공통 빈은 부모 컨테이너에 등록하고, 각각의 요청마다 생성되는 빈은 자식 컨테이너에 등록하여 사용할 수 있습니다.
- ListableBeanFactory: 요청에 따라 하나씩 ``BeanFactory`` 조회하지 않고 모든 빈들을 목록화하고 조회하는 기능을 제공합니다. 
- MessageSource: 다양한 메시지소스로부터 메시지를 검색하고, 해당 메시지를 다국어로 변환하는 기능을 제공합니다. 여러 언어로 번역된 메시지를 관리하고, 애플리케이션에서 필요한 곳에서 해당 메시지를 조회하여 사용할 수 있습니다. 
- ResourcePatternResolver: 클래스패스나 파일 시스템에서 특정 패턴에 매칭되는 리소스를 찾는 기능을 제공하는 인터페이스입니다. ``ResourceLoader`` 인터페이스를 확장하여 리소스를 패턴으로 검색하는 기능을 추가로 제공합니다.
- ApplicationEventPublisher: 이벤트를 발행하고 구독자들에게 이벤트를 전달하는 역할을 담당하는 인터페이스입니다. 
- EnvironmentCapable: 애플리케이션의 환경 정보를 제공하는 기능을 정의한 인터페이스입니다. 애플리케이션의 실행 환경에 관련된 정보를 제공하고, 설정 및 프로파일 관리 등에 활용됩니다.

# 싱글톤 컨테이너 & @Configuration

# @Component & @ComponentScan

``@Component`` 어노테이션은 스프링 컨테이너에서 관리하는 객체, 즉 스프링 빈으로 등록하는 객체를 나타내기 위해 클래스 레벨에 작성되는 어노테이션입니다. 


# BeanDefinition

# 스프링 컨테이너 생성 과정

1.

오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.

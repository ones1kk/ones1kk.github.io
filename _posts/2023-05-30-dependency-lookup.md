---
title: DL(Dependency Lookup)
date: 2023-05-30 21:50:00 +09:00
categories: [ Spring, Core ]
tags: [ DL, Dependency-Lookup, ObjectFactory, ObjectProvider, JSR-330 Provider ]
---

# 의존성 조회(Dependency Lookup: DL)이란?

의존성 조회(Dependency Lookup: DL)은 제어의 역전(Inversion of Control: IoC) 원칙을 구현한 디자인 패턴 중 하나입니다.
IoC 컨테이너가 객체 생성과 의존성 주입을 담당하는 의존성 주입(Dependency Injection: DI)과는 다르게 DL은 개발자가 필요한 시점에 의존성을 요청하여 컨테이너로부터 해당 의존성을 가져옵니다.
DL은 컨택스트별 의존성 조회(Contextualized Dependency Lookup: CDL)를 사용하여 의존성을 조회하는데, 이는 결국 스프링에서 관리하는 컨택스트인 ``ApplicationContext``를
기반으로 빈과 빈들의 의존성을 조회한다는 뜻입니다.  
즉, 특정 설정 파일에서 의존성에 사용할 인스턴스를 가져오는 DI와 달리 DL은 자원을 관리하는 ``context(container)``에서 의존성을 조회하며, 항상 수행되는 것이 아닌 개발자가 필요한 시점에
가져오는 것이 가장 큰 특징입니다.

# 스프링에서 제공하는 의존성 조회 방법

애플리케이션 코드에서 의존성 조회가 필요한 경우 가장 기본적으로는 스프링 컨테이너인 ``ApplicationContext``에서 제공하는 ``getBean()`` 메소드를 통해 원하는 빈을 조회할 수 있습니다.
하지만 거대한 인터페이스인 ``ApplicationContext``를 주입 받아 사용하는 것은 너무 무겁고 단위 테스트 작성할 때 Mocking에 많은 어려움을 겪습니다.  
때문에 스프링에서는 직접 ``ApplicationContext``에서 빈을 조회하는 것이 아닌, 다양한 의존성 조회 방법들을 제공해줍니다.

## ObjectProvider

``org.springframework.beans.factory.ObjectFactory``를 상속 받아 구현한 ``ObjectProvider``는 개발자가 지정한 클래스 타입의 빈을 ``context``에서
조회하는 서비스를 제공해줍니다.
``ObjectProvider, ObjectFactory``는 별도의 라이브러리가 필요 없는 스프링에서 제공해주는 객체로 ``ObjectFactory``에서 제공해주는 의존성 조회뿐만 아니라 컬렉션 객체, 선택적
주입 기능을 제공합니다.

- 컬렉션 의존성: 단일 객체 뿐만 아니라 컬렉션 객체도 제공할 수 있습니다.
- 선택적 주입: 선택적 주입(optional injection)을 지원합니다. 즉, 해당하는 객체의 인스턴스가 존재하지 않는 경우에도 예외를 발생시키지 않고 null을 반환합니다.

```java
package org.springframework.beans.factory;

public interface ObjectProvider<T> extends ObjectFactory<T>, Iterable<T> {

T getObject(Object... args) throws BeansException;

@Nullable
T getIfAvailable() throws BeansException;

default T getIfAvailable(Supplier<T> defaultSupplier) throws BeansException {
  T dependency = getIfAvailable();
  return (dependency != null ? dependency : defaultSupplier.get());
}

default void ifAvailable(Consumer<T> dependencyConsumer) throws BeansException {
  T dependency = getIfAvailable();
  if (dependency != null) {
    dependencyConsumer.accept(dependency);
  }
}

@Nullable
T getIfUnique() throws BeansException;

default T getIfUnique(Supplier<T> defaultSupplier) throws BeansException {
  T dependency = getIfUnique();
  return (dependency != null ? dependency : defaultSupplier.get());
}

default void ifUnique(Consumer<T> dependencyConsumer) throws BeansException {
  T dependency = getIfUnique();
  if (dependency != null) {
    dependencyConsumer.accept(dependency);
  }
}

@Override
default Iterator<T> iterator() {
  return stream().iterator();
}

default Stream<T> stream() {
  throw new UnsupportedOperationException("Multi element access not supported");
}

default Stream<T> orderedStream() {
  throw new UnsupportedOperationException("Ordered element access not supported");
}

}
```

## JSR-330 Provider

``JSR-330(Java Specification Request 330)``은 Java Community Process(JCP)에서 제안된 Java Specification Request(JSR) 중 하나입니다.
자바 표준이므로 스프링 프레임워크뿐만 아닌 다른 Google Guice, CDI(Java Contexts and Dependency Injection) 등과 같은 프레임 워크에서도 사용할 수 있습니다.

```java
package javax.inject;

public interface Provider<T> {

  T get();
  
}
```  

``get()`` 메소드 하나로 기능이 매우 단순하며, 해당 메소드로 의존성을 조회할 수 있습니다.

## @Lookup

# ObjectProvider 동작 코드

## Provider 동작 코드 

``JSR-330``의 ``javax.inject.Provider``를 처리하기 위해 스프링은 내부적으로  ``javax.inject`` API에 대한 Hard Dependency를
방지하기 위해 ``org.springframework.beans.factory.support.DefaultListableBeanFactory``의 내부 클래스로 분리하여 구현했습니다. 

> Separate inner class for avoiding a hard dependency on the javax.inject API. 
> Actual javax.inject.Provider implementation is nested here in order to make it invisible for Graal's introspection of DefaultListableBeanFactory's nested classes.  
 
![inner-provider.png](/assets/img/spring/core/dependency-lookup/inner-provider.png)   

또한 빈을 조회하는 것은 기존 ``DefaultListableBeanFactory``에 구현된 ``doResolveDependency()`` 메소드를 통해 ``context``에 접근, 빈을 조회한 후 반환하게 됩니다.   


오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.

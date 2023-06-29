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

``org.springframework.beans.factory.annotation.@LookUp``해당 어노테이션이 명시된 메소드는 스프링 컨테이너에서 해당 메소드를 재정의하여 ``org.springframework.beans.factory.BeanFactory``로 리디렉션하는 역할을 합니다.

스프링 애플리케이션이 구동을 시작하면서 ``org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons()`` 스프링 컨테이너는 메소드를 통해서 등록된 빈들에 대하여 초기화 작업을 진행합니다. 

![override-lookup](/assets/img/spring/core/dependency-lookup/override-lookup.png)  

그 중 ``@Autowired, @Value, @LookUp..``어노테이션이 명시된 메소드 또는 필드를 처리하는 빈 후처리기인 ``org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor``를 통해서 ``@LookUp`` 메소드 재정의를 진행합니다. 

![instantiate-bean](/assets/img/spring/core/dependency-lookup/instantiate-bean.png)  

그 후 정상적으로 빈이 생성되어 반환됩니다. 

추가로 ``@LookUp`` 어노테이션을 사용하는데는 권장 및 주의 사항이 있는 해당 내용은 아래와 같습니다.  
- 스프링 컨테이너는 CGLIB를 통해 메소드를 포함하는 해당 클래스의 런타임 서브클래스를 생성하여 컨테이너에서 관리하기 때문에 동적으로 서브클래스를 제공할 수 없는 팩토리 메소드에서 반환된 빈에 대해서는 교체할 수 없습니다.
- 특정 시나리오에서 구체적인 클래스가 필요할 때, lookup 메소드의 스텁 구현을 제공하는 것을 고려해야합니다. 
- 설정 파일의 ``@Bean`` 어노테이션이 선언된 메소드에서 반환된 빈에는 작동하지 않습니다. 

# ObjectProvider 동작 코드

![default-listable-bean-factory](/assets/img/spring/core/dependency-lookup/default-listable-bean-factory.png)  

``org.springframework.beans.factory.support.DefaultListableBeanFactory``는 스프링의 ``ConfigurableListableBeanFactory``와 ``BeanDefinitionRegistry`` 인터페이스의 기본 구현체로 빈 정의 메타 데이터를 기반으로 한 완전한 빈 팩토리입니다. 
또한 빈의 생성 및 초기화 전후에 추작인 작업을 처리하는 빈 후처리기(Post-Processors) 기능을 제공하는 Spring Context(컨테이너)입니다. 

스프링에서 제공하는 ``org.springframework.beans.factory.ObjectProvider`` 인터페이스는 ``DefaultListableBeanFactory``의 내부 클래스 ``DependencyObjectProvider``를 통해 구현 되었습니다. 

``DependencyObjectProvider``는 ``DefaultListableBeanFactory``의 ``doResolveDependency()`` 메소드를 호출함으로써 springContext의 의존성을 조회합니다. 
아래는 동작하는 코드에 대한 설명입니다. 

![find-autowire-candidates](/assets/img/spring/core/dependency-lookup/find-autowire-candidates.png)  

먼저 ``ObjectProvider`` 타입 파라미터로 넘겨진 클래스 정보를 기반으로 context에서 관리 중인 빈을 찾기 위한 메소드를 실행시킵니다.    

![find-canidates-part-1](/assets/img/spring/core/dependency-lookup/find-canidates-part-1.png)    

그 후 ``BeanFactoryUtls.beanNamesForTypeIncludingAncestors()`` 메소드를 통해서 조상 팩토리에서 정의된 빈을 포함한 주어진 타입의 모든 빈 이름을 가져옵니다.(빈 정의가 재정의된 경우 고유한 이름을 반환합니다.)  

![do-resolve-dependency](/assets/img/spring/core/dependency-lookup/do-resolve-dependency.png)  

![get-Object](/assets/img/spring/core/dependency-lookup/get-Object.png)

정상적으로 매칭되는 빈을 찾았다면, Null, NullBean, Class Type, Assignable 여부를 확인 한 후 context에서 조회 된 빈 객체를 개발자가 의존성을 조회한 코드상으로 반환합니다.     

## Provider 동작 코드 

스프링은 내부적으로  ``javax.inject`` API에 대한 Hard Dependency를 방지하기 위해 ``DefaultListableBeanFactory``의 내부 클래스로 분리하여 ``javax.inject.Provider``를 구현했습니다. 

> Separate inner class for avoiding a hard dependency on the javax.inject API. 
> Actual javax.inject.Provider implementation is nested here in order to make it invisible for Graal's introspection of DefaultListableBeanFactory's nested classes.  
 
![inner-provider](/assets/img/spring/core/dependency-lookup/inner-provider.png)   

또한 빈을 조회하는 것은 기존 ``DefaultListableBeanFactory``에 구현된 ``doResolveDependency()`` 메소드를 통해 ``context``에 접근, 빈을 조회한 후 반환하게 됩니다.   

오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.

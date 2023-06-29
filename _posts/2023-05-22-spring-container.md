---
title: 스프링 컨테이너(Spring Container)
date: 2023-05-22 20:10:00 +09:00
categories: [ Spring, Core ]
tags: [ Spring-Container, Singleton-Container, Configuration, Component, ComponentScan, BeanDefinition ]
---

# 스프링 컨테이너

스프링 컨테이너는 스프링에서 객체의 생성, 관리, 조립 등을 담당하는 컨테이너입니다.
``org.springframework.context.ApplicationContext`` 인터페이스를 스프링 컨테이라 부르며, XML 또는 어노테이션 기반의 자바 설정 클래스로 생성이 가능합니다.

![application-context](/assets/img/spring/core/spring-container/application-context.png)

``ApplicationContext``는 다양한 인터페이스를 상속받으며 빈 관리 기능 & 어플리케이션 부가 기능을 제공합니다.
``ApplicationContext``가 상속 받는 인터페이스의 제공 기능은 아래와 같습니다.

- ``BeanFactory``: 빈 객체를 생성하고 빈의 인스턴스화, 의존성 주입, 라이프사이클 관리하는 역할을 담당하는 인터페이스입니다.
- ``HierarchicalBeanFactory``: 부모-자식 관계를 가지는 ``BeanFactory``를 표현하고 다룰 수 있게 해줍니다. 전역적으로 사용되는 설정 정보나 공통 빈은 부모 컨테이너에 등록하고,
  각각의 요청마다 생성되는 빈은 자식 컨테이너에 등록하여 사용할 수 있습니다.
- ``ListableBeanFactory``: 요청에 따라 하나씩 ``BeanFactory`` 조회하지 않고 모든 빈들을 목록화하고 조회하는 기능을 제공합니다.
- ``MessageSource``: 다양한 메시지소스로부터 메시지를 검색하고, 해당 메시지를 다국어로 변환하는 기능을 제공합니다. 여러 언어로 번역된 메시지를 관리하고, 애플리케이션에서 필요한 곳에서 해당 메시지를
  조회하여 사용할 수 있습니다.
- ``ResourcePatternResolver``: 클래스패스나 파일 시스템에서 특정 패턴에 매칭되는 리소스를 찾는 기능을 제공하는 인터페이스입니다. ``ResourceLoader`` 인터페이스를 확장하여
  리소스를 패턴으로 검색하는 기능을 추가로 제공합니다.
- ``ApplicationEventPublisher``: 이벤트를 발행하고 구독자들에게 이벤트를 전달하는 역할을 담당하는 인터페이스입니다.
- ``EnvironmentCapable``: 애플리케이션의 환경 정보를 제공하는 기능을 정의한 인터페이스입니다. 애플리케이션의 실행 환경에 관련된 정보를 제공하고, 설정 및 프로파일 관리 등에 활용됩니다.

# 싱글톤 컨테이너

스프링 컨테이너에서는 기본적으로 사용되는 모든 객체들을 싱글톤으로 관리합니다.
즉 동일한 인스턴스를 호출할 때마다 생성하는 것이 아닌 하나의 인스턴스를 공유하여 메모리 사용을 최적화하고, 객체 간의 일관성을 유지합니다.

하지만 싱글톤 패턴은 아래와 같은 몇가지 문제점을 가지고 있습니다.

- private 생성자로 인해 상속을 통해 다형성을 적용하기 힘듭니다.
- 내부 속성을 변경하거나 초기화 하기 어렵습니다.
- 객체 대체 및 객체 주입의 어려움으로 인해 테스트하기 어렵습니다.
- 클라이언트가 구체 클래스에 의존함으로 ``DIP``를 위반합니다.

스프링은 해당 싱글톤 패턴의 문제점을 ``org.springframework.beans.factory.config.SingletonBeanRegistry``와 ``CGLIB``를 통해 해당 싱글톤 문제를
해결했습니다.

먼저 ``@Configuration`` 어노테이션을 통해 자바 설정 파일을 명시하고, 설정 파일 내에서 빈으로 등록할 객체의 팩토리 메소드(``@Bean`` 어노테이션을 명시)를 통해 빈으로 등록하게 되는데,
이 때 스프링은 ``CGLIB``라는 바이트 코드를 조작하는 라이브러리를 사용해 자바 설정 파일을 상속받은 임의의 다른 클래스를
만들고, ``SingletonBeanRegistry.registerSingleton()`` 메소드를 통해 그 다른 클래스를 스프링 빈으로 등록합니다.
> 스프링 v2.2.0부터 ``CGLIB`` 기술이 안정화 되었다고 판단해, 기본 프록시 생성 방식으로 지정됐습니다.

이를 통해, 스프링 컨테이너는 싱글톤 패턴의 장점을 활용하여 객체를 유지하고 단점으로 여겨지는 부분을 해결해 싱글톤 컨테이너로서 유지되며 관리됩니다.

# @Component & @ComponentScan

## @Component

``@Component`` 어노테이션은 스프링 컨테이너에서 관리하는 객체, 즉 스프링 빈으로 등록하는 객체를 나타내기 위해 클래스 레벨에 작성되는 어노테이션입니다.
``@Component`` 어노테이션이 선언 되어 있는 클래스는 스캔 대상이 되어 자동으로 빈으로 등록할 수 있지만, 수동으로 빈을 등록하는 것보다는 우선 순위가 떨어집니다.  
기본적으로 빈 이름은 클래스명의 앞 첫글자를 소문자로 변환 후 등록되며 ``value`` 속성을 이용하여 개별적으로도 지정할 수 있습니다.  
또 추가로 스프링에서는 ``@Component`` 어노테이션을 활용하여 파생되는 다양한 어노테이션이 아래와 같이 존재합니다.

- ``@Controller``: 스프링 MVC 컨트롤러로 인식
- ``@Service``: 특별한 기능은 없으며, 비즈니스 계층을 인식하는데 이용합니다.
- ``@Repository``: 스프링 데이터 접근 계층으로 인식하고, 데이터 계층의 예외를 스프링 예외로 변환합니다.
- ``@Configuration``: 스프링 설정 정보로 인식하고, 스프링 빈이 싱글톤을 유지하도록 추가 처리합니다.

> 위의 어노테이션들은 ``@Component``를 상속받는 것이 아닌, 상속 관계처럼 사용할 수 있도록 스프링에서 제공해주는 기능입니다.

## @ComponentScan

``@ComponentScan`` 어노테이션은 주로 설정 클래스에 작성되며 컴포넌트 스캔이 수행되어야 하는 패키지를 지정하여 자동으로 스프링 빈으로 등록할 클래스들을 찾아내는 역할을 합니다.
``@ComponentScan`` 어노테이션은 보다 컴포넌트 스캔 동작을 세밀하게 제어할 수 있도록 제공하는 주요 속성은 아래와 같습니다.

- ``basePackages``: 스캔 대상이 되는 패키지를 지정합니다. 문자열 배열 형태로 여러 개의 패키지를 지정할 수 있습니다. 만약 지정하지 않으면 ``@ComponentScan`` 이 선언된 클래스의
  패키지가 시작 위치가 됩니다.
- ``value``: ``basePackages``의 Alias로 ``basePackages`` 대신 속성명을 적지 않고 패키지명만 간결하게 적을 때 사용되는 속성입니다. ``basePackages``와 함께
  사용되지 않을 때만 동일한 역활로 사용됩니다.
- ``includeFilters``: 특정 조건에 맞는 컴포넌트만 스캔 대상으로 포함시킵니다. Filter 타입의 배열로 지정하며, @ComponentScan.Filter 어노테이션을 사용하여 조건을 지정합니다.
- ``excludeFilters``: 특정 조건에 맞는 컴포넌트를 스캔 대상에서 제외시킵니다. Filter 타입의 배열로 지정하며, @ComponentScan.Filter 어노테이션을 사용하여 조건을 지정합니다.

## Component Scanning

개발자가 설정한 빈들은 애플리케이션이 구동하기 시작하면서 스프링 컨테이너 내부에 빈들이 저장되어 관리됩니다.
빈을 등록하는 방법은 크게 4가지가 있는데, 아래와 같습니다.

1. 자동 등록: ``@Component`` 어노테이션을 포함한 어노테이션을 클래스에 선언 시, 해당 어노테이션이 명시된 클래스 파일을 찾아 자동으로 등록됩니다.
2. 수동 등록(``@Configuration``): ``@Configuration`` 어노테이션을 선언한 자바 클래스 파일은 스프링이 설정 파일로 인식합니다. 해당 설정 파일에 ``@Bean`` 어노테이션이 명시된
   메소드들을 스프링은 빈으로 등록하여 관리합니다.
3. 수동 등록(XML(Bean)): ``<bean>, <property>`` 태그로 등록할 빈을 지정합니다.
4. 수동 등록(XML(ComponentScan)): ``<context:component-scan>`` 태그에 지정한 패키지 기준으로 ``@Component`` 어노테이션이 명시된 클래스를 찾아 등록합니다.

스프링은 어떤 방법으로 ``@Component`` 어노테이션이 선언되어 있는 클래스 파일을들을 스캔하여 빈으로 등록하는지 코드를 살펴보겠습니다.

먼저 애플리케이션이 구동되면 여러 동작을 수행하는데 그 중 애플리케이션 컨택스트(Application Context)를 새로고침합니다.

### SpringApplication.refreshContext(ConfigurableApplicationContext)

> Load or refresh the persistent representation of the configuration, which might be from Java-based configuration, an
> XML file, a properties file, a relational database schema, or some other format.
> As this is a startup method, it should destroy already created singletons if it fails, to avoid dangling resources. In
> other words, after invocation of this method, either all or no singletons at all should be instantiated.

Java 설정 파일, XML 파일, 프로퍼티 파일, 관계형 DB 설정을 불러오기 또는 새로고침하며, 이 과정에서 ``BeanFactory``의 초기화 & 빈이 등록되는 과정이 포함되어 있습니다.

![do-process-configuration-class](/assets/img/spring/core/spring-container/do-process-configuration-class.png)

그 중 ``org.springframework.context.annotation.ConfigurationClassParser`` 클래스는 자바 기반 설정 클래스를 파싱하고 구성 요소를 등록하는 클래스입니다.  
해당 클래스는 스프링의 ``@Configuration`` 어노테이션이 적용된 설정 클래스를 처리하며, 빈 정의(Bean Definition) 및 구성 요소 정보를 수집하여 스프링 애플리케이션 컨텍스트에 등록합니다.
또한 ``@Configuration``을 처리하는 과정에서 ``@ComponentScan`` 어노테이션이 선언된 파일을 찾아, Component Scanning을 즉시 실행하여 ``BeanFactory``
또는 ``ApplicationContext``에 빈 정의들을 등록하게 됩니다.

![do-scan](/assets/img/spring/core/spring-container/do-scan.png)

``@ComponentScan`` 어노테이션의 설정된 ``basePackages``를
기반으로 ``org.springframework.context.annotation.ClassPathBeanDefinitionScanner.scan()``메소드를 호출해
``@Component`` 어노테이션이 선언된 클래스 파일을 찾아 빈 정의를 담은 집합을 반환합니다.

# BeanDefinition

![bean-definition](/assets/img/spring/core/spring-container/bean-definition.png)

스프링은 빈을 효율적으로 관리하기 위해 빈의 메타 정보들을 추상화하여 관리합니다.
스프링은 다양한 방식으로 빈을 등록할 수 있는데, 스프링 컨테이너는 구현 방식에 상관없이 빈 정의 정보를 받아 관리하기 위한 메타 정보입니다.

![bean-definition-diagram](/assets/img/spring/core/spring-container/bean-definition-diagram.png)
> [출처](https://www.cnblogs.com/ninth/p/6404317.html)

설정 파일로 등록된 빈, 어노테이션이 스캔된 빈, XML에서 등록된 빈 등 다양한 방법으로 등록된 빈을 하나의 추상화된 인터페이스로 용이하게 관리하도록 돕는 역활을 합니다.

## BeanDefinition 속성

- ``beanClassName``: 생성할 빈 클래스 이름
- ``factoryBeanName``: 팩토리 빈 클래스 이름
- ``factoryMethodName``: 팩토리 메소드 이름
- ``scope``: 싱글톤(기본값)
- ``lazylnit``: 실제 빈을 사용할 때 생성 여부
- ``lnitMethodName``: 호출되는 초기화 메소드 이름
- ``destroyMethodName``: 제거될 때 호출되는 메소드 이름
- ``constructor arguments, properties`` : 의존 관계 주입에서 사용될 값

![bean-definition-object-example](/assets/img/spring/core/spring-container/bean-definition-object-example.png)

오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.

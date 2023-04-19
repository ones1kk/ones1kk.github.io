---
title: IOC(Inversion of Control) & DI(Dependency Injection) & Spring Bean
date: 2023-04-20 00:02:00 +09:00
categories: [Spring, Core]
tags: [IOC, DI, Bean]
---

# 들어가기에 앞서

스프링 프레임워크(Spring Framework)는 자바 기반의 대표적인 프레임워크로, 애플리케이션 개발에 필요한 다양한 기능을 제공합니다.  
이 중에서도 IOC(Inversion of Control), DI(Dependency Injection), Bean은 스프링의 가장 핵심적인 개념 중 하나로 스프링의 특징이자 장점을 이해하는데 중요한 역할을 합니다.

# IOC(Inversion of Control)이란?

IOC는 Inversion of Control의 약어로, 번역하자면 ***제어의 역전***입니다.  
기존의 프로그램 개발에서는 개발자가 코드를 작성하고 제어의 흐름을 조작하여 객체를 생성하고 관리하는 것이 일반적이었지만 스프링의 IOC는 객체 생성, 관리의 제어 흐름을 프레임워크가 담당하게 되는 개념입니다.

스프링의 IOC는 개발자가 객체의 생성, 관리, 의존성 주입 등을 직접 처리하는 것이 아닌, 스프링 컨테이너에서 객체의 생명 주기를 관리하고 객체 간의 의존성 주입 문제를 해결합니다.   
이를 통해 개발자는 객체의 생성과 관리에 집중하는 것이 아니라 비즈니스 로직에 더 집중할 수 있게 됩니다.
또한 스프링의 IOC는 객체의 결합도를 낮추고 유연성을 높여줌으로써 코드의 재사용성과 테스트 용이성을 향상시킵니다.

# DI(Dependency Injection)란?

DI는 Dependency Injection의 약어로, 번역하자면 ***의존성 주입***입니다.  
객체 간의 의존성을 코드 내에서 직접 해결하는 것이 아니라 외부에서 의존성을 주입해주는 개념입니다.  
스프링의 DI는 스프링 컨테이너가 객체 간의 의존성을 해결해주므로 객체 간의 결합도를 낮추고 유연성을 높여줍니다.

스프링의 DI는 크게 3가지 방식으로 이루어질 수 있습니다.

## 생성자 주입(Constructor Injection)

스프링 빈의 생성자에 ``` @Autowired ``` 어노테이션을 사용하여 의존 객체를 주입하는 방식입니다.  
자바 객체는 생성자를 통해서 생성되는 점을 착안하여 객체 생성 시점에 의존성 주입도 함께 진행합니다.

> 단 하나의 필드를 가지고 있는 생성자는 ``` @Autowired ``` 생략 가능합니다.  
> 또 필드에 ``` final ``` 키워드 사용 시  ``` @Autowired ``` 없이 의존성 주입이 됩니다.

## 필드 주입(Field Injection)

스프링 빈의 필드에 ``` @Autowired ``` 어노테이션을 사용하여 의존 객체를 주입하는 방식입니다.  
필드에 직접 주입하므로 생성자나 setter 메서드가 필요하지 않아 편리하게 사용할 수 있지만, 주입되는 의존성을 명시적으로 확인하기 어렵다는 단점이 있습니다.

## 세터 주입(Setter Injection)

setter 메서드에 @Autowired 애노테이션을 사용하여 의존성을 주입할 수 있습니다.  
필드 주입과 달리 명시적으로 세터 메서드를 호출하여 주입되는 의존성을 확인할 수 있습니다.


하지만 이 중에서도 생성자를 통한 의존성 주입이 가장 일반적이면서 선호회는 방식인데, 그 이유는 테스트 용이성을 높여 효과적인 단위 테스트와 모의 객체(Mock Object)를 사용한 테스트를 보다 수월하게 구성할 수 있습니다.

# Spring Bean이란?

스프링 컨테이너에서 관리하는 객체를 스프링 빈(Spring Bean)이라고 부르며, 스프링의 핵심 기능 중 하나인 DI와 관련있습니다.  
스프링 빈은 일반적으로 싱글톤(Singleton)으로 생성되어 해당 객체를 재사용으로 효율성을 올리고, 여러 개의 빈이 동일한 인스턴스를 참조하게 하므로 객체 생성과 관리를 효율적으로 할 수 있습니다.

스프링 빈 등록은 XML, 어노테이션 또는 설정 파일을 사용하여 정의 할 수 있으며, 스프링 컨테이너는 이 설정 정보를 바탕으로 스프링 빈을 생성하고, 의존성 주입을 통해 필요한 객체들 간의 관계를 자동으로 설정합니다.

스프링 빈은 빈 생명 주기(lifecycle)를 관리할 수 있는 콜백 메소드와 스코프(scope) 개념을 가지고 있습니다.  
스프링 빈이 초기화나 소멸 시점에 스프링에서 제공되는 ``` @PostConstruct, @PreDestroy ``` 어노테이션이 작성된 메소드를 통해 다양한 설정 및 관리를 할 수 있습니다.

또 빈이 존재할 수 있는 범위에 대한 설정으로 스크프를 지원합니다.

1. 싱글톤(singleton): 스프링 컨테이너 시작과 종료까지 유지되는 가장 넓은 범위이며, 항상 같은 인스턴스를 가지고 있는다.
2. 프로토타입(prototype): 빈의 생성과 의존관계 주입까지만 스프링 컨테이너가 관여하며, 항상 새로운 인스턴스를 반환한다.
3. 리퀘스트(request): 웹 요청이 들어오고 나갈때까지 유지되는 스코프.
4. 세션(session): 웹 세션이 생성되고 종료될까지 유지되는 스코프.
5. 애플리케이션(application): 웹 서블릿 컨택스트(Servlet Context)와 같은 범위로 유지되는 스코프.

> 스프링 컨테이너의 또다른 표현으로 IOC 컨테이너, DI 컨테이너와 같은 다양한 명칭이 있습니다.  
> 컨테이너를 어떤 관점으로 보느냐에 따라 명칭이 다를뿐, 결국 같은 컨테이너를 지칭합니다.



# IOC와 DI 사용하기

스프링 프레임워크에서 IOC와 DI를 사용하는 방법은 다양하지만, 가장 기본적인 방법은 스프링의 빈 개념을 활용하는 것입니다.  
스프링은 빈을 통해 객체의 생명 주기를 관리하고, 빈들 간의 의존성을 해결하여 IOC와 DI를 구현합니다.  
아래의 예제 코드를 통해 살펴 보겠습니다.

```java

public interface MemberService {
    Member findMemberByName(String name);
    void saveMember(Member member);
}

public class MemberServiceImpl implements MemberService {

    private final MemberRepository memberRepository;

    public MemberServiceImpl(MemberRepository memberRepository) {
    	this.memberRepository = memberRepository;
    }

    @Override
    public Member findMemberByName(String name) {
        return memberRepository.findMemberByName(name);
    }

    @Override
    public void saveMember(Member member) {
        memberRepository.save(member);
    }
}

public interface MemberRepository {

    Member findMemberByName(String name);
    void save(Member member);
}

public class MemberRepositoryImpl {

    public Member findMemberByName(String name) {
        return new Member(name);
    }

    public void save(Member member) {
        System.out.println(member);
    }
}

@Configuration
public class AppConfig {

    @Bean
    MemberService memberService(MemberRepository memberRepository) {
        return new MemberServiceImpl(memberRepository);
    }

    @Bean
    MemberRepository memberRepository() {
        return new MemberRepositoryImpl();
    }
}

```   

``` @Configuration ```이 선언 되어 있는 ``` AppConfig ```는 ``` @Bean ``` 어노테이션으로 선언되어 있는 메소드를 통해서 스프링 컨테이너가 관리할 빈들의 설정 정보가 작성되어 있는 자바 클래스입니다.   
해당 클래스에서는 각 빈들의 의존 관계 및 빈의 scope와 같은 설정을 작성할 수 있습니다.

``` MemberService ```를 생성하기 위해서는 ``` MemberRepository ```가 필요(의존적 관계)합니다.  
이 부분 또한 각 클래스에서 제공하는 적절한 방식을 사용하여 자동으로 DI(의존성 주입)가 이루어집니다.


결국 의존 관계까지 해결한 ``` MemberService, MemberRepository ``는 단 하나의 인스턴스를 가지는(싱글톤) 객체로 스프링 컨테이너애 등록됩니다.    
결과적으로 해당 설정을 통해서 ``` MemberService, MemberRepository ```는 스프링 빈으로서 스프링 컨테이너의 관리 대상이 되며 제어권을 개발자들에게서 가져감으로써(제어의 역전) 개발자들은 해당 객체 관리의 번거로움에서 벗어나게 됩니다.

또한 ``` AppConfig  ```와 같은 설정 파일이 없어도, ``` @Component ``` 어노테이션을 빈으로 등록할 클래스에 선언해 놓으면 스프링 컨테이너가 해당 클래스들을 추적해, 제공하는 적절한 DI 방식으로 DI를 해결하게 됩니다.

# 결론

이와 같이 스프링 프레임워크를 사용하면 객체 간의 결합도를 낮추고 유연한 구조를 구현할 수 있으며 코드의 가독성을 높일 수 있습니다.  
스프링의 IOC와 DI는 현대적인 자바 개발에서 필수적인 개념이며, 스프링 프레임워크를 사용하여 손쉽게 적용할 수 있습니다.  
이렇게 스프링의 IOC와 DI에 대해 간략하게 소개해보았습니다.  
더 자세한 내용은 [스프링 공식 문서](https://spring.io/)나 다양한 블로그 및 서적을 참고하시기 바랍니다.

오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다. 

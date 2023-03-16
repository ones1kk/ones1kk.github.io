---
title: Gradle Java Library plugin Configuration(api, implementation)
date: 2023-03-16 23:13:10 +09:00
categories: [Gradle, Plugin]
tags: [gradle]
---
# 초안(2023-03-16)

최근 멀티 모듈 프로젝트 환경을 구성하면서 겪었던 trouble이 몇가지 있었습니다.  
그 중 Gradle에서 제공하는 Java Library Dependency 관련된 trouble이 있었는데, 해당 프로젝트는 각 Layer, feature 별로 서브 모듈들을 가지고 있고 모듈에서 필요한 모듈의 의존성을 주입 받아 사용하는 환경이였습니다.   
이 시점에서, 모듈간 의존성을 설정하며 겪은 Trouble을 요약해보자면 아래와 같습니다.
1. 코드 작성 시점에서 문제 없이 하위 모듈의 소스들을 가져와 작성은 되나 Runetime 시점에서는 NotFoundClassException이 발생한다.
2. 코드 작성 시점에서 실제 해당 모듈 클래스를 import 해오나 해당 클래스가 의존하고 있는 다른 class의 정보를 가져 오지 못한다.  
   ...(생략)


하지만 각 서브 모듈의 의존 관계를 나타내는 다이어그램에서는 의도한대로 서로 주입을 받고 있어서 문제가 없는 것 처럼 보였습니다.


사실 처음 구성해보는 멀티 모듈 환경이라 정확한 원인을 파악하기 어려웠고, 저의 주변 상황도 마찬가지였습니다.  
(Maven에서 제공해주는 ```<scope>``` 태그처럼 Gradle도 분명히 있을 것이라고 추측은 했으나, 키워드나 function을 찾지 못한 상황이였습니다😭)

결국 Gradle에서 제공해주는 [공식 문서](https://docs.gradle.org/current/userguide/what_is_gradle.html)를 처음부터 천천히 살펴 보기로 했습니다.  
그 중 [The Java Library Plugin](https://docs.gradle.org/current/userguide/java_library_plugin.html#sec:java_library_configurations_graph) 페이지에서 원하는 내용을 확인 할 수 있었는습니다.  
제가 이해한 개념, 느낀점을 요약하여 적자면 아래와 같습니다.

보통 IDE에서 Gradle을 이용하여 Java 또는 Spring Project를 구성할 때, **build.gradle** 파일의 **dependency block**에 필요로 하는 라이브러리, 프레임워크의 의존성을 주입 받습니다.  
예를 들어 아래와 같이, Spring initializr를 활용해 Spring Boot 프로젝트를 생성하게 되면 **dependency block**에 자동으로 의존성을 주입을 받는 것을 많이 본 적이 있습니다.
```groovy
dependencies {

    implementation 'org.springframework.boot:spring-boot-starter-web'

    compileOnly 'org.projectlombok:lombok'

    annotationProcessor 'org.projectlombok:lombok'

    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

```

사실 저같은 경우는 Spring Boot에서 제공해주는 의존성 & 버전 관리에 익숙해져 유심히 본 적이 없는 block이였습니다.  
그저 추가할 라이브러리가 있으면 ```implementation 'org....'``` 기계적으로 추가만 했지 키워드들에는 크게 관심이 없었기 때문입니다.

  





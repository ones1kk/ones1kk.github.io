---
title: Java Plugin & Java Library Plugin
date: 2023-03-16 23:13:10 +09:00
categories: [Build Tool, Gradle]
tags: [gradle, java-plugin, java-library-plugin]
---

# 최근 멀티 모듈 프로젝트 환경을 구성하면서 겪었던 trouble이 몇 가지 있었습니다.

그 중 Gradle에서 제공하는 Dependency 관련된 trouble이 있었는데, 해당 프로젝트는 각 Layer, feature 별로 서브 모듈들을 가지고 있고 모듈에서 필요한 모듈의 의존성을 주입 받아 사용하는
환경이었습니다.   
이 시점에서, 모듈 간 의존성을 설정하며 겪은 Trouble의 케이스를 적어 보면 아래와 같습니다.

1. 코드 작성 시점에서 문제 없이 하위 모듈의 소스들을 가져와 작성은 되나 Runetime 시점에서는 NotFoundClassException이 발생한다.
2. 코드 작성 시점에서 실제 해당 모듈 클래스를 import 해오나 해당 클래스가 의존하고 있는 다른 class의 정보를 가져 오지 못한다.  
   ...(이하 생략)

또 각 서브 모듈의 의존 관계를 나타내는 다이어그램에서는 의도한 대로 서로 주입 받고 있어서 문제가 없는 것처럼 보였습니다.

사실 처음 구성해보는 멀티 모듈 환경이라 정확한 원인을 파악하기 어려웠고, 저의 주변 상황도 마찬가지였습니다.  
(Maven에서 제공해주는 ```<scope>``` 태그처럼 Gradle도 분명히 있을 것이라고 추측은 했으나, 키워드나 function을 찾지 못한 상황이었습니다😭)

결국 Gradle에서 제공해주는 [공식 문서](https://docs.gradle.org/current/userguide/what_is_gradle.html)를 처음부터 천천히 살펴보기로 했습니다.  
그
중 [The Java Library Plugin](https://docs.gradle.org/current/userguide/java_library_plugin.html#sec:java_library_configurations_graph)
페이지에서 원하는 내용을 확인 할 수 있었습니다.

# 제가 이해한 개념을 요약 & 정리하여 적자면 아래와 같습니다.

일반적으로 IDE에서 Gradle을 이용하여 Java 또는 Spring Project를 구성할 때, **build.gradle** 파일의 **dependency block**에 필요로 하는 라이브러리, 프레임워크의
의존성을 주입 받습니다.  
예를 들어 Spring initializr를 활용해 Spring Boot 프로젝트를 생성하게 되면 **dependency block**에 아래와 같은 스크립트가 자동으로 생성되어 의존성 주입을 받는 것을 많이 본
적이 있습니다.

```groovy
dependencies {

    implementation 'org.springframework.boot:spring-boot-starter-web'

    compileOnly 'org.projectlombok:lombok'

    annotationProcessor 'org.projectlombok:lombok'

    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

```

```{group-id}:{artifact-id}:{version}``` 이런 포맷으로 작성하게 되면 Maven Central Repository에 등록되어 있는 jar 파일을 다운받아 프로젝트에 의존성을 주입받게
됩니다.

``` groovy 
* 사용할 원격 저장소를 Maven Central Repository로 지정하는 build.gradle의 block
repositories {
    mavenCentral()
}
```

## 그럼 단지 저 **dependencies block과 repsoitories block**만 지정 해주면 프로젝트 의존성을 설정 할 수 있을까요??🤔

정답은 아닙니다.

```groovy
plugins {
    id "java"
}
 ``` 

이와 같이 명시적으로 Java Plugin을 사용한다고 .gradle 파일에 설정을 해야합니다.  
추가로 [The Java Plugin 공식 문서](https://docs.gradle.org/current/userguide/java_plugin.html#java_plugin)에서 내용을 확인해 보면 이렇게 적혀져
있습니다.
> The Java plugin adds Java compilation along with testing and bundling capabilities to a project. It serves as the
> basis for many of the other JVM language Gradle plugins...(이하 생략)


좀 더 보충하자면, Java Plugin은 프로젝트를 빌드하고 관리하기 위한 작업 및 구성을 제공하는 내장 플러그인입니다.  
Java Plugin은 크게 아래 내용들을 제공합니다.

1. Java 소스 코드를 컴파일한다.
2. Junit(Default) 테스트를 실행한다.
3. 컴파일 된 클래스와 resources를 포함하는 JAR 아카이브를 어셈블한다.
4. Java 소스 코드(main, test 등)에 대한 소스 디렉토리를 정의한다.
5. 프로젝트 의존성을 지정한다.

또한 Java Plugin을 사용함으로써 얻는 이점들이 몇 가지 있습니다.

1. 빌드 프로세스의 단순화(ex: build.gradle을 통한 설정)
2. 프로젝트 구조 표준화(ex: Java 개발자들에게 친숙한 표준 프로젝트 구조 제공)
3. 다양한 빌드 유형 지원(ex: debug 및 release 빌드)
4. IDE와 통합

Java Plugin은 위와 같이 특징 및 강력한 이점들이 있습니다.   
하지만 그 중에서도 눈 여겨 볼 점은 **Java Plugin**이 제공해주는 기능 중 5번째 **프로젝트 의존성을 지정한다.** 입니다❗️❗️❗️  
``` dependencies  block ```에서 ``` implementation, compileOnly, testImplementation``` 등과 같은 키워드들은 Java Plugin을 통해서 의존성
구성을 설정하게 되는 것입니다.

## 그럼 해당 키워드들은 어떤 의미를 가질까요??

아래의 다이어그램으로 해당 키워드들이 어떤 방식으로 의존성 구성을 하는지 확인할 수 있습니다.  
![main-configurations](/assets/img/gradle/java-main-configurations.png)
> Main source set dependency configurations  
[출처](https://docs.gradle.org/current/userguide/java_plugin.html#java_plugin)

![test-configurations](/assets/img/gradle/java-test-configurations.png)
> Test source set dependency configurations  
[출처](https://docs.gradle.org/current/userguide/java_plugin.html#java_plugin)

먼저 이미지들에 있는 박스에 대해서 색깔별로 설명을 하자면 아래와 같습니다.

1. 녹색 박스: 종속성을 선언할 수 있는 구성
2. 청회색 박스: Task에서 사용하기 위한 구성
3. 밝은 파란색 박스: Task

특히 청회색 박스를 보면 해당 의존성이 어느 시점까지 구성되어 생명 주기를 가지는지 대한 설정도 확인 할 수 있습니다.

1. complieClassPath: 지정한 sourceSets(main)의 경로에서 컴파일 시점까지의 생명 주기를 가집니다.
2. runtimeClassPath: 지정한 sourceSets(main)의 경로에서 소스 코드를 실행한(RunTime) 시점까지의 생명 주기를 가집니다.
3. testComplieClassPath: 지정한 sourceSets(test)의 경로에서 컴파일 시점까지의 생명 주기를 가집니다.
4. testRuntimeClassPath: 지정한 sourceSets(test)의 경로에서 소스 코드를 실행한(RunTime) 시점까지의 생명 주기를 가집니다.

## 이제 이정도면 어느 정도 ```Gradle```에서 의존성 구성을 어떤 식으로 처리하는지 알게 되었습니다.

그럼 이제 ``` spring-data-jpa를 ``` ``` implementation``` 받고있는 ``` domain-module ```을 의존성 주입 받아 사용해 보곘습니다. 아래와 같이 말이죠.

``` groovy 
* another-module > build.gradle
dependencies {
	implementation project(":domain-module")
}
```

## 오잉! 이상합니다! ``` another-module ```에서 ``` JpaRepository를 구현 받은 domain-module의 Repository```의 method들을 가져오지 못합니다!!!

아래와 같이 모든 classPath에 ``` domain-module ```이 들어 왔는데 말이죠 🤔

![gradle-dependency-tab](/assets/img/gradle/gradle-dependency-tab.png)

많은 시행 착오를 겪으면서 결국 해결한 것이 서두에
작성한 [The Java Library Plugin](https://docs.gradle.org/current/userguide/java_library_plugin.html#sec:java_library_configurations_graph)
페이지를 통해서 입니다.

결국 문제는 Java Plugin의 문제였습니다. 아래 사진을 같이 보시죠.  
![java-library-ignore-deprecated-main.png](/assets/img/gradle/java-library-ignore-deprecated-main.png)
> java-library-ignore-deprecated-main  
[출처](https://docs.gradle.org/current/userguide/java_library_plugin.html#sec:java_library_configurations_graph)

![java-library-ignore-deprecated-test](/assets/img/gradle/java-library-ignore-deprecated-test.png)
> java-library-ignore-deprecated-test  
[출처](https://docs.gradle.org/current/userguide/java_library_plugin.html#sec:java_library_configurations_graph)

Java Plugin에서 표현한 다이어그램과는 다른 박스가 하나 추가되었습니다. 핑크색 박스는 어떤 의미일까요?
> 분홍색 박스: 구성 요소가 컴파일되거나 라이브러리에 대해 실행될 떄 사용되는 구성  
> apiElements: 라이브러리에 대해 컴파일하는데 필요한 모든 요소를 검색하기 위한 요소들  
> runtimeElements: 라이브러리에 대해 실행하는 데 필요한 모든 요소를 ​​검색하기 위한 요소들

``` domain-module은 another-module```에서 DB 접근 및 handling과 관련된 로직을 수행하는 모듈입니다. 마치 라이브러리처럼 말이죠.  
``` another-module```은 ```spring-data-jpa```에 대한 의존성을 주입 받지 않은 상태입니다. 해당 의존성에 대해서 아무런 정보도 가지고 있지 않은
상태이고, ```spring-data-jpa```에 대한 의존성은 오직 ``` domain-module ```만 가지고 있는 상태입니다.

그렇기 때문에 ``` another-module ```은 ``` domain-module ```에서 의존 받고 있는 ```spring-data-jpa```의 elements까지 가져와야 하는 상황입니다.  
``` implementation ``` 키워드가 아닌 ``` api ``` 키워드로 변경합니다.

``` groovy 
* another-module > build.gradle
dependencies {
	api project(":domain-module")
}
```

추가로 이것도 수정해야 합니다.

```groovy
plugins {
    id "java" -> id "java-library"
}
 ```

> plugins block은 맨 위에 작성하는 것이 관례적입니다.  
> 만일 맨 위가 아닌 특정 위치에 작성하게 된다면 최소한 각 plugin이 제공하는 block 위에 해당 plugins block이 먼저 작성되어야 합니다.

변경 후, Gradle를 새로 고침하면 정상적으로 의존성을 끌고 옵니다!!🥳

## 이것으로 제가 겪었던 trouble에 대한 정리를 마치겠습니다.

> 추가로 개인적인 생각이지만 org.springframework.boot:spring-boot-starter-** 관련 의존성을 주입 받을 때 아래 케이스들은 org.springframework.boot:
> spring-boot-starter-**.jar자체에 해당 외부 라이버리를 이미 jar로 갖고 있기 때문인 것 같습니다???
> 1. 스프링 내부에서 사용하는 외부 라이브러리(ex: jackson)에 대한 의존성 관리 및 버전 관리를 명시적으로 해주지 않아도 되는 부분
> 2. implementation 키워드를 사용해도 외부 라이브러리의 elements들이 정상적으로 주입 되는 부분  
     ![spring-boot-start-web-internal](/assets/img/gradle/spring-boot-start-web-internal.png)

추가로 Gralde 공식 문서에는 아래와 같은 가이드를 두고 ``` api, implementation ``` 사용을 권장하기 때문에 확인하고 사용하시면 좀 더 유용합니다.

> Prefer the implementation configuration over api when possible.

### implementation

1. 타입이 메소드 바디 안에서만 쓰이는 경우
2. 타입이 private 맴버(변수/메소드 등등)에서만 쓰이는 경우
3. 타입이 인터널 클래스에서만 쓰이는 경우

### api

1. 타입이 인터페이스나 슈퍼 클래스에서 쓰이는 경우
2. 타입이 public/protected/package/private 메소드의 파라미터(메소드의 인자, 반환 타입 및 타입 파라미터)에서 쓰일 때
3. 타입이 public 필드에서 쓰일 때
4. public 어노테이션 타입일 때

> 또한 api, implementation 키워드들은 위에 사진에서 봐왔던 것과 같이 compileClassPath, runtimeClassPath, testCompileClassPath,
> testRuntimeClassPath를 전부 포함하기 때문에 의존성 충돌로 인한 문제가 발생할 수 있습니다.   
> 그렇기 때문에 최대한 의존성을 줄여서 설계하고 관리하는 것이 중요하곘습니다!!


오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.

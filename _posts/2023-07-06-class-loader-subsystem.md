---
title: 클래스 로더 서브시스템(Class Loader SubSystem)
date: 2023-07-06 00:01:00 +09:00
categories: [ Language, Java ]
tags: [ Class-Loader, JVM ]
---

# Class Loader SubSystem

![class-loader-subsystem](/assets/img/language/java/jvm/class-loader-subsystem.png)

> [출처](https://chazool.medium.com/jvm-class-loader-subsystem-e6bc4f539066)

클래스 로더(Class Loader SubSystem)는 자바 바이트 코드(.class)를 로드하고 JVM의 메모리 영역인 Runtime Data Areas로 동적 로딩하는 역활을 수행합니다.
클래스 로더는 필요에 따라 클래스 파일을 검색하고 로드하며, 로드된 클래스는 메모리 영역에 할당됩니다.

클래스를 로딩하는 과정은 3단계로 이루어집니다.

- 로드(Loading): 클래스 로딩은 먼저 해당 클래스 파일을 찾아서 메모리에 로드하는 과정입니다. 클래스 로더는 클래스 파일을 파일 시스템, JAR 파일, 네트워크 등에서 찾아옵니다.
- 링크(Linking): 로드된 클래스 파일의 바이트 코드는 아래 3단계의 링크 과정을 거쳐서 최종적으로 실행 가능한 형태로 변환됩니다.
  - 검증(Verification): 클래스 파일의 구조와 유효성을 검사하여 올바른 형식인지 확인합니다. 즉 문법 구문 오류를 찾아내는 역활을 합니다. 문법 오류, 타입 오류, 참조 오류 등을 확인하고 오류가
    있다면 컴파일 오류 메세지를 출력합니다.
  - 준비(Preparation): 클래스의 정적 변수를 위한 메모리 공간을 할당하고 초기값을 설정합니다.
  - 분석(Resolution): 클래스의 상수 풀(Constant Pool)에 있는 심볼릭 레퍼런스들을 실제 메모리 상의 레퍼런스로 변환합니다. 즉 클스 파일의 심볼릭한 참조들이 실제로 실행되는 데 필요한 메모리
    주소와 연결되어 클래스 간의 상호 작용이나 메소드 호출 등을 수행할 수 있습니다.
- 초기화(Initialization): 클래스의 초기화는 정적 변수의 초기화와 정적 블록의 실행을 담당합니다.

또한 클래스 로더는 계층 구조로 이루어져있습니다.

![class-loader-structure](/assets/img/language/java/jvm/class-loader-structure.png)

> [출처](https://chazool.medium.com/jvm-class-loader-subsystem-e6bc4f539066)

클래스 로더는 각 계층마다 역활과 책임이 나눠져있으며 가장 최상위 로더인 부트스트랩 클래스 로더로 부터 시작하여 하위로는 확장 클래스 로더와 애플리케이션 클래스 로더가 순차적으로 실행합니다.
하지만 자바 9 버전부터 모듈 시스템 도입에 맞춰 각각의 클래스 로더의 이름과 범위, 구현 내용등이 바뀌었기 때문에 각 버전에 맞게 설명하겠습니다.

## Java 8

- 부트스트랩 클래스 로더(Bootstrap Class Loader): Native C로 구현되어 있으며 ``jre/lib/rt.jar``에 담긴 JDK 클래스 파일을 로딩합니다.
- 확장 클래스 로더(Extension Class Loader): 자바로 구현되어 있으며 ``jre/lib/ext`` 폴더나 ``java.ext.dirs`` 환경 변수로 지정된 폴더에 있는 클래스 파일을 로딩합니다.
- 애플리케이션 클래스 로더(Application Calss Loader): 자바로 구현되어 있으며 ``-classpath(또는 -cp)``나 JAR 파일 안에 있는 Manifest 파일의 ``Class-Path``
  속성 값으로 지정된 폴더에 있는 클래스를 로딩합니다. 개발자가 애플리케이션 구동을 위해 직접 작성한 대부분의 클래스는 해당 클래스 로더에의해 로딩됩니다.

## Java 9

- 부트스트랩 클래스 로더(Bootstrap Class Loader): ``rt.jar``, ``tools.jar`` 등이 삭제됨에 따라 로딩할 수 있는 클래스의 범위가 전체적으로 축소되었습니다. 또한 이름의 변경
  없이 유지 되었습니다.
- 플랫폼 클래스 로더(Platform Class Loader): ``jre/lib/ext``, ``java.ext.dirs``를 더이상 로딩하지 않고 Java SE의 모든 클래스와 JCP(Java Community
  Process)에 의해 표준화 된 모듈 내의 클래스를 로딩하며 자바 8에 비해 담당 범위가 확장되었습니다. 또한 이름이 변경 되었습니다.
- 시스템 클래스 로더(System ClassLoader): 클래스패스, 모듈패스에 있는 클래스를 로딩합니다. 또한 이름이 변경 되었습니다.

[참고(JDK 9 Migration Guide)](https://docs.oracle.com/javase/9/migrate/toc.htm#JSMIG-GUID-A78CC891-701D-4549-AA4E-B8DD90228B4B)

또 3개의 클래스 로더는 다음 3가지 원칙을 통해 동작합니다.

## Delegation Principle

위임 원칙은 클래스 로드 시 윗방향으로 클래스 로딩을 위임하는 것을 의미합니다.
최초 클래스 로드 요청을 받은 애플리케이션 클래스 로더는 가장 최상위 로더인 부트스트랩 클래스 로더까지 확장 클래스 로더를 걸쳐 클래스 로딩을 요청합니다.
그 후 부트스트랩 클래스 로더에서부터 최하위인 애플리케이션 클래스 로더까지 각자가 담당하는 대상 파일에서 로드 요청이 들어온 클래스를 찾습니다.
각 클래스 로더가 담당하는 클래스 파일일 시 담당 클래스 로더에서 클래스를 반환하지만 3개의 클래스 로더에서 해당 클래스 파일을 찾지 못한다면 ``ClassNotFoundException`` 예외가 발생합니다.

## Visibility Principle

가시성 원칙은 상위 계층의 클래스 로더와 하위 계층의 클래스 로더간의 구분을 위한 원칙으로 하위 클래스 로더는 상위 클래스 로더의 로드 클래스를 볼 수 있지만, 상위 클래스 로더는 반대로 하위 클래스 로더의 로드
클래스를 볼 수 없는 원칙입니다.
가시성 원칙을 통해 각 클래스 로더들은 독립적으로 클래스를 로드할 수 있으며 서로 다른 클래스들 간의 충돌을 방지하고 의존성을 제어할 수 있게 됩니다.

## Uniqueness Principle

유일성 원칙은 각 클래스 로더들이 클래스를 로드할 때 클래스의 유일성을 보장하는 원칙입니다.
하위 클래스 로더는 상위 클래스 로더가 로드한 클래스를 다시 로딩하지 않으며 동일한 이름의 클래스는 동일한 클래스 로더에 의해 한 번만 로드됩니다.
또한 이미 로드한 클래스는 캐싱 되어 다시 요청될 경우 캐싱된 클래스를 반환합니다.

오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다. 

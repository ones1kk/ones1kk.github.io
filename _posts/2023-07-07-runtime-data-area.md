---
title: 런타임 데이터 영역(Runtime Data Area)
date: 2023-07-07 00:00:00 +09:00
categories: [ Language, Java ]
tags: [ Runtime-Data-Area, Method-Area, Heap-Area, Stack-Area, PC-Register, Native-Method-Stack, JVM ] 
---

# Runtime Data Area

Runtime Data Area는 OS로 부터 할당받은 JVM의 메모리 영역으로 자바 애플리케이션이 실행하는 동안 데이터를 저장하고 관리하는 영역입니다.
Runtime Data Area는 총 5가지 영역으로 나뉘어져있으며 스레드 공유 여부에 따라 각 영역의 성격이 다릅니다.

Method Area, Heap Area는 모든 스레드가 공유하는 영역이고, 나머지 Stack Area, PC Register, Native Method Stack은 각 스레드에 따라 생성되는 개별 영역입니다.

![runtime-data-area](/assets/img/language/java/jvm/runtime-data-area.png)

> [출처](https://blog.devgenius.io/java-virtual-machine-architecture-9009d864fc72)

## Method Area

★ 확인 필요 ★

Method Area는 Class Area 또는 Static Area로도 불리며 모든 스레드가 공유하는 영역 중 하나로 클래스 로더에 의해 로드된 클래스의 바이트 코드, 메소드, 필드, 메소드, 상수 풀이 저장되는
영역입니다.
JVM 시작 시 적재되며 해당 영역에 저장된 데이터는 애플리케이션이 종료되거나, 명시적으로 null 선언 시 GC의 대상이 되며 사라지게 됩니다.

### Runtime Constant Pool

Method Area에 포함되는 영역이지만 따로 분리되어 관리하는 만큼 중요한 부분들이 있습니다.
Runtime Constant Pool은 클래스와 인터페이스의 메소드, 필드, 문자열 상수 등의 레퍼런스가 저장되며 이들의 물리적인 메모리 위치를 참조할 경우에 사용합니다.
Runtime Constant Pool은 다음과 같은 초기화 코드 정보들이 저장됩니다.

- 필드 정보: 맴버 변수의 이름, 데이터 타입, 접근 제어자 정보
- 메소드 정보: 메소드명, 반환 타입, 메소드 매개변수, 접근 제어자 정보
- 타입 정보: Class 또는 Interface 여부, 타입 속성, 이름, Super Class 이름

위와 같은 데이터 정보들을 통해 JVM은 Runtime Constant Pool에 존재하는 데이터를 참조하는 다른 데이터들과 연결시킵니다.

★ 확인 필요 ★

## Heap Area

Heap Area는 모든 스레드가 공유하는 영역 중 하나로 애플리케이션에서 생성되는 모든 객체와 배열이 할당되는 메모리로 동적으로 할당하고 해제하는데 사용하는 메모리 영역입니다.
Heap Area에서 생성된 객체와 배열의 참조 값은 JVM Stack Area의 변수나 다른 객체의 필드에 의해서만 핸들링될 수 있습니다.

![heap-area](/assets/img/language/java/jvm/heap-area.png)

> [출처](https://medium.com/platform-engineer/understanding-java-memory-model-1d0863f6d973)

또한 Heap Area는 객체의 생명 주기와 메모리를 효율적으로 관리하기 위해 Young Generation과 Old Generation으로 나눠서 관리합니다.

### Young Generation

대부분의 객체는 생성 후 일시적으로 사용되다가 더 이상 참조되지 않게 되는데 이러한 객체들을 관리하기 위한 영역이 Young Generation입니다.  
Young Generation은 다시 세분화된 영역으로 Eden 영역과 Survivor Spaces라고 불리는 S0(From Area), S1(To Area) 영역으로 나뉩니다.   
Young Generation에서는 일반적으로 GC가 더 빈번하게 발생하며 이를 Minor GC라고 합니다.
Eden 영역은 객체들이 생성 당시 최초로 할당되는 공간입니다.
Minor GC는 Eden 영역이 가득 차 검사하게 되고 더 이상 참조 되지 않는 객체들이 발견되면 메모리를 회수하고 이 중 아직 참조되는 객체는 Survivor Spaces의 S0 영역으로 복사가 되고 해당
Eden 영역의 메모리는 회수됩니다.
그 후 다음 Minor GC에서도 참조되는 객체는 S0에서 S1복사하고 나머지 영역에서의 메모리는 회수됩니다.

### Old Generation

Old Generation은 Young Generation에서 살아남아 오랜 시간 동안 참조되는 객체들이 할당되는 영역입니다.
Old Generation은 Young Generation보다 크고, 오랜 시간 동안 살아남은 객체들이 저장되는 곳입니다.
Old Generation에서 GC가 발생하는 빈도는 상대적으로 적으며 Old Generation에서의 GC를 Major GC라고 합니다.
Major GC는 일반적으로 Young Generation에 비해 더 시간이 오래 걸리지만 참조되는 객체들이 적기 때문에 Minor GC보다 덜 빈번하게 발생합니다.

## JVM Stack

스레드 별로 생성되며 메소드 실행 관련 정보를 저장하는 영역(프레임 저장)

## PC Register

스레드 별로 생성되며 실행 중인 명령(오프셋)을 저장하는 영역

## Native Method Stack

스레드 별로 생성되며 네이티브 코드 실행에 관련 정보를 저장하는 영역

오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다. 

---
title: 실행 엔진(Execution Engine)
date: 2023-07-06 00:01:00 +09:00
categories: [ Language, Java ]
tags: [ Execution-engine, JVM, JIT-Compiler, GC ]
---

# Execution Engine

![execution-engine](/assets/img/language/java/jvm/execution-engine.png)

Execution Engine은 바이트 코드를 읽고 실제로 실행하는 역활을 담당합니다.
Runtime Data Areas에 있는 자바 바이트 코드를 읽고 운영체제에 맞게 기계어로 번경하여 해당하는 명령어(instruction) 단위로 실행합니다.
Execution Engine은 위의 수행 과정에서 **인터프리터**와 **JIT 컴파일러** 두 가지 방식을 혼합하여 바이트 코드를 실행합니다.

## Interpreter

자바는 인터프리트, 컴파일 두 방식을 혼합하여 사용하는 하이브리드 모델입니다.
컴파일을 통해서 자바 바이트 코드를 생성하고 코드의 명령어를 인터프리터를 사용해 하나씩 읽어서 해석하고 바로 실행합니다.
이는 빠른 시작 속도와 동적인 실행 환경을 제공하지만 반복적으로 실행되는 코드의 성능은 상대적으로 떨어집니다.

## JIT(Just-in-Time) Compiler

인터프리터의 단점을 보완하기 위해 도입된 방식으로 인터프리터가 반복적으로 실행하는 메소드(Hot Method)를 실시간으로 컴파일하여 네이티브 코드(Native Code)로 실행시킵니다.
이렇게 변환된 네이티브 코드는 캐싱 처리되어 이후에 해당 메소드가 실행될 때 재사용됩니다.
이뿐만 아니라 JIT 컴파일러는 프로파일링 정보, 코드 흐름, 하드웨어 정보 등을 활용해 프로그램 실행 중 동적으로 네이티브 코드 최적화를 실행합니다.

- 자주 실행되는 메소드
- 조건문, 반복문, 메서드 호출 등의 제어 흐름
- 하드웨어 정보

JIT 컴파일러는 이러한 실행 환경 정보를 수집하고 분석하여 최적화 전략을 결정합니다.

## Garbage Collection

GC는 Heap 메모리 영역에서 사용하지 않는 객체를 식별하여 메모리를 자동으로 회수합니다.
참조되지 않는 객체를 제거하고 공간을 비우는 이벤트를 "Stop the world"라고 부르며 가비지가 수집될 때까지 모든 애플리케이션의 스레드가 보류됩니다.

일반적으로 GC는 다음 두 단계를 통해 동작합니다.

### Marking

GC가 사용 중인 객체와 사용하지 않는 객체를 식별하는 단계를 마킹이라고합니다.
사용되지 않는 객체를 식별하기 위해서 모든 객체를 스캔합니다.

![marking-phase](/assets/img/language/java/jvm/marking-phase.png)

> [출처](https://www.tothenew.com/blog/java-garbage-collection-an-overview/)

### Deletion

삭제 단계에서는 표시된 객체가 삭제되고 메모리가 회수됩니다.
또한 참조되지 않는 객체는 두 가지 방법으로 삭제될 수 있습니다.

1. 일반 삭제(Normal Deletion): 사용되지 않는 모든 객체를 제거하고 새로운 객체를 저장할 수 있게끔 메모리를 할당합니다.
2. 삭제 및 압축(Deletion and Compaction): 성능을 더욱 향상시키기 위해 참조되지 않은 객체를 삭제하는 것 외에도 나머지 참조된 객체를 압축합니다.

GC는 위와 같이 두 단계를 통해 사용되지 않는 객체를 찾아 메모리를 회수합니다.

애플리케이션 실행 시간이 흐름에 따라 점점 많은 객체들이 생성되고 사용됩니다.
하지만 매번 힙 메모리에 있는 모든 객체를 GC가 스캔하는 것은 매우 비효율적이기 때문에 자바는 Heap 메모리를 여러 세대(Generation)으로 나누어 스캔 전략을 관리합니다.  
Heap 메모리는 크게 Young Generation, Old Generation, Permanent Generation(java 8 이후는 Metaspace) 세 가지 세대로 나눕니다.
Young Generation에서 가비지 컬렉션이 발생할 때 살아남은 객체들이 Old Generation으로 이동합니다.
가비지 컬렉션은 Young Generation의 Eden 영역과 Survivor 영역을 스캔하여 살아남은 객체들을 찾고 그 중에서도 메모리 사용량 등과 같은 일정 조건을 충족한 객체들이 Old Generation으로
이동합니다.
이렇게 Heap 메모리를 나누어 관리함으로써 GC는 주로 Young Generation에서 가비지 컬렉션이 빈번하게 발생하며 매번 모든 객체를 스캔해야하는 비효율로부터 벗어날 수 있게 되었습니다.

> 추가로 GC는 데몬 스레드로 동작하기 때문에 명시적으로 호출해도 즉시 실행이 되지 않을 수 있습니다.

오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다. 

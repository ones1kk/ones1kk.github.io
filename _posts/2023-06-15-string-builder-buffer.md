---
title: String & StringBuilder & StringBuffer
date: 2023-06-16 00:40:01 +09:00
categories: [ Language, Java ]
tags: [ String, StringBuilder, StringBuffer ]
---

# String

``String``은 각각의 문자를 나열한 배열을 나타내는 객체입니다.
Java에서 제공하는 Primitive Type(원시 타입)을 제외한 모든 나머지들은 객체로 구성되어 있으며 객체 생성은 생성자(construct)를 통해 이루어집니다.
하지만 ``String``은 객체를 생성할 때 생성자를 사용하는 방법 외에도 바로 값을 할당하여 생성할 수 있는데 이를 **문자열 리터럴(String Literal)**이라 부릅니다.

```java
public static void main(String[] args) {
    String str1 = new String("a"); // 생성자
    String str2 = "a"; // 문자열 리터럴
}
```

``String``은 위와 같이 두가지 방법으로 객체를 생성할 수 있습니다.
첫번째 방법은 일반적으로 객체를 생성하는 방법인 생성자를 사용하는 방법이며 두번째 방법이 문자열 리터럴을 사용해 객체를 생성하는 방법입니다.
두 변수에는 동일안 ``'a'``라는 값을 가지고 있는 객체이지만, 객체가 할당되는 메모리 영역에는 차이가 있습니다.
Java에서 동적으로 객체를 생성하게 되면 객체(참조값)를 메모리에 저장하게 되는데 자바의 메모리는 이를 모든 쓰레드에서 접근 가능한 ``Heap`` 영역에 할당하게 됩니다.
하지만 문자열 리터럴로 생성한 객체는 **String Constant Pool** 라는 특별한 영역에 할당하여 관리하게 됩니다.

## String Constant Pool

String Constant Pool은 자바의 문자열 상수를 관리하기 위해 사용되는 특별한 메모리 영역입니다.
내부적으로 각 String Constant를 hashing하여 key로 value를 조회하기때문에 탐색에서 좋은 성능을 보여줍니다.
과거 버전 Java의 String Constant Pool은 Permenent Generation에 있어 고정된 크기의 저장 공간이였기 때문에 OOM(OutOfMemory) 제약이 있었지만,
버전이 업그레이드 되면서 위치가 Heap 영역에서 Metaspace 영역으로 이동됐습니다. 

> Metaspace 영역은 JVM의 Native Memory를 사용하며 JVM이 관리합니다. Perm영역과의 결정적인 차이는 메모리가 동적으로 관리되며 필요할 경우 OS에게 요청하여 메모리를 추가 할당할 수 있습니다. 

1. Java 6 이전: String Constant Pool은 클래스와 관련된 메타 데이터를 저장하는 JVM의 메모리 영역의 Permanent Generation 저장되었습니다.
2. Java 7: Permanent Generation이 아닌 Heap 영역에 저장하면서 메모리 관리 측면에서 유연성을 높혔습니다.
3. Java 8: Permanent Generation가 완전히 제거되었고, 네이티브 메모리 영역인 Metaspace라는 새로운 메모리 영역이 도입되면서 String Constant Pool은 Metaspace에
   저장됩니다. 또한 8버전을 기점으로 내부 구현도 HashTable에서 HashMap으로 변경됐습니다.
4. Java 9 이후: UTF-16이 아닌 바이트 배열로 문자열을 저장하는 Compact Strings라는 새로운 구현이 도입되었습니다.

> Permenent Generation 영역은 고정된 사이즈이기 때문에 Runtime 시점에 사이즈를 확장할 수 없습니다.
> 7 버전에서 Heap 영역으로 이동시킴으로써 GC의 대상으로 될 수 있는 이점을 얻습니다.
> 하지만 8버전부터 Metaspace 영역으로 이동되면서 GC의 관리 대상에서 벗어났지만, 문자열 참조 대상이 없는 경우 선택적으로 GC 대상이 되기도합니다.
> 또한, ``-XX:+UseStringDeduplication`` 옵션 사용을 통해 중복 문자열에 대해 GC를 수행하도록 트리거를 줄 수 있습니다.
> 자세한 내용은 [이곳](https://blog.gceasy.io/2018/12/23/usestringdeduplication/)에서 확인할 수 있습니다.

또한 동적으로 생성되는 문자열의 경우에는 String Constant Pool에 저장되는 대상이 아닙니다.
그러므로 이런 동적인 문자열은 동등성(equals) 비교를 해야합니다.

### Compact Strings

Java 8 버전 이전까지는 ``String`` 객체 내부적으로 UTF-16으로 인코딩된 문자 배열(char[])을 사용했지만, Java 9 버전부터는 Compact Strings를 제공함으로써 문자열의 압축 여부에
따라 문자열 배열과 바이트 배열를 선택하여 인코딩합니다.
때문에 사용하는 문자열(언어)별로 1 바이트, 2 바이트 메모리를 동적으로 할당할 수 있게 됐습니다.
이로써 Heap 메모리 사용량을 줄여 GC(Garvage Collector) 오버헤드(overhead)가 줄어드는 효과를 얻습니다.

### Intern

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    ...
    public native String intern();
    ...   
    }
```

``intern()`` 메소드는 String Constant Pool에 해당 문자열이 있는지 ``equals()`` 메소드를 통해 판별하고 없으면 상수 풀에 저장한 후 참조값을 전달하고, 있으면 기존 Pool에 있는
참조 값을 넘겨줍니다.
해당 메소드는 강제로 String Constant Pool에 넣기 때문에 동적으로 생성되는 문자열에 ``intern()`` 메소드를 사용하게 된다면 OOM(OutOfMemory) 에러가 발생합니다.
즉, 메모리 누수 발생 가능성을 높이는 메소드이므로 사용에는 주의를 필요로합니다.

## Immutability

``String``은 객체로 생성된 후 상태를 변경할 수 없는 불변(Immutable) 객체입니다.
다음과 같은 이유로 불변 객체로 설계되었습니다.

1. 스레드 안전성: 객체가 불변하면 여러 스레드에서 동시에 접근해도 변경할 필요가 없기 때문에 동기화가 필요하지 않습니다. 이는 멀티스레드 환경에서 안전하게 사용할 수 있습니다.
2. 보안성: 문자열은 많은 애플리케이션에서 중요한 데이터를 나타내는 경우가 많습니다. 문자열을 불변으로 설계하면 다른 객체에서 문자열을 변경할 수 없으므로, 악의적인 변경을 방지하고 데이터 무결성을 보장할 수
   있습니다.
3. 메모리 최적화: 불변 객체는 생성 후 변경이 불가능하므로, 한 번 생성된 객체는 재사용될 수 있습니다.
4. 캐시와 문자열 연산 최적화: 불변 문자열은 문자열 연산 시에 캐시로 활용될 수 있습니다. 예를 들어, 문자열 연결 연산에서 이미 생성된 문자열을 재사용하여 연산 속도를 향상시킬 수 있습니다.

## Construct VS Literal

생성자를 통해 객체를 생성하면 일반적인 객체와 같이 Heap 영역에 새로운 객체를 생성하여 저장합니다.
하지만 리터럴을 사용하여 객체를 생성하는 경우, 해당 문자열은 String Constant Pool에 저장이 되고 이미 존재하는 문자열이라면 기존에 생성한 객체를 반환합니다.
때문에 특별한 경우가 아니라면 ``String`` 객체는 생성자를 통한 생성은 지양해야합니다.  

![string-pool-concept](/assets/img/language/java/string/string-pool-concept.png)
> [출처](https://www.geeksforgeeks.org/how-to-initialize-and-compare-strings-in-java/)

# StringBuilder

# StringBuffer

오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다. 

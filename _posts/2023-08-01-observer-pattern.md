---
title: 옵저버 패턴(Observer Pattern)
date: 2023-08-01 00:00:00 +09:00
categories: [ Software Architecture, Design Pattern ]
tags: [ GoF, Design-Pattern, Observer-Pattern ]
---

# Observer Pattern

옵저버 패턴(Observer Pattern)은 Gang of Four(GoF) 디자인 패턴 중
하나로 [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.yes24.com/Product/Goods/17525598) 책에서
소개 된 23가지 디자인 패턴 중 하나입니다.
옵저버 패턴은 **동작 패턴**으로써 객체의 상태 변화가 생길 때 각 객체를 관찰하는 옵저버들에게 통지하고 옵저버들은 받은 인지한 상태 변화에 따른 조치를 취하는 디자인 패턴입니다.  
옵저버 패턴은 객체 간의 일대다 의존성 관계를 정의하며 Pub/Sub(발행/구독) 모델로서 주로 분산 이벤트 핸들링 시스템을 구현하는데 사용됩니다.

# Structure

![observer-pattern](/assets/img/software-architecture/design-pattern/observer-pattern/observer-pattern.png)

- Subject
  - Observer의 관찰 대상자 인터페이스입니다.
  - 주제를 나타내는 인터페이스로 각 주제별 여러 Observer를 가질 수 있으며 Observer와는 일대다 관계입니다.
  - Subject의 상태가 변하거나 어떤 동작을할 때 Observer들에게 알림을 발행(notify)합니다.
- Observer
  - Subject의 상태 변화나 동작을 추적하는 인터페이스로 Subject를 묶는 인터페이스입니다.
- ConcreteObserver
  - Observer 인터페이스를 구현한 클래스로 Subject의 변경된 상태를 받습니다.
  - Subject로 부터 받은 상태에 대한 정보를 처리합니다.

# Example of Observer Pattern

## Sample Code

옵저버 패턴에서는 한 개의 관찰 대상자(Subject)와 여러 개의 관찰자(Observer)가 일 대 다로 구성 되어있습니다.
Subject의 변경된 상태는 일괄적으로 여러 관찰자들에게 메시지를 발행할 수 있으며 메세지를 수신한 Observer들은 그 다음 액션에 대해 적절히 대응합니다.
또한 언제든지 Observer들은 Subject 그룹에서 추가 또는 삭제될 수 있습니다.

다음은 수학 선생님(Subject)의 수업을 듣는 학생들(Observers)을 관계를 예를 들어 옵저버 패턴을 적용한 코드의 예입니다.

먼저 Subject이자 이벤트 발행 대상자인 ``MathTeacher`` 클래스에 대한 코드입니다.

```java
    // Subject
    interface Publisher {

        void subscribe(Subscriber subscriber);

        void unsubscribe(Subscriber subscriber);

        void notifySubscribers();

    }

    // ConcreteSubject
    static class MathTeacher implements Publisher {

        // Subject를 관찰하는 Observers
        private final List<Subscriber> subscribers = new ArrayList<>();

        // 추가
        @Override
        public void subscribe(Subscriber subscriber) {
            System.out.println(subscriber + "이(가) 수강했습니다.");
            subscribers.add(subscriber);

        }

        // 삭제
        @Override
        public void unsubscribe(Subscriber subscriber) {
            System.out.println(subscriber + "이(가) 수강을 취소했습니다.");
            subscribers.remove(subscriber);
        }

        // 메시지 발행
        @Override
        public void notifySubscribers() {
            for (Subscriber subscriber : subscribers) {
                subscriber.display();
            }
        }

    }

```

다음은 Subject인 수학 선생님을 관찰하는 Observer인 ``Student`` 클래스에 대한 코드입니다. 

```java
    // Observer
    interface Subscriber {

        // 메세지를 처리하는 메소드
        void display();
    }

    // ConcreteObserver
    static class Student implements Subscriber {

        final String name;

        public Student(String name) {
            this.name = name;
        }

        // 메시지 대한 처리
        @Override
        public void display() {
            System.out.println(name + "에게 알림이 왔습니다.");
        }

        @Override
        public String toString() {
            return this.name;
        }
    }

```

다음은 해당 Subject에 Observer들을 추가시켜 동작하는 코드입니다.  

```java 
    public static void main(String[] args) {
        Publisher mathTeacher = new MathTeacher();
        Student studentA = new ObserverPattern.Student("학생 A");
        Student studentB = new Student("학생 B");

        mathTeacher.subscribe(studentA);
        mathTeacher.subscribe(studentB);

        mathTeacher.notifySubscribers();

        mathTeacher.unsubscribe(studentB);
        
        mathTeacher.notifySubscribers();
    }
```

![console-result](/assets/img/software-architecture/design-pattern/observer-pattern/console-result.png)

위와 같이 옵저버 패턴을 적용해 코드를 작성하면 Subject와 Observer들은 강하게 결합되지는 않으며 상호 의존성이 낮은 [Loose Coupling 상태](https://ones1kk.github.io/posts/coupling/)를 유지하게 됩니다.

## HttpSessionAttributeListener

```java
public interface HttpSessionAttributeListener extends EventListener {


    default void attributeAdded(HttpSessionBindingEvent se) {
    }

    default void attributeRemoved(HttpSessionBindingEvent se) {
    }
    
    default void attributeReplaced(HttpSessionBindingEvent se) {
    }
}

```

``javax.servlet.http.HttpSessionAttributeListener``는 ``HttpSession`` 객체의 속성(attribute)이 변경되는 이벤트를 처리하기 위한 Subject 클래스입니다.
``HttpSessionAttributeListener``를 구현하면 세션의 속성이 추가되거나 제거되는 등의 상태 변화를 감지하고 원하는 동작을 수행할 수 있습니다.


오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.
  

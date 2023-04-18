---
title: 어노테이션 ModelAttribute vs RequestBody
date: 2023-04-09 11:10:00 +09:00
categories: [Spring]
tags: [spring-mvc]
---

# 들어가기에 앞서

Spring Framework는 웹 요청을 처리하기 위한 몇 가지 어노테이션(Annotation)을 제공합니다.  
가장 일반적으로 사용되는  ``` @ModelAttribute 와 @RequestBody ``` 두 어노테이션은 모두 Spring MVC 컨트롤러의 메소드 매개 변수에 요청 데이터를 매핑하는데 사용합니다.  
이 글에서는 ``` @ModelAttribute 와 @RequestBody ``` 각각의 어노테이션이 요청 데이터를 어떤 식(흐름)으로 사용이 되는지 살펴보고자 합니다.

# @ModelAttribute

## 특징

``` @ModelAttribute ``` 어노테이션은 메소드 매개 변수 또는 메소드 반환 값을 명시된 모델 속성(Model Attribute)에 바인딩하는데 사용됩니다.   
***메소드 매개 변수에 사용되는 경우*** 매개 변수가 명시된 모델 속성에 바인딩되어야 함을 나타내며, ***메소드에 사용되는 경우*** 반환 값이 모델 속성 값으로 사용되어야 함을 나타냅니다.

## @ModelAttribute의 동작 방식

``` @ModelAttribute ``` 선언된 매개 변수 매핑을 처리하기 위해서는 ``` ModelAttributeMethodProcessor ```라는 ``` ArgumentResolver ```를 사용합니다.

![ModelAttributeMethodProcessor](/assets/img/argument/model-attribute-method-processor.png)

``` ModelAttributeMethodProcessor  ```은 ``` HandlerMethodArgumentResolver, HandlerMethodReturnValueHandler ``` 2개의 인터페이스를 상속 받아 구현되어 있지만, 매개 변수를 어떤 식으로 매핑하는지 확인하기 위해 ``` ArgumentResolver ```의 내용만 다루도록 하겠습니다.


이 중 ***매개 변수 매핑 처리***에서 핵심적으로 살펴 볼 메소드는 ``` Object resolveArgument(MethodParamete, ModelAndViewContainer, NativeWebRequest, WebDataBinderFactory) ``` 입니다.  
해당 메소드는 내부에 핵심 로직을 담당하는 ``` createAttribute(name, parameter, binderFactory, webRequest) 와 bindRequestParameters(binder, webRequest) ```를 가지고 있습니다.

![createAttribute](/assets/img/argument/create-attribute.png)   
먼저 ``` createAttribute() ```를 살펴보면, 이 메소드의 핵심 메소드는 ``` constructAttribute() ```입니다.  
``` constructAttribute() ``` 는 컨트롤러의 매개 변수로 들어온 객체를 생성하는 메소드인데, 크게 두가지 부분으로 나누어서 볼 수 있습니다.

![constructAttribute-1](/assets/img/argument/construct-attribute-1.png)

- 먼저 적절한 생성자가 있는지 확인합니다.
- 적절한 생성자가 없다면 새로운 객체 인스턴스를 생성해 반환하고 메소드를 종료합니다.

![constructAttribute-2](/assets/img/argument/construct-attribute-2.png)

- 만일 적절한 생성자가 있다면, ``` HttpRequest ```의 파라미터 값과 파라미터 이름을 해당 생성자의 타입과 이름을 비교하여 생성한 객체를 반환합니다.

위의 과정을 걸쳐 생성된 모델 속성 객체(attribute)는 ``` constructAttribute() ``` 메소드를 활용하여 생성자를 통한 객체 생성 및 값 바인딩만 된 상태로, 생성자에서 값이 바인딩 되지 않은 필드가 있다면 아직 해당 필드는 null(또는 primitive 기본값) 값인 상태입니다.

예를 들어 아래와 같은 ```Member ``` 객체를 이용했다면, 선언되어 있는 생성자를 사용하여 ``` constructAttribute() ``` 메소드가 해당 객체를 생성 후 반환합니다.

![member](/assets/img/argument/member.png)

![call-bind-request-parameters](/assets/img/argument/call-bind-request-parameters.png)

디버그 창에서 보이듯이 ``` age ``` 필드의 값은 0인 상태고, ```age ```값을  바인딩하기 위해서는 ``` bindRequestParameters() ```를 통해 이루집니다.

![bind-request-parameters](/assets/img/argument/bind-request-parameters.png)

매개 변수로 넘겨 받은 ``` WebDataBinder ```를 ``` ServletRequestDataBinder ``` 다운 캐스팅 후 ``` bind(ServletRequest) ```를 호출합니다.

![bind](/assets/img/argument/bind.png)

그 후 Model의 property에 접근하여 값 변경을 돕는 ``` AbstractPropertyAccessor ```의 값들을 셋팅하게 됩니다.

![process-local-property](/assets/img/argument/process-local-property.png)

``` propertyHanlder ```를 통해 해당 모델에 존재하는 모든 property를 ``` setValue() ```해주게 됩니다.

![set-value](/assets/img/argument/set-value.png)

기본적으로 reflection으로 생성한 Method의 접근 제어자를 접근 가능하게 변경해주고, ``` getWriteMethod() ```를 통해 추출한 프로퍼티 접근 메소드를 통해 전달받은 value를 설정합니다.

> 주의  
> 비록 ```consructAttribute() ```를 통해 ``` username ``` 값은 주입이 된 후 생성이 되었지만, ``` List<MutablePropertyValues> propertyValues ```에는 모델의 모든 수정 가능한 프로퍼티가 담기기 때문에 property 갯수만큼의 ``` setPropertyValue() ```가 실행이 됩니다.  
> 결과적으로 ``` username, age ``` 2개의 프로퍼티를 가지고 있는 Member 모델은 ``` username ```의 값이 있는데도 불구하고, 총 2번의 setter를 호출합니다.

이렇게 생성된 객체는 정상적으로 반환이 되어 컨트롤러 매개 변수 타입으로 바인딩 후 전달됩니다.

![model-attribute-return](/assets/img/argument/model-attribute-return.png)

요약하자면
1. ``` ModelAttributeMethodProcessor ```는  ``` @ModelAttribute ```가 선언 되어있는(해당 어노테이션이 없어도 자동으로) 컨트롤러 메소드 매개 변수에 있는 객체를 HttpServletRequest로 넘어온 파라미터와 자동으로 바인딩합니다.
2. 해당 바인딩은 먼저 컨트롤러 메소드 매개 변수의 적절한 생성자를 찾아 바인딩하고, 그 후 프로퍼티에 대한 접근법(setter, 접근 제어자...)이 제공이 된다면 해당 접근법을 사용하여 값을 바인딩합니다.
3. 프로퍼티 접근법이 제공되지 않는다면, 생성자를 통해서 생성된 객체만을 반환합니다.
4. 프로퍼티 접근법이 제공된다면, 값이 이미 있어도 접근 가능한 모든 프로퍼티에 값을 바인딩합니다.
5. 당연히 필드값이 일치하지 않거나 타입이 일치하지 않는다면 오류를 뱉습니다.

# @RequestBody

## 특징

@RequestBody 어노테이션은 HTTP request body를 Spring MVC 컨트롤러의 메소드 매개 변수에 매핑하는데 사용됩니다.  
일반적으로 요청 데이터가 JSON 또는 XML과 같은 형식일 때 사용됩니다.  
Spring은 ``` org.springframework.http.converter.HttpMessageConverter ```를 사용하여 request body를 매개 변수 유형으로 변환합니다.

## @RequestBody의 동작 방식

![request-response-body-method-processor](/assets/img/argument/request-response-body-method-processor.png)

``` @RequestBody ```가 선언된 매개 변수 매핑을 처리하기 위해서는 ``` RequestResponseBodyMethodProcessor ```라는 ``` ArgumentResolver ```를 사용합니다.

해당 클래스에서 핵심적으로 살펴볼 메소드는 ``` resolveArgument(MethodParameter, ModelAndViewContainer, NativeWebRequest, WebDataBinderFactory) ```로 request로 넘어온 body를 해당 메소드에서 처리합니다.  
해당 메소드의 핵심 로직을 담당하는 ``` <T> Object readWithMessageConverters(NativeWebRequest, MethodParameter, Type paramType ``` 를 살펴 보겠습니다.

![read-with-message-converters](/assets/img/argument/read-with-message-converters.png)

``` WebRequest ```로 넘어온 값을 ``` ServletServerHttpRequest ``` 변환 하고, ``` @RequestBody ```를 사용한 메소드의 파라미터 정보들을 내부 protected 메소드인 ``` <T> Object readWithMessageConverters(HttpInputMessage, MethodParameter, Type) ``` 반환 값을 validation 후 최종적으로 반환합니다.

![find-converter](/assets/img/argument/find-converter.png)

``` messageConverters ``` 중 인자로 넘어온 ``` inputMessage(HttpServletRequest) ```를 converting 할 수 있는 converter를 찾습니다.
converting을 처리할 수 있는 converter는  ``` org.springframework.http.converter.json.MappingJackson2HttpMessageConverter ```로 해당 converter의  ``` T read(Type,Class<?>, HttpInputMessage) ```메소드를 호출하여 주어진 입력 메시지에서 주어진 유형의 객체를 읽고 반환합니다.

아래는 ``` MappingJackson2HttpMessageConverter ``` 의 내부 구현에 대한 설명입니다.

![read](/assets/img/argument/read.png)

``` org.springframework.http.converter.json.AbstractJackson2HttpMessageConverter.read(Type, Class<?>, HttpInputMessage) ```는 내부 pirvate 메소드인 ``` Object readJavaType(JavaType, HttpInputMessage) ```를 호출합니다.

![read-java-type](/assets/img/argument/read-java-type.png)

``` MappingJackson2HttpMessageConverter ```은 내부적으로 ObjectMapper를 활용하여 각 분기문의 조건에 따라 readValue()를 사용하여 request body 값을 매개 변수에 값을 바인딩합니다.

> ObjectMapper의 매핑 우선 순위
> 1. ``` @JsonProperty, @JsonSetter ``` 등 클래스, 필드, 메소드 레벨에서 사용되는 어노테이션을 우선적으로 참고합니다.
> 2. 클래스 내부에 매핑될 필드와 일치하는 이름을 가진 setter 메서드가 있다면, setter 메소드를 활용하여 값을 할당합니다.
> 3. 필드의 접근 제어를 우회하여 리플렉션을 사용하거나, 필드가 public인 경우에 직접 값을 설정합니다.
> 4. 생성자 매개변수와 매핑될 이름이 일치하는 경우, ObjectMapper는 생성자를 통해 객체를 생성하고 매개변수에 값을 할당합니다.
> 5. ObjectMapper는 기본 생성자를 사용하여 객체를 생성하고, setter 메서드나 필드 접근을 통해 값을 할당합니다.

ObjectMapper는 어노테이션과 setter 메서드가 가장 높은 우선 순위를 가지며, 필드 접근 및 생성자는 보조적으로 사용되는 방법입니다.

결국 클라이언트에서 요청한 request body는 적절한 converter를 활용하여 매개 변수 타입으로 매핑하여 ``` @RequestBody ```를 사용한 컨트롤러 메소드에 반환되게 됩니다.

# 결론

``` @ModelAttribute ```를 사용하여 HTTP 요청 매개 변수 및 세션 특성과 같은 다양한 소스의 데이터를 바인딩할 수 있는 반면 ``` @RequestBody ```는 일반적으로 JSON 또는 XML 형식의 데이터에 사용됩니다.  
또한 ``` @ModelAttribute ```는 GET 요청에 사용되는 반면 ``` @RequestBody ```는 일반적으로 POST 및 PUT 메소드에 사용됩니다.

``` @ModelAttribute와 @RequestBody ```는 모두 Spring MVC에서 활용되는 중요한 어노테이션입니다.  
두 어노테이션 모두 요청 데이터를 메소드 매개 변수에 매핑하는데 사용되지만 ``` @ModelAttribute ```는 다양한 소스의 데이터를 모델 특성으로 바인딩하는 데 사용되는 반면 ``` @RequestBody ``` 는 HTTP request body를 메소드에 매핑하는데 사용됩니다.

또한 ``` @ModelAttribute ```는 적절한 생성자가 없다면 setter를 필수로 가져야하지만, ``` @RequestBody ```는 내부적으로 ObjectMapper를 사용하기 때문에 setter 메소드가 필요 없다는 것이 큰 차이입니다.

오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.  

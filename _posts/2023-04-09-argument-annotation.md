---
title: 어노테이션 ModelAttribute vs RequestBody
date: 2023-04-09 11:10:00 +09:00
categories: [Spring]
tags: []
---

# 들어가기에 앞서

Spring Framework는 웹 요청을 처리하기 위한 몇 가지 어노테이션(Annotation)을 제공합니다.  
가장 일반적으로 사용되는 두 개의 어노테이션은 ``` @ModelAttribute 와 @RequestBody ```가 있습니다.  
두 어노테이션은 모두 Spring MVC 컨트롤러의 메소드 매개 변수에 요청 데이터를 매핑하는데 사용합니다.  
이 글에서는 ``` @ModelAttribute 와 @RequestBody ``` 각각의 어노테이션이 요청 데이터를 어떤 식(흐름)으로 매핑하는지에만 중점적으로 살펴보고자 합니다.

# @ModelAttribute

## 특징

``` @ModelAttribute ``` 어노테이션은 메소드 매개 변수 또는 메소드 반환 값을 명시된 모델 속성(Model Attribute)에 바인딩하는데 사용됩니다.   
***메소드 매개 변수에 사용되는 경우*** 매개 변수가 명시된 모델 속성에 바인딩되어야 함을 나타내며, ***메소드에 사용되는 경우*** 반환 값이 모델 속성 값으로 사용되어야 함을 나타냅니다.

## @ModelAttribute의 동작 방식

![ModelAttributeMethodProcessor](/assets/img/argument/model-attribute-method-processor.png)

``` ModelAttributeMethodProcessor  ```은 ``` HandlerMethodArgumentResolver, HandlerMethodReturnValueHandler ``` 2개의 인터페이스를 상속 받아 구현되어 있습니다.  
인터페이스명과 같이 매개 변수로 들어오는 값과 반환 값을 어떤 식으로 처리할지 구현하고 있지만 오늘은 ,서두에서 말씀드렸다싶이, 매개 변수를 어떤 식으로 매핑하는지 확인하기 위해 ``` ArgumentResolver ```의 내용만 다루도록 하겠습니다.

``` @ModelAttribute ```를 처리하기 위해서는 ``` ModelAttributeMethodProcessor ```라는 ``` ArgumentResolver ```를 사용합니다.

이 중에서 핵심적으로 살펴 볼 메소드는 ``` Object resolveArgument(MethodParamete, ModelAndViewContainer, NativeWebRequest, WebDataBinderFactory) ``` 입니다.  
해당 메소드는 내부에 ``` createAttribute(name, parameter, binderFactory, webRequest) 와. bindRequestParameters(binder, webRequest) ```를 가지고 있습니다.

![createAttribute](/assets/img/argument/create-attribute.png)   
먼저 ``` createAttribute() ```를 살펴보면, 이 메소드의 핵심 메소드는 ``` constructAttribute() ```입니다.  
``` constructAttribute() ``` 는 컨트롤러의 매개 변수로 들어온 객체를 생성하는 메소드인데, 크게 두가지 부분으로 나누어서 볼 수 있습니다.

![constructAttribute-1](/assets/img/argument/construct-attribute-1.png)

1. 먼저 적절한 생성자가 있는지 확인합니다. 적절한 생성자가 없다면 새로운 객체 인스턴스를 생성해 반환합니다.

![constructAttribute-2](/assets/img/argument/construct-attribute-2.png)

2. 만일 적절한 생성자가 있다면, ``` HttpRequest ```의 파라미터 값과 해당 생성자를 조합해 생성한 객체를 반환합니다.

# @RequestBody

## 특징

@RequestBody 주석은 HTTP request body를 Spring MVC 컨트롤러의 메소드 매개 변수에 매핑하는데 사용됩니다.  
일반적으로 요청 데이터가 JSON 또는 XML과 같은 형식일 때 사용됩니다.  
Spring은 ``` org.springframework.http.converter.HttpMessageConverter ```를 사용하여 request body를 매개 변수 유형으로 변환합니다.

## @RequestBody의 동작 방식
작성예정

# 차이

``` @ModelAttribute ```를 사용하여 HTTP 요청 매개 변수 및 세션 특성과 같은 다양한 소스의 데이터를 바인딩할 수 있는 반면 ``` @RequestBody ```는 일반적으로 JSON 또는 XML 형식의 데이터에 사용됩니다.  
또한 ``` @ModelAttribute ```는 GET 요청에 사용되는 반면 ``` @RequestBody ```는 일반적으로 POST 및 PUT 메소드에 사용됩니다.
- @RequestBody에서는 Setter 사용이 불가능하겠네??(작성예정)

# 결론

@ModelAttribute와 @RequestBody는 모두 Spring MVC에서 활용되는 중요한 어노테이션입니다.  
요청 데이터를 메소드 매개 변수에 매핑하는 데 사용되지만 목적이 다릅니다.  
``` @ModelAttribute ```는 다양한 소스의 데이터를 명명된 모델 특성으로 바인딩하는 데 사용되는 반면 ``` @RequestBody ``` 는 HTTP 요청 본문을 메소드에 매핑하는데 사용됩니다.


오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.

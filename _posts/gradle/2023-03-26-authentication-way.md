---
title: JWT(JSON Web Token) & Cookie & Session
date: 2023-03-20 00:28:10 +09:00
categories: [Web]
tags: [jwt, session, cookie, httpSession]
---

### 들어가기에 앞서
서버에서 보안, 인증, 권한 부여 등을 확인하고 관리하는 방식은 대표적으로 ***쿠키, 세션, 토큰 3가지 방식***이 있습니다.  
이 글에서는 3가지 방식(토큰은 JWT 기반)에 대해서 알아보고자 합니다.  
추가로 Spring Framework에서는 어떤 방식으로 해당 내용을 처리할 수 있을지 간단한 예제도 소개하도록 하겠습니다.

### JWT(JSON Web Token)

#### 역사
JWT는 JSON Web Token의 줄임말로 일반적으로 웹 애플리케이션에서 인증 및 권한 부여 목적으로 사용되는 표준입니다.  
XML을 이용하여 인터넷을 통해 데이터를 전송하던 2000년대 초반 때, 웹 애플리케이션이 점점 가볍고 이동성이 높은 JSON 데이터 전송 방식을 선호하게 되는 시점과 함께 JWT는 JSON 형식으로 데이터를 안전하게 전송하기 위한 수단으로 개발 되었습니다.

#### 배경
웹 애플리케이션이 더욱 복잡해지고 여러 사용자가 이용함에 따라 안전하고 명백한 방식으로 웹 애플리케이션을 처리할 필요성이 대두 되었습니다.  
또한 이전에는 서버가 세션 기반 인증으로 사용하는 것이 일반적이었지만, 이 방법은 확장성, 상태 관리에 어려움을 겪는 문제가 있었습니다.  
JWT는 상태 비저장, 확장성 및 다양한 플랫폼에서 사용할 수 있는 장점을 필두로 클라이언트에게 토큰을 발급합니다.  
토큰을 발급 받은 클라이언트는 토큰 자체에 포함된 권한 정보나 서비스를 사용하기 위한 정보(Self-contained)를 이용하기 때문에 따로 서버에 데이터를 저장할 필요 없이 서버에 대한 향후 요청을 할 수 있게 되었습니다.

#### 특징
JWT는 Header, Payload, Signature 이렇게 3개의 부분으로 구성되어 있습니다.

각 부분에 대해서 간략하게 설명하자면

Header: 토큰의 유형과 토큰 서명에 사용되는 알고리즘에 대한 정보를 포함합니다.

Payload: 토큰이 나타내는 실제 데이터를 포함하는 부분입니다.  
실제 데이터들은 claim이라고 불리며, JWt는 JSON을 이용해서 claim을 정의합니다.  
ID 또는 Role과 같이 사용자 정보를 서버에 저장하는 방식이 아닌, 토큰 자체가 정보를 가지고 있는 방식입니다.

Signature: 토큰이 변경됐는지 확인하는데 사용하는 부분입니다.  
Signature는 서버에 저장한 키를 사용하여 생성합니다.

> Byte를 적게 차지하는 것이 유리하기 때문에 독특하게도 JWT는 JSON 형식으로 데이터를 저장할 때 key 값을 3글자로 줄이는 관행이 있습니다.
> - sub: 인증 주체(subject)
> - iss: 토큰 발급처
> - typ: 토큰의 유형(type)
> - alg: 서명 알고리즘(algorithm)
> - iat: 발급 시각(issued at)
> - exp: 말료 시작(expiration time)
> - aud: 클라이언트(audience)

![jwt-anatomy](/assets/img/spring/web/jwt-anatomy.png)
> [출처](https://www.ibm.com/docs/en/cics-ts/6.1?topic=cics-json-web-token-jwt)

> Claim(메세지)을 JWT는 위 사진 우측의 Header, Payload, Signature 3개의 부분과 같이 JSON 형태로 표현한 것인데, > JSON은 개행 문자가 있기 때문에, REST API 호출 시 HTTP > Header에 넣기가 불편합니다.  
> 그래서 JWT는 claim JSON 문자열을 base64 인코딩을 통해 하나의 문자열로 변환합니다.

아래는 JWT가 Authentication을 처리하는 흐름입니다.

![token-based-authentication](/assets/img/spring/web/token-based-authentication.jpg)
> [출처](https://www.freecodecamp.org/news/how-to-sign-and-validate-json-web-tokens/)

#### 장점

1. 보안(Security): JWT는 토큰의 변조를 탐지할 수 있는 서명으로 안전하도록 설계되었습니다.  
   서명은 서버에만 알려진 키를 사용하여 생성되므로 서버만이 토큰을 발급하고 확인할 수 있습니다.
2. 무상태(Stateless): JWT를 사용하게 되면 서버가 세션 데이터를 저장할 필요가 없으므로 여러 서버에 걸쳐 확장 가능하고 쉽게 관리할 수 있습니다.
3. 상호 운용성(Interoperability): JWT는 JSON을 기반으로 사용하기 때문에 다양한 플랫폼 및 언어에서 작업이 용이합니다.
4. 사용자 정의(Customizable): JWT의 payload는 모든 데이터를 포함할 수 있으므로 다양한 사용 사례에 맞게 사용자 정의에 용이합니다.

#### 단점

1. Cookie, Session과는 다르게 base64 인코딩을 통한 정보를 전달하므로 전달량이 많아 질 수 있기 때문에 전송 데이터 양으로 인한 부하가 생길 수 있습니다.
2. payload는 암호화가 되어있지 않기 때문에 민감한 정보를 저장할 수 없습니다.
3. 토큰이 탈취 당한다면 만료될 때까지 대처가 불가능합니다.


오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.


---
title: Java Plugin & Java Library Plugin
date: 2023-03-16 23:13:10 +09:00
categories: [Spring, Library]
tags: [library, spring-boot-library, java-library]
---

Git 저장소(ex: GitGub, GitLab, Bitbucket...)를 통해서 코드를 관리할 때, private repository인 경우는 해당없지만, public repository로 관리하다 보면 민감한 정보에 대한 처리는 필수적입니다.
소스 코드에는 DB URL, Principal, Credential, Token Secret Key ... 등등 많은 민감한 정보를 담고 있기 때문입니다.

이를 위해 .gitignore file을 통해 Git 버전 관리에서 제외할 파일 목록을 지정 할 수 있지만,
1. 협업을 하는 상황
2. 사용하는 pc 환경이 자주 바뀌는 경우   
   ...

ignore 된 파일의 변경 사항을 추적하고 동시에 관리하기란 여간 힘든 일이 아닙니다.

이런 외부에 공개되면 안 되는 정보들을 암호화하여 용이하게 관리할 수 있는 오픈 소스 라이브러리인 Jasypt(Java Simplified Encryption) SpringBoot를 소개합니다.

# 사용법
먼저 [개발자 GitHub 페이지](https://github.com/ulisesbocchio/jasypt-spring-boot)입니다.  
이번 글에서는 간단한 사용법, 유용한 팁(?)을 함께 기술할 예정입니다.  
보다 자세한 내용이 궁금하신 분들은 해당 사이트에서 확인하시길 바라겠습니다.
### 먼저 당연히 해당 프로젝트에 의존성을 주입 받아야합니다.(2022년 12월 15일에 release된 가장 최신 버전 3.0.5)

```groovy
dependencies {
   implementation "com.github.ulisesbocchio:jasypt-spring-boot-starter:3.0.5"
}
```

```xml
<dependency>
  <groupId>com.github.ulisesbocchio</groupId>
  <artifactId>jasypt-spring-boot-starter</artifactId>
  <version>3.0.5</version>
</dependency>

```

### Jasypt Encryptor가 사용할 Password를 생성해야 합니다.
[Jasypt Online Encryption and Decryption(Free)](https://www.devglan.com/online-tools/jasypt-online-encryption-decryption) 사이트를 통해서 Two way Encryption Password를 생성할 수 있습니다.

### Configration class를 만든 후 작성해야 합니다.

```java 
@Configuration
@EnableEncryptableProperties
public class JasyptConfig {

    private static final String ITERATION = "1000";

    private static final String POOL_SIZE = "2";

    private static final String GENERATOR_CLASS_NAME = "org.jasypt.salt.RandomSaltGenerator";

    private static final String ALGORITHM = "PBEWithMD5AndDES";

    private static final String OUTPUT_TYPE = "BASE256";

    @Bean
    @Primary
    public StringEncryptor jasyptStringEncryptor(Environment environment) {
        String password = environment.getProperty("jasypt.encryptor.password");
        StandardPBEStringEncryptor encryptor = new StandardPBEStringEncryptor();
        SimpleStringPBEConfig config = new SimpleStringPBEConfig();
        config.setPassword(password);
        config.setAlgorithm(ALGORITHM);
        config.setKeyObtentionIterations(ITERATION);
        config.setPoolSize(POOL_SIZE);
        config.setSaltGeneratorClassName(GENERATOR_CLASS_NAME);
        config.setStringOutputType(OUTPUT_TYPE);
        encryptor.setConfig(config);
        return encryptor;
    }

}
```
> @EnableEncryptableProperties은 해당 라이브러리에서 제공하는 어노테이션으로 해당 어노테이션을 꼭 명시해주셔야 합니다.

### 2번째 스텝에서 만든 password가 ```SimpleStringPBEConfig```의 비밀번호로 사용됩니다.
여기서 시스템 환경 변수를 통해서 password를 주입을 받는 이유는 뒤에 설명하겠습니다.

> 그 후 Properties를 암호화합니다.

```java
class JasyptConfigTest {

    private static final String ITERATION = "1000";

    private static final String POOL_SIZE = "2";

    private static final String GENERATOR_CLASS_NAME = "org.jasypt.salt.RandomSaltGenerator";

    private static final String ALGORITM = "PBEWithMD5AndDES";

    private static final String OUTPUT_TYPE = "BASE256";

    @ParameterizedTest
    @ValueSource(strings = {"id", "password", "token-key"})
    void test(String value) {
        StandardPBEStringEncryptor encryptor = new StandardPBEStringEncryptor();
        SimpleStringPBEConfig config = new SimpleStringPBEConfig();
        config.setPassword("생성한 password");
        config.setAlgorithm(ALGORITM);
        config.setKeyObtentionIterations(ITERATION);
        config.setPoolSize(POOL_SIZE);
        config.setSaltGeneratorClassName(GENERATOR_CLASS_NAME);
        config.setStringOutputType(OUTPUT_TYPE);
        encryptor.setConfig(config);
        
        
        String encryptedValue = encryptor.encrypt(value);
        String decryptedValue = encryptor.decrypt(encryptedValue);

        Assertions.assertThat(value).isEqualTo(decryptedValue);

        System.out.println(value + " : " + encryptedValue);
    }

}
```

### 혹은 위 코드가 복잡하다고 생각이 드시면, @SpringBootTest를 통한 통합 테스트 환경에서 DI 받아 암호화하는 방법도 있습니다.

```java 
@SpringBootTest
class JasyptConfigTest {

    @Autowired
    private StringEncryptor encryptor;

    @ParameterizedTest
    @ValueSource(strings = {"id", "password", "token-key"})
    void test(String value) {
        String encryptedValue = encryptor.encrypt(value);
        String decryptedValue = encryptor.decrypt(encryptedValue);

        Assertions.assertThat(value).isEqualTo(decryptedValue);

        System.out.println(value + " : " + encryptedValue);
    }

}
```
> 다만 정상적인 실행을 위해서, ```Run/Debug Configurations ```에서 해당 테스트 클래스에 vm option ``` -Djasypt.encryptor.password=${생성한 password}```를 추가한 후 실행해야합니다.

위의 방법과 같이 암호화가 완료되면, console 창에 이렇게 출력이 됩니다.  
![output-of-encrypted](/assets/img/spring/library/result-of-encrypted.png)

> 저는 위와 같은 방식으로 진행했는데, 더 좋은 방법이 있으면 알려주십시오❗️❗️❗️  
해당 테스트 코드를 실행하면 암.복호화가 정상적으로 되는지 확인할 수 있고, 암호화된 값이 console창에 print됩니다.

### 암호화가 된 값을 기존 Raw Application Properties 값에 대체해줍니다.

```yml
(전)
spring:
  datasource:
    url: 127.0.0.1
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver

  (후)
spring:
  datasource:
    url: ENC(암호화된 값)
    username: ENC(암호화된 값)
    password: ENC(암호화된 값)
    driver-class-name: com.mysql.cj.jdbc.Driver

```  
> 암호화 된 값은 특정 ```prefix, suffix ```로 감싸서 표현해야합니다.     
기본으로 ``` ENC(, ) ```를 가지며, 커스텀 하기 위해선 아래와 같이 적용함으로써 변경 가능합니다.
```yml
jasypt:
  encryptor:
    property:
      prefix: 변경할 prefix
      suffix: 변경할 suffix
```

### 그 후 ``` Run/Debug Configurations ```에서 vm option을 추가합니다.
![add-vm-option](/assets/img/spring/library/configuration-vm-option.png)

### 정상적으로 실행이 되는 것을 확인 할 수 있으실겁니다❗️❗️❗️


# 유용한 팁(?)
작성 예정

## 오탈자 및 오류 내용을 댓글 또는 메일로 알려주시면, 검토 후 조치하겠습니다.

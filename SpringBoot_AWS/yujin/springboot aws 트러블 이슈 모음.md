# springboot aws 트러블 이슈 모음



## 3-1 JSON 구문 분석 오류 : Cannot deserialize instance of `java.lang.Long` out of START_OBJECT token;

[[SpringBoot & AWS] 3. 스프링 부트에서 JPA로 DB 다루기](https://velog.io/@redcarrot01/SpringBoot-AWS-3.-%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B6%80%ED%8A%B8%EC%97%90%EC%84%9C-JPA%EB%A1%9C-DB-%EB%8B%A4%EB%A3%A8%EA%B8%B0) 에서 만난 오류다.

<br>

**JSON 구문 분석 오류 : Cannot deserialize instance of `java.lang.Long` out of START_OBJECT token;**

> Error while extracting response for type [class java.lang.Long] and content type [application/json;charset=UTF-8]; nested exception is org.springframework.http.converter.HttpMessageNotReadableException: JSON parse error: Cannot deserialize instance of `java.lang.Long` out of START_OBJECT token; nested exception is com.fasterxml.jackson.databind.exc.MismatchedInputException:
>
> ...
>
> at com.yujin.service.springboot.web.PostsApiControllerTest.Posts_등록된다(PostsApiControllerTest.java:54)

오류를 직역하면 ~ 유형과 ~ 타입의 응답 추출하는 과정에서 오류가 발생했다.

중첩된 예외는 HttpMessageNotReadableException : JSON parse error 이다.

이것은 START_OBJECT 토큰에서 ~ 인스턴스를 역직렬화 할 수 없다..

<br>

~~~JAVA
// (PostsApiControllerTest.java:54) 오류 부분
ResponseEntity<Long> responseEntity = restTemplate.
                postForEntity(url, requestDto, Long.class);
~~~

<br>
postForEntity -> 즉, json으로 받는 post매핑 부분에서 뭔가 잘못되었다는 것 같다. 

post 매핑되는 곳들을 살펴보면,

 PostsApiController의 postmapping 부분을 보면  다음과 같고, 

```java
@PostMapping("api/v1/posts")
```

테스트코드를 보면 

~~~java
 String url = "http://localhost:"+port+"/api/v1/posts ";
~~~

<br> 

### 원인

`"/api/v1/posts "`에서 공백이 생겨서 생긴 오류였다.

즉, 테스트할 때 post method로 테스트하기 위해 postForEntity를 사용했는데, 

호출한 컨트롤러 메소드가 post의 형식을 제대로 갖추고 있지 않았기 때문에 발생했다..

### 요약

1. 오류 로그를 읽어보면 post mapping 부분에 에러가 발생했음
2. post를 날리는 곳, 받는 곳을 살핀다.
3. post -> put 메소드라고 적은 것은 아닌지,  url에서 오타가 있었는지 확인해본다.

### 해결

`"/api/v1/posts " -> "/api/v1/posts"` 공백제거 





## 5-5 Failed to serialize object using DefaultSerializer; DefaultSerializer requires a Serializable payload ..

[5. 스프링 시큐리티와 OAuth2.0으로 로그인 구현](https://velog.io/@redcarrot01/SpringBoot-AWS-5.-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0%EC%99%80-OAuth2.0%EC%9C%BC%EB%A1%9C-%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EA%B5%AC%ED%98%84) 에서 만난 오류다.

<br>

**Failed to serialize object using DefaultSerializer; DefaultSerializer requires a Serializable payload ..**

<img src="https://user-images.githubusercontent.com/38436013/119092391-58324300-ba49-11eb-855d-16a54434d8c7.png" alt="image" style="zoom:67%;" />

> Caused by: org.springframework.core.serializer.support.SerializationFailedException: Failed to serialize object using DefaultSerializer; nested exception is java.lang.IllegalArgumentException: DefaultSerializer requires a Serializable payload but received an object of type [com.yujin.service.springboot.config.auth.dto.SessionUser]

직역하면,

SerializationFailedException은 직렬화 오류 발생이고,

DefaultSerializer는 직렬화된 페이로드를 받아야 하지만 그렇지 않았다.

<BR>

### 도데체 자바 직렬화가 무엇인지 몰라서 찾아보았다.

[참고했습니다1](https://woowabros.github.io/experience/2017/10/17/java-serialize.html)

[참고했습니다2](https://woowabros.github.io/experience/2017/10/17/java-serialize2.html)

### what?

자바 시스템 내부에서 사용되는 객체 or 데이터를 외부 자바 시스템에서도 사용할 수 있도록

**byte 형태로 변환해주는 기술이 직렬화**이며

그 반대가 역직렬화하고 한다.

> 시스템 적으로 설명하면, 
>
> JVM의 메모리에 상주되어 있는 객체 데이터를 직렬화, 혹은 역직렬화하여
>
> JVM으로 상주시키는 형태를 의미한다. 

### how? 어떻게 사용?

 `java.io.Serializable` 인터페이스를 상속받고, 

`java.io.ObjectOutputStream` 객체를 이용하면 직렬화 가능하다.

### why? 왜 사용?

문자열 형태의 직렬화 방법

- 직접 데이터를 문자열 형태로 확인 가능한 직렬화 방법이다
- api나 데이터 변환할 때 많이 사용
- 다량의 데이터 직렬화시 csv, json 등으로 많이 사용함 -> `csv, json은 자바 직렬화는 아니다!`

### 자바 직렬화 장점

- 자바 시스템 개발 최적화

### when, where 자바 직렬화 언제 어디서 사용?

JVM의 메모리에서만 상주되어있는 객체 데이터를 그대로 영속화(Persistence)가 필요할 때 사용

- 서블릿 세션

  - 서블릿 기반의 WAS(톰캣, 웹로직 등)들은 대부분 세션의 자바 직렬화를 지원

  - 세션을 파일로 저장하거나 세션 클러스터링, DB를 저장하는 옵션 등을 선택하게 되면 

    세션 자체가 직렬화가 되어 저장되어 전달

  -  세션에 필요한 객체는 `java.io.Serializable` 인터페이스를 구현(`implements`) 해두는 것을 추천

    > 이부분이 나의 코드에서 직렬화 상속 받아야 이유이다.

- 캐시

  - 캐시([Ehcache](http://www.ehcache.org/), [Redis](https://redis.io/), [Memcached](https://memcached.org/), …) 라이브러리에서 많이 사용

  - DB를 조회한 후 가져온 데이터 객체 같은 경우 실시간 형태로 요구하는 데이터가 아니라면 메모리, 외부 저장소, 파일 등을 저장소를 이용해서 데이터 객체를 저장한 후 동일한 요청이 오면 DB를 다시 요청하는 것이 아니라 저장된 객체를 찾아서 응답하게 하는 형태를 보통 캐시를 사용
  - 캐시를 이용하면 DB에 대한 리소스를 절약 가능

### 원인

세션 객체를 직렬화 하지 않아서 생긴 오류다.

세션 객체를 직렬화 하는 이유는 세션을 db에 저장, 유지해야 하는데, 이때 세션 자체를 직렬화해줘야 한다.

### 해결
java.io.Serializable 인터페이스를 상속

이전코드

~~~java
package com.yujin.service.springboot.config.auth.dto;
//...
@Getter
public class SessionUser {
    private String name;
//...
~~~

수정코드

~~~java
package com.yujin.service.springboot.config.auth.dto;

//...
import java.io.Serializable;
@Getter
public class SessionUser implements Serializable{
    private String name;
//...
~~~



## 5-7 No qualifying bean of type :  expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {}

[5. 스프링 시큐리티와 OAuth2.0으로 로그인 구현](https://velog.io/@redcarrot01/SpringBoot-AWS-5.-%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0%EC%99%80-OAuth2.0%EC%9C%BC%EB%A1%9C-%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EA%B5%AC%ED%98%84) 에서 만난 오류다.



> Failed to load ApplicationContext
> java.lang.IllegalStateException: Failed to load ApplicationContext
> 	at org.springframework.test.context.cache.DefaultCacheAwareContextLoaderDelegate.loadContext(DefaultCacheAwareContextLoaderDelegate.java:125)
> ...
>
> : org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'com.yujin.service.springboot.config.auth.CustomOAuth2UserService' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {}



**No qualifying bean of type. expected at least 1 bean which qualifies as autowire candidate**

직역하면, 만족하는 빈이 없다. autowire할 빈이 적어도 1개 필요하다.



엥? 어노테이션 @Repository, @Service .. 건드린 부분은 없었다.

이문제는 아닌 것 같고,  SecurityConfig 스캔 제거 코드 추가하고 나서 난 오류라, 이부분을 살폈다.

<br>

#### 드디어 발견

에러 로그 귀찮지만, 다시 보았다.

~~~
Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'securityConfig' defined in file [C:\MyProject_and_Study\springboot-aws-webservice\build\classes\java\main\com\yujin\service\springboot\config\auth\SecurityConfig.class]: Unsatisfied dependency expressed through constructor parameter 0; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'com.yujin.service.springboot.config.auth.CustomOAuth2UserService' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {}
...
~~~

직역하면, 내 로컬에 있는 securityConfig는 빈 생성에 문제가 있다, CustomOAuth2UserService에 적합한 빈이 없다..

바로 import를 확인했다.



난 엉뚱하게도 카타리나 친구를 import하고 있었다.

~~~java
import org.apache.catalina.security.SecurityConfig;
~~~



***똥 멍 청 이 !!***



### 원인

에러 로그 귀찮지만, 다시 보기

**빈 생성에 문제가 있는 경우, 어노테이션 OR IMPORT 위치 확인하기**

import 오류는 가끔씩 있는 편, 꼭 확인!



### 해결

~~~java
import com.yujin.service.springboot.config.auth.SecurityConfig;
~~~


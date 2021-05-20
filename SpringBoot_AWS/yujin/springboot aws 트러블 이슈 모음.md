# springboot aws 트러블 이슈 모음



## 3-1 JSON 구문 분석 오류 : Cannot deserialize instance of `java.lang.Long` out of START_OBJECT token;

~~~java
Error while extracting response for type [class java.lang.Long] and content type [application/json;charset=UTF-8]; nested exception is org.springframework.http.converter.HttpMessageNotReadableException: JSON parse error: Cannot deserialize instance of `java.lang.Long` out of START_OBJECT token; nested exception is com.fasterxml.jackson.databind.exc.MismatchedInputException:

...
    
at com.yujin.service.springboot.web.PostsApiControllerTest.Posts_등록된다(PostsApiControllerTest.java:54)
~~~

오류를 직역하면 ~ 유형과 ~ 타입의 응답 추출하는 과정에서 오류가 발생했다.

중첩된 예외는 HttpMessageNotReadableException : JSON parse error 이다.

이것은 START_OBJECT 토큰에서 ~ 인스턴스를 역직렬화 할 수 없다..

~~~JAVA
// (PostsApiControllerTest.java:54) 오류 부분
ResponseEntity<Long> responseEntity = restTemplate.
                postForEntity(url, requestDto, Long.class);
~~~

postForEntity -> 즉, json으로 받는 post매핑 부분에서 뭔가 잘못되었다는 것 같다. post 매핑되는 곳들을 살펴보면,

 PostsApiController의 postmapping 부분을 보면  다음과 같고, 

```java
@PostMapping("api/v1/posts")
```

테스트코드를 보면 

~~~java
 String url = "http://localhost:"+port+"/api/v1/posts ";
~~~

#### 원인

`"/api/v1/posts "`에서 공백이 생겨서 생긴 오류였다.

즉, 테스트할 때 post method로 테스트하기 위해 postForEntity를 사용했는데, 

호출한 컨트롤러 메소드가 post의 형식을 제대로 갖추고 있지 않았기 때문에 발생했다..

#### 요약 

1. 오류 로그를 읽어보면 post mapping 부분에 에러가 발생했음
2. post를 날리는 곳, 받는 곳을 살핀다.
3. post -> put 메소드라고 적은 것은 아닌지,  url에서 오타가 있었는지 확인해본다.

#### 해결

`"/api/v1/posts " -> "/api/v1/posts"` 공백제거 



## 4-3 404,\"error\":\"Not Found\",\"message\":\"No message available\",\"path\":\"/posts/api/v1/posts\"}"

  


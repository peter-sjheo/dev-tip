# MockMvc란?
> MockMvc는 웹 어플리케이션을 애플리케이션 서버에 배포하지 않고 테스트용 MVC환경을 만들어 요청 및 전송, 응답기능들을 제공해준다.
> 즉 Spring에서 MVC패턴으로 작성한 프로그램의 컨트롤러를 시작해서 테스트를 해주는 프레임워크라고 볼수 있는것이다.

간단히 이야기하자면
내가 구현한 컨트롤러에 있는 기능을 테스트해야하는데, 화면이 없거나 Client가 없거나 등의 사유로 기능 테스트를 하기 힘든 환경일때,
서버에 어플리케이션을 올리지 않고 테스트하려고 할 경우에, 테스트 MVC환경을 지원해주는 프레임워크라고 볼수 있다.


# MockMvc 사용법
간단히 아래처럼 구현한 컨트롤러 api들을 테스트한다고 하자.
```Java


@Getter
@Builder
public class TestMessage
{
	private String message1;
	private String message2;
}


public class TestController {
	@GetMapping("/success")
	public ResponseEntity success() {	
		return new ResponseEntity<>("success", HttpStatus.OK);
	}


	@GetMapping("/serverError", produces = "application/json")
	public ResponseEntity<TestMessage> serverError() {	
		TestMessage message = TestMessage.builder()
			.message1("message1 send")
			.message2("message2 send")
			.build();

		return new ResponseEntity<>(message, HttpStatus.INTERNAL_SERVER_ERROR);	//강제로 500오류 발생.
	}
}


```
> 위의 코드는 아래코드 (MockMvc 활용)를 통해 테스트 가능하다.

```Java

class ControllerTest {
	
    @Autowired
    private MockMvc mockMvc; // mockMvc 생성
    
    @Test
    public void testSuccess() throws Exception{
    	
        
        //mockMvc에게 테스트 대상 컨트롤러 api("URI : success") 정보 입력.
        
        mockMvc.perform(
        get("/success") 							// 해당 url로 요청을 한다.
        .contentType(MediaType.APPLICATION_JSON) 	// Json 타입으로 지정
        .andExpect(status().isOk()) 				// 응답 status를 ok로 테스트
		.andExpect(content().string("success")) 	// 응답 status를 ok로 테스트
        .andDo(print()); 							// 응답값 print
        );
        
	}
	
		    
    @Test
    public void testServerError() throws Exception{
    	        
        //mockMvc에게 테스트 대상 컨트롤러 api("URI : success") 정보 입력.
        
        mockMvc.perform(
        get("/serverError")
        .contentType(MediaType.APPLICATION_JSON)
        .andExpect(status().is5xxServerError())									//Http 500대 오류 확인.
		.andExpect(content().contentType(MediaType.APPLICATION_JSON_UTF8))		//api 리턴 contentType 확인. (produce)
		.andExpect(jsonPath("$.message1").value("message1 send"))				//api 리턴값의 실제값 확인.
		.andExpect(jsonPath("$.message2").value("message2 send"))
        .andDo(print())
		.andReturn();
        );
        
	}
}


```
소스보면 대충 짐작하겠지만 
.contentType() 등과 같이 다양한 요청에 대한 설정을 지원해준다.

```Java

@Test
public void test() throws Exception{
	
	mockMvc.perform(
	get("/serverError")
	.contentType(MediaType.APPLICATION_JSON)
	.param("blahblah")									//Http 응답 검증
	.header("key", "12345")
	.content("hello")
	.andDo(print())
	.andReturn();
	);
	
}

/*
param / params 
-: 쿼리 스트링 설정

cookie 
-: 쿠키 설정

requestAttr 
-: 요청 스코프 객체 설정

sessionAttr 
-: 세션 스코프 객체 설정

content
-: 요청 본문 설정

header / headers 
-: 요청 헤더 설정

contentType
-: 본문 타입 설정

이외에 get, post, put, delete등도 지원한다.

*/

```



또한 andExcept 메소드는 다양한 검증방법을 지원해준다.

```Java

@Test
public void test() throws Exception{
	
	mockMvc.perform(
	get("/serverError")
	.contentType(MediaType.APPLICATION_JSON)
	.andExpect(status().is5xxServerError())									//Http 응답 검증
	.andExpect(header().string("Key", "12345"))								//응답 헤더값 검증			
	.andExpect(content().contentType(MediaType.APPLICATION_JSON_UTF8))		//응답양식 검증
	.andExpect(jsonPath("$.message1").value("message1 send"))				//data 검증
	.andExpect(jsonPath("$.message2").value("message2 send"))
	.andDo(print())
	.andReturn();
	);
	
/*	
status
-HTTP 상태 코드 검증

header
-응답 해더의 상태 검증

cookie
-쿠키 상태 검증

content
-응답 본문 내용 검증 jsonPath나 xpath와 같은 특정 콘텐츠를 위한 메서드도 제공

view
-컨트롤러가 반환한 뷰 이름 검증

forwardedUrl
-이동대상의 경로를 검증 패턴으로 검증할떄는 forwardedUrlPattern 메서드를 사용

redirectedUrl
-리다이렉트 대상의 경로 또는 URL 검증 패턴으로 검증할때는 redirectedUrlPattern 메서드를 사용

model
-스프링 MVC 모델 상태 검증

flash
-플래시 스코프의 상태 검증

request
-서블릿 3.0부터 지원되는 비동기 처리의 상태나 요청 스코프의 상태, 세션 스코프의 상태 검증
*/

}





```

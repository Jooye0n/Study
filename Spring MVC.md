### 1. 웹 서버, 웹 애플리케이션 서버
웹은 전부 HTTP 프로토콜을 기반으로 통신한다. 웹 서버와 웹 애플리케이션 서버는 둘다 HTTP 기반으로 동작하는 서버이다. 다만 웹 서버는 요청 전달과 정적 리소스(파일) 제공이 주 목적, 웹 애플리케이션 서버(WAS) 는 애플리케이션 코드의 실행이 주 목적인 서버를 말한다. NGINX, Apache, IHS (IBM HTTP Server)는 웹 서버이고 WebSphere, 톰캣, Jetty, Undertow 는 WAS 이다. 
은행처럼 규모가 큰 시스템에서는 일반적으로 웹 서버와 WAS를 분리해서 운영한다. WAS 과부하가 우려되기도 하고, 정적 파일 제공은 비용이 저렴하기 때문.
로드밸런서가 여러 웹 서버 중 하나로 요청을 분산하며, 웹 서버는 여러 WAS 에 요청을 분산해준다. 
==사용자(브라우저/앱) → L4 또는 L7 로드밸런서 → 웹 서버(IHS / Apache) → WAS(WebSphere / Tomcat) → 업무 처리(DB / 대외계 API) → WAS → 웹 서버 → 사용자==
* L4 (전송 계층) 로드밸런서: IP 주소와 포트, TCP/UDP 연결 정보 등을 기준으로 분산. 예를 들어 `10.0.0.10:8080`으로 들어온 연결을 여러 WAS 중 하나에 전달할 수 있다.
* L7 (응용 계층) 로드밸런서: HTTP URL, 헤더, 쿠키 등 요청 내용을 기준으로 분산. 예를 들어 `/loan/*`은 대출 서비스로, `/account/*`는 계좌 서비스로 전달할 수 있다. 
### 2. HTML Form 데이터 전송
POST 전송 시엔 웹 브라우저가 요청 HTTP 메시지를 생성한다. 데이터를 Body 에 담아 요청을 전송하며, `Content-Type`으로 Body에 담긴 데이터의 형식을 알려준다. 

| Content-Type                        | Body 형식                      |
| ----------------------------------- | ---------------------------- |
| `application/x-www-form-urlencoded` | `userId=hong&age=30`         |
| `application/json`                  | `{"userId":"hong","age":30}` |
| `multipart/form-data`               | 파일 및 폼 데이터를 여러 부분으로 구분해 전송   |
웹 애플리케이션 서버를 직접 구현한다면, Form 데이터 하나를 전송하기 위해 소켓 연결부터 시작해야 한다. 운영체제에게 파일열기를 요청할 때 `new FileInputStream`을 하듯, 통신을 요청할 땐 `new Socket`먼저 해야한다. 다음으로 HTTP 요청을 파싱하고, `Content-Type` 에 따라 바디 내용을 파싱하고 ..등등 복잡하고 비효율적인 과정을 거쳐야 한다. 
그래서 등장한 것이 서블릿이다. 서블릿이란 웹 서버에서 HTTP 요청을 처리하고 응답을 생성하는 자바 프로그램이다. HTTP 요청을 처리하고, HTTP 응답에 들어갈 HTML 을 직접 만든다. 개발자는 애플리케이션 로직만 작성한다. 
WAS는 Request, Response 객체를 새로 만들어서 서블릿 객체를 호출한다. 서블릿 객체는 누가 생성하고 호출할까? 바로 서블릿 컨테이너 이다. 톰캣처럼 서블릿을 지원하는 WAS를 서블릿 컨테이너라고 한다. WAS는 웹 애플리케이션을 실행하는 서버이고, 서블릿 컨테이너는 그 안에서 서블릿의 생명주기와 요청 처리를 관리하는 구성 요소이다. 서블릿 객체를 생성, 초기화, 호출, 종료하는 생명주기를 관리한다. 멀티 쓰레드까지 관리해준다. 서블릿 객체는 싱글톤으로 관리한다. 

### 3. 동시 요청 - 멀티 쓰레드
서블릿은 누가 호출해줄까? 쓰레드가 호출한다. 쓰레드가 애플리케이션 코드를 하나하나 순차적으로 실행한다. 동시 처리가 필요하면 쓰레드를 추가로 생성해야 한다. 요청마다 쓰레드를 생성해도 되지만, 쓰레드 생성 비용이 고가이다.  또한 쓰레드가 많아지면 응답 속도가 늦어지며, 컨텍스트 스위칭 비용이 발생한다. 생성에 제한이 없으므로 고객 요청이 너무 많으면 CPU, 메모리 임계점을 넘어서 서버가 죽을 수 있다. 
따라서 대부분의 WAS 는 `쓰레드 풀`을 구성한다. 쓰레드를 사용한 뒤 풀에 반납하는 구조이다. 

### 4. CSR, CSR
SSR: 서버에서 최종 HTML을 생성해서 클라이언트에게 전달. 주로 정적인 화면에 사용. 관련기술: JSP, 타임리프
CSR: HTML 결과를 자바스크립트를 사용해 웹 브라우저에서 동적으로 생성해서 적용. 주로 동적인 화면에 사용. 관련기술: React, Vue.js

### 5. 서블릿부터  JSP까지
서블릿과 자바 코드만으로 HTML을 만들 수 있었다. 서블릿 덕분에 동적으로 원하는 HTML을 마음껏 만들 수 있다. 그런데, 코드에서 보듯이 이것은 매우 복잡하고 비효율 적이다. 자바 코드로 HTML을 만들어 내는 것 보다 차라리 HTML 문서에 동적으로 변경해야 하는 부분만 자바 코드를 넣을 수 있다면 더 편리할 것이다. 이것이 바로 템플릿 엔진이 나온 이유이다. 템플릿 엔진을 사용하면 HTML 문서에서 필요한 곳만 코드를 적용해서 동적으로 변경할 수 있다. JSP도 결국 서블릿으로 변환된다. WAS가 JSP를 서블릿으로 변환된 다음, 그 서블릿이 Java 컴파일된다. 
ex) JSP, Thymeleaf, Freemarker, Velocity  

### 6. MVC 패턴
하나의 서블릿이나 JSP 만으로 비즈니스 로직과 뷰 랜더링까지 모두 처리하게 되면, 너무 많은 역할을 하게되고 결과적으로 유지보수가 어려워진다. 진짜 문제는 둘 사이에 변경의 라이프 사이클이 다르다는 점이다. 기능 특화 관점에서도 특히 JSP 같은 뷰 템플릿은 화면을 렌더링 하는데 최적화 되어 있기 때문에 이 부분의 업무만 담당하는 것이 가장 효과적이다. 
MVC 패턴은 지금까지 학습한 것 처럼 하나의 서블릿이나, JSP로 처리하던 것을 컨트롤러와 뷰라는 영역으로 서로 영역을 나눈 것을 말한다. 
- 컨트롤러: HTTP 요청을 받아서 파라미터를 검증하고, 비즈니스 로직을 실행한다. 그리고 뷰에 전달할 결과 데이터를 조회해서 모델에 담는다. 
- 모델: 뷰에 출력할 데이터를 담아둔다. 뷰가 필요한 데이터를 모두 모델에 담아서 전달해주는 덕분에 뷰는 비즈니스 로직이나 데이터 접근을 몰라도 되고, 렌더링에만 집중할 수 있다. 
- 뷰: 모델에 담겨있는 데이터를 사용해서 화면을 그리는 일에 집중한다. 여기서는 HTML을 생성하는 부분을 말한다. 
```java
request.setAttribute("member", member); //request객체에 데이터를 보관해서 뷰에 전달한다. HttpServletRequest 를 Model 로 사용한다. 
String viewPath = "/WEB-INF/views/new-form.jsp"; //WEB-INF 경로안에 JSP가 있으면 외부에서 직접 JSP를 호출할 수 없다. 컨트롤러를 통해서 JSP가 호출되도록 한다.
RequestDispatcher dispatcher = request.getRequestDispatcher(viewPath);
dispatcher.forward(request, response); //다른 서블릿이나 JSP로 이동할 수 있는 기능이다. 서버 내부에서만 다시 호출이 발생한다.
```
위 코 드는 항상 반복된다. 이를 개선하기 위해 ==프론트 컨트롤러==도 도입하였다. View를 분리하고, 서블릿 기술을 몰라도 동작 및 테스트를 진행할 수 있도록 Model을 추가하였다. 컨트롤러에서 지정하는 뷰 이름에도 중복이 있으므로, 뷰의 논리 이름을 반환하고, 실제 물리 위치의 이름은 뷰 리졸버에서 처리하도록 단순화 하였다.
==**프레임워크나 공통 기능이 수고로워야 사용하는 개발자가 편리해진다.**==

### 7. 어댑터 패턴
![[Pasted image 20261017220557.png|659]]
- 핸들러: 컨트롤러의 이름을 더 넒은 범위인 핸들러로 변경했다. 그 이유는 이제 어댑터가 있기 때문에 꼭 컨트롤러의 개념 뿐만 아니라 어떠한 것이든 해당하는 종류의 어댑터만 있으면 다 처리할 수 있기 떄문이다. 

### 8. 스프링 MVC
![[Pasted image 20261017224809.png|656]]
스프링 MVC의 프론트 컨트롤러가 바로 DispatcherServlet 이다. DispatcherServlet 도 부모 클래스에서 HttpServlet 을 상속 받아서 사용하고, 서블릿으로 동작한다. 
스프링 부트는 DispatcherServlet 을 서블릿으로 자동으로 등록하면서 모든 경로("/") 에 대해 매핑한다. 더 자세한 경로가 우선순위가 높으므로 기존에 등록한 서블릿도 함께 동작한다.
가장 우선순위가 높은 핸들러 매핑과 핸들러 어댑터는 `RequestMappingHandlerMapping` ,`RequestMappingHandlerAdapter` 이다. 
`@RequestMapping`이 있으면 `RequestMappingHandlerMapping` ,`RequestMappingHandlerAdapter` 이다. 

### 9. War 와 Jar
Jar 를 사용하면 항상 내장 서버(톰캣 등)을 사용하고, `webapp`경로도 사용하지 않는다. 내장 서버 사용에 최적화 되어 있는 기능이다. 최근에는 주로 Jar 를 사용한다. 
War를 사용하면 내장 서버도 사용 가능하지만, 주로 외부 서버에 배포하는 목적으로 사용한다. 주로 톰캣을 별도로 사용하고 빌드된 파일을 넣을 때, jsp 쓸 때 사용한다. 

### 10. 로깅
로그 라이브러리는 Logback, Log4J, Log4J2 등등 수 많은 라이브러리가 있는데, 그것을 통합해서 인터페이스로 제공하는 것이 바로 SLF4J 라이브러리다. 
쉽게 이야기해서 SLF4J는 인터페이스이고, 그 구현체로 Logback 같은 로그 라이브러리를 선택하면 된다. 실무에서는 스프링 부트가 기본으로 제공하는 Logback을 대부분 사용한다.
```java
@Slf4j

log.trace("trace log={}", name);
log.debug("debug log={}", name);
log.info(" info log={}", name);
log.warn(" warn log={}", name);
log.error("error log={}", name);

// LEVEL: TRACE > DEBUG > INFO > WARN > ERROR
```

application.properties 에서 로그 레벨 설정이 가능하다.
```shell
#전체 로그 레벨 설정(기본 info)
logging.level.root=info
#hello.springmvc 패키지와 그 하위 로그 레벨 설정
logging.level.hello.springmvc=debug
```
`log.debug("data="+data)` 로 사용하지 말 것. 의미없는 문자 더하기 연산이 일어나지 않도록 주의하자.

### 11. Argument Resolver 와 ReturnValueHandler
- 메시지 컨버터: HTTP의 Body에 문자 내용을 직접 반환할 때, 뷰 리졸버 대신에 `HttpMessageConverter`가 동작한다. @RequestBody, @ResponseBody 등을 사용하는 경우에 HTTP 메시지 컨버터를 적용한다. 
  ( HTTP Body를 읽거나 쓰는 공통적인 처리 역할을 포함해서 Java 객체를 JSON으로 직렬화하고, HTTP 응답 Body에 기록하는 역할을 한다. )
![[Pasted image 20261018030731.png|482]]
메시지 컨버터는 핸들러 어댑터와 관련이 있다. 생각해보면 애노테이션 기반의 컨트롤러는 매우 다양한 파라미터를 사용할 수 있다. 이는 `ArgumentResolver` 덕분이다. 애노테이션 기반 컨트롤러를 처리하는 `RequestMappingHandlerAdapter`는 바로 `ArgumentResolver`를 호출해서 컨트롤러(핸들러)가 필요로 하는 다양한 파라미터의 값(객체)을 생성한다. 그리고 이렇게 파라미터의 값이 모두 준비되면 컨트롤러를 호출하면서 값을 넘겨준다.  스프링은 30개가 넘는 `ArgumentResolver`를 제공한다. 
- 요청: `@RequestBody` 를 처리하는 `ArgumentResolver` 가 있고, `HttpEntity` 를 처리하는 `ArgumentResolver` 가 있다. 이 `ArgumentResolver` 들이 HTTP 메시지 컨버터를 사용해서 필요한 객체를 생성하는 것이다. 
- 응답:  `@ResponseBody` 와 `HttpEntity` 를 처리하는 `ReturnValueHandler` 가 있다. 그리고 여기에서 HTTP 메시지 컨버터를 호출해서 응답 결과를 만든다.

### 12. BindingResult 검증처리
```java
@PostMapping("/add")
public String addItemV1(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {

	if (!StringUtils.hasText(item.getItemName())) {
		bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다."));
	}
	
	//특정 필드 예외가 아닌 전체 예외
	if (item.getPrice() != null && item.getQuantity() != null) {
		int resultPrice = item.getPrice() * item.getQuantity();
		if (resultPrice < 10000) {
			bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은 10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
		}
	}
	
	if (bindingResult.hasErrors()) {
		log.info("errors={}", bindingResult);
		return "validation/v2/addForm";
	}
	
	//성공 로직
	Item savedItem = itemRepository.save(item);
	redirectAttributes.addAttribute("itemId", savedItem.getId());
	redirectAttributes.addAttribute("status", true);
	return "redirect:/validation/v2/items/{itemId}";
	
}
```
`BindingResult bindingResult` 파라미터의 위치는 `@ModelAttribute Item item` 다음에 와야 한다. 
ModelAttribute 객체의 바인딩 결과를 담고 있기 때문이다. 

복잡한데 왜 쓸까? "핵심은 구체적인 것에서 덜 구체적인 것으로" `MessageCodesResolver`는 required.item.itemName 처럼 구체적인 것을 먼저 만들어주고, required 처럼 덜 구체적인 것을 가장 나중에 만든다. 이렇게 하면 앞서 말한 것 처럼 메시지와 관련된 공통 전략을 편리하게 도입할 수 있다. "왜 이렇게 복잡하게 사용하는가?" 모든 오류 코드에 대해서 메시지를 각각 다 정의하면 개발자 입장에서 관리하기 너무 힘들다. 크게 중요하지 않은 메시지는 범용성 있는 requried 같은 메시지로 끝내고, 정말 중요한 메시지는 꼭 필요할 때 구체적으로 적어서 사용하는 방식이 더 효과적이다. 
스프링은 검증을 체계적으로 제공하기 위해 `Validator`인터페이스를 제공한다. 이를 사용하여 검증기를 만들면 `@Validated` 애노테이션으로 검증이 가능하다. 
```java
@PostMapping("/add")
public String addItemV6(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes) {
	if (bindingResult.hasErrors()) {
		log.info("errors={}", bindingResult);
		return "validation/v2/addForm";
	}
	//성공 로직
	Item savedItem = itemRepository.save(item);
	redirectAttributes.addAttribute("itemId", savedItem.getId());
	redirectAttributes.addAttribute("status", true);
	return "redirect:/validation/v2/items/{itemId}";
}
```

### 13. Bean Validation
필드별 간단한 애노테이션으로 검증이 가능하다. 
```shell
implementation 'org.springframework.boot:spring-boot-starter-validation'
```
위 라이브러리를 넣으면 스프링 부트가 자동으로 Bean Validator 를 인지하고 스프링에 통합한다.
`LocalValidatorDactoryBean`을 글로벌 Validator 로 등록한다. 이 Validator 는 @NotNull 같은 애노테이션을 보고 검증을 수행한다. 
```java
public class Item {

private Long id;

@NotBlank
private String itemName;

@NotNull
@Range(min = 1000, max = 1000000)
private Integer price;

@NotNull
@Max(9999)
private Integer quantity;

//...
}
```
먼저 Bean Validation은 특정한 구현체가 아니라 Bean Validation 2.0(JSR-380)이라는 기술 표준이다. 쉽게 이야기해서 검증 애노테이션과 여러 인터페이스의 모음이다. 마치 JPA가 표준 기술이고 그 구현체로 하이버네이트가 있는것과 같다.
Bean Validation을 구현한 기술중에 일반적으로 사용하는 구현체는 하이버네이트 Validator이다. 이름이 하이버네이트가 붙어서 그렇지 ORM과는 관련이 없다.
- `jakarta.validation-api` : Bean Validation 인터페이스
- `hibernate-validator` : 구현체

**검증은 어떤 순서로 해야 좋을까?**
1. @ModelAttribute 각각의 필드에 타입 변환 시도
	1. 성공하면 다음으로
	2. 실패하면 typeMismatch 로 FieldError 추가
2. Validator 적용
오류 메시지 변환은 마찬가지로 errors.properties 에서 LEVEL 별로 변경해주면 된다.
또한, ==오브젝트 오류는 @ScriptAssert 를 지원한다.== 그러나 오브젝트 오류(글로벌 오류)의 경우 `@ScriptAssert` 을 억지로 사용하는 것 보다는 직접 자바 코드로 작성하는 것을 권장한다.
추가로, Bean Validation은 RequestBody 에도 적용할 수 있다.
```java
@Slf4j
@RestController
@RequestMapping("/validation/api/items")
public class ValidationItemApiController {

    @PostMapping("/add")
    public Object addItem(
            @RequestBody @Validated ItemSaveForm form,
            BindingResult bindingResult) {

        log.info("API 컨트롤러 호출");

        if (bindingResult.hasErrors()) {
            log.info("검증 오류 발생 errors={}", bindingResult);
            return bindingResult.getAllErrors();
        }

        log.info("성공 로직 실행");
        return form;
    }
}
```

### 14. 동일한 모델 객체를 등록할 때와 수정할 때 각각 다르게 검증하려면?
두 가지 방법이 있다.
1. BeanValidation의 groups 기능을 사용한다. 
	```java
	package hello.itemservice.domain.item;

	import lombok.Data;
	import org.hibernate.validator.constraints.Range;
	
	import javax.validation.constraints.Max;
	import javax.validation.constraints.NotBlank;
	import javax.validation.constraints.NotNull;
	
	@Data
	public class Item {
	
	    @NotNull(groups = UpdateCheck.class) // 수정 시에만 적용
	    private Long id;
	
	    @NotBlank(groups = {SaveCheck.class, UpdateCheck.class})
	    private String itemName;
	
	    @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
	    @Range(
	        min = 1000,
	        max = 1000000,
	        groups = {SaveCheck.class, UpdateCheck.class}
	    )
	    private Integer price;
	
	    @NotNull(groups = {SaveCheck.class, UpdateCheck.class})
	    @Max(value = 9999, groups = SaveCheck.class) // 등록 시에만 적용
	    private Integer quantity;
	
	    public Item() {
	    }
	
	    public Item(String itemName, Integer price, Integer quantity) {
	        this.itemName = itemName;
	        this.price = price;
	        this.quantity = quantity;
	    }
	}
	
	@PostMapping("/add")
	public String addItemV2(
	        @Validated(SaveCheck.class) @ModelAttribute Item item,
	        BindingResult bindingResult,
	        RedirectAttributes redirectAttributes) {
	
	    // ...
	}
	```
2. Item을 직접 사용하지 않고, ItemSaveForm, ItemUpdateForm 같은 폼 전송을 위한 별도의 모델 객체를 만들어 사용한다. 
	==실무에선 보통 이 방법을 사용한다.== 이유는 등록시 폼에서 전달하는 데이터가 Item 도메인 객체와 딱 맞지 않기 때문이다. 그래서 보통 Item 을 직접 전달받는 것이 아니라, 복잡한 폼의 데이터를 컨트롤러까지 전달할 별도의 객체를 만들어서 전달한다. 

### 15. 서블릿 필터, 스프링 인터셉터
로그인 한 사용자만 진입 가능하게 하려면, 상품 관리 컨트롤러에서 로그인 여부를 체크하는 로직을 하나하나 작성하면 되겠지만 번거로우며, 향후 로그인과 관련된 로직이 변경될 때 마다 모든 로직을 수정해야 한다. 이렇게 애플리케이션의 여러 로직에서 공통으로 관심이 있는 것을 공통 관심사 라고 한다. 이러한 공통의 관심사는 스프링의 AOP 로도 해결할 수 있지만, 웹과 관련된 공통 관심사는 지금부터 설명할 서블릿 필터 또는 스프링 인터셉터를 사용하는 것이 좋다. 웹과 관련된 공통 관심사를 처리할 때는 HTTP의 헤더나 URL의 정보들이 필요한데, ==서블릿 필터나 스프링 인터셉터는 `HttpServletRequest`를 제공한다. ==
서블릿 필터는 서블릿이 지원하는 수문장이다. 서블릿 필터를 적용하면 필터가 호출 된 다음에 서블릿이 호출된다. 필터도 로직이다. 특정 URL 패턴에 적용할 수 있다. 필터는 체인으로 구성되는데, 중간에 필터를 자유롭게 추가할 수 있다. 예를 들어서 로그를  남기는 필터를 먼저 적용하고, 그 다음에 로그인 여부를 체크하는 필터를 만들 수 있다. 필터 인터페이스를 구현하고 등록하면, ==서블릿 컨테이너가 필터를 싱글톤 객체로 생성하고, 관리한다. ==
스프링 인터셉터도 서블릿 필터와 같이 웹과 관련된 공통 관심 사항을 효과적으로 해결할 수 있는 기술이다. 그러나 적용되는 순서와 범위 사용 방법이 다르다. 인터셉터가 더 많은 기능을 제공한다. 서블릿 필터가 서블릿이 제공하는 기술이라면, 스프링 인터셉터는 스프링 MVC 가 제공하는 기술이다. 
스프링 인터셉터는 스프링의 URL 경로에 매우 정교하게 패턴을 지정할 수 있다. 스프링이 제공하는 URL 경로는 서블릿 기술이 제공하는 URL 경로와 완전히 다르다. *`PathPatter`공식 문서 참고*
코드를 보면 알겠지만, 개발자 입장에서는 스프링 인터셉터가 훨씬 편리하다. 
```
HTTP 요청 → WAS → 필터 → 서블릿 → 스프링 인터셉터 → 컨트롤러
```
![[Pasted image 20261021003100.png|535]]
- 정상 흐름: `preHandle`이 핸들러 어댑터 호출 전에 호출되고, 응답값이 true 면 다음으로 진행하고 false면 진행을 멈춘다. (그림 1번에서 끝난다. ) `postHandle`은 컨트롤러 호출 후에 호출된다. `afterCompletion`은 뷰가 렌더링 된 이후에 호출된다.
- 예외 흐름: `preHandle`이 컨트롤러 호출 전에 호출되고, 만약 컨트롤러에서 예외가 발생하면 `postHandle`은 호출되지 않는다. `afterCompletion`은 항상 호출된다. 
==최종적으로는 ArgumentResolver를 활용하여 애노테이션으로 관리하는 것이 가장 편리하다.==
### 16. API 예외 처리
`BasicErrorController`를 확장하면 JSON 메시지도 변경할 수 있다. 그런데 API 오류는 `@ExceptionHandler`가 제공하는 기능을 사용하는 것이 더 나은 방법이다. HTML과는 달리 API 오류 처리는 각각의 컨트롤러나 예외마다 서로 다른 응답 결과를 출력해야 할 수 도 있다. 예를 들어서 회원과 관련된 API에서 예외가 발생할 때 응답과, 상품과 관련된 API에서 발생하는 예외에 따라 그 결과가 달라질 수 있다. 
스프링 MVC는 컨트롤러(핸들러) 밖으로 예외가 던져진 경우 예외를 해결하고, 동작을 새로 정의할 수 있는 방법을 제공한다. 컨트롤러 밖으로 던져진 예외를 해결하고, 동작 방식을 변경하고 싶으면 `HandlerExceptionResolver`를 사용하면 된다. 줄여서 `ExcetionResolver`라 한다. 
![[Pasted image 20261019021912.png|711]]



---
title: DispatcherServlet
date: 2025-09-19
categories:
  - spring
tags:
  - spring
---
### DispatcherServlet의 필요성과 동작 프로세스에 대한 설명을 합니다.
> DispatcherServlet은 클라이언트가 서버로 요청했을 경우, 해당 요청을 가장 먼저 받고, 적합한 컨트롤러를 찾아 요청에 대한 처리를 위임하는 프론트 컨트롤러이다. 과거 servlet의 경우 클라이언트 요청에 대한 URL 매핑을 직접 해야했으나,
> DispatcherServlet은 이런 번거로운 작업이나, 공통 작업 처리를 한다.

<!-- more -->
## DispatcherServlet?
- SpringMVC에서 핵심 Servlet역할을 담당하는 FrontController로써, 모든 HTTP요청을 중앙에서 제어하고 적절한 Controller로 분배하는 역할을 담당한다.
- 클라이언트로부터 어떤 요청이 들어올 경우, Tomcat과 같은 WAS영역내에 있는 서블릿컨테이너의 DispatcherServlet이 가장 먼저 요청 정보를 받게된다. DispatcherServlet은 기존 Servlet이 담당했던 공통적인 작업을 처리하고, 해당 요청을 처리해야되는 Handler(Controller)를 찾아 작업을 위임한다.
## 장점
과거 모든 서블릿을 URL을 위해 web.xml에 모두 등록하고 사용하여야 했으나, **DispatcherServlet이 해당 어플리케이션으로 들어오는 모든 요청을 핸들링**하고, **공통 작업을 처리**하면서 개발자는 편의성 있게 개발할 수 있게 되었다.
## 동작 구조
![](https://velog.velcdn.com/images/baekhk1006/post/36a0f7c3-967c-44bf-bb05-2dd623e20ec3/image.png)
### 1. 클라이언트의 요청을 DispatcherServlet이 받음
DispatcherServlet은 가장 먼저 요청을 받는 프론트 컨트롤러이다. Servlet Context(Web Context)에서 필터(Filter Class)를 지나 Spring Context영역에서 DispatcherServlet이 가장 먼저 요청을 받게 된다.
![](https://velog.velcdn.com/images/baekhk1006/post/3de09622-6ef3-4b0c-8323-07d1c5222ef0/image.png)
### 2. 요청 정보를 통해 요청을 위임한 핸들러(컨트롤러)를 찾음
DispatcherServlet은 요청을 처리할 핸들러를 찾고, 해당 객체의 메소드를 호출한다. 따라서 요청된 정보에 대해 처리할 수 있는 핸들러가 어떤 해들러인지 식별해야 하는데, 해당 역할을 HandlerMapping이 담당한다.

@Controller에 @RequestMapping관련 어노테이션을 사용하여, 핸들러(컨트롤러)를 작성하는 것이 일반적이며, @Controller방식은 RequestMappingHandlerMappging이 처리한다.

RequestMappingHandlerMappging는 @Controller로 작성된 모든 핸들러를 HashMap 형태로 <요청정보, 처리할 대상> 관리한다. 여기서 처리할 대상은 HandlerMethod객체로 컨트롤러, 메소드 등의 정보를 갖고 있는데, 이는 스프링이 리플렉션을 이용하여 위암하기 때문이다.

요청이 오면 HTTPMethod, URI 등을 사용해 요청정보를 만들고, HashMap에 요처을 처리할 대상을 찾은 후 HandlerExecutionChain으로 Wrapping하여 반환한다.HandlerExecutionChain으로 Wrapping하는 이유는 컨트롤러로 요청을 넘겨주기 전에 처리해야하는 **Interceptor 등을 포함하기 위해서**이다.
### 3. 요청을 핸들러(컨트롤러)로 위임한 핸들러 어댑터를 찾아서 전달
이후, 핸들러(컨트롤러)에 요청을 위임해야하는데, DispatcherServlet은 핸들러로 요청을 직접 위임하는 것이 아니라, HandlerAdapter를 통해 위임한다.
- 컨트롤러의 구현 방식이 다양하기 때문에 Adapter패턴을 이용하여 DispatcherServlet과 Handler 사이에 HandlerAdapter를 두어 위임
### 4. 핸들러 어댑터가 핸들러로 요청을 위임한다.
HandlerAdapter는 Handler(컨트롤러)로 요청을 위힘하기 **전/후로 공통적인 처리 과정이 필요하다.**
- 전처리 과정
    - Interceptor 처리
        - HandlerAdapter가 Handler(컨트롤러)로 요청을 전달하기전, **Interceptor**를 이용해 공통 작업을 할 수 있다.
        - 예) 인증/인가 체크, 로깅, 요청 정보 수정 등
    - ArgumentResolver 사용
        - 컨트롤러의 메소드 파라미터를 적절한 객체로 변환
        - 예)
            - @RequestParam: URL 쿼리 파라미터를 메소드의 인자로 변환
            - @RequestBody: JSON 요청 본문을 객체로 변환
- 핸들러(컨트롤러) 호출
    - HandlerAdapter는 적절한 핸들러 메소드를 찾아 호출
- 후처리 과정
    - ReturnValueHandler 사용
        - 컨트롤러의 반환 값을 처리하여 적절한 응답 방식으로 변환
            - ResponseEntity객체 타입을 HTTP 응답으로 변환
            - 반환 데이터를 JSON 타입으로 직렬화
### 5. 핸들러 내부에서 비즈니스 로직을 처리한다.
요청을 위임받은 Handler는 Service를 호출하고, 개발자가 작성한 비즈니스 로직들이 실행된다.
### 6. 핸들러는 로직 처리이후 반환 값을 반환한다.
요청을 위임받은 Handler(컨트롤러)는 서비스를 호출하고 개발자가 작성한 비즈니스 로직을 실행한다.

응답 데이터를 사용하는 경우에 주로 ResponseEntity를 반환하게 되어 있고, 응답 페이지를 보여주는 경우 String으로 View의 이름을 반환할 수 있다.
### 7. 핸들러 어댑터가 반환 값을 처리한다.
HandlerAdpater는 Handler(컨트롤러)로 부터 받은 응답을 응답 처리기인 ReturnValueHandler가 후처리한 후에 DispatcherServlet으로 반환한다.

만약 Handler가 ResponseEntity를 반환할 경우 HttpEntityMethodProcessor가 MessageConvter를 통해 응답 객체를 직렬화하고 HttpStatus를 설정한다.
그리고 Handler가 View이름을 반환할 경우 ViewResolver를 통해 View를 반환한다.
### 8. 서버의 응답을 클라이언트로 반환한다.
DispatcherServlet을 통해 반환되는 응답은 다시 필터들을 거쳐 클라이언트에게 응답한다.
이때, 응답이 데이터일 경우 그대로 반환되지만, 응답이 View일 경우 이름에 맞는 View를 찾아서 반환해주는 ViewResolver가 적절한 화면을 내려준다.
---
layout: post
title: "[가지마켓] #6 회원과 상품 연관관계 매핑"
date: 2021-02-11 00:50:59
author: me
categories: Project
tags: gagi
cover: "/assets/instacode.png"
---


> 중고 거래 플랫폼 **가지마켓 프로젝트** 의 진행상황 및 이슈를 공유합니다.<br/>
> 프로젝트 저장소는 [이곳](https://github.com/GagiMarket/gagi)에 확인할 수 있습니다.


## 목차
* 회원과 상품 연관관계 매핑
  + 회원 로그인, 권한 확인하기 
* 로그인 체크 중복 로직 Interceptor 처리
* 테스트 코드 작성(세션 테스트)
* References

## 회원과 상품 연관관계 매핑
회원과 상품은 다음과 같은 연관관계를 가지고 있습니다.

* 회원은 여러개의 상품을 등록할 수 있다.
* 권한을 가진 회원은 상품을 수정할 수 있다.
* 권한을 가진 회원은 상품을 삭제할 수 있다.

따라서, 회원과 상품은 **1:N 관계**에 있습니다. 그리고 연관관계의 주인은 1:N의 N인 상품(item)으로 하였습니다.

```java
/*
 * Member Class
 */
package com.gagi.market.member.domain;
public class Member {
    ...(생략)...
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_id")
    private Long memberId;

    @OneToMany(mappedBy = "member")
    private List<Item> items = new ArrayList<>();
}
/*
 * Item Class
 */
package com.gagi.market.item.domain;
public class Item {
    ...(생략)...
    @ManyToOne
    @JoinColumn(name = "member_id")
    private Member member;

    //==연관관계 메소드==//
    public void setMember(Member member) {
        this.member = member;
        member.getItems().add(this);
    }
}
```

#### 회원 로그인, 권한 확인하기
상품 등록, 수정, 삭제 시 **사용자의 로그인 여부와 상품 조작 권한을 확인**해야합니다.

사용자 로그인 확인은 이전 게시글 [#5 회원 도메인 구현과 리팩토링](https://doorisopen.github.io/project/2021/01/29/gagi-project-member-domain-implementation-and-refactoring.html)에서 구현한 세션값을 가져오는 어노테이션 `@LoginMember` 으로 로그인 여부를 확인할 수 있게 하였습니다.

```java
@PostMapping
public ResponseEntity<ItemResponseDto> createItem(@LoginMember SessionMember member, @RequestBody ItemRequestDto requestDto) {
    if (member == null) { // 로그인 여부 확인
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
    }
    Item item = itemService.createItem(requestDto.toEntity(), member.getMemberEmail());
    return ResponseEntity
            .created(URI.create(ITEM_API_URI + "/" + item.getItemId()))
            .body(ItemResponseDto.of(item));
}
```

그리고 **상품 수정, 삭제시 필요한 상품 조작 권한 확인**은 사용자의 세션값을 활용해서 사용자 정보를 조회하고 등록된 상품의 권한과 일치하는지 비교하여 권한을 확인하였습니다.

```java
package com.gagi.market.item.service;

@Transactional
@Service
public class ItemService {
    public boolean checkPermissionOfItem(Long itemId, String memberEmail) {
        Member findMember = memberRepository.findMemberByMemberEmail(memberEmail).orElse(null);
        Item findItem = itemRepository.findById(itemId).orElse(null);
        return findItem
                .getMember()
                .getMemberEmail()
                .equals(findMember.getMemberEmail());
    }
}
```

## 로그인 체크 중복 로직 Interceptor 처리
상품 등록, 수정, 삭제 그리고 로그아웃 등의 기능을 수행할때 전제되어야 하는 조건이 로그인이 되어있어야 한다는 것입니다. 그러나 다음과 같이 불필요하게 **로그인 여부를 확인하는 중복 코드가 발생**하여 이를 개선해보겠습니다.

```java
@PostMapping
public ResponseEntity<ItemResponseDto> createItem(@LoginMember SessionMember member, @RequestBody ItemRequestDto requestDto) {
    if (member == null) { // 로그인 여부 확인
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
    }
    ...(생략)
}
```

로그인 확인 등 중복을 없애기 위한 방법을 찾아보니 2가지가 있었습니다.

* AOP로 로그인 확인하기
* Interceptor로 로그인 확인하기

처음에 로그인 여부를 확인하는 중복 코드를 없애기 위해 AOP로 처리하려고 했습니다. 그러나 일반적으로 AOP는 Controller 에서 처리하지 않고 Interceptor로 처리하는 것이 일반적이라는 것을 보고 해당 방법으로 적용하였습니다.

<a href="{{ site.2021_project_img }}/project-gagi-application-request-and-response-flow.png" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.2021_project_img }}/project-gagi-application-request-and-response-flow.png" title="Check out the image">
</a>

그리고, 위의 그림에서 **filter, interceptor, aop**의 위치를 보면 Dispatcher Servlet 호출 이후에 호출되는 interceptor에서 처리하는것이 적절해 보였습니다.

Interceptor를 만들기 위해서 **HandlerInterceptor 인터페이스를 구현**하는 방법과 **HandlerInterceptorAdapter를 상속**하는 방법 2가지가 있습니다. 

필자는 **HandlerInterceptor 인터페이스의 preHandle 메소드를 구현한 LoginCheckInterceptor 클래스를 생성**하여 Interceptor를 구현하였습니다.(HandlerInterceptor 인터페이스의 메소드들은 default 메소드이기 때문에 전부 Override할 필요가 없고 필요한 것만 구현하면 됩니다.)

```java
package com.gagi.market.config.auth;
...(생략)
public class LoginCheckInterceptor implements HandlerInterceptor {
    public List<String> pathPatters = Arrays.asList(
            "/api/v1.0/items/**",
            "/api/v1.0/members/**");
    public List<String> excludePathPatters = Arrays.asList(
            "/api/v1.0/items/search",
            "/api/v1.0/members/login",
            "/api/v1.0/members/duplicated");

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        SessionMember session_member = (SessionMember) request.getSession().getAttribute("SESSION_MEMBER");
        if (isGet(request)) {
            return true;
        }
        if (session_member == null) {
            response.setStatus(401);
            response.setHeader("message", "UnAuthorized!!");
            return false;
        }
        return true;
    }

    private boolean isGet(HttpServletRequest request) {
        return (request.getRequestURI().equals("/api/v1.0/items")) && (request.getMethod().equals("GET")) ||
                (request.getRequestURI().equals("/api/v1.0/members")) && (request.getMethod().equals("GET"));
    }
}
```

위의 코드를 살펴 보면, 우선 멤버 변수 `pathPatters`는 **로그인을 검증할 URL 패턴 리스트** 입니다. 그리고 `excludePathPatters`는 **검증에서 제외할 패턴 리스트**입니다. 이는 이후 WebMvcConfigurer에 위의 interceptor를 등록할때 URL 패턴을 변수로 관리하기 위함입니다.

**preHandle() 메소드**는 컨트롤러에 요청이 넘겨지기 이전에 호출되는 메소드 입니다.


참고로 **인터셉터가 처리하는 패턴을 동일한 URL의 HTTP Method GET, POST를 구분할 수 가 없습니다.** 다시말해 `/api/v1.0/items` 해당 URL의 GET Method는 인터셉터 처리를 제외하고 POST는 인터셉터가 처리하도록 구분할 수 없다는 의미입니다. 그래서 isGet() 메소드를 구현하여 예외 허용 처리를 하였습니다.

그리고 위의 인터셉터를 아래와 같이 **WebMvcConfigurer를 구현한 WebConfig 클래스에 등록**해줍니다.

```java
package com.gagi.market.config;
...(생략)
@Configuration
public class WebConfig implements WebMvcConfigurer {

    ...(생략)
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        LoginCheckInterceptor loginCheckInterceptor = new LoginCheckInterceptor();
        registry.addInterceptor(loginCheckInterceptor)
                .addPathPatterns(loginCheckInterceptor.pathPatters)
                .excludePathPatterns(loginCheckInterceptor.excludePathPatters);
    }
}
```

## 테스트 코드 작성(세션 테스트)
회원과 상품의 연관관계를 연결한 상태에서 API 테스트하기 위해서는 사용자의 로그인이 되어있어야 합니다. 그래서 `MockHttpSession`을 이용해서 JUnit에서 테스트를 수행하였습니다. 

```java
protected MockHttpSession session;

@BeforeEach
public void setup() {
    mvc = MockMvcBuilders.webAppContextSetup(context)
            .build();
    Member member = memberRepository.save(Member.builder()
            .memberEmail("test@gagi.com")
            .memberPw("test")
            .memberAddress("가지특별시 가지동")
            .memberPhoneNumber("010-1234-5678")
            .build());
    session = new MockHttpSession();
    session.setAttribute("SESSION_MEMBER", new SessionMember(member));
}
```

위와 같이 session 정보를 가진 Mock 객체를 생성해줍니다. 그리고 다음과 같이 session 멤버 변수를 활용하여 테스트를 수행하였습니다.

```java
package com.gagi.market.item.api;
...(생략)
@Transactional
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ItemApiControllerTest {

    ...(생략)
    @BeforeEach
    public void setup() {
        mvc = MockMvcBuilders.webAppContextSetup(context)
                .build();
        Member member = memberRepository.save(Member.builder()
                .memberEmail("test@gagi.com")
                .memberPw("test")
                .memberAddress("가지특별시 가지동")
                .memberPhoneNumber("010-1234-5678")
                .build());
        session = new MockHttpSession();
        session.setAttribute("SESSION_MEMBER", new SessionMember(member));
    }

    ...(생략)

    @DisplayName("회원은 상품 등록을 성공한다.")
    @Test
    public void memberSuccessInCreateItem() throws Exception {
        //given
        String url = LOCALHOST_URI + port + ITEM_API_URI;
        ItemRequestDto requestDto = ItemRequestDto.builder()
                .itemName("m1 맥북 프로")
                .itemDescription("2021 신형 애플 노트북")
                .itemCategory("노트북")
                .itemPrice(10000)
                .itemLocation("강남역")
                .build();
        //when
        mvc.perform(
                post(url)
                        .contentType(MediaType.APPLICATION_JSON)
                        .session(session)
                        .content(new ObjectMapper().writeValueAsString(requestDto)))
                .andExpect(status().isCreated());
        //then
        List<Item> list = itemRepository.findAll();
        assertThat(list.size()).isEqualTo(1);
        assertThat(list.get(0).getItemName()).isEqualTo("m1 맥북 프로");
        assertThat(list.get(0).getItemCategory()).isEqualTo("노트북");
    }

    @DisplayName("비회원은 상품 등록을 실패한다.")
    @Test
    public void nonMemberFailInCreateItem() throws Exception {
        //given
        String url = LOCALHOST_URI + port + ITEM_API_URI;
        ItemRequestDto requestDto = ItemRequestDto.builder()
                .itemName("m1 맥북 프로")
                .itemDescription("2021 신형 애플 노트북")
                .itemCategory("노트북")
                .itemPrice(10000)
                .itemLocation("강남역")
                .build();
        //when
        mvc.perform(
                post(url)
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(new ObjectMapper().writeValueAsString(requestDto)))
                .andExpect(status().isUnauthorized());
        //then
    }
}
```

## References
* [MockHttpSession](https://shinsunyoung.tistory.com/70)
* [HandlerInterceptor1](https://taetae0079.tistory.com/18)
* [HandlerInterceptor2](https://ecsimsw.tistory.com/entry/Spring-Interceptor-Spring-boot)
* [facade pattern](https://imasoftwareengineer.tistory.com/29)
* [WebTestClient1](https://www.baeldung.com/spring-5-webclient)
* [WebTestClient2](https://pjh3749.tistory.com/257)
* [WebTestClient3](https://expert0226.tistory.com/376)
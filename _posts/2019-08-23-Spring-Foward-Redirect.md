---
layout: post
title:  "[Spring] 7 - Forward와 Redirect"
date:   2019-08-23 9:59:40
author: me
categories: Spring
tags: Spring Framework web Forward Redirect
cover:  "/assets/images/Spring/springcover.png"
---

<br />
<br />

>> 해당 게시글의 전체 소스코드는 이곳 <a href="https://github.com/doorisopen/SpringSpring/tree/3ecbbf611fbc64dd06ea8f88e2257b88ae3323c3"><strong>Github</strong></a>를 참고해 주세요

<br />

### 목차
>> 1. __Forward와 Redirect 동작 방식 이해__

<br />
<br />

<hr />

## Forward와 Redirect
* __src/test/java/urlController.java 내용__

### Forward
* __Forward는 서버 내 페이지 전환__
__실생활 예시로 들면,__ <br />
1. 철수는 과자를 사러 집앞 A 편의점을 방문했다
2. 편의점을 갔는데 내가 원하는 과자를 찾지 못해서 편의점 직원에게 도움을 요청
3. 직원은 내가 원하는 과자를 창고에서 찾아서 줬다

<a href="{{ site.spring_img }}/spring_fwd_rdt_1.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_fwd_rdt_1.JPG" title="Check out the image">
</a>

```
// Forward TEST
@RequestMapping(value="/tryFwd", method = RequestMethod.GET)
public String getUserTest4( @ModelAttribute("msg") String msg ) {
	logger.info(msg);
	logger.info(" /tryFwd URL called. then getUserTest4 method executed.");
	return "forward:/tryB";
}
```

<br />

### Redirect
* __Redirect는 브라우저를 경유한다__
__실생활 예시로 들면,__ <br />
1. 철수는 과자를 사러 집앞 A 편의점을 방문했다
2. 편의점에 들어가니 직원이 다음과 같이 말했다. "현재 편의점이 리모델링 중이니 B 편의점으로 가주세요"
3. 철수는 B 편의점으로 가서 과자를 산다

<a href="{{ site.spring_img }}/spring_fwd_rdt_2.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.spring_img }}/spring_fwd_rdt_2.JPG" title="Check out the image">
</a>

```
// Redirect TEST
@RequestMapping(value="/tryRdt", method = RequestMethod.GET)
public String getUserTest5( @ModelAttribute("msg") String msg ) {
	logger.info(msg);
	logger.info(" /tryRdt URL called. then getUserTest5 method executed.");
	return "redirect:/tryB";
}
```

<hr />

## 결론
* URL의 변화여부가 필요하다면 __Redirect__ 를 사용하는 것이 좋다
* 객체를 재사용하거나 공유해야한다면 __Forward__ 를 사용하는 것이 좋다


<hr />

### References
* 대학교 웹 프레임워크 수업 정리한 내용 입니다.

> * <a href="https://developer.mozilla.org/ko/docs/Web/HTTP/Redirections">https://developer.mozilla.org/ko/docs/Web/HTTP/Redirections<a>
> * <a href="https://nesoy.github.io/articles/2018-04/Redirect-Forward">https://nesoy.github.io/articles/2018-04/Redirect-Forward<a>
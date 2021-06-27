---
layout: post
title:  "[Spring] POJO(Plain Old Java Object)"
date:   2019-09-05 10:34:40
author: me
categories: Spring
tags: Spring Framework POJO
cover:  "/assets/images/Spring/springcover.png"
---

<br />
<br />

### 목차
>> 1. __POJO 이해 하기__

<br />
<br />


<hr />

POJO = Java Bean <br />

여기서 Java Bean은 Sun의 Java Beans나 EJB의 Bean을 뜻하는 것이 아닌 <br />

순수하게 setter, getter 메소드로 이루어진 Value Object성의 Bean을 의미 <br />


예를 들면 이와 같은 코드이다 <br />

```
public class SimpleBean {
    private String name;
    private String age;
 
    public void setName(String name) {
        this.name = name;
    }
    public String getName() {
        return this.name;
    }
 
    public void setAge(String age) {
        this.age = age;
    } 
    public String getAge() {
        return this.age;
    }
 
}
```

즉, 이클립스를 통해 자동으로 생성하던 깡통 빈 클래스를 통해서 생성된 객체 <br />


그것이 바로 스프링에서 말하는 POJO인 것이다 <br />

그렇다면 POJO가 스프링의 중요한 특징 중에 하나인 이유는 무엇인가? <br />


그것은 클래스 상속을 강제하지 않고, 인터페이스 구현을 강제하지 않으며, 애노테이션 사용을 강제하지 않는다 <br />


즉 개발자는 특정한 라이브러리나 컨테이너의 기술에 종속적이지 않고, 가장 일반적인 형태로 코드를 작성할 수 있다는 것이다 <br />

이것은 생산성에도 유리하고, 코드에 대한 테스트 작업 역시 좀 더 유연하게 할 수 있다는 장점이 생긴다 <br />
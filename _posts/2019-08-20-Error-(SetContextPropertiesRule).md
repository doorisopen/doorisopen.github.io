---
layout: post
title:  "[ERROR] [SetContextPropertiesRule]{Context} Setting property 'source' to 'org.eclipse.jst.jee.server:project' did not find a matching property"
date:   2019-08-20 18:20:59
author: me
categories: Error
tags: Spring Error Web Server Tomcat
cover:  "/assets/images/Spring/springcover.png"
---


Tomcat 서버를 돌리다가 아래와 같은 경고 메시지 받았다 <br />

[ERROR] [SetContextPropertiesRule]{Context} Setting property 'source' to 'org.eclipse.jst.jee.server:project' did not find a matching property <br />


구글링 해본 결과 다른 블로그 에서 <br />
Servers Tab 에서 <br />

Tomcat v6.0 Server at localhost(서버중지 시키고) 더블클릭하면 아래와 같이 나오고 <br />

Server Options 에서 <br />

publish module context to separate XML files를 체크해주고 저장하면 <br />

해결 된다고 했지만 나의 경우는 해결되지 않았고 아래와 같은 경고 메시지 2개를 동시에 받았다 <br />

<a href="{{ site.error_img }}/spring_error1.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.error_img }}/spring_error1.JPG" title="Check out the image">
</a>

<br />

<a href="{{ site.error_img }}/spring_error2.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.error_img }}/spring_error2.JPG" title="Check out the image">
</a>

```
경고: [SetContextPropertiesRule]{Context} Setting property 'source' to 'org.eclipse.jst.jee.server:project명' did not find a matching property.
```

<br />

```
경고: The path attribute with value [/myspring] in deployment descriptor[C:\spring-tool-suite-4-4.1.2.RELEASE-e4.10.0-win32.win32.x86_64\workspace\.metadata\.plugins\org.eclipse.wst.server.core\tmp2\conf\Catalina\localhost\myspring.xml] has been ignored
```


또한, <br />

프로젝트 우클릭 properties -> Deployment Assembly -> Java Build Path Entries 추가 -> 프로젝트 Clean <br />

이 방법 으로도 해결되지 않았다. <br />

좀더 찾아본결과 ... <br />

<hr />

* __출처 https://powerhan.tistory.com/182__

Eclipse에서 Tomcat 구동시 로그에 위와같은 경고가 나오는 것은 "정상"이라는 글을 보았다. 내용은 아래와 같다. <br />


정상적인인 배포 환경에서는 나오지 않지만, Eclipse 안에서 Tomcat Server를 구동하는 경우에는 Eclipse의 WTP 플러그인이 임의로 source 태그를 집어넣기 때문에 나오는 경고메세지 입니다. <br />



더 정확히 얘기하면, Project Explorer의 Server에서 server.xml을 열어보면 하단에 아래와 같은 설정이 있는데요, <br />

(Server에 webapp을 추가한 경우에만 생깁니다.) <br />

```
<Context docBase="test-dynamic-web-project" path="/test-dynamic-web-project" reloadable="true"  
<!-- 주석 아래 -->
source="org.eclipse.jst.jee.server:test-dynamic-web-project"/>
```

Tomcat 6 부터는 server.xml에 정의되지 않는 속성이 있는 경우 WARNING을 뱉어내도록 바꼈습니다. 위에서 __주석 아래__ 부분이 WTP에서 임의로 추가한 속성입니다. <br />


'Servers' view > Tomcat v7.0 Server at localhost 더블클릭 > Server Options > 'Publish module contexts to separate XML files' check 하면 해결된다는 사람도 있는데요, 그래도 WARNING은 나옵니다. <br />

라고 한다...

<hr />


아래와 같은 단순한 방법으로 경고 메시지가 사라졌었지만 <br />

다음 날(2019-08-22) 확인해보니 똑같이 __경고 메시지가 출력되는것을 보았다..__ <br />

__publish module context to separate XML files를 체크를 해제__ 한 상태여야 한다 <br />

<a href="http://stackoverflow.com/questions/7753409/importing-dynamic-web-project-in-eclipse/7754620#7754620">http://stackoverflow<a>


> 1. Remove project from Tomcat (rightclick Tomcat, Add/Remove project, remove project)
> 2. Close project in Eclipse (rightclick project, Close)
> 3. Clean Tomcat (rightclick Tomcat, Clean)
> 4. Reopen project in Eclipse (rightclick project, Open)
> 5. Clean project in Eclipse (Project > Clean... > Clean selected projects below, select project)
> 6. Add project to Tomcat (rightclick Tomcat, Add/Remove projects, add project)
> 7. Start Tomcat (rightclick Tomcat, Start).

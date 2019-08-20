---
layout: post
title:  "[ERROR] [SetContextPropertiesRule]{Context} Setting property 'source' to 'org.eclipse.jst.jee.server:project' did not find a matching property"
date:   2019-08-20 18:20:59
author: me
categories: Error
tags: Spring Error Web Server Tomcat
cover:  "/assets/images/Spring/springcover.png"
---

Tomcat 서버를 돌리다가 아래와 같은 경고 메시지 2개를 동시에 받았다

<a href="{{ site.error_img }}/spring_error1.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.error_img }}/spring_error1.JPG" title="Check out the image">
</a>

<br />

<a href="{{ site.error_img }}/spring_error2.JPG" data-lightbox="falcon9-large" data-title="Check out the image">
  <img src="{{ site.error_img }}/spring_error2.JPG" title="Check out the image">
</a>

__경고: [SetContextPropertiesRule]{Context} Setting property 'source' to 'org.eclipse.jst.jee.server:project명' did not find a matching property.__
<br />
__경고: The path attribute with value [/myspring] in deployment descriptor[C:\spring-tool-suite-4-4.1.2.RELEASE-e4.10.0-win32.win32.x86_64\workspace\.metadata\.plugins\org.eclipse.wst.server.core\tmp2\conf\Catalina\localhost\myspring.xml] has been ignored__

구글링 해본 결과 다른 블로그 에서 <br />
Servers Tab 에서 <br />

Tomcat v6.0 Server at localhost(서버중지 시키고) 더블클릭하면 아래와 같이 나오고 <br />

Server Options 에서 <br />

publish module context to separate XML files를 체크해주고 저장하면 해결. <br />

된다고 했지만 나의 경우는 해결되지 않았다. <br />

또한, <br />

프로젝트 우클릭 properties -> Deployment Assembly -> Java Build Path Entries 추가 -> 프로젝트 Clean <br />

이 방법 으로도 해결되지 않았다. <br />

그러나 아래와 같은 단순한 방법으로 해결했다 <br />

__publish module context to separate XML files를 체크를 해제__ 한 상태여야 한다 <br />

<a href="http://stackoverflow.com/questions/7753409/importing-dynamic-web-project-in-eclipse/7754620#7754620">http://stackoverflow.com/questions/7753409/importing-dynamic-web-project-in-eclipse/7754620#7754620<a>

> 1. Remove project from Tomcat (rightclick Tomcat, Add/Remove project, remove project)
> 2. Close project in Eclipse (rightclick project, Close)
> 3. Clean Tomcat (rightclick Tomcat, Clean)
> 4. Reopen project in Eclipse (rightclick project, Open)
> 5. Clean project in Eclipse (Project > Clean... > Clean selected projects below, select project)
> 6. Add project to Tomcat (rightclick Tomcat, Add/Remove projects, add project)
> 7. Start Tomcat (rightclick Tomcat, Start).
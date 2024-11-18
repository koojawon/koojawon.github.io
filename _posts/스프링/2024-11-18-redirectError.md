---
title: "Aws ALB를 사용시 Spring OAuth2.0 redirection URL Scheme이 http일 때!!"
excerpt: ""
categories:
  - SpringSecurity
  - Spring
  - AWS
toc: true
toc_sticky: true

---

## 문제 상황!

AWS ALB를 이용해 https 요청을 http프로토콜로 전환해 스프링이 있는 백엔드 서버로 전달하고 있었다.

이때 구글에서 OAuth2.0 성공 시 redirection url은 https://{uri}/api/login/oauth2/code/google로 설정해둔 상태이나, 스프링에서 http://{uri}/api/login/oauth2/code/google로 리디렉션 시키려 하였다!



### 원인!

스프링은 SSL을 적용하지 않아 Http 프로토콜을 사용하고 있었다. 이때, ALB에서 X-Forwarded-For와 X-Forwarded-Proto, X-Forwarded-Port 헤더를 보내 주는데, 이것이 클라이언트(유저)가 LB에 연결할 때 사용한 ip/protocol/port이다. 하지만 스프링이 기본적으로 이를 사용하지 않고 있었다!!



### 해결방법

application-properties.yml에

``` yaml
server:
	forward-headers-strategy: NATIVE
```

를 작성한다!!!

forward-headers-strategy를 native로 설정하면, 톰캣의 자동 설정을 따라 처리하며, 일반적으로 사용하는 X-Forwarded-For 와 X-Forwarded-Proto 헤더를 redirection URL을 생성할 때 사용한다.

이때, https에서 ALB를 타고 온 요청은 proto헤더가 https이므로 oauth redirection url을 생성할 때에도 https 프로토콜을 사용하게 된다.

(FRAMEWORK옵션을 이용하면, ForwardedHeaderFilter를 직접 커스텀하여 사용할 수 있다.)
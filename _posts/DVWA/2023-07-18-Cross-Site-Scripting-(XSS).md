---
title: "XSS"
categories: DVWA
toc: true  
toc_sticky: true 
---

## Groot Security Study - Week 8
 - DVWA의 내용들에 대해 직접 실습하고 그에 대한 보고서를 정리하는 블로그

### 개요
  9주차는 XSS(Cross-Site Scripting), 그 중에서도 Stored 방식에 관한 내용이다. XSS는 공격자가 악성 코드/스크립트 등을 웹사이트를 통하여 유포 할 수 있는 공격을 말한다. 이를 이용해 공격자가 일반 사용자의 쿠키/세션등을 탈취할 수 있고 비정상적인 행동을 유발할 수 있다. 주로 다른 웹사이트와 정보를 교환하는 방식을 이용해서 사이트 간(Cross-Site) 이라는 이름이 붙어있다.<br/>
  이중 Stored 방식은 공격자가 악성 코드/스크립트를 공격 대상인 웹 사이트에 상시적으로 유지되게 하여(Stored) 해당 웹사이트에 접근하는 일반 사용자들에 대해 자동적으로/지속적으로 공격을 수행할 수 있다.<br/>
  <center><img src="/assets/230718/Stored_XSS.png" width="50%" height="50%" alt="XSS_Diagram"></center><br/>
  실습은 DVWA를 이용 하였으며, 실습 환경은 작성자의 가상머신에서 이루어졌다(ARM / VMWare Fusion - Kali Linux)<br/>
  
### 본론
**실습 과정**
  1. Low<br/>
  2. Medium<br/>
  3. High<br/>

### 결론
  1. **원인 분석**<br/>

  2. **예상 대응 방안**<br/>
  
### 마치며
  원래 진행대로라면 DOM(Document Object Model) 기반 시스템의 XSS공격에 대한 내용을 다루었어야 했으나, 무슨 이유인지 쿠키 내용을 조회할 때 session ID가 출력되지 않아서(Burp Suite에서 조회 되기는 함) 다른 기반의 XSS공격에 관한 내용을 먼저 다루었다.

### 참조
  * [Cross-site scripting](https://en.wikipedia.org/wiki/Cross-site_scripting)
  * [Cross Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/)
  * [Owasp Top 10](https://owasp.org/www-project-top-ten/)
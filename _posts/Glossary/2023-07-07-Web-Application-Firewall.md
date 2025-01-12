---
title: "Web Application Firewall"
categories: Glossary
toc: true  
toc_sticky: true 
---

## Additional Knowledge
 - Groot Security 스터디를 진행하면서 각 주차별 학습 내용 이외에 기록해 놓으면 좋을 정보들

### 개요
  <center><img src="/assets/230707/diagram_of_waf.png" width="100%" height="100%" alt="Diagram_of_WAF"></center><br/>
  WAF(Web Application Firewall)은 일반적인 네트워크 방화벽과는 다르게 웹 애플리케이션 보안에 특화되어 있는 솔루션이다.

### 본론
  **구성 요소**<br/>
  1. 정책
      - 원본 관리, 보호 규칙 설정, 봇 감지 기능을 비롯한 전반적인 WAF 서비스 구성
  2. 원본
      - 보호 규칙 또는 기타 기능을 설정하도록 설계된 웹 애플리케이션의 원본 호스트 서버
  3. 보호 규칙
      - 지정된 보호 규칙 기준을 충족할 때 네트워크 요청을 허용, 차단 또는 기록하도록 구성
      - 시간 경과에 따라 웹 애플리케이션에 대한 트래픽을 관찰하고 적용할 새 규칙을 제안
  4. 봇 관리
      - 웹 애플리케이션에 대해 식별된 봇 트래픽을 감지하여 차단하거나 허용
      - JavaScript 인증 질문, CAPTCHA 인증 질문, GoodBot 허용 목록이 포함
      - IP 비율 제한, CAPTCHA, 장치 핑거프린팅 및 인간 상호 작용 인증 질문과 같은 감지 기술을 사용하여 웹 애플리케이션에서 의심스러운 봇 활동을 식별하고 차단
        - 게시된 봇 제공자의 합법적인 봇 트래픽을 허용하여 이러한 제어를 무시할 수도 있음

  **기능**<br/>
  1. DNS(Domain Name System)를 통한 동적 트래픽 라우팅
      - 수천 개에 이르는 전역 위치에서 사용자 대기 시간을 고려하는 DNS 기반 트래픽 라우팅 알고리즘을 활용하여 대기 시간이 가장 짧은 경로를 파악
  2. WAF 서비스의 고가용성
      - 여러 원본 서버를 추가할 수 있는 여러 가지 고가용성 구성 옵션을 제공
      - 주 원본 서버가 오프라인 상태이거나 상태 검사에 올바르게 응답하지 않는 경우에 사용
  3. 유연한 정책 관리 방법
      - 조직의 요구 사항을 해결하기 위해 기능을 구성하고 관리
  4. 모니터링 및 보고
      - 규정 준수 및 분석을 위해 콘텐츠 라이브러리와 관련된 보고에 액세스할 수 있는 기능을 사용자에게 제공
  5. 에스컬레이션
      - 지원 팀이 긴급성에 따라 티켓을 발행하고 에스컬레이션할 수 있는 기능을 제공

### 마치며
  크게 어려운 내용 없이 기존의 네트워크 방화벽을 각종 웹 애플리케이션에 알맞게 최적화 시킨 버전이라고 생각하면 편할 것 같다. 특히 요즘 클라우드 관련 기술을 이용하여 서비스가 이루어지는 조직이 많다보니 클라우드 기반의 WAF도 수요가 올라가고 있는 것으로 보인다.

### 참조
  * [WAF란 무엇일까?(Akamai)](https://www.akamai.com/ko/glossary/what-is-a-waf)
  * [WAF란 무엇일까?(Oracle)](https://www.oracle.com/kr/security/cloud-security/what-is-waf/)
  * [웹방화벽이란?](https://www.pentasecurity.co.kr/web-application-firewall/)
  * [AWS WAF](https://aws.amazon.com/ko/waf/)
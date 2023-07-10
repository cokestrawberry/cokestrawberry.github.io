# Groot Security Study - Week 8

## DVWA의 내용들에 대해 직접 실습하고 그에 대한 보고서를 정리하는 블로그

### 개요
  8주차는 Weak Session IDs라는 주제이다. 우선 session ID가 무엇인지 살펴보면 우리가 서버에 연결할때 수많은 사용자들 사이에서 각각을 구분하기 위해 정보(쿠키)를 생성하여 사용자가 저장하게 되는데, 이때 외부에 노출되면 안되는 정보들에 대해서는 session ID를 발급하여 사용자가 아닌 서버쪽에서 저장하게 된다. 이때 session ID를 발급하는 기준이나 방법이 예측 가능하다면(weak) 다른 사용자를 가장하여 서버에 연결할 가능성이 생긴다. 이를 노려 다른 session ID를 추론해 해당 사용자인 척 서버에 연결하는 공격을 weak session id라고 할 수 있다.<br/>
  <center><img src="/assets/230710/weak_session_IDs.png" width="50%" height="50%" alt="Weak_Session_IDs_Diagram"></center>
  실습은 DVWA를 이용 하였으며, 실습 환경은 작성자의 가상머신에서 이루어졌다(ARM / VMWare Fusion - Kali Linux)<br/>

### 본론
**실습 과정**
    
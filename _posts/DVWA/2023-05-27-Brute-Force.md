## Groot Security Study - Week 1
 - DVWA의 내용들에 대해 직접 실습하고 그에 대한 보고서를 정리하는 블로그

### 개요
  * 1주차는 Brute Force 공격에 관한 내용이다.<br/>
  * 실습은 DVWA를 이용 하였으며, 실습 환경은 작성자의 가상머신에서 이루어졌다(ARM / VMWare Fusion - Kali Linux)<br/>

### 본론
  1. **취약점(Brute Force) 설명**<br/>
    * Brute Force는 단어 뜻 그대로 원하는 결과를 얻을 때 까지 무차별적으로 값을 대입하는 공격 방식이다.<br/>
    * 보통 사용자의 id는 암호화 되어 있지 않으므로 사용자 id를 알아낸 뒤 접속이 될때까지 무작위 패스워드를 입력하고, 올바른 패스워드가 입력되었을때 얻은 권한으로 시스템에 접속하여 이후 권한 상승등을 통해 시스템을 장악할 수 있게 한다.<br/>

  2. **실습 과정**<br/>
    - Burpsuite에서 제공되는 기능을 이용하여 공격을 진행하였다.<br/>
    - 관리자 계정 admin에 대해 brute force 공격을 진행하기 위해 kali linux에서 제공하는 흔히 사용되는 패스워드 목록 파일을 payload로 업로드 하였다.<br/>
    <center><img src="/assets/230527/230527_screenshot_1.png" width="50%" height="50%" alt="Screenshot_of_uploaded_password_list"></center><br/>
    - 이후 start attack을 눌러 공격을 시작하면 준비된 패스워드 목록에 대해 각각 로그인을 시도한다.<br/>
    - 이때 다른 패스워드와는 다른 값을 리턴하는 패스워드가 있는데, 이 패스워드를 이용하여 로그인을 시도해 본다.<br/><br/>
    <center><img src="/assets/230527/230527_screenshot_2.png" width="50%" height="50%" alt="Screenshot_of_trying_password_list"></center><br/>
    - 해당 패스워드를 이용하여 DVWA에서 id = admin, password = password로 로그인을 시도하면 로그인에 성공했다는 화면이 뜬다.<br/><br/>
    <center><img src="/assets/230527/230527_screenshot_3.png" width="50%" height="50%" alt="Screenshot_of_result_of_brute_force_attack"></center><br/>

### 결론
  1. **원인 분석**<br/>
    - 다음의 코드가 DVWA의 brute force 실습 페이지이다.<br/>
    <center><img src="/assets/230527/230527_screenshot_4.png" width="50%" height="50%" alt="Screenshot_of_low.php"></center><br/>
    - '$query'로 시작하는 문장이 데이터베이스를 쿼리문을 통해 조회하는 역할을 하는데, 데이터베이스에서 'user' 정보가 입력받은 username($user)이고 'password' 정보가 입력받은 password($pass)인 조합을 찾는(SELECT)다는 뜻이라고 보면 될 듯 하다.<br/><br/>
    - 조회 결과에 따라 $result의 값이 달라지고, 이를 조회한 결과와 비교해서 일치하는 것이 있으면(이때 username-password 조합은 유일하다고 가정) 입력된 username의 보호구역(protected area)에 들어왔다는 문구와 위의 사진에서 볼 수 있는 이미지가 뜬다.<br/><br/>
    - 이때 입력받은 username과 password 값($user, $pass)에 대해, 별다른 과정 없이 해당 조합이 데이터베이스에 존재하는지 여부만 확인하는 방식으로 동작하므로, 몇번이고 로그인을 시도 할 수 있기 때문에 가능한 모든 조합의 경우를 시도할 수 있다.<br/><br/>
    - 따라서 해당 페이지는 시큐어 코딩이 전혀 되어있지 않은 상황이고, 마음만 먹으면 모든 username에 대해 password를 알아 낼 수도 있다.(물론 굉장히 많은 시간이 소요될 것이다.)<br/><br/>
    - 추가적으로, brute force 공격에 해당되는 내용은 아니지만, 입력 스트링에 대해 쿼리문의 변혈을 유발하여 password 없이 로그인 하거나 다른 정보를 알아 낼 수도 있는 특수문자의 사용을 확인/방지하는 코드도 없어 SQL Injection 공격도 가능해 보인다.<br/>

  2. **예상 대응 방안**<br/>
    - 로그인 시도 횟수를 제한하여 그 이상 로그인을 시도할 수 없게 하거나 비밀번호를 강제로 변경하게 한다.
      * 새로운 변수 login_trial와 account_lock 을 도입한다.<br/>
      * login_trial : 로그인을 시도한 횟수(초기 값: 0)<br/>
      * account_lock : 계정 잠금에 대한 boolean 값(초기 값: FALSE)<br/>
      * 로그인에 실패할 때 마다 login_trial의 값이 1씩 증가한다.<br/>
      * 지정된 횟수 (예: 5회)에 도달하면 account_lock의 값 TRUE로 변경. 즉, 계정을 잠궈 접속 시도 불가능 하게 만든다.<br/>

### 마치며<br/>
  * 이번 주차는 환경 설정에서 속된말로 전을 굽는 바람에 많이 헤맸다.
  * 그래서 정말 기본적인 수준의 실습만을 진행했고, 해당 문제(DVWA에 대한 brute force 공격)의 진행 과정을 위주로 write-up을 작성하였다.
  * 아직 피드백을 받지는 않았지만, 스터디 미팅 시간에 받은 간이 피드백을 바탕으로 내용을 좀 추가 했다.
  * 공격 툴을 만드는 과정은 진행하지 않았는데, 다음 주차 실습에서 툴 제작까지는 아니더라도 구글링해 찾은 툴들을 분석(동작 방식 등)하는 과정이 있어도 괜찮을 것 같다.

### 레퍼런스<br/>
  * [OWASP - Brute Force Attack](https://owasp.org/www-community/attacks/Brute_force_attack)
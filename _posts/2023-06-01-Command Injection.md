# Groot Security Study - Week 2

## DVWA의 내용들에 대해 직접 실습하고 그에 대한 보고서를 정리하는 블로그

### 개요
  * 2주차는 Command Injection 공격에 관한 내용이다.<br/>
  * Command Injection은 OWASP 10 에서 상위권에 꼽히는 Injection 공격의 한 종류로, 2017년에는 1위, 2023년 조사에서는 3위를 기록하였다.<br/>
  * Command Injection이란, 취약한 애플리케이션에 대해 임의의 명령어들을 전달하여 실행시키는 공격이다.<br/>
  * 일반적인 서비스가 다 그런거 아닌가? 할 수 있지만, 블특정 사용자가 시스템에서 돌아다니고 혹여 보안에 위협적인 행위를 하게 된다면 서비스와 시스템이 위험에 처할 수 있다.<br/>
  * 실습은 DVWA를 이용 하였으며, 실습 환경은 작성자의 가상머신에서 이루어졌다(ARM / VMWare Fusion - Kali Linux)<br/>

### 본론
**실습 과정**
  1. Low
      * 다음은 Low 단계 command injection의 공격을 수행할 서비스의 소스코드이다.<br/><br/>
    <center><img src="/assets/230601/230601_screenshot_1.png" width="50%" height="50%" alt="Screenshot_of_source_code(low)"></center><br/><br/>
    <center><img src="/assets/230601/230601_screenshot_2.png" width="50%" height="50%" alt="Screenshot_of_return:127.0.0.1"></center><br/><br/>
      * 소스코드를 확인하면, 입력받은 ip주소를 $target으로 저장하여 os에 맞게 'ping (ip주소)' 명령을 터미널에서 실행시키는 방식으로 동작 한다는 것을 알 수 있다.<br/>
      * 이때 입력받은 문구를 ip주소라고 가정하고 단순히 앞에 ping 명령어를 붙인 후 터미널에 전달하는 방식이라, ip주소가 아닌 다른 명령어를 연결 해주면 함께 실행 시킬 수 있을 것 같다.<br/><br/>
    <center><img src="/assets/230601/230601_screenshot_3.png" width="50%" height="50%" alt="Screenshot_of_return:127.0.0.1;cat /etc/passwd"></center><br/><br/>
      * 위는 입력값으로 '127.0.0.1 && cat /etc/passwd'을 전달한 결과이다.<br/>
      * 여러 CLI명령어를 동시에 입력하게 해주는(그리고 그 명령어들을 연결해주는) 방법들 중 ';'을 이용하여 'cat /etc/passwd'명령을 연결해 주었고, 그 결과 위의 사진처럼 id와 패스워드들의 저장 정보를 알 수 있었다.
      * 현재 동작하는 id를 알기 위해 같은 방식으로 whoami명령을 실행하면, 다음과 같이 www-data인 것을 알 수 있다.<br/><br/>
    <center><img src="/assets/230601/230601_screenshot_3-1.png" width="50%" height="50%" alt="Screenshot_of_return:whoami"></center><br/><br/>

  2. Medium
      * 다음은 Medium 단계 command injection의 공격을 수행할 서비스의 소스코드이다.<br/><br/>
    <center><img src="/assets/230601/230601_screenshot_4.png" width="50%" height="50%" alt="Screenshot_of_source_code(medium)"></center><br/><br/>
      * 소스코드를 확인하면, 이전 단계에서 사용한 ';'나 '&'가 서비스의입력되면 공백문자로 치환해 사용할 수 없게 하는 코드가 추가되었다.
      * 하지만 이 두가지 이외에도 여러 명령어를 함께 입력할 수 있는 방법이 많이 있는데, 그 중 하나가 파이프(&#124;)이다.
      * 파이프는 두 명령에 대해 앞선 명령의 결과를 후위 명령의 인자로 전달해 주는 역할을 한다.
      * Low에서 시도한 것과 유사하게 '127.0.0.1 &#124; cat /etc/passwd'를 실행한다면, 소스코드에서의 'ping -c 4'와 결합하여 다음의 두가지 명령으로 나누어 진다.
          - ping -c 4 127.0.0.1
          - cat /etc/passwd
      * 이때 파이프에 의해 ping명령의 결과가 cat명령의 인자로 전달되는데, cat명령에는 인자로 /etc/passwd가 이미 연결되어 있으므로 ping명령의 결과는 무시된다.
      * 따라서 입력값으로 '127.0.0.1 &#124; cat /etc/passwd'을 전달하면 다음과 같은 결과를 받을 수 있다.<br/><br/>
     <center><img src="/assets/230601/230601_screenshot_5.png" width="50%" height="50%" alt="Screenshot_of_return:127.0.0.1 &#124; cat /etc/passwd"></center><br/><br/>

  3. High
      * 다음은 High 단계 command injection의 공격을 수행할 서비스의 소스코드이다.<br/><br/>
    <center><img src="/assets/230601/230601_screenshot_6.png" width="50%" height="50%" alt="Screenshot_of_source_code(high)"></center><br/><br/>
      * 소스코드를 확인하면, 다른 특수문자들도 모두 공백문자로 치환되게 설정되어 있다.
      * 하지만 코드를 잘 확인해 보면, 파이프를 공백문자로 치환하는 부분에 있어 '&#124; '과 같이 공백이 한 칸 들어가 있다.
      * 따라서 '127.0.0.1 &#124;&#124; cat /etc/passwd'를 입력하면 먼저 '&#124; '이 공백으로 치환되어 '127.0.0.1 &#124;cat /etc/passwd'이 되어 실행되게 된다.
      * 이떼 '&#124;&#124;'의 탐지보다 '&#124; '의 탐지가 먼저 이루어져 위와 같은 결과가 발생하게 된다.
      * 결과적으로 다음과 같은 결과를 받을 수 있다.<br/><br/>
    <center><img src="/assets/230601/230601_screenshot_7.png" width="50%" height="50%" alt="Screenshot_of_return:127.0.0.1 &#124;&#124; cat /etc/passwd"></center><br/><br/>

### 결론
  1. **원인 분석**
      * Low : 보호가 전혀 되어있지 않다. 입력되는 문자열을 받아 이를 터미널에 그대로 전달해 주는 단순한 방식이기에 입력되는것이 보안에 위협이 되더라도 막을 수 있는 방법이 없다.
      * Medium : 어느 정도 보호가 이루어져 있기는 하나, 부족하다. 많이 사용되는 '&'와 ';'에 대해서만 방어가 이루어지고 나머지에 대해서는 보호가 전혀 되어있지 않다.
      * High : 웬만해서 많이 사용되는 특수문자들에 대해 대부분 차단이 이루어 지고 있으나, 문제 풀이에서 처럼 서로 충돌하는 부분이 있다. 이 때문에 입력되는 특수문자들의 배열 순서를 적절히 조절하면 유효한 공격이 이루어 질 수 있다.

  2. **예상 대응 방안**
      * 다음은 impossible 단계 소스코드이다.<br/><br/>
    <center><img src="/assets/230601/230601_screenshot_8.png" width="50%" height="50%" alt="Screenshot_of_source_code(impossible)"></center><br/><br/>
      * 코드를 보면, 이전 단계처럼 공격에 사용될 것 같은 특수문자를 공백으로 치환하여 보호하는 것이 아니라, 처음부터 IPv4 주소의 형식처럼 '숫자.숫자.숫자.숫자'로 입력이 들어와야만 유효한 결과를 반환할 수 있게 설계되어 있다.
      * 이처럼 '하면 안되는 행동 못하게 하기'방식이 아니라 '할 수 있는 행동 정해주기' 방식의 시큐어 코딩이 필요해 보인다.

### 마치며
  * 지난주차 스터디가 끝난 후 일단 블로그 설정부터 마무리 했다. (알고 보니 포스팅 될 컨텐츠로의 경로에 오타가 있어서 아무 내용이 보이지 않았던 것이었다.)
  * 소스코드를 먼저 보고 시스템이 어떤 방식으로 동작하는지 이해하는게 가장 중요한 것 같다. 지난 주차에는 이미 실습을 진행 하고 나서 소스코드를 보게 되어 단순히 취약점 발생 원인을 밝히는 수준에서 끝났는데, 이번엔 먼저 소스코드를 확인하여 실습을 좀 더 수월하게 진핼할 수 있었다.
  * 지난 주차를 마치며 다음부턴 해당 주차학습에 관련된 공격 툴을 분석해 보는 과정을 추가해야겠다 했었는데, 이번 주차처럼 단순히 '유효한 취약점 공격 방법을 찾는' 실습에 대해서는 굳이? 라는 생각이 든다.(툴이라는게 결국 사람이 하기 귀찮은 부분을 대신 해주는 역할이지 않나 싶다.)

### 참조
  * [OWASP - Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
# Groot Security Study - Week 8

## DVWA의 내용들에 대해 직접 실습하고 그에 대한 보고서를 정리하는 블로그

### 개요
  8주차는 Weak Session IDs라는 주제이다. 우선 session ID가 무엇인지 살펴보면 우리가 서버에 연결할때 수많은 사용자들 사이에서 각각을 구분하기 위해 정보(쿠키)를 생성하여 사용자가 저장하게 되는데, 이때 외부에 노출되면 안되는 정보들에 대해서는 session ID를 발급하여 사용자가 아닌 서버쪽에서 저장하게 된다. 이때 session ID를 발급하는 기준이나 방법이 예측 가능하다면(weak) 다른 사용자를 가장하여 서버에 연결할 가능성이 생긴다. 이를 노려 다른 session ID를 추론해 해당 사용자인 척 서버에 연결하는 공격을 weak session id라고 할 수 있다.<br/>
  <center><img src="/assets/230710/weak_session_IDs.png" width="30%" height="30%" alt="Weak_Session_IDs_Diagram"></center>
  실습은 DVWA를 이용 하였으며, 실습 환경은 작성자의 가상머신에서 이루어졌다(ARM / VMWare Fusion - Kali Linux)<br/>

### 본론
**실습 과정**
  1. Low
      먼저 소스코드를 확인해 보자.

      ```php
      <?php
      $html = "";

      if ($_SERVER['REQUEST_METHOD'] == "POST") {
        if (!isset ($_SESSION['last_session_id'])) {
          $_SESSION['last_session_id'] = 0;
        }
        $_SESSION['last_session_id']++;
        $cookie_value = $_SESSION['last_session_id'];
        setcookie("dvwaSession", $cookie_value);
      }
      ?>
      ```

      코드를 보면, 이전에 발급받은 session ID가 존재하지 않는 경우 session ID로 0을 부여하고, 이후에 발급되는 session ID들은 이전 값에 1을 더한 값으로 설정된다. 이 경우 무작위나 특수한 방법으로 session ID가 발급된 것이 아니므로 단순히 자신 이전의 값을 sesssion ID로 전달하는 것 만으로 다른 사용자로 가장하고 서버에 연결할 수 있다.<br/>
      <img src="/assets/230710/230710_screenshot_1.png" width="50%" height="50%" alt="Weak_Session_IDs_low"><br/>

  2. Medium
      역시 소스코드를 먼저 확인해 보자.

      ```php
      <?php
      $html = "";

      if ($_SERVER['REQUEST_METHOD'] == "POST") {
        $cookie_value = time();
        setcookie("dvwaSession", $cookie_value);
      }
      ?>
      ```

      low와는 다르게 session ID가 연속된 수열이 아닌 session ID가 생성되는 시간으로 정해진다. 하지만 session ID를 여러번 생성해보면 1초마다 값이 1씩 커진다는 것을 알 수 있는데 결국 이 방법도 비 연속적인 선형 수열로 값을 생성하기에 무작위 대입 등의 방법을 통해서 다른 사용자를 가장하여 서버에 연결할 수 있게 된다.<br/>
      <img src="/assets/230710/230710_screenshot_2.png" width="50%" height="50%" alt="Weak_Session_IDs_medium"><br/><br/>
      <img src="/assets/230710/230710_screenshot_3.png" width="50%" height="50%" alt="Weak_Session_IDs_medium"><br/>

  3. High
      역시 소스코드부터 확인해 보자.

      ```php
      <?php
      $html = "";

      if ($_SERVER['REQUEST_METHOD'] == "POST") {
        if (!isset ($_SESSION['last_session_id_high'])) {
          $_SESSION['last_session_id_high'] = 0;
        }
        $_SESSION['last_session_id_high']++;
        $cookie_value = md5($_SESSION['last_session_id_high']);
        setcookie("dvwaSession", $cookie_value, time()+3600, "/vulnerabilities/weak_id/", $_SERVER['HTTP_HOST'], false, false);
      }
      ?> 
      ```

      코드를 확인해보면 session ID을 생성한 후 md5를 이용하여 해시값을 만든 후 사용하고 있는데, 정작 해당 해시값의 원본이 Low단계에서의 연속적 선형 수열과 같아 쉽게 유추될 수 있다.<br/>
      <img src="/assets/230710/230710_screenshot_4.png" width="50%" height="50%" alt="Weak_Session_IDs_high"><br/><br/>
      <img src="/assets/230710/230710_screenshot_5.png" width="50%" height="50%" alt="Weak_Session_IDs_high_hash"><br/><br/>
      <img src="/assets/230710/230710_screenshot_6.png" width="50%" height="50%" alt="Weak_Session_IDs_high"><br/><br/>
      <img src="/assets/230710/230710_screenshot_7.png" width="50%" height="50%" alt="Weak_Session_IDs_high_hash"><br/><br/>
      따라서 이 방법도 그리 안전한 방법은 아닌것으로 보인다.

### 결론
  1. **원인 분석**<br/>
      난이도가 상승할수록 session ID를 보호하는 수준이 높아지나, session ID를 생성하는 방법에 취약점이 존재하여 딱히 의미가 없는 절차가 되었다. MD5를 이용하여 해시값을 생성하던, 생성 시간을 인자로 넣던, 결국 선형 수열을 이용한 session ID 생성 방법이라 공격자가 무작위 대입 방법등을 통하여 본인이 할당받은 값 이전의 session ID들을 충분히 유추하거나 탐지해 낼 수 있다.

  2. **예상 대응 방안**<br/>
      Impossible 난이도의 코드를 확인해보자.

      ```php
      <?php
      $html = "";

      if ($_SERVER['REQUEST_METHOD'] == "POST") {
        $cookie_value = sha1(mt_rand() . time() . "Impossible");
         setcookie("dvwaSession", $cookie_value, time()+3600, "/vulnerabilities/weak_id/", $_SERVER['HTTP_HOST'], true, true);
      }
      ?> 
      ```

      코드를 확인해보면, session ID생성 과정에서부터 무작위 난수를 사용하고, 해당 seed의 유출을 대비하여 생성 시간까지 이용한 조합을 sha1을 이용해 해싱하고 그 결과값을 session ID로 사용하고 있다. 이처럼 복합적인 요소 / 무작위 요소를 이용한 후 이를 암호화 하여 사용하면 가능한 session ID가 비선형적으로 생성되기에 현실적 시간 내에 계산되지 않을 수 있다.

### 마치며
  이번 주차는 특정 암호를 찾아내는 것이 아니라, 말 그대로 현재 시스템의 취약점을 찾아 분석하는 것이라 다른 주차에 비해 간단하면서도 그만큼 여러가지 가능성을 고려해야 해서 생각해 볼 것이 많았다. 그래도 코드를 분석하면 대체로 답을 얻을 수 있어서 그렇게 어렵지는 않았던 것 같다.

### 참조
  * [Session Prediction](https://owasp.org/www-community/attacks/Session_Prediction)
  * [쿠키/세션에 대하여](https://medium.com/@cute_mustard_sardine_17/쿠키-cookie-세션-session-에-대하여-e8a974d76df8)
  * [Securing Session IDs](https://www.hacksplaining.com/prevention/weak-session)
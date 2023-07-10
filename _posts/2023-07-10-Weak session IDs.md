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
      <img src="/assets/230710/230710_screenshot_2.1.png" width="50%" height="50%" alt="Weak_Session_IDs_medium"><br/>
      <img src="/assets/230710/230710_screenshot_2.2.png" width="50%" height="50%" alt="Weak_Session_IDs_medium"><br/>

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
      <img src="/assets/230710/230710_screenshot_3.1.png" width="50%" height="50%" alt="Weak_Session_IDs_high"><br/>
      <img src="/assets/230710/230710_screenshot_3.1.1png" width="50%" height="50%" alt="Weak_Session_IDs_high_hash"><br/>
      <img src="/assets/230710/230710_screenshot_3.2.png" width="50%" height="50%" alt="Weak_Session_IDs_high"><br/>
      <img src="/assets/230710/230710_screenshot_3.2.1png" width="50%" height="50%" alt="Weak_Session_IDs_high_hash"><br/>

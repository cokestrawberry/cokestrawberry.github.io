---
title: "XSS"
categories: DVWA
toc: true  
toc_sticky: true 
---

## Groot Security Study - Week 8
 - DVWA의 내용들에 대해 직접 실습하고 그에 대한 보고서를 정리하는 블로그

### 개요
  9주차는 XSS(Cross-Site Scripting), 그 중에서도 Stored 방식에 관한 내용이다.<br/>
  XSS는 공격자가 악성 코드/스크립트 등을 웹사이트를 통하여 유포 할 수 있는 공격을 말한다. 이를 이용해 공격자가 일반 사용자의 쿠키/세션등을 탈취할 수 있고 비정상적인 행동을 유발할 수 있다. 주로 다른 웹사이트와 정보를 교환하는 방식을 이용해서 사이트 간(Cross-Site) 이라는 이름이 붙어있다.<br/>
  이중 Stored 방식은 공격자가 악성 코드/스크립트를 공격 대상인 웹 사이트에 상시적으로 유지되게 하여(Stored) 해당 웹사이트에 접근하는 일반 사용자들에 대해 자동적으로/지속적으로 공격을 수행할 수 있다.<br/>
  <center><img src="/assets/230718/Stored_XSS.png" width="50%" height="50%" alt="XSS_Diagram"></center><br/>
  실습은 DVWA를 이용 하였으며, 실습 환경은 작성자의 가상머신에서 이루어졌다(ARM / VMWare Fusion - Kali Linux)<br/>
  
### 본론
**실습 과정**
  1. Low<br/>
      <img src="/assets/230718/230718_screenshot_1.png" width="100%" height="100%" alt="XSS_WebSite_Form"><br/>
      사진을 보면 공격 대상이 대시보드에 각각의 사용자가 자신의 이름과 남기고 싶은 메세지를 적으면 방명록을 남길 수 있는 시스템이라는 것을 알 수 있다.
      소스코드를 확인해 보면 다음과 같다.

      ```php
      <?php

      if( isset( $_POST[ 'btnSign' ] ) ) {
        // Get input
        $message = trim( $_POST[ 'mtxMessage' ] );
        $name    = trim( $_POST[ 'txtName' ] );

        // Sanitize message input
        $message = stripslashes( $message );
        $message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

        // Sanitize name input
        $name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

        // Update database
        $query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

        //mysql_close();
      }

      ?>
      ```

      코드를 살펴보면, 입력된 이름과 메세지를 데이터베이스에 INSERT해 주는 동작을 수행하는데 SQL Injection에 대한 보호만 되어있고 XSS에 대해서는 여타 별다른 보호가 되어있지 않은 듯 하다. 따라서 아래와 같은 스크립트를 메세지로 입력하였다.

      ```javascript
      <script>alert(document.cookie)</script>
      ```

      그 결과, 해당 내용이 데이터베이스에 추가되고, 다시 페이지가 로드 되었을때 해당 부분이 스크립트로 인식되어 실행되므로 다음과 같은 팝업이 뜬다.<br/>
      <img src="/assets/230718/230718_screenshot_2.png" width="100%" height="100%" alt="XSS_Popup"><br/>
      본 실습에서는 바로 결과를 알 수 있게 이를 팝업으로 띄웠지만, 실제 공격에서는 공격자의 PC로 해당 정보를 전송하는 등의 방식으로 데이터를 취득하게 된다.

  2. Medium<br/>
      medium 난이도는 소스코드를 먼저 확인해 보면 다음과 같다.

      ```php
      <?php

      if( isset( $_POST[ 'btnSign' ] ) ) {
        // Get input
        $message = trim( $_POST[ 'mtxMessage' ] );
        $name    = trim( $_POST[ 'txtName' ] );

        // Sanitize message input
        $message = strip_tags( addslashes( $message ) );
        $message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
        $message = htmlspecialchars( $message );

        // Sanitize name input
        $name = str_replace( '<script>', '', $name );
        $name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

        // Update database
        $query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

        //mysql_close();
      }

      ?> 
      ```

      코드를 확인해 보면, 메세지 입력에는 역슬래쉬를 자동으로 삽입하여 특수문자들을 처리해주는 함수(addslashes)가 사용되었고, 이름 입력에는 &gt;script&lt;이 입력될 경우 공백으로 치환되게 하는 부분(str_replace)이 추가되었다.
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
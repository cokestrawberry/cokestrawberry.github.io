# Groot Security Study - Week 6

## DVWA의 내용들에 대해 직접 실습하고 그에 대한 보고서를 정리하는 블로그

### 개요
  6주차는 SQL Injection 공격에 관한 내용이다. SQL Injection 공격은 데이터베이스 내부의 데이터를 조회하기 위해 쿼리(Query)문을 입력하는데, 이때 탐색 조건을 무력화 하는 등의 방법으로 악의적인 쿼리문을 입력하여 정해진 범위 이외의 데이터를 조회할 수 있게 되는 공격이다.
  <img src="/assets/230629/SQL_Injection_Diagram.png" width="100%" height="100%" alt="SQL_Injection_Diagram"><br/>
  21년도 조사에서는 순위가 조금 낮아졌으나, 이전까지 OWASP top 10 목록에서 1위를 놓치지 않던, 공격 난이도에 비해 파급력이 큰 공격이다.<br/>
  실습은 DVWA를 이용 하였으며, 실습 환경은 작성자의 가상머신에서 이루어졌다(ARM / VMWare Fusion - Kali Linux)<br/>

### 본론
**실습 과정**
  1. Low<br/>
      우선 소스코드를 확인해보자.

      ```php
      <?php
      if( isset( $_REQUEST[ 'Submit' ] ) ) {
        // Get input
        $id = $_REQUEST[ 'id' ];

        switch ($_DVWA['SQLI_DB']) {
          case MYSQL:
            // Check database
            $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
            $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

            // Get results
            while( $row = mysqli_fetch_assoc( $result ) ) {
              // Get values
              $first = $row["first_name"];
              $last  = $row["last_name"];
 
              // Feedback for end user
              echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
            }

            mysqli_close($GLOBALS["___mysqli_ston"]);
            break;
          case SQLITE:
            global $sqlite_db_connection;

            #$sqlite_db_connection = new SQLite3($_DVWA['SQLITE_DB']);
            #$sqlite_db_connection->enableExceptions(true);

            $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
            #print $query;
            try {
                $results = $sqlite_db_connection->query($query);
            }
            catch (Exception $e) {
              echo 'Caught exception: ' . $e->getMessage();
              exit();
            }

            if ($results) {
              while ($row = $results->fetchArray()) {
                // Get values
                $first = $row["first_name"];
                $last  = $row["last_name"];

                // Feedback for end user
                echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
              }
            }
            else {
              echo "Error in fetch ".$sqlite_db->lastErrorMsg();
            }
            break;
        } 
      }
      ?>
      ```

      코드를 보면 id라는 파라미터를 통해 id를 입력받아서 id라는 변수에 저장한다. 이후 id변수의 값(입력받은 id)를 데이터베이스에서 조회하게 되는데, 그 중 다음의 문장이 쿼리문 조회를 수행한다.

      ```sql
      $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
      ```

      위 문장은 'users'테이블에서 'user_id'의 값이 id 변수의 값과 같은 데이터들에 대해 'first_name'과 'last_name'을 SELECT해 결과를 알려준다는 의미이다. 이때 id 변수가 입력된 값으로 대치될때 쿼리문의 다른 부분과 같이 텍스트로 대체 되는데, 사용자의 입력이 이루어지는 이 부분에서 취약한 SQL 쿼리문을 삽입하는 SQL Injection 공격이 이루어 질 수 있을것으로 보인다.<br/><br/>
      따라서 입력으로 [' or '1' = '1]을 입력하면 쿼리문이 다음과 같이 설정된다.

      ```sql
      $query  = "SELECT first_name, last_name FROM users WHERE user_id = '' or '1' = '1';";
      ```

      이렇게 되면 탐색 조건이 <'user_id'의 값이 공백이거나 '1' = '1'인 경우>가 되어 테이블의 모든 데이터들에 대해 <'user_id'의 값이 공백>은 거짓이지만 <'1' = '1'>이 참이 되어 모든 데이터들이 SELECT의 대상이 되고, 그 결과 다음과 같이 모든 데이터들이 출력된다.<br/>
      <img src="/assets/230629/230629_screenshot_1.png" width="50%" height="50%" alt="Screenshot_of_query_request_result"><br/>

  2. Medium<br/>
      Medium 난이도 역시 코드를 먼저 확인해보자.

      ```php
      <?php
      if( isset( $_POST[ 'Submit' ] ) ) {
        // Get input
        $id = $_POST[ 'id' ];
        $id = mysqli_real_escape_string($GLOBALS["___mysqli_ston"], $id);

        switch ($_DVWA['SQLI_DB']) {
          case MYSQL:
            $query  = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
            $result = mysqli_query($GLOBALS["___mysqli_ston"], $query) or die( '<pre>' . mysqli_error($GLOBALS["___mysqli_ston"]) . '</pre>' );

            // Get results
            while( $row = mysqli_fetch_assoc( $result ) ) {
              // Display values
              $first = $row["first_name"];
              $last  = $row["last_name"];

              // Feedback for end user
              echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
            }
            break;
          case SQLITE:
            global $sqlite_db_connection;

            $query  = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
            #print $query;
            try {
              $results = $sqlite_db_connection->query($query);
            }
            catch (Exception $e) {
              echo 'Caught exception: ' . $e->getMessage();
              exit();
            }

            if ($results) {
              while ($row = $results->fetchArray()) {
                // Get values
                $first = $row["first_name"];
                $last  = $row["last_name"];

                // Feedback for end user
                echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
              }
            }
            else {
              echo "Error in fetch ".$sqlite_db->lastErrorMsg();
            }
            break;
        }
      }

      // This is used later on in the index.php page
      // Setting it here so we can close the database connection in here like in the rest of the source scripts
      $query  = "SELECT COUNT(*) FROM users;";
      $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

      $number_of_rows = mysqli_fetch_row( $result )[0];
      mysqli_close($GLOBALS["___mysqli_ston"]);
      ?>
      ```

      코드를 보면 mysqli_real_escape_string()라는 함수를 이용해서 입력에 대해 이스케이핑을 하는것으로 보인다. 그 외에는 입력을 드롭다운으로 받는다는 특징이 있기에 우선 bursuite를 이용하여 입력값을 드롭다운에서 선택된 값이 아닌 SQL Injection 공격문으로 대체되도록 하였다. 하지만 다음과 같이 SQL구문 오류가 뜨는것을 보면 특수문자인 작은따옴표(')가 이스케이핑 되는것 같다.
      <img src="/assets/230629/230629_screenshot_2.png" width="50%" height="50%" alt="Screenshot_of_intercept_query"><br/><br/>
      <img src="/assets/230629/230629_screenshot_3.png" width="50%" height="50%" alt="Screenshot_of_escape"><br/><br/>
      하지만 굳이 '1'='1' 이라는 조건을 만들어 줄 필요는 없기에 다음과 같이 조건을 변경하여 진행하였다.

      ```sql
      id=1+or+1=1
      ```

      그 결과 다음과 같이 원하는 결과를 얻을 수 있다. 작은따옴표 없이도 공격이 수행되는 이유는, 코드가 Low 단계와는 다르게 입력값을 id 변수에 텍스트로 집어넣는것이 아닌 정수로 집어넣기 때문이다.(그리고 이는 인터페이스가 드롭다운 형식으로 되어있기 때문으로 보인다.)<br/>
      <img src="/assets/230629/230629_screenshot_4.png" width="50%" height="50%" alt="Screenshot_of_query_request_result"><br/>

  3. High<br/>
      High 난이도의 코드는 다음과 같다.

      ```php
      <?php
      if( isset( $_SESSION [ 'id' ] ) ) {
        // Get input
        $id = $_SESSION[ 'id' ];

        switch ($_DVWA['SQLI_DB']) {
          case MYSQL:
            // Check database
            $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id' LIMIT 1;";
            $result = mysqli_query($GLOBALS["___mysqli_ston"], $query ) or die( '<pre>Something went wrong.</pre>' );

            // Get results
            while( $row = mysqli_fetch_assoc( $result ) ) {
              // Get values
              $first = $row["first_name"];
              $last  = $row["last_name"];

              // Feedback for end user
              echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
            }

            ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);        
            break;
          case SQLITE:
            global $sqlite_db_connection;

            $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id' LIMIT 1;";
            #print $query;
            try {
              $results = $sqlite_db_connection->query($query);
            }
            catch (Exception $e) {
              echo 'Caught exception: ' . $e->getMessage();
              exit();
            }

            if ($results) {
              while ($row = $results->fetchArray()) {
                // Get values
                $first = $row["first_name"];
                $last  = $row["last_name"];

                // Feedback for end user
                echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
              }
            }
            else {
              echo "Error in fetch ".$sqlite_db->lastErrorMsg();
            }
            break;
        }
      }
      ?> 
      ```

      코드를 확인해보면 id 변수에 할당되는 값을 'user_id'와 비교할때 길이를 1로 제한하여 어떤 값이 들어오든 맨 앞 한글자만 받아들인다. 따라서 해당 부분을 주석처리 할 수 있다면 원하는대로 SQL Injection을 수행할 수 있을것으로 보인다. 이를 위해 다음과 같은 쿼리문을 입력하였다.

      ```sql
      1' or '1'='1'; -- 
      #주석표시(--)뒤 한칸 공백 필요
      ```

      위의 문장을 입력으로 넣으면, 세미콜론에서 문장이 끝나고 그 이후 부분인 [LIMIT 1;";]은 주석처리되어 무시된다. 그 결과 다음과 같이 원하는 결과를 얻을 수 있다.
      <img src="/assets/230629/230629_screenshot_5.png" width="50%" height="50%" alt="Screenshot_of_query_request_result"><br/>

### 결론
  1. **원인 분석**
      각 난이도의 입력방식을 보면 입력값에 대한 검사가 되지 않거나 변조에 취약하게 설정되어 있다. 심지어 주어진 값 중에서 골라야 하는 드롭다운에서도 변조가 가능하다는건 조금 놀라웠다.

  2. **예상 대응 방안**
      입력값들에 대해서 가능한 위험 요소들을 제거하는 것이 아닌, 처음부터 유저들의 입력을 시스템의 운영자가 원하는 형식으로 강제하는 방식의 시큐어 코딩이 필요해 보인다.

### 마치며
  * 놓치고 지나간 주차들에 대해서 write-up을 준비하고 있는데, 뭐가 문제인지 제대로 공격이 이루어지지 않아서 열심히 공부해보는 중이다. 그리고 지난번 스터디에서 다른분들의 write-up중에서 참고할만한 형식(내용 X)에 대해 벤치마킹을 해보려고 하니 그에 따라 점점 새로 공부할게 늘어가는것 같다.

### 참조
  * [OWASP - SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
  * [OWASP - SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
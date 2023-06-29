# Groot Security Study - Week 6

## DVWA의 내용들에 대해 직접 실습하고 그에 대한 보고서를 정리하는 블로그

### 개요
  6주차는 SQL Injection 공격에 관한 내용이다. SQL Injection 공격은 데이터베이스 내부의 데이터를 조회하기 위해 쿼리(Query)문을 입력하는데, 이때 탐색 조건을 무력화 하는 등의 방법으로 악의적인 쿼리문을 입력하여 정해진 범위 이외의 데이터를 조회할 수 있게 되는 공격이다.
  21년도 조사에서는 순위가 조금 낮아졌으나, 이전까지 OWASP top 10 목록에서 1위를 놓치지 않던, 공격 난이도에 비해 파급력이 큰 공격이다.
  </br>
  실습은 DVWA를 이용 하였으며, 실습 환경은 작성자의 가상머신에서 이루어졌다(ARM / VMWare Fusion - Kali Linux)<br/>

### 본론
**실습 과정**
  1. Low
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
    코드를 보면 id라는 파라미터를 통해 id를 입력받아서 id라는 변수에 저장한다. 이후 id변수의 값(입력받은 id)를 데이터베이스에서 조회하게 되는데, 
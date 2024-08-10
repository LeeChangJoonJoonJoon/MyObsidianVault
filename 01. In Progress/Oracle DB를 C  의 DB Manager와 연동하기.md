

2023-05-05

----
#Server #OracleDB #Manager #Unreal #MVC #Cpp #PANIM #CLion #지선생 #ChatGPT #Bard 

## 개요
오늘부터 며칠 만에 오라클과 C++의 MVC 시스템을 연결하는 작업을 해야 한다.
여기엔 그 구체적인 과정을 적어보고자 한다.

## 내용
우선, 소스 파일들의 상속을 정하여 정리를 하자.
.cpp 파일과 .h의 정리된 모습은 다음과 같다.
![[스크린샷 2023-05-15 오후 4.30.41.png|300]]

그리고 이 클래스들을 사용하여 실행시키는 메인함수는 `DBManager.h` 헤더파일을 상속 받는다.
[[C++ 파일 분할하는 법]]에서 자세한 방법을 다룬다.

그리고 이제 `OracleConnectionPool`에서 오라클과 연결을 시켜줘야 하는데...
이 플러그인을 어떻게 써야할지 모르겠다.
![[스크린샷 2023-05-15 오후 5.05.16.png]]

그래서 우리의 지선생에게 여쭈어봤다.
`Database` 버튼이 있으니 찾아보랜다.
그래서 다음과 같이 팝업창에 내용을 입력하고 연결해 봤다.
![[스크린샷 2023-05-15 오후 5.17.28.png]]

여기서 주의할 점은, `Host`와 `User`를 구분해야 한다는 것이다.
`Host`는 내가 기본값인 `localhost`로 두었고, `User`는 일전에 권한을 부여한 `panim`으로 써줬다.

접속에 성공했고, `Apply`를 누르면 다음과 같이 SQL문을 실행할 수 있는 콘솔이 뜬다...!
![[스크린샷 2023-05-15 오후 5.24.25.png]]

그리고 `Refresh` 버튼을 누르면 다음과 같이 SQL Developer에서 만든 테이블도 확인이 된다.
![[스크린샷 2023-05-15 오후 5.34.36.png]]

오라클과의 연동을 하기 위해서는 C++가 오라클을 조작할 수 있도록 해주는 오라클의 API인 OCCI를 이용해야 한다.
자세한 과정은 [[Apple Silicon에서 Oracle Client Instant 패키지 사용하기]]에 나와 있다.

이제 이 테이블에 정보를 편집하고 읽어오는 기능을 만들 차례이다.
바드에게 물어봤다.
![[스크린샷 2023-05-16 오후 6.28.16.png]]

구체적인 클래스와 헤더는 다음 순으로 작성할 것이다.
```cpp
OracleConnectionPool.h
OracleConnectionPool.cpp
DBDebug.cpp
OracleConnection.h
OracleConnection.cpp
```

[[01. OracleConnection.cpp]], [[02. DBDebug.cpp]], [[03. OracleConnectionPool.cpp]]에서 자세히 다루겠다.
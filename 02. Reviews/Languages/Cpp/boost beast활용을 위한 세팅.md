

2024-01-13

----
#Cpp #HTTP #REST #Mac #CLion

## 개요
맥북으로 HTTP 요청을 하는 C++ 클라이언트를 만들어 보고자 한다.
이를 위해 CMake와 CLion에 대한 이해가 필요하다.

## 내용
먼저 boost 라이브러리를 다운 받고, 터미널에 `open library`를 입력하여 시스템 라이브러리들이 들어 있는 곳에 넣어준다.
![[Pasted image 20240113223642.png]]

그리고 여기서 경로복사를 하고 아래 나오는 코드에 넣어준다.
```cpp
set(BOOST_ROOT "/Library/boost_1_84_0")  
  
find_package(Boost 1.57.0)  
  
if(NOT Boost_FOUND)  
    message(FATAL_ERROR "Could not find boost!")  
endif()

include_directories(${BOOST_ROOT})
```

이를 CLion으로 구성된 프로젝트의 CMake 파일에 붙여넣어주고 새로고침하면 된다.
프로젝트가 정상적으로 라이브러리를 찾았다면 CMake에서 정의한 분기를 타지 않을 것이다.

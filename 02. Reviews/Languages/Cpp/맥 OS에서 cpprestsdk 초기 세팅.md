

2024-02-17

----
#Cpp #라이브러리 #문제해결 #Mac #REST #cpprestsdk

## 개요
회사에서 진행하는 언리얼 프로젝트에서 언리얼 프레임워크가 제공하는 웹소켓 통신 라이브러리가 작동하지 않는 문제가 발생.
이를 해결하고자 원래 C++ 코드를 프로젝트 내에 삽입하여 여기로 웹소켓 통신을 하고자 한다.

## 내용
먼저 `Homebrew`를 통해 `cpprestsdk`를 다운 받는다.
![[Pasted image 20240217201950.png]]

`cpprestsdk`는 `boost.asio`, `openssl`을 참조하므로 둘 다 설치해야 한다.
`boost.asio`는 이미 [[boost beast활용을 위한 세팅]]에서 세팅해 줬다.
`openssl`를 설치하자.
![[Pasted image 20240217204817.png]]

CMake를 다음과 같이 설정해 주면 된다
```cpp
cmake_minimum_required(VERSION 3.28)  
project(WebSocket_test)  
  
set(CMAKE_CXX_STANDARD 26)  
  
set(BOOST_ROOT "/Library/Cpp3rdPartyLibraries/boost_1_84_0")  
set(OPENSSL_ROOT_DIR "/usr/local/opt/openssl")  
  
find_package(cpprestsdk REQUIRED)  
find_package(OpenSSL REQUIRED)  
  
# Boost 라이브러리 참조  
find_package(Boost REQUIRED COMPONENTS system thread)  
  
# 프로젝트 소스 파일 추가  
add_executable(main main.cpp)  
  
# cpprestsdk 및 Boost 라이브러리 링크  
target_link_libraries(main PRIVATE cpprestsdk::cpprest Boost::system Boost::thread OpenSSL::SSL OpenSSL::Crypto)
```

이제 여기서 [[cpprestsdk로 클라이언트 통신 객체 만들기]]를 해볼 수 있겠다.
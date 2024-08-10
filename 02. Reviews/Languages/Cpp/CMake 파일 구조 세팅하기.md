

2023-11-05

----
#Cpp #CLion 

## 개요
이제는 Visual Studio가 맥에서 지원되지 않는 관계로, 현재 맥 환경에서 원활하게 돌아가는 CLion을 C++ 에디터로 사용해 보려고 했다.
근데, Visual Studio 프로젝트에서 C++을 다루는 것과 달리, 몇가지 문제가 있기에 이를 해결하는 과정을 적어보고자 한다.

## 내용
각 파일 별로 빌드에서 제외할 수 있는 Visual Studio 프로젝트와 달리, CMake 컴파일러를 사용하는 CLion에서는 빌드 타깃을 CMakeLists.txt 파일에서 결정한다.
현재 내가 책을 공부하기 위해 만든 파일 구조는 다음과 같다.
![[Pasted image 20231105165816.png]]

문제는, 컴파일러가 두개의 `main` 함수를 인식하여 에러를 뿜어댄다는 것이다.
이를 막기 위해 디렉토리를 분리하여 `.cpp`파일을 구성했는데도.

물론 디렉토리를 분리하고 디렉토리마다 CMakeLists.txt 파일을 배치하여 컴파일 하는 방법도 있다.
하지만 이 방법은 정확히 알지 못하여 더 쉬운 방법을 이용할 것이다. 

현재 cmake-build-debug 파일까지를 포함하는 파일명이 CppBasic이다.
그리고 처음 CMakeLists.txt를 열어보면 다음과 같이 구성돼 있다.
```cpp
cmake_minimum_required(VERSION 3.26)  
project(CppBasic)  
  
set(CMAKE_CXX_STANDARD 20) 

add_executable(CppBasic main.cpp)
```

`add_executable()`의 괄호 내에 인수로 들어가는 앞의 변수는 현재 가장 큰 파일인 `CppBasic`이다.
그리고 거기에 기본으로 포함된 `main.cpp`가 실행 타깃으로 설정돼 있는 것이다.

이를 다음과 같이 바꿔보자.
일단 내가 새로 추가한, `main` 함수가 포함된 파일들은 각각 HelloCpp.cpp, CoutSample.cpp이다. 
```cpp
cmake_minimum_required(VERSION 3.26)  
project(CppBasic)  
  
set(CMAKE_CXX_STANDARD 20)  
  
add_executable(HelloCpp HelloCpp.cpp)  
add_executable(CoutSample CoutSample.cpp)
```

이렇게 파일을 편집하고 가장 상위 파일인 CppBasic을 우클릭하여 다음 버튼을 누른다.
![[Pasted image 20231105170554.png|500]]

그러면 다음과 같이 빌드 및 실행 파일을 정할 수 있게 된다.
![[Pasted image 20231105170640.png|500]]

이제 신나게 코딩을 하면 된다.

출처: https://medium.com/@mfkhao2009/clion-one-project-multiple-executable-file-4091d07c3936
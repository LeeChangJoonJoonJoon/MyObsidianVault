

2023-05-16

----
#PANIM #Cpp #Mac #OracleDB #UNIX #터미널 #ChatGPT #지선생 #Rosetta2 #문제 #문제해결 

## 개요
오라클에서 제공하는 API를 .cpp파일에서 사용하려면 그 파일이 `x86` 기반에서 빌드되어야 한다.
현재 오라클과의 연결을 담당하는 `OracleConnection.cpp`를 이를 통해 빌드해야 한다.
하지만 다음 두가지 문제가 있다.
1. 참조 자체가 파일이 깨진다. 안된다는 얘기.
2. 그 깨지는 파일도 엄청 느리게 참조된다. 대략 37초 정도 걸린다. 파일을 객체화하고 데이터를 끌어오는 데에 이 정도 시간이 걸린다는 건 못쓰는 것과 다를 바 없다.

## 내용

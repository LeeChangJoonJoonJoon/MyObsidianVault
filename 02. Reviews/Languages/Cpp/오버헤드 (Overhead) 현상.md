

2023-05-23

----
#Cpp #CS #지선생 #ChatGPT #라이브러리 #메모리 

## 개요
[[무브 시맨틱 (Move Semantic)]]을 사용하는 이유로, 오버헤드(Overhead)를 감소시키기 위함이라는 설명을 들었다.

## 내용
다음과 같은 간단한 코드를 살펴보자.
```cpp
#include <iostream>

class MyObject {
public:
    MyObject() {
        std::cout << "MyObject 생성자 호출" << std::endl;
    }
    
    MyObject(const MyObject& other) {
        std::cout << "MyObject 복사 생성자 호출" << std::endl;
    }
};

void myFunction(MyObject obj) {
    std::cout << "myFunction 호출" << std::endl;
}

int main() {
    MyObject obj1; // MyObject 생성자 호출
    
    myFunction(obj1); // myFunction 호출, 복사 오버헤드 발생
    
    return 0;
}

```

여기서 메인함수는 `myFunction` 함수를 호출하기 위해 프로그램의 실행 흐름이 끊고, 함수를 실행시키는 데에 필요한 스택 메모리를 할당한다.
매개변수가 있을 경우 대입연산이 일어난다.
이처럼 예상치 못한 리소스의 소모가 일어나는 것을 오버헤드라고 부른다.

우리가 무브 시멘틱을 사용하는 이유는 이런 오버헤드 현상이 적기 때문이었다.

여기서 생긴 복사 오버헤드는 "다른 객체를 인수로 전달받"는 과정에서 발생한다.
C++에선 기본적으로 어떤 변수(이 경우 `obj1`)를 함수의 인수로 전달할 때 값의 복사가 발생한다.
이 전달하는 행위 자체만으로 복사생성자가 호출되었던 것이다.


2023-05-23

----
#Cpp #CS #지선생 #ChatGPT #라이브러리 #메모리 #객체 #객체지향 

## 개요
[[unique_ptr,  make_unique]]을 공부하던 중, `unique_ptr`의 특성 중 하나를 설명하면서 Move Sementic이란 용어를 보았다.
다음 글은 https://kukuta.tistory.com/426 를 읽고 필자가 이해한 내용을 정리한 것이다.

## 내용
먼저 C++11 공식문서에서 설명하고 있는 자원의 '이동'이 뭔지 알아야 한다.
이 '이동'은 `move` 연산으로 코드 상에서 만나볼 수 있다.
```cpp
#include <iostream>
#include <string>

int main() 
{
    std::string s1 = "Hello World";
    std::string s2 = s1;
    std::string s3 = "Hello World";
    std::string s4 = std::move(s3);
    
    std::cout << "s1:" << s1 << std::endl;
    std::cout << "s2:" << s2 << std::endl;
    std::cout << "s3:" << s3 << std::endl;
    std::cout << "s4:" << s4 << std::endl;
    
    return 0;
}

// OUTPUT
// s1:Hello World
// s2:Hello World
// s3:
// s4:Hello World
```

여기서 `s1`을 `s2`에 대입연산자 `=`을 실행하고 있는데, 이때 컴파일러 단에서는  기존 값인 `"Helllo World"`를 새로운 메모리 공간에 복사하며, 새로운 변수 `s2`는 이 새로 생긴 `"Hello World"`를 가리킨다.
(정확히 말하자면 변수명 자체는 그 값이 저장된 주소를 가리키진 않고, 그 주소를 찾기 위한 레퍼런스로 인식된다.)
하지만 `move` 함수를 쓰게 되면, 기존에 `s3`에 할당된 값은 사라지고 `s4`로 자리를 옮기게 된다.
즉, 이 '자원의 이동'은 얕은 복사를 한 뒤 원본의 참조를 제거하는 것을 말한다.
*깊은 복사(Deep Copy)는 객체 자체를 복사하는 것을 말하고, 얕은 복사(Shallow Copy)는 객체가 가진 주소 값을 하나 더 생성하는 것을 말한다.

그럼 이걸 왜 사용하는가?
다음과 같이 복사 [[오버헤드 (Overhead) 현상]]을 일으키는 경우를 살펴보자.
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

이에 대한 ChatGPT의 설명은 다음과 같다.
![[스크린샷 2023-05-23 오후 4.48.34.png]]

단지 함수의 정의 부분에서 `MyClass`의 형을 가진 인수를 선언했을 뿐인데 호출하지도 않은 함수가 실행되는 것을 알 수 있다.
여기서 `MyObject obj`라는 선언 자체만으로도 `MyObject`가 객체화되며 그에 속한 메서드들이 호출된다.
코드의 결과를 보아도 알 수 있는 사실이다.
```unix
$ cd "/Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/PANIM_DB/ScratchPads/" && g++ -std=c++14 ScratchPad_02.cpp -o ScratchPad_02 && "/Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/PANIM_DB/ScratchPads/"
ScratchPad_02
MyObject 생성자 호출
MyObject 복사 생성자 호출
myFunction 호출
```

처음 `obj1`을 객체화할 때는 생성자만 호출되고 끝났지만, 
다음 `myFunction`이 호출되는 시점에 그 인수 `MyObject obj`가 기존의 `obj1`을 복사하여 복사생성자를 호출시키기 때문이다.

이는 복사 오버헤드 때문이다.
무브 시멘틱은 임시 변수 생성 시와 메모리 복사 시 바로 이 오버헤드를 감소시키기 위해 사용한다.
이를 통해 성능 향상을 꾀할 수 있다.
앞서 보았듯, 단순히 값을 복사하는 행위가 아니라 값을 옮기는 행위이기 때문이다.

그럼 이 예제를 무브 시멘틱을 이용하여 수정하면 어떨까?
```cpp
#include <iostream>

using namespace std;

class MyObject {
public:
    MyObject() {
        std::cout << "MyObject 생성자 호출" << std::endl;
    }
    
    MyObject(const MyObject& other) {
        std::cout << "MyObject 복사 생성자 호출" << std::endl;
    }
    
    MyObject(MyObject&& other) {
        std::cout << "MyObject 이동 생성자 호출" << std::endl;
    }
    
    MyObject& operator=(const MyObject& other) {
        std::cout << "MyObject 복사 할당 연산자 호출" << std::endl;
        return *this;
    }
    
    MyObject& operator=(MyObject&& other) {
        std::cout << "MyObject 이동 할당 연산자 호출" << std::endl;
        return *this;
    }
};

void myFunction(MyObject&& obj) {
    std::cout << "myFunction 호출" << std::endl;
}

int main() {
    MyObject obj1; // MyObject 생성자 호출
    myFunction(std::move(obj1)); // myFunction 호출, 이동 시멘틱 적용
    
    return 0;
}
```

그러면 다음과 같이 복사생성자가 호출되지 않는 결과를 확인할 수 있다.
```unix
$ cd "/Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/PANIM_DB/ScratchPads/" && g++ -std=c++14 ScratchPad_02.cpp -o ScratchPad_02 && "/Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/PANIM_DB/ScratchPads/"
ScratchPad_02
MyObject 생성자 호출
myFunction 호출
```

하지만 단순하게 복사 대신 참조를 통해 수정하는 방법도 있다.
```cpp
// 다음은 무브 시멘틱을 통해 복사 오버헤드를 피하도록 수정한 예제이다.  
  
#include <iostream>  
  
using namespace std;  
  
class MyObject {  
	public:  
	MyObject() {  
		std::cout << "MyObject 생성자 호출" << std::endl;  
	}  
	  
	MyObject(const MyObject& other) {  
		std::cout << "MyObject 복사 생성자 호출" << std::endl;  
	}  
};  
  
void myFunction(MyObject& obj) {  
	std::cout << "myFunction 호출" << std::endl;  
}  
  
int main() {  
	MyObject obj1; // MyObject 생성자 호출  
	myFunction(obj1); // myFunction 호출, 복사 오버헤드 발생  
	  
	return 0;  
}
```

그러면 다음과 같은 결과가 나온다.
```unix
$ cd "/Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/PANIM_DB/ScratchPads/" && g++ -std=c++14 ScratchPad_02.cpp -o ScratchPad_02 && "/Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/PANIM_DB/ScratchPads/"
ScratchPad_02
MyObject 생성자 호출
myFunction 호출
```

복사 오버헤드가 발생하지 않는 것을 볼 수 있다.
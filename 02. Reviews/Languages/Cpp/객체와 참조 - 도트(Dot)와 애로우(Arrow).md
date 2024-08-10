

2023-05-19

----
#CS #Cpp #닷넷정복 #메모리 #객체 #객체지향 #참조

## 개요
[[01. OracleConnection.cpp]]을 구현하던 도중, `->`라는 표현을 발견했고, 이것이 도트(Dot) 즉, `.`와 비교되어 이해할 수 있다는 걸 알게 되었다.

## 내용
다음 예제부터 살펴 보자.
```cpp
#include <iostream> // string class 등을 담고 있는 헤더파일  
  
using namespace std;  
  
struct MyStruct  
{  
	int age = 20;  
	int num = 2000;  
	const char* name = "김김김";  
};  

class MyClass{  
public:  
	int age = 10;  
	int num = 1000;  
	const char* name = "홍길동";  
};  
  
int main(int argc, const char * argv[])  
{  
	MyStruct myStruct1;  
	MyStruct* myStruct2 = new MyStruct;  
	myStruct2->num = 3000;  
	  
	MyClass myClass1;  
	myClass1.num = 4000;  
	  
	MyClass* myClass2 = new MyClass();  
	  
	cout << myStruct1.num << endl; // 2000
	cout << myStruct2->num << endl; // 3000
	cout << myStruct2 << endl; // 0x121606b80
	cout << myClass1.num << endl; // 4000
	cout << myClass2->num << endl; // 1000
	  
	return 0;  
}
```

출처: https://dbstndi6316.tistory.com/386

이를 실행하면 다음과 같이 출력된다.
```unix
$ cd "/Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/PANIM_DB/ScratchPads/
" && g++ -std=c++14 ScratchPad_04.cpp -o ScratchPad_04 && "/Users/changjoonlee/GitHub/Pro
ject_PANIM_GitHub/CppMyPractice/PANIM_DB/ScratchPads/"ScratchPad_04
2000
3000
0x121606b80
4000
1000
```

위의 예제에서 알 수 있듯이, `myStruct2`는 포인터이다. 
[[객체와 참조]]에서 `int* a = b;`와 같은 정의에서 봤듯이, `MyStruct*`는 일종의 형(Type)이며, `MyStruct`의 포인터 변수이다.
`*MyStruct myStruct2 = new MyStruct;`는 우변에서 동적할당/객체화를 한 뒤 그것의 주소 값을 포인터 변수에 집어넣는 행위였던 것이다.

그러면 이 도트와 애로우가 포인터와 어떤 관계를 가지는지 알아보자.
다음은 내가 만든 예제이다.
```cpp
#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일  
  
using namespace std;  
  
class MyClass{  
public:  
	void myFunction()  
	{  
		cout << "Hello World" << endl;  
	}  
};  
  
int main(int argc, const char *argv[])  
{  
	MyClass* p_myClass = nullptr;  
	p_myClass = new MyClass;  
	  
	//p_myClass.myFunction(); // expression must have class type but it has type "MyClass *"C/C++(153)  
	p_myClass->myFunction();  
	  
	return 0;  
}
```

아주 간단한 클래스를 만들어 보았다.
앞서 설명했다시피 도트는 직접 접근이다.
하지만 `p_myClass`에 담긴 것은 객체화된 `MyClass`의 주소 값이기에 직접 접근해도 거기엔 포인터만 있을 뿐 `myFunction()` 같은 건 없다.
하지만 애로우를 서서 주소로 접근하는 건 가능하다.
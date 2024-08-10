

2023-05-30

----
#Cpp #CS #수업 #C #객체지향 #객체

## 개요
우리는 이제 기본적인 언어에서 추가된 C++문법을 배워보려고 한다.

## 내용
```cpp
//  
// Created by Changjoon Lee on 2023/05/30.  
//  
#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일  
  
using namespace std;  
  
struct Point  
{  
	int x;  
	int y;  
	  
	void ShowPoint()  
	{  
		printf("%d, %d\n", x, y);  
	}  
};  
  
int main(int argc, const char *argv[])  
{  
	Point pt = {10, 20};  
	pt.ShowPoint();  
	  
	return 0;  
}
```

구조체에 함수를 넣어서 쓸 수 있게 되었다.
그럼에도 (우리가 언리얼에서도 보았듯이) `struct`는 변수들의 집합으로 주로 쓰인다.

C언어에서는 `<<`는 연산자(해당 값을 해당 위치만큼 이동하는 연산자)였는데 C++에서는 연산자의 의미를 재정의할 수 있다. → 연산자 오버로딩!
마치 함수처럼 사용할 수 있다는 것이다.
```cpp
#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일  
  
using namespace std;  
  
int main(int argc, const char * argv[])  
{  
	int num0, num1;  
	cout << "정수 2개 입력" << endl;  
	cin >> num0 >> num1;  
	cout << num1 << " + " << num1 << " = " << num0 + num1 << endl;  
	  
	return 0;  
}
```

콘솔 아웃(`cout`)이라는 객체에다가 이 값을 전달한다는 의미이다.
그리고 콘솔 상에서의 개행 문자인 `endl`이 마지막에 붙은 것을 볼 수 있다.

다음은 네임스페이스에 대해 알아볼 것이다.
```cpp
#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일  
  
using namespace std;  
  
namespace A  
{  
	int num0, num1;  
  
	int Sum()  
	{  
		return num0 + num1;  
	}  
}  
  
namespace B  
{  
	float num0, num1;  
	  
	float Sum()  
	{  
		return num0 + num1;  
	}  
}  
  
int main(int argc, const char *argv[])  
{
	A::num0 = 100, A::num1 = 200;
	cout << A::Sum() << endl;
	
	B::num0 = 3.14f, B::num1 = 2.5f;
	cout << B::Sum() << endl;
	
	return 0;  
}
```

`class`만 가지고 그룹화시켰을 때 중복되는 클래스명이 많아졌다.
그래서 상위 분류 개념은 `namespace`를 나중에 C++ 문법에 추가시켰다.
Java는 `namespace`를 `package`로 변화시키고, C#은 `namespace`를 그대로 사용한다.

그러면 이 `namespace`를 `using` 문을 통해 사용하는 법을 알아보자.
```cpp
#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일  
  
using namespace std;  
  
int main(int argc, const char * argv[])  
{  
	int num0, num1;  
	  
	cout << "2개의 정수 입력 >> ";  
	cin >> num0 >> num1;  
	cout << num0 << " + " << num1 << " = " << num0 + num1 << endl;  
	  
	return 0;  
}
```

위와 같이 간단하게 해주는 방법이 있고, 좀 세밀하게 다음과 같이 지정해주는 방법도 있다.
```cpp
using std::cin;
using std::cout;
using std::endl;
```

C++에 이르러 산술형값과 논리형값의 혼재를 해결하게 되었다.
즉, 참이면 `1`, 거짓이면 `0`이 되는 식의 값을 가지는 것을 `true`와 `false`로 쓸 수 있게 했다.
하지만 기존에 쓰던 관습대로 `1` 또는 `0`을 쓰는 것도 계속 가능하다.
C언어와의 연계를 위해서 그렇게 쓰는 것을 남겨둔 것이다.
```cpp
#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일  
  
using namespace std;  
  
int main(int argc, const char * argv[])  
{  
	int num = 0;  
	cout << "정수 입력 : ";  
	cin >> num;  
	  
	bool b = num > 0;  
	if (b)  
	{  
		cout << num << "은 0보다 큽니다" << endl;  
	}  
	else
	{  
		cout << num << "은 0이거나 작습니다" << endl;  
	}  
	  
	cout << "sizeof(bool) : " << sizeof(bool) << endl;  
	
	b = true;
	cout << b << endl;
	b = false;
	cout << b << endl;
	
	return 0;  
}
```

Java에서는 이러한 애매함이 설계/논리의 정합성을 해친다고 판단하여 산술형과 논리형을 엄격하게 구분하였다.
C#도 이 부분은 Java와 동일하게 가져갔다.

우리는 앞서 포인터를 이용하여 다음 예제를 이해해 보았다.
```cpp
#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일  
  
using namespace std;  
  
void GetHourMinSec(int sec, int* p_Hour, int* p_Min, int* p_Sec)  
{  
	*p_Sec = sec % 60;  
	*p_Min = sec / 60;  
	*p_Hour = *p_Min / 60;  
	*p_Min = *p_Min % 60;  
}  
  
int main(int argc, const char * argv[])  
{  
	int hour, min, sec;  
	GetHourMinSec(3720, &hour, &min, &sec);  
	cout << hour << ":" << min << ":" << sec << endl;  
	GetHourMinSec(4836, &hour, &min, &sec);  
	cout << hour << ":" << min << ":" << sec << endl;  
	  
	return 0;  
}
```

C++에 이르러 다음과 같이 표현할 수 있게 되었다.
```cpp
#define ref

#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일  
  
using namespace std;  
  
void GetHourMinSec(int sec, int& rHour, int& rMin, int& rSec)  
{  
	rSec = sec % 60;  
	rMin = sec / 60;  
	rHour =rMin / 60;  
	rMin = rMin % 60;  
}  
  
int main(int argc, const char * argv[])  
{  
	int hour, min, sec;  
	GetHourMinSec(3720, ref hour, ref min, ref sec);  
	cout << hour << ":" << min << ":" << sec << endl;  
	GetHourMinSec(4836, ref hour, ref min, ref sec);  
	cout << hour << ":" << min << ":" << sec << endl;  
	  
	return 0;  
}
```

다음 예제를 통해 위 코드를 이해해 보고자 한다.
```cpp
#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일  

using namespace std;  
  
int main(int argc, const char * argv[])  
{  
	int num = 100;  
	int* ptr = &num;  
	int& rNum = num;  
	  
	cout << num << endl << *ptr << endl << rNum << endl;  
	num = 999;  
	cout << num << endl << *ptr << endl << rNum << endl;  
	num = 1234;  
	cout << num << endl << *ptr << endl << rNum << endl;  
	num = 9876;  
	  
	return 0;  
}
```

출력값을 보면 다음과 같다.
```unix
$ cd "/Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/CppLec/_001_CExtend/" && g++ -std=c++14 _09_ref.cpp -o _09_ref && "/Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/CppLec/_001_CExtend/"_09_ref
100
100
100
999
999
999
1234
1234
1234
```

`int* ptr = &num;`에서는 포인터에 `num`의 주소값을 넣어준다.
프로그래머들은 해당 변수가 참조를 받아오는 것이라는 의미를 명확하게 하고자 `#define ref`로 설정한 `ref` 값을 변수 앞에 써 주기도 한다.
이는 일종의 꼬리표이다.

다음과 같은 표현은 `rNum1`이 참조하는 값이 없어서 `int& rNum1;`라고 선언하는 부분에 컴파일 에러가 난다.
```cpp
#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일  
  
using namespace std;  
  
int main(int argc, const char * argv[])  
{  
	int num = 100;  
	int& rNum = num;  
	cout << rNum << endl;  
	  
	int& rNum1;  
	int& rNum1 = num;  
	cout << rNum1 << endl;  
	  
	return 0;  
}
```

다음은 함수의 파라미터 값에 따라서 적절한 함수를 찾아서 작동시켜주는 함수 오버로딩에 대한 예제이다.
```cpp
#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일  
  
using namespace std;  
  
int Add(int num0, int num1)  
{  
	return num0 + num1;  
}  
  
double Add(double num0, double num1)  
{  
	return num0 + num1;  
}  
  
int main(int argc, const char * argv[])  
{  
	cout << Add(10, 28) << endl;  
	cout << Add(10.1, 28.1) << endl;  
	  
	return 0;  
}
```

원래 C언어에서는 함수가 같은 이름을 가지면 에러가 났다.
하지만 여기선 이름이 같아도 파라미터의 형이 다르기에 다른 함수로 취급된다.

다음은 디폴트 파라미터에 대한 예제이다.
```cpp
#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일  
  
using namespace std;  
  
int Add(int num0 = 10, int num1 = 20)  
{  
	return num0 + num1;  
}  
  
int main(int argc, const char * argv[])  
{  
	cout << Add(10, 28) << endl;  
	cout << Add() << endl;  
	  
	return 0;  
}
```

이를 실행하면 다음과 같은 값이 출력된다.
```unix
$ cd "/Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/CppLec/_001_CExtend/" && g++ -std=c++14 _12_defaultParam.cpp -o _12_defaultParam && "/Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/CppLec/_001_CExtend/"_12_defaultParam
38
30
```

이제 `inline` 키워드를 살펴볼 것이다.
원래 우리가 `#define`을 통해 정의한 매크로 함수는 전처리기에 의해 치환됨으로 인해 함수를 호출할 필요가 없어서 속도가 빠르다는 장점이 있었다.
하지만 이것이 복잡한 코드를 구현할 때는 원하지 않는 값이 출력되는 문제가 있었다.
이 단점을 해결하고 장점은 그대로 취하고자 `inline` 키워드가 고안된 것임.
```cpp
#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일  
  
#define ADD(x, y) ((x) + (y))  
  
using namespace std;  
  
inline int Add(int a, int b)  
{  
	return a + b;  
}  
  
int main(int argc, const char * argv[])  
{  
	cout << Add(10, 20) << endl;  
	cout << ADD(100, 200) << endl;  
	  
	return 0;  
}
```

다만 주의할 점은, `inline`을 사용해도 컴파일러에 의해 그것이 정말 실행될지 결정이 된다.
```cpp
#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일  
  
using namespace std;  
  
#define ARR_NUM 5  
int main(int argc, const char * argv[])  
{  
	int* num = new int[ARR_NUM];  
	for (int i = 0; i < ARR_NUM; i++)  
	{  
		num[i] = i * 10;  
	}  
	for (int i = 0; i < ARR_NUM; i++)  
	{  
		cout << num[i] << ", ";  
	}  
	cout << endl;  
	  
	delete[] num;  
	  
	return 0;  
}
```

`new`는 힙 공간에 동적할당을 하는 연산자이다.
이 할당한 메모리는 `delete`를 통해 해제해 줘야 한다.
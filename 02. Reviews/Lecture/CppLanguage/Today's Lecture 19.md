

2023-06-01

----
#Cpp #CS #수업 #C #객체지향 #객체

## 개요
어제 복사생성자까지 배워 보았다.

## 내용
`private`에 `static` 변수를 선언하면 아래와 같이 객체화 이전에 초기화를 해줄 수 있고, 이 클래스에서 객체화된 모든 객체들이 이 하나의 변수만 공동으로 사용하도록 만들어줄 수 있다.
```cpp
#include <iostream>  
  
using namespace std;  
  
class Theater  
{  
private:  
	// 클래스 소속으로 모든 객체가 공유하는 변수  
	static int enterCount;  
	char name[30];  
	int seat;  
	  
public:  
	Theater(const char *p_Name, int _seat)  
	: seat(_seat)  
	{  
		strncpy(name, p_Name, strlen(p_Name));  
	}  
	~Theater() {}  
	  
public:  
	static void AddEnterCount()  
	{  
		enterCount++;  
	}  
	  
	void ShowTheater()  
	{  
		cout << "극장명 : " << name << endl;  
		cout << "좌석수 : " << seat << endl;  
	}  
};  
  
int Theater::enterCount = 0; // 객체가 만들어지기 전의 static 변수의 초기화.  
  
int main(int argc, const char *argv[])  
{  
	Theater th0("종로 CGV", 3500);  
	th0.ShowTheater();  
	Theater th1("명동 CGV", 3500);  
	th1.ShowTheater();  
	Theater th2("종각 CGV", 3500);  
	th2.ShowTheater();  
	  
	return 0;  
}
```

위의 메인함수에서 `private`이고 `static`인 변수에 접근할 때 직접접근이 아니라 `ShowTheater()` 메서드를 통해 간접적으로 접근하는 것을 볼 수 있다.

다음은 `this`를 사용하는 예제이다.
```cpp
#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일  
  
using namespace std;  
  
class Point  
{  
public:  
	int x, y;  
	  
public:  
	void SetPoint(int x, int y)  
	{  
		this->x = x;  
		this->y = y;  
	}  
	
	void ShowPoint()  
	{  
		cout << "x : " << this->x << ", y : " << this->y << endl;  
	}  
};  
  
void ShowPoint(Point* pt)  
{  
	cout << "x : " << pt->x << ", y : " << pt->y << endl;  
}  
  
int main(int argc, const char *argv[])  
{  
Point pt0, pt1;  
  
	pt0.SetPoint(10, 20);  
	pt1.SetPoint(100, 200);  
	  
	pt0.ShowPoint();  
	pt1.ShowPoint();  
	  
	cout << "---------------------------" << endl;  
	ShowPoint(&pt0);  
	ShowPoint(&pt1);  
	  
	return 0;  
}
```

멤버변수의 경우, 그 특정한 객체에만 속하게 된다.
반면 멤버함수의 경우 동일한 클래스에서 파생된 모든 객체는 첫번째 객체화된 객체의 멤버함수를 공유한다.
`pt0.SetPoint(10, 20);`와 같이 우리가 써주긴 했지만, 컴파일러에 의해 메모리에 올라갈 때는 `SetPoint(Point* this, int x, int y);`와 같이 올라간다.
우리는 같은 메서드를 쓰기 때문에 그 안에 들어가는 멤버변수가 `SetPoint` 메서드를 호출한 객체의 멤버변수임을 명시하고자 `this`라고 가리켜 주는 것이다.

그러면 메인함수에서 다음과 같이 해주면...
```cpp
cout << "---------------------------" << endl;
cout << "&pt0 : " << &pt0 << endl;
pt0.PointThis();
cout << "---------------------------" << endl;
cout << "&pt1 : " << &pt1 << endl;
pt1.PointThis();
```

다음과 같은 결과가 나온다.
```cpp
---------------------------
&pt0 : 0x16eee6f88
this : 0x16eee6f88
---------------------------
&pt1 : 0x16eee6f80
this : 0x16eee6f80
```

만약 `SetPoint` 내에서 `this->x = x`가 아니라 `x = x`와 같이 코딩하면 어떻게 될까?
그러면 지역변수에 지역변수를 대입하는 상황이 벌어진다.
왜냐하면 지역변수가 멤버변수보다 우선순위가 높기 때문이다.
그러므로 멤버변수와 동일한 이름의 멤버변수가 있을 때는 `this`를 써야 한다.

이제 상속에 대해서 알아볼 것이다.
C++에서 접근권한은 세가지이다.
가장 기본적인 두가지는 `private`과 `public`이 있다.
전자는 클래스의 멤버만 다른 `private` 멤버를 접근 가능하고, 외부에서는 안된다.
후자는 모든 곳에서 접근 가능하다.

그러면 세번째 것은?
바로 `protected`라는 것이다.
이는 자식 클래스가 생겼을 경우 부모의 멤버를 접근하게 해주는 것이다.
자식에게는 `public`으로 동작하고 외부에서는 `private`을 가지고 동작한다.

C++에서의 상속은 `public`, `private`, `protected` 세가지이다.
하지만 대부분 `public`만 사용한다.
Java, C#은 `public` 상속만을 지원한다.
```cpp

```

상속받은 자식 객체 내부에는 부모 객체가 포함된다.
그러므로 부모 객체가 먼저 생성된 후 자식 객체의 구현부분이 만들어지므로 부모 생성자 호출 후 생성자 호출이 이루어진다.

자식 생성자에서 부모 생성자의 초기화를 하려면 멤버 이니셜라이즈 영역에 부모 생성자 초기화를 한다.

여기서 잠시 객체지향의 3대 속성을 복기하고 가자.
1. 캡슐화 (Encapsulation, has ~ a): 외부의 접근을 제한하는 방식으로 클래스를 설계한다.
2. 상속성 (Inheritance, is ~ a/is kind of): 기능의 확장 및 재사용을 위해 원래 있던 클래스를 물려 받아 사용한다.
3. 다형성 (Polymorphism): 같은 부모 클래스에서 나왔더라도 부모와 다른 형태를 가질 수 있다.
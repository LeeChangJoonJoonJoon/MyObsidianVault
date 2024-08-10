

2023-05-31

----
#Cpp #CS #수업 #C #객체지향 #객체

## 개요
오늘은 C++에 새롭게 도입된 클래스에 대해 알아볼 것이다.

## 내용
다음은 `struct`와 `class`의 차이를 보여주는 예제이다.
```cpp
#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일  
  
using namespace std;  
  
struct Point0  
{  
	int x, y;  
	void ShowPoint()  
	{  
		cout << "x: " << x << ", y: " << y << endl;  
	}  
};  
  
class Point1  
{  
public:  
	int x, y;  
	void ShowPoint()  
	{  
		cout << "x: " << x << ", y: " << y << endl;  
	}  
};  
  
int main(int argc, const char *argv[])  
{  
	Point0 pt0;  
	Point0 pt1;  
	  
	pt0.x = 10;  
	pt0.y = 10;  
	pt0.ShowPoint();  
	  
	pt1.x = 10;  
	pt1.y = 10;  
	pt1.ShowPoint();  
	  
	return 0;  
}
```

`Point1`의 `public:`이라는 접근지정자를 지우면 작동하지 않는 것을 볼 수 있다.
왜냐하면 C++에선 접근지정자가 없으면 기본적으로 `private`이기 때문이다.
C++에서는 `struct`는 주로 변수들의 집합으로, `class`는 우리가 잘 아는 그 객체로 보아 많이 쓴다.

다음은 클래스의 선언부와 구현부를 나누어 놓는 예제이다.
```cpp
#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일  
  
using namespace std;  
  
class Point  
{  
public:  
	int x, y;  
	void ShowPoint();  
};  
  
void Point::ShowPoint()  
{  
	cout << "x: " << x << ", y: " << y << endl;  
}  
  
int main(int argc, const char *argv[])  
{  
	Point pt1;  
	  
	pt1.x = 10;  
	pt1.y = 10;  
	pt1.ShowPoint();  
	  
	return 0;  
}
```

만약 `Point::`와 같이 `ShowPoint` 앞에 써주지 않았으면 그냥 전역 함수로 취급된다.

C++에서 클래스에서 객체를 생성할 때는 `Point pt;`와 같이 하면 지역변수로서 스택에 객체가 올라간다.
그리고 `Point* pt = new Point();`와 같이 하면 동적할당으로서 힙에 객체가 올라간다.
반면 C#과 같은 언어에서는 전부 힙에 올라갔다.
다만 후자와 같이 동적할당 후엔 `delete`로 메모리 해제를 해줘야 한다.
다음 살펴볼 예제는 소멸자에서 메모리 해제를 해주는 것을 보여준다.

전자와 같이 메인함수에서 선언했을 경우, 그 객체에 대해 소멸자를 만들어 두면 는 메인함수가 끝나면 자동으로 스택에서 해제된다.
하지만 동적할당을 했을 경우, 소멸자에 동적할당될 멤버에 대해 `delete`를 써줘야 한다. 

다음 코드에서는 전달되는 이름마다 길이가 다르므로 그냥 배열을 사용하지 않고 여기서는 이름의 길이 + 1만큼의 동적배열을 생성하였다.
+1을 한 이유는 마지막에 `NULL` 값을 넣어줘야 하기 때문이다.

먼저 헤더파일.
```cpp

```

위 코드에서 `Human any = hong;`으로 복사생성자가 호출되고, 컴파일러 내부에선 `Human any(hong)`으로 바뀌어서 이해된다.
우선 `hong`과 같은 객체들은 동적할당이 아닌 방식으로 객체화되었기에 스택에 생성되었다.
그리고 `any`도 마찬가지로 스택에 생성되었다.
하지만 우린 이미 이 클래스 내부에 `p_Name`을 통해 동적할당을 해줬다.
그래서 스택 내에서 힙의 배열을 가리키는 포인터 `p_Name`까지 `any`가 복사해 갔다.

`hong`이 메인함수 실행이 끝남으로서 메모리에서 해제될 시점에 소멸자에서 구현한 `delete` 구문에 의해 힙에 있는 배열이 해제되었다.
하지만 바로 다음에 `any`에서 또 한번 소멸자의 `delete`가 작동하게 되고, 없는 배열을 가리키는 포인터 `p_Name`은 해제할 대상을 잃은 상태이기에 다음과 같은 메시지가 뜬 것이다.

이를 해결하는 방법이 있는데, 그것이 바로 복사생성자를 만드는 것이다.
```cpp
Human(Human& human);
```

이렇게 만들어 주면, 컴파일러가 만든 디폴트 복사생성자가 호출되어서 `Human any = hong;`로 읽히는 게 아니라, 우리가 만들어준 복사생성자가 호출되어 `any`의 멤버 포인터 `p_Name`이 가리킬 새 객체가 자동으로 힙에 동적할당된다.
즉, `any`가 해제할 힙 객체가 만들어져서 위에서 우리가 했던 식으로 (이미 메모리에서 해제되어서) 없는 객체를 삭제하는 일은 없게 된다.

생성자를 만드는 다른 방법을 알아보자.
```cpp
Human::Human()
	: p_Name(NULL), age(0), height(0)
{
	memset(_job, 0, sizeof(_job));
}
```

이렇게 표현할수도 있으니 알아두자.


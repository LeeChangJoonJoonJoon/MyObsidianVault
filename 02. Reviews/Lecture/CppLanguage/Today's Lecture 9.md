

2023-05-16

----
#Server #수업 #CSharp 

## 개요
오늘은 클라이언트 상에서 어떻게 서버와 연결되는지에 대해 배울 것이다.

## 내용
유니티에서는 메인 스레드 외의 스레드로의 직접 접근을 허용하지 않는다.
근데, 우리가 이런 클라이언트와 연결하고자 하는 서버는 멀티스레드로 작동한다.
그러면 어떻게 할까?

유니티에서 나온 정보를 큐를 만들어서, 하나씩 전달해 준다.
그 역할을 하는 클래스가 `CFreeNetEventMangaer`이다.
여기선 큐를 만들고 이 안에 있는 객체를 넣고 빼는 작업을 한다.
내부에는 엔큐와 디큐 처리를 해주는 메서드들이 있다.

큐는, 구체적으로 여기서는 `NetworkEvent`와 `CPacket`이 있다.
```cs
Queue<NetorkEvent> network_events;
Queue<CPacket> network_message_events;
```

상황에 따라 스택을 이용할 수도 있다.
(C++에서는 이게 STL이라는 라이브러리에 있다.)

여기 내의 `CConnector`는 서버와 접속을 하는 역할의 객체이다.
서버와 마찬가지로 비동기 호출을 사용하기에 `pending`의 참 거짓 여부로 직접 접근할 수 있는 기능을 넣어줬다.

그리고 `CFreeNetUpdateService` 클래스에 가면, 프리넷에 있는 패킷이나 상태정보가 존재하면 그 안의 데이터를 `Update` 문에서 꺼내는 역할을 한다.
`FreeNet` 내부에는 `IPeer` 내의 `peer`에서 값을 가져와서 `on_message`에 담는다.
그 메서드 내에서는 패킷으로 데이터를 변환하여 큐에 저장한다.

----

### C++ 수업
언어의 탄생 배경을 잠시 살펴보자.

태초에 기계어가 있었다. 
이는 이진수로 구성된 것이다

그리고 이것을 기반으로 한 어셈블리어가 있다.

어셈블리어를 함수를 기반으로 동작하게 만든 게 C 언어이다.
여기서 객체지향을 추가한게 C++.
이 객체지향의 영향을 받아서 더 현대적으로 만들어진 것이 C#과 Java이다.

여기서 C++ 컴파일러 안에서 C 컴파일러가 있다.
우리가 수업 때 배우는 것은 C 언어부터이다.
왜냐하면, C++ 언어의 배경을 알면 더 근본적인 접근이 가능하기 때문이다.

```c
#include <stdio.h>

int main(int argc, const char * argv[])
{
	printf(HelloWorld\n);
	return 0;
}
```

`printf`함수를 사용하려면 컴파일러한테 함수의 원형정보와 함수를 전달해야 한다.
함수의 원형정보는 `stdio.h`에 들어 있고 함수 자체는 dll 파일 내부에 들어 있다.
어쨌든, `printf` 함수를 쓰기 위해 추가해 주는 거라고 생각하면 된다.

C 컴파일러는 언제나 `main` 함수를 찾아서 진입점을 찾는다.
참고: Xcode 상에서는 `int main`과 `return 0`대신 `void main`을 써도 된다.
```c
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int a, b, c;  
	  
	a = 10;  
	b = 20;  
	c = 30;  
	  
	printf("%d %d %d", a, b, c);  
	return 0;  
}
```

`%d`는 뒤에 따르는 변수들을 순서대로 출력되도록 하는 서식문자이다.
그리고 `d`라는 표기는, 10진수로 출력하라는 뜻이다.

```c
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	char a = 'A';  
	char str[6] = "Korea";  
	printf("%c, %s\n", a, str);  
	return 0;  
}
```

```c
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int a = 0;  `
	float b = 3.14f;  
	double c = 4.23424;  
	char d = 'Z';  
	char str[6] = "Korea";  
	  
	printf("%d %f %1f %c %s\n", a, b, c, d, str);  
	printf("size of a : %d %d\n", sizeof(int), sizeof(a));  
	printf("size of a : %d %d\n", sizeof(float ), sizeof(b));  
	printf("size of a : %d %d\n", sizeof(double), sizeof(c));  
	printf("size of a : %d %d\n", sizeof(char ), sizeof(d));  
	printf("size of a : %d\n", sizeof(str));  
	return 0;  `
}
```

`sizeof`라는 연산자를 통해 자료형의 크기를 반환하는 연산자이다.
이는 다음과 같이 바이트의 크기를 출력해 준다.
```cc
/Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/_04_sizeof
0 3.140000 4.234240 Z Korea
size of a : 4 4
size of a : 4 4
size of a : 8 8
size of a : 1 1
size of a : 6

Process finished with exit code 0
```

이 경우 실행은 되지만, 컴파일러가 다음과 같은 메시지를 던진다.
```cc
cc -o _04_sizeof /Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/CLec/_04_sizeof.c
/Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/CLec/_04_sizeof.c:12:35: warning: format specifies type 'int' but the argument has type 'unsigned long' [-Wformat]
    printf("size of a : %d %d\n", sizeof(int), sizeof(a));
                        ~~        ^~~~~~~~~~~
                        %lua
```

우리가 쓰는 함수 `printf`에서의 `f`는, 출력을 하는 데에 있어서 `%`로 시작하는 서식과 인수(변수)를 대응시켜 조립(Formatting)하는 과정을 거친 뒤 문자열로 출력한다는 뜻이다.
그래서 여러가지 옵션이 있다.
![[스크린샷 2023-05-16 오후 1.30.33.png]]

다음과 같이 바꿔주면 된다.
```c
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int a = 0;  
	float b = 3.14f;  
	double c = 4.23424;  
	char d = 'Z';  
	char str[6] = "Korea";  
	  
	printf("%d %f %1f %c %s\n", a, b, c, d, str);  
	printf("size of a : %lu %lu\n", sizeof(int), sizeof(a));  
	printf("size of a : %lu %lu\n", sizeof(float ), sizeof(b));  
	printf("size of a : %lu %lu\n", sizeof(double), sizeof(c));  
	printf("size of a : %lu %lu\n", sizeof(char ), sizeof(d));  
	printf("size of a : %lu\n", sizeof(str));  
	return 0;  
}
```

위 코드에서, `unsigned int e = -10;`를 추가하면, 이를 출력했을 때 양수 10이 출력된다.
`int` 자료형과 같이 정수 자료형에 해당되는 것들은 4바이트 중 맨 앞의 비트가 부호로 쓰이지 않고 숫자 값으로 쓰고 싶을 때 `unsigned`를 앞에 붙여 주면된다.
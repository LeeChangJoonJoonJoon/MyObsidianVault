

2023-05-19

----
#C #지선생 #ChatGPT #수업 #문제 #문제해결 

## 개요
어제까지 C 언어의 함수에 대해 공부하였다.

## 내용
```c
//  
// Created by Changjoon Lee on 2023/05/19.  
//  
#include <stdio.h>  
  
int num = 100;  
  
int getNum()  
{  
	int num = 10;  
	return num;  
}  
  
int getGNum()  
{  
	return num;  
}  
  
int main(int argc, const char * argv[])  
	{  
	int ret = getNum();  
	printf("%d\n", ret);  
	ret = getGNum();  
	printf("%d\n", ret);
	
	return 0;
}
```

위의 `return num;`과 같이 변수를 반환해주면 메모리 상의 레지스터라는 곳에 임시로 저장을 해놓고, 이것을 받아주는 `int ret = getNum();`와 같은 게 없으면 그 레지스터에서 삭제한다.

컴파일러는 코드를 기계어로 바꾸기 위해 전역변수 `int = 100;`을 전역영역에 올린다.
다음에는 메인함수에서 선언한 `ret` 변수가 먼저 스택 공간에 올라간다.
이제 `getNum()`이 실행되어서 그 변수 내에 10이라는 수를 스택 내 `ret` 위치에 저장한다.
이때 CPU에 소속된 레지스터에 10이라는 값을 올린다.
그리고 메인함수가 실행될 때 스택 공간에 위치한 `ret`에 값을 전해주고 레지스터에선 사라진다.

지역변수 내의 구역에 들어오면, 지역변수에 저장된 값이 우선된다.
이 때, 전역변수에서 같은 이름의 변수에 저장되어 있던 값은 무시된다.
다만 지역변수 밖으로 나왔을 때는 전역변수에 저장했던 값이 우선된다.

다음은 전역변수를 사용하는 예제이다.
```c
//  
// Created by Changjoon Lee on 2023/05/19.  
//  
#include <stdio.h>  
  
// countFunc 함수의 호출횟수 저장.  
int count = 0;  
  
void countFunc()  
{  
	count++;  
}  
  
void callNumFunc()  
{  
	printf("함수 호출 횟수는 %d입니다.\n", count);  
}  
  
int main(int argc, const char * argv[])  
{  
	for (int i = 0; i < 10; ++i) {  
		countFunc();  
	}  
	  
	callNumFunc();  
	  
	return 0;  
}
```

여기서 `count`는 지역변수이기 때문에 `countFunc`이 호출될 때 Stack에 생성되고 `return count`할 때 레지스터의 임시변수에 값을 전달하고 Stack에서 소멸된다.
다음에 `countFunc`이 호출될 때 다시 위와 같은 작업을 반복한다.

결과로는 다음이 나온다.
![[스크린샷 2023-05-19 오전 11.14.37.png]]

다만 이렇게 사용하게 되면, 어느 함수에서나 가져다 사용할 수 있다.
이 전역변수를 남발하면 함수 간의 모듈화를 방해한다.
따라서 유지보수가 어려워진다.

이런 맥락에서 Java/C#은 아예 전역변수를 없앴다.
Python/Javascript는 편의성을 중요시하기에 전역변수가 있다.

반면 지역변수로 하면 어떤가?

다음과 같이 전역변수를 `countFunc`의 지역변수로 이동시켜 주었다.
```c
//  
// Created by Changjoon Lee on 2023/05/19.  
//  
#include <stdio.h>  
  
void countFunc()  
{  
	int count = 0;  
	count++;  
}  
  
void callNumFunc()  
{  
	printf("함수 호출 횟수는 %d입니다.\n", count);  
}  
  
int main(int argc, const char * argv[])  
{  
	for (int i = 0; i < 10; ++i) {  
		countFunc();  
	}  
	  
	callNumFunc();  
	  
	return 0;  
}
```

여기 `councFunc` 함수의 지역변수는 메인함수에서 `for`문에 의해서 호출 될 때마다 새롭게 지역변수를 초기화하고 영역이 끝나면 소멸시킨다.
이 지역 변수는 지역 내에서만 사용가능하다.
함수나 `{}` 영역에 들어서면 Stack에 올라갔다가, 해당 지역이 끝나면 Stack에서 소멸한다.
수시로 생성했다가 소멸시키므로 Stack 메모리를 작게 만들 수 있는 장점이 있다.
모듈화/분업화가 유리하다.
허나 함수의 호출이 끝나면 값을 보관할 수 없다.
위의 예에선 그 단점이 잘 드러난다.

결과로는 다음과 같이 나온다.
![[스크린샷 2023-05-19 오전 11.14.55.png]]

위와 같은 단점들을 보완하고자 나온 것이 `static` 지역변수이다.
```c
//  
// Created by Changjoon Lee on 2023/05/19.  
//  
#include <stdio.h>  
  
void countFunc()  
{  
	static int count = 0;  
	count++;  
}  
  
void callNumFunc()  
{  
	printf("함수 호출 횟수는 %d입니다.\n", count);  
}  
  
int main(int argc, const char * argv[])  
{  
	for (int i = 0; i < 10; ++i) {  
		countFunc();  
	}  
	  
	callNumFunc();  
	  
	return 0;  
}
```

이 `static` 지역변수는 해당 지역에서만 사용가능하고, 전역변수가 저장되는 Data Area 영역에 저장되므로 함수 호출이 끝나도 소멸되지 않는다.
처음에 함수가 호출될 때 메모리에 올라가고, 그 함수가 호출될 때마다 접근할 수 있다.
다만, 다른 파일에서도 접근할 수 있는 전역변수와 달리, 이 `static` 지역변수는 그게 안된다.
그래서 다음과 같이 우리가 원하는 결과를 얻을 수 있다.
![[스크린샷 2023-05-19 오전 11.30.48.png]]

이제 C언어에서의 메모리 구조를 알아볼 것이다.
프로그램이 시작되면, 다음과 같이 메모리에 분할된 공간이 생긴다.
![[스크린샷 2023-05-19 오전 11.44.15.png|400]]

Stack, Heap, Data는 프로그램의 데이터가 저장되는 공간이고, Code Segment는 함수가 저장되는 공간이다.
Stack은 지역변수가 저장되며, 생성과 소멸이 빈번하다.
Heap은 동적할당이라고 해서, 필요하면 사용하는 영역이다. Stack의 공간보다 훨씬 크다.
Data는 전역변수, static변수, 문자열, 상수를 저장한다.
CPU의 클럭에 따라 Code Segment에 있는 함수 코드를 올린다.
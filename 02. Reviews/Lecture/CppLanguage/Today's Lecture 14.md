

2023-05-23

----
#지선생 #ChatGPT #수업 #문제 #문제해결 

## 개요
수업을 통해 포인터를 통해 지역변수를 전역변수에 넘겨주는 방법을 고민하던 중이었다.

## 내용
그럼에도 원래 의도했던 세개의 값인 `10, 50, 90`가 모두 나오진 않는다.
그래서 다음 두가지 방법으로 해결할 수 있다.
1. 매개변수에 포인터를 사용하면 가능하다.
2. 여러개 변수를 포인터 변수로 받아서 구조체에 담고 이 구조체 하나만 리턴하면 된다.

먼저 두번째 용법을 살펴보자.
```c
//  
// Created by Changjoon Lee on 2023/05/23.  
//  
  
#include <stdio.h>  
  
void GetHourMinSec(int totalsec, int* pHour, int* pMin, int* pSec)  
{  
	int hour = 0, min = 0, sec = 0;  
	  
	sec = totalsec % 60;  
	min = totalsec / 60;  
	hour = min / 60; // 몫을 먼저 계산해서 나머지를 뽑아 내야 덮어쓰이지 않는다.  
	min = min % 60;  
	  
	*pHour = hour;  
	*pMin = min;  
	*pSec = sec;  
}  
  
int main(int argc, const char * argv[])  
{  
	int hour = 0, min = 0, sec = 0;  
	GetHourMinSec(3725, &hour, &min, &sec);  
	printf("%02d, %02d, %02d\n", hour, min, sec);  
	GetHourMinSec(4000, &hour, &min, &sec);  
	printf("%02d, %02d, %02d\n", hour, min, sec);  
	  
	return 0;  
}
```

그러면 다음과 같이 결과값이 도출된다.
![[스크린샷 2023-05-23 오전 10.59.24.png]]

스택 메모리 상에, `hour`, `min`, `sec`가 올라간다.
이는 메인 함수에 속한 지역변수이다.

그리고 함수가 호출되는 시점에, 그 함수의 인수인 `totalSec`, `pHour`, `pMin`, `pSec`와 그 함수의 본체 내에 초기화된 `sec`, `min`, `hour`가 스택 메모리에 올라간다.
`totalSec`는 `3725`와 `4000`이 저장되고, 호출된 함수 내에서 연산이 이루어져 `sec`, `min`, `hour`에 각각 `5, 2, 1` 혹은 (두번째의 경우) `40, 6, 1`이 들어가게 된다.

하지만, 이는 `GetHourMinSec` 함수의 지역변수이기에 이를 메인함수에 넘기기 어렵다.
리턴 값으로는 하나의 값만 내놓을 수 있다.
그래서, `GetHourMinSec` 내에서 포인터 변수에 인수 값을 전달하도록 해서 메인 함수의 세 변수 `hour, min, sec`를 포인터가 가리키는 변수(역참조 연산자 `*`를 통해서!)에 대입한다.
이렇게 하면 `GetHourMinSec` 내부에서 메인함수의 지역변수에 접근할 수 있다.

다음은 2차원 배열을 배워볼 것이다.
다음 예제에서 `kor` 배열은 `kor[0][0]`부터 시작해서 `kor[2][4]`까지의 요소들을 가진다.
```c
//  
// Created by Changjoon Lee on 2023/05/23.  
//  
  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int kor[3][5];  
	  
	for (int i = 0; i < 3; ++i) {  
		for (int j = 0; j < 5; ++j) {  
			kor[i][j] = i * 5 + j;  
		}  
	}  
	  
	for (int i = 0; i < 3; ++i) {  
		for (int j = 0; j < 5; ++j) {  
			printf("%2d", kor[i][j]);  
		}  
		printf("\n");  
	}  
	  
	return 0;  
}
```

이를 그림으로 표현하면 다음과 같다.

| 1   | 2   | 3   | 4   | 5   |
| --- | --- | --- | --- | --- |
| 5   | 6   | 7   | 8   | 9   |
| 10  | 11  | 12  | 13  | 14  | 

출력은 다음과 같다.
![[스크린샷 2023-05-23 오전 11.26.38.png]]

그리고 함수 포인터를 배워보자.
```c
//  
// Created by Changjoon Lee on 2023/05/23.  
//  
  
#include <stdio.h>  
  
void KorPresentation()  
{  
	printf("점수 입력 >> ");  
}  
  
void EngPresentation()  
{  
	printf("Input Score >> ");  
}  
  
int InputValue(void (*fptr)())  
{  
	int num = 0;  
	fptr();  
	scanf("%d", &num);  
	return num;  
}  
  
int main(int argc, const char * argv[])  
{  
	int val = InputValue(KorPresentation);  
	printf("입력 값은 %d\n", val);  
	val = InputValue(EngPresentation);  
	printf("Input Score is %d\n", val);  
	  
	return 0;  
}
```

이것이 포인터의 세번째 용법이다.
함수를 포인터 변수에 저장하고, 그 포인터를 함수처럼 호출하면 퐁니터가 가리키는 함수가 호출된다.
프로그램 내부가 아니라 외부에서 전달하는 함수에 따라 동작을 변화시킬 수 있다.
이 함수 포인터가 C#에서의 `delegate`로 발전하게 된 것이다.
함수 포인터와 C#의 `delegate`의 용법은 동일하다.

이를 쓰면 뭐가 좋은가?
프로그램의 유연성이 증가하고, 이벤트 핸들러 함수를 제작할 수 있다.

그래서 위 코드를 살펴보면, `void (*fptr)()`은 포인터 변수이다. 리턴형은 `void`이고, 매개변수 없는 `void` 함수만 가리킬 수 있다.
만약 `int (*fptr)(int, int)`와 같은 포인터 변수는 리턴형은 `int`, 매개변수도 `int`가 두개인 함수만 가리킬 수 있다.
우리가 기존 변수 포인터가 그 포인터가 선언될 당시의 형(Type)에 맞게 가리킬 수 있듯이 말이다.
함수는 메모리의 Code Segment 영역에 로딩된다.
그 함수의 시작위치를 저장하게 된다.

*깨알 팁: 여기서 프로그램이 실행되기 전에 전역변수는 Data 영역에 올락가고, 스택에는 지역변수가, Code Segment에는 함수가, 힙에는 동적할당된 값들이 올라간다.

위 프로그램의 결과 값은 다음과 같이 나온다.
![[스크린샷 2023-05-23 오전 11.45.21.png]]

우리는 이제 이 동적할당이 뭔지 알아보고자 한다.
```c
//  
// Created by Changjoon Lee on 2023/05/23.  
//  
  
#include <stdio.h>  
#include <stdlib.h>  
  
#define KOR_NUM 3  
  
int main(int argc, const char * argv[])  
{  
	int* pKor = (int*)malloc(sizeof(int) * KOR_NUM);  
	for (int i = 0; i < KOR_NUM; ++i) {  
		printf("%d 번째 국어점수 입력 >> ", i + 1);  
		scanf("%d", (pKor + 1));  
	}  
	  
	printf("\n");  
	  
	for (int i = 0; i < KOR_NUM; ++i) {  
		printf("%d 번째 국어점수 : %d\n", i + 1, *(pKor + i));  
	}  
	free(pKor);  
	  
	return 0;  
}
```

여기서 `#define KOR_NUM 3`이라고 한 부분에서 넣는 값의 형을 적어줄 필요는 없다.
`KOR_NUM`은 변수가 아니라, 이 프로그램의 메타적인 위치에서 `KOR_NUM`이라고 쓰인 부분에 값을 대입해 주는 것이기 때문이다.
즉, 컴파일 되기 전에 '전처리'의 과정으로서 컴파일러의 시점에서는 이미 그 `3`이라는 값이 들어가 있는 것이다.

결과는 다음과 같다.
![[스크린샷 2023-05-23 오후 12.12.55.png]]

`malloc`은 힙에 메모리를 할당한다는 뜻이다.
그리고 `free`는 그 할당된 메모리를 해제한다는 뜻이다.

현재 스택에는 `pKor`라는 포인터가 저장돼 있으며, 힙에 있는 
왜 이런 문법을 사용하는가?
우리의 예제에서 보듯이, 몇명의 국어점수를 입력할지 모르는 상황에 메모리의 크기를 효율적으로 사용하기 위해 사용한다.
만약 `static`이나 전역 변수의 선언을 통해 우리가 예상할 수 있는 가장 큰 크기를 스택에 담을 경우, 이것이 프로그램이 실행되는 내내 그 공간을 차지하기 때문에 메모리의 낭비가 심해진다.
반대로 지역변수를 사용할 경우, 그 변수에 할당되는 스택의 크기가 너무 작고, 범위에서 벗어나면 소멸한다.
따라서 필요할 때 코드를 통해 메모리의 크기를 제어할 수 있도록 만들고자 동적할당이 생겨난 것이다.

위 예제에서는 `pKor`를 선언한 부분에서 힙 메모리에 4 * 3 = 12byte를 할당한 것이고, 지역변수인 `pKor`이 할당한 메모리의 시작주소를 가리킨다.
즉, 힙메모리에 3개짜리 `int` 배열을 동적으로 생성한다.
그래서 우리는 여기서 배열처럼 이 변수를 다룰 수 있는 것이다.

여기서 메모리 해제를 적절하게 해주지 못한다면 어떤 일이 벌어질까?
필요 없는 메모리를 적절하게 해제해 주지 않는다면 그 서버는 갑자기 메모리 부족으로 다운된다.
이 때 서버 프로그램이 종료되는 이유를 찾기까지 많은 시간과 비용이 소모되게 된다.
그래서 C, C++에서는 힙에 할당한 동적 메모리를 적절한 시점에 잘 해제하는 것이 매우 중요하게 된다.

Java/C#은 프로그램에서 힙을 사용하는 동적메모리를 해제하지 않는 상황이 종종 많은 sw 에러의 원인이 되는 것을 발견하고 과감하게 VM/CLR 내에 GC(Garbage Collector)를 두고 더 이상 참조되지 않는 동적 메모리를 자동 해제해 줌으로써 C/C++에 비해 메모리 누수애 의한 에러를 많이 줄였다.

다음은 동적할당의 다른 방법인 `calloc`을 사용하는 예제이다.
```c
//  
// Created by Changjoon Lee on 2023/05/23.  
//  
  
#include <stdio.h>  
#include <stdlib.h>  
  
#define KOR_NUM 3  
  
int main(int argc, const char * argv[])  
{  
	int* pKor = (int*)calloc(sizeof(int) * KOR_NUM);  
	for (int i = 0; i < KOR_NUM; ++i) {  
		printf("%d 번째 국어점수 입력 >> ", i + 1);  
		scanf("%d", (pKor + 1));  
	}  
	  
	printf("\n");  
	  
	for (int i = 0; i < KOR_NUM; ++i) {  
		printf("%d 번째 국어점수 : %d\n", i + 1, *(pKor + i));  
	}  
	free(pKor);  
	  
	return 0;  
}
```

결과는 다음과 같다.
![[스크린샷 2023-05-23 오후 12.35.34.png]]

`malloc`과 동일하게 돌아가는 것을 알 수 있다.

이제 구조체에 대해 배워보자.
구조체는 하나의 추상적 개념(아무개의 신상정보 등)을 표현하는 여러 개의 변수들을 1개의 그룹으로 묶어준다.
이 1개의 그룹으로 된 변수를 선언하면 프로그램을 훨씬 단순하게 표현할 수 있다.

`struct`는 C++에 와서 `class`로 발전하게 된다.
C언어에서의 구조체는 변수의 집합이다.
C++의 `struct, class`는 변수와 함수의 집합이다.

홍길동의 신상정보를 입력받아 출력하는 예제를 만들어 볼 것이다.
```c
//  
// Created by Changjoon Lee on 2023/05/23.  
//  
  
#include <stdio.h>  
#include <string.h>  
  
#define HUMAN_NUM 3  
  
struct HUMAN  
{  
	char name[20];  
	int age;  
	float height;  
};  
  
int main(int argc, const char * argv[])  
{  
	struct HUMAN human;  
	  
	printf("이름 >> ");  
	fgets(human.name, strlen(human.name), stdin);  
	human.name[strlen(human.name) - 1] = '\0';  
	printf("나이 >> ");  
	scanf("%d\n", &human.age);  
	printf("키 >> ");  
	scanf("%f", &human.height);  
	  
	printf("*******************\n");  
	printf("이름 : %s\n", human.name);  
	printf("나이 : %d\n", human.age);  
	printf("키 : %.f\n", human.height);  
	printf("*******************\n");  
	  
	return 0;  
}
```

*여기서 `fgets`는 공백이 삽입된 부분까지만 값을 읽는 함수이다.

결과를 보면 다음과 같다.

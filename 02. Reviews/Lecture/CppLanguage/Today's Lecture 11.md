

2023-05-18

----
#C #지선생 #ChatGPT #수업 #문제 #문제해결 

## 개요
어제 비트연산자를 배우고 끝났었다.
오늘은 다른 연산자를 배워볼 것이다.

## 내용
다음 코드는 Shift 연산자라고 부른다.
```c
//  
// Created by Changjoon Lee on 2023/05/18.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	unsigned char cNum = 19;  
	printf("%d\n", cNum << 1);  
	printf("%d\n", cNum << 2);  
	printf("%d\n", cNum << 3);  
	printf("%d\n", cNum >> 1);  
	printf("%d\n", cNum >> 2);  
	printf("%d\n", cNum >> 3);  
	  
	return 0;  
}
```

이는 십진수인 19에 `<<` 연산을 하면, 이진수로 표현된 `00010011`을, 다음과 같이 왼쪽으로 자리수를 하나씩 늘려준다.
`00100110`이 되는 것이다. 
즉, 십진수로 생각하면 2가 곱해지게 되는 것이다.

결과를 보면 다음과 같다.
![[스크린샷 2023-05-18 오전 9.54.26.png]]

다음은 조건연산자이다.
```c
//  
// Created by Changjoon Lee on 2023/05/18.  
//  
  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int val1 = 10, val2 = 20;  
	int result = 0;  
	result = val1 >= 10? val1 : val2;  
	printf("%d\n", result);  
	return 0;  
}
```

삼항연산자로고도 하는데, 다음과 같이 항으로 표현된다.
D = A ? B : C;
A항이 참이면 B항에 대입, 아니면 C항에 대입한다.

결과는 다음과 같다.
![[스크린샷 2023-05-18 오전 10.02.57.png]]

다음은 mask 연산의 예시이다.
```c
//  
// Created by Changjoon Lee on 2023/05/18.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	unsigned char val1 = 0x07, val2 = 0x09;  
	unsigned char result = 0;  
	result = val1 << 4;  
	printf("%#x\n", result);  
	result = result | val2;  
	printf("%#x\n", result & 0xf0);  
	printf("%#x\n", result & 0x0f);
	  
	return 0;  
}
```

16진수 1자리는 2진수 4자리 범위를 차지한다.
2진수는 컴퓨터에 적합한 숫자이고, 10진수는 사람에게 적합한 숫자이다.
2진수는 작은 숫자도 표현하려면 범위가 커진다.
그러므로 프로그래밍에서 표현하려면 사용하기가 좋지 않다.
그러므로 16진수를 쓰면 작은 범위로도 큰 숫자를 표현할 수 있다.
정리하면, 2진수로 작동하는 프로그래밍에서는 16진수를 사용한다.

`unsinged`를 기준으로 `256` → `0xff`와 같은 식으로 말이다.
어제 배웠지만, 뒤에 붙는 `a, b, c, d ...`와 같은 기호는 각각 `10, 11, 12, 13 ...`를 표현한 것이다.
`|` 연산을 하면 비트 표현식이 그대로 내려오고, 
`&` 연산을 하면 비트 표현식이 반대로 되어서 내려온다.

위 코드의 결과를 보면 다음과 같다.
![[스크린샷 2023-05-18 오전 10.17.07.png]]

연산자는 여기까지가 끝이다.
연산자 우선순위를 잠깐 정리하고 마무리 해보자.
1. 먼저 연산하고 싶으면 괄호`()`로 묶자.

다음은 제어문을 배울 차례이다.
```c
//  
// Created by Changjoon Lee on 2023/05/18.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int num = 0;  
	printf("정수 입력 >> ");  
	scanf("%d", &num);  
	
	if (num > 10)  
		printf("10보다 큼\n");  
	else if(num == 10)  
		printf("10과 같음\n");  
	else  
		printf("10보다 작음\n");  
	  
	return 0;  
}
```

결과는 다음과 같다.
![[스크린샷 2023-05-18 오전 10.27.43.png]]

C#에서와 마찬가지로, 여기에도 `switch case`문이 있다.
```c
//  
// Created by Changjoon Lee on 2023/05/18.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	char ch;  
	printf("연산자를 입력하세요 : ");  
	scanf("%s", &ch);  
	  
	switch (ch) {  
		case '+':  
			printf("+ 연산자입니다.");  
			break;  
		case '-':  
			printf("- 연산자입니다.");  
			break;  
		case '*':  
			printf("* 연산자입니다.");  
			break;  
		case '/':  
			printf("/ 연산자입니다.");  
			break;  
		case '%':  
			printf("% 연산자입니다.");  
			break;  
		default:  
			printf("이것은 산술연산자가 아닙니다.");  
			break;  
	}  
	  
	return 0;  
}
```

다음과 같이 결과가 나온다.
![[스크린샷 2023-05-18 오전 10.36.36.png]]

그렇게 어렵지 않은 내용이다.
하지만, C#과 달리 문자열 비교는 여기선 안된다.
왜냐하면, C언어는 문자들도 숫자로 저장이 되기 때문이다.

이제 반복문을 배울 차례이다.
```c
//  
// Created by Changjoon Lee on 2023/05/18.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int i = 1;  
	while (i <= 10){  
		printf("%d\n", i++);  
	}  
	  
	i = 1;  
	while (1){  
		printf("%d\n", i++);  
		if(i > 10)  
			break;  
	}  
	  
	return 0;  
}
```

이를 실행시켜 보면 다음과 같다.
![[스크린샷 2023-05-18 오후 12.22.44.png]]

다음은 `do while`문이다.
```c
//  
// Created by Changjoon Lee on 2023/05/18.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int i = 1;  
	do {  
		printf("%d\n", i++);  
	} while (i <= 10);  
	  
	return 0;  
}
```

결과는 다음과 같다.
![[스크린샷 2023-05-18 오후 12.25.20.png]]

이제 `for`문.
```c
//  
// Created by Changjoon Lee on 2023/05/18.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int i = 0;  
	for (;i <= 10;)  
		printf("%d\n", i++);  
	  
	for (int j = 0; j <10; ++j) {  
		printf("%d\n", j++);  
	}  
	  
	int k = 0;  
	for (;;) {  
		printf("%d\n", k++);  
		if(i > 10)  
		break;  
	}  
	  
	return 0;  
}
```

결과는 다음과 같다.
![[스크린샷 2023-05-18 오후 12.29.48.png]]

이제 함수를 배울 차례.
다음은 함수를 만든 예제이다.
```c
//  
// Created by Changjoon Lee on 2023/05/18.  
//  
#include <stdio.h>  
#include <stdlib.h>  
#include "osXgetch.h"  
  
void showMenu()  
{  
	printf("----연산 선택----\n");  
	printf("1. 더하기\n");  
	printf("2. 빼기\n");  
	printf("3. 곱하기\n");  
	printf("4. 나누기\n");  
	printf("5. 종료\n");  
}  
  
int getSelectMenu()  
{  
	int num;  
	printf("번호 선택 >> ");  
	scanf("%d", &num);  
	return num;  
}  
  
double Add(double dNum0, double dNum1)  
{  
	return dNum0 + dNum1;  
}  
  
double Sub(double dNum0, double dNum1)  
{  
	return dNum0 - dNum1;  
}  
  
double Mul(double dNum0, double dNum1)  
{  
	return dNum0 * dNum1;  
}  
  
double Div(double dNum0, double dNum1)  
{  
	return dNum0 / dNum1;  
}  
  
double getDoubleNum()  
{  
	double num;  
	printf("실수 입력 >> ");  
	scanf("%lf", &num);  
	return num;  
}  
  
void printResult(double result)  
{  
	printf("\n");  
	printf("결과는 %.2f입니다\n", result);  
}  
  
void cls()  
{  
	// 콘솔 창의 출력을 모두 지운다.  
	system("clear");  
}  
  
int main(int argc, const char * argv[])  
{  
	int isRun = 1;  
	  
	double dNum0 = 0, dNum1 = 0;  
	while (isRun)  
	{  
		cls();  
		showMenu();  
		int sel = getSelectMenu();  
		  
		int result = 0;  
		switch (sel) {  
		case 1:  
			dNum0 = getDoubleNum();  
			dNum1 = getDoubleNum();  
			result = Add(dNum0, dNum1);  
			printResult(result);  
			break;  
		case 2:  
			dNum0 = getDoubleNum();  
			dNum1 = getDoubleNum();  
			result = Sub(dNum0, dNum1);  
			printResult(result);  
			break;  
		case 3:  
			dNum0 = getDoubleNum();  
			dNum1 = getDoubleNum();  
			result = Mul(dNum0, dNum1);  
			printResult(result);  
			break;  
		case 4:  
			dNum0 = getDoubleNum();  
			dNum1 = getDoubleNum();  
			result = Div(dNum0, dNum1);  
			printResult(result);  
			break;  
		case 5:  
			isRun = 0;  
			break;  
		}  
		  
		get_ch(); // 키보드 아무거나 입력될 때까지 대기  
	}  
	return 0;  
}
```

여기서 세가지 문제가 있었다.

첫째로는 수업시간에 소개된 `system("cls")`가 맥에서는 쓸 수 없는 명령어였다는 것이다.
`"cls"`를 시스템에 전달해도 맥은 알아듣지 못한다.
간단하게 대체할 수 있는 키워드는 `"clear"`로써, `system("clear")`를 써서 간단하게 해결.

둘째로는 `getch()`가 맥 환경에서는 쓸 수 없는 함수라는 것이다.
대신할 방법을 찾다가, 그냥 헤더파일을 만들어서 참조하기로 했다.
지선생에게 만들어 달라 했더니, 
![[스크린샷 2023-05-18 오후 2.40.02.png]]

그래서 헤더 파일의 이름을 `osXgetch.h`로 하고 참조를 해줬다.
자세히 뜯어보진 않으려 한다.

실행 도중 문제가 하나 더 생겼다.
`get_ch();` 부분에서는 다음 키보드 입력이 있을 때까지 대기해야 하는데, 결과를 출력하고 재빨리 콘솔창을 지워버리는 것이었다.
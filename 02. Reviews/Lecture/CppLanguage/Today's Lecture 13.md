

2023-05-22

----
#C #지선생 #ChatGPT #수업 #문제 #문제해결 

## 개요
오늘부터는 배열과 포인터에 대해 배워볼 것이다.

## 내용
```c
//  
// Created by Changjoon Lee on 2023/05/22.  
//  
  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int total = 0;  
	double avg;  
	int kor0, kor1, kor2;  
	printf("3명의 학생 국어 점수 입력 >> ");  
	scanf("%d %d %d", &kor0, &kor1, &kor2);  
	  
	total = kor0 + kor1 + kor2;  
	avg = (double)total / 3;  
	printf("총합은 %d, 평균은 %.21f\n", total, avg);  
	  
	return 0;  
}
```

위 코드는 비효율적이다.
만약 3명의 학생이 아니라 100명의 학생이라면?
우리는 국어점수 뒤에 숫자가 붙는다는 규칙성을 발견했으니, 다음과 같이 좀 더 효율적으로 짜 볼 수 있을 것이다.
```c
//  
// Created by Changjoon Lee on 2023/05/22.  
//  
  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int total = 0;  
	double avg;  
	int kor[3];  
	for (int i = 0; i < 3; ++i) {  
		printf("%d번째 학생 국어 점수 입력 >> ");  
		scanf("%d", &kor[i]);  
	}  
	  
	for (int i = 0; i < 3; ++i) {  
		total += kor[i];  
	}  
	  
	avg = (double)total / 3;  
	  
	printf("총합은 %d, 평균은 %.21f\n", total, avg);  
	  
	return 0;  
}
```

이는 다음을 의미한다.`int` 자료형을 가진 변수가 `kor`이라는 대표명으로 나란히 3개 이어져 있다.
그리고 이 `kor`에는 시작위치,  즉 주소가 담긴다.

근데 이를 실행해 보니 다음과 같은 이상한 값(`1875670156`)이 나왔다.
![[스크린샷 2023-05-22 오전 10.37.58.png]]

이는 `printf("%d번째 학생 국어 점수 입력 >> ");` 뒷부분에 `i + 1`을 주지 않아서 생기는 현상이었다.
다음과 같이 고쳐주면 정상적으로 작동한다.
```c
//  
// Created by Changjoon Lee on 2023/05/22.  
//  
  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int total = 0;  
	double avg;  
	int kor[3];  
	for (int i = 0; i < 3; ++i) {  
		printf("%d번째 학생 국어 점수 입력 >> ", i + 1);  
		scanf("%d", &kor[i]);  
	}  
	  
	for (int i = 0; i < 3; ++i) {  
		total += kor[i];  
	}  
	  
	avg = (double)total / 3;  
	  
	printf("총합은 %d, 평균은 %.21f\n", total, avg);  
	  
	return 0;  
}
```

근데 여기서 우리가 보는 배열이 C#의 배열과 모습이 다르다.
C#에서는 다음과 같이 썼었다.
```cs
int[] kor = new int[3];
```

하지만 C언어에서 이는 다음 두가지를 의미한다.
`kor`은 힙에 있는 `int` 배열 객체를 가리키고 있다. 
`int` 배열 객체는 `int`가 나란히 3개가 있다.

그리고 여기서 주의할 점
```c
scanf("%d", &kor[i]);
```

위 부분을 작성할 때, `"%d "`라고 작성하면 입력해도 실행이 안된다...
형식을 맞춰서 입력 당시에 `87 `과 같이 공백을 추가해서 넣어도 여전히 작동하지 않는다...
![[스크린샷 2023-05-22 오전 10.58.23.png]]

근데 만약 학생의 숫자가 바뀌면 어떻게 편하게 모두 바뀌게 해줄 수 있을까?
첫째로, `#define`을 이용하는 방법이 있다.
```c
//  
// Created by Changjoon Lee on 2023/05/22.  
//  
  
#include <stdio.h>  
  
#define KOR_NUM 7
  
int main(int argc, const char * argv[])  
{  
	int total = 0;  
	double avg;  
	int kor[3];  
	for (int i = 0; i < KOR_NUM; ++i) {  
		printf("%d번째 학생 국어 점수 입력 >> ", i + 1);  
		scanf("%d ", &kor[i]);  
	}  
	  
	for (int i = 0; i < KOR_NUM; ++i) {  
		total += kor[i];  
	}  
	  
	avg = (double)total / KOR_NUM;  
	  
	printf("총합은 %d, 평균은 %.21f\n", total, avg);  
	  
	return 0;  
}
```

이 `#define`을 '전처리기'라고 하고, 컴파일 시점 이전에 `KOR_NUM`을 만나면 `7`로 숫자를 넣어주는 식으로 작동한다.

두번째 방법은 다음과 같은 게 있다.
```c
//  
// Created by Changjoon Lee on 2023/05/22.  
//  
  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int total = 0;  
	double avg;  
	int kor[3];  
	  
	int ARR_LEN = sizeof(sizeof(kor) / kor[0]); // 배열의 길이가 이 배열에 들어가게 된다.  
	// int ARR_LEN = sizeof(kor) / sizeof(int); // 위와 상동이다.  
	  
	for (int i = 0; i < ARR_LEN; ++i) {  
		printf("%d번째 학생 국어 점수 입력 >> ", i + 1);  
		scanf("%d ", &kor[i]);  
	}  
	  
	for (int i = 0; i < ARR_LEN; ++i) {  
		total += kor[i];  
	}  
	  
	avg = (double)total / ARR_LEN;  
	  
	printf("총합은 %d, 평균은 %.21f\n", total, avg);  
	  
	return 0;  
}
```

이렇게 전체 데이터 크기를 한 요소의 데이터 크기로 나누는 방식이 있다.

이제 배열의 초기화에 대해 배워보자.
```c
//  
// Created by Changjoon Lee on 2023/05/22.  
//  
  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int kor = { 99, 89, 100, 78, 67 };  
	char *name[] = { "홍길동", "임꺽정", "장길산", "차돌바위", "일지매" };  
	int ARR_LEN = sizeof(kor) / sizeof(kor[0]);  
	  
	for (int i = 0; i < ARR_LEN; ++i) {  
		printf("%s : %d\n", name[i], kor[i]);  
	}  
	  
	return 0;  
}
```

초기화 값들이 있으면 컴파일러가 그 값들의 갯수를 알아낼 수 있으므로 `sizeof(kor)`에서 알 수 있듯이 배열의 크기를 안써줘도 된다.
그리고 여기서 배열에 값을 넣어주는 행위를 초기화라고 부른다.

```c
//  
// Created by Changjoon Lee on 2023/05/22.  
//  
  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	char str0[] = { 'k', 'o', 'r', 'e', 'a', '\0' };  
	char str1[] = { 'k', 'o', 'r', 'e', 'a', };  
	char str2[] = { 'k', 'o', 'r', 'e', 'a', 0 };  
	char str3[] = "korea";  
	char str4[6] = "korea";  
	  
	printf("%s\n", str0);  
	printf("%s\n", str1);  
	printf("%s\n", str2);  
	printf("%s\n", str3);  
	printf("%s\n", str4);  
	  
	for (int i = 0; i < 6; ++i) {  
		printf("%c, %d\n", str0[i], str0[i]);  
	}  
	  
	return 0;  
}
```

(왜 컴파일러가 `NULL`을 못읽지;;;)

C언어에서의 `char`는 다음을 의미한다.
1. 정수형의 경우 -127 ~ 128
2. 문자형의 경우 아스키코드표
어차피 문자는 메모리에 저장될 때는 숫자로 변환되어 저장된다.

하지만 C#에서의 `char`는 오직 문자형으로만 사용하고, 대신 -127 ~ 128을 표현할 때는`byte`를 쓴다.

다시 예제를 보면, 여기 초기화된 문자열 모두는 동일한 결과를 출력한다.
그리고, C언어에서는 `\0`과 `0`을 `NULL`과 동일한 의미로 사용한다.
C언어의 컴파일러는 문자열을 읽다가 이 세 키워드를 만나면 읽기를 멈춘다.
이게 크기를 주지 않은 문자열에 대해서 끝 부분을 처리해 주는 방식이다.

그럼, 크기를 선언하지 않은 문자열 끝에 이런 `NULL` 값을 주지 않으면 어떻게 될까?
다음 예제를 보자.
```c
//  
// Created by Changjoon Lee on 2023/05/22.  
//  
  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	char str0[] = { 'k', 'o', 'r', 'e', 'a'};  
	char str1[] = "korea";  
	str1[5] = 'i';  
	  
	printf("%s\n", str0);  
	printf("%s\n", str1);  
	  
	return 0;  
}
```

결과는 다음과 같다.
![[스크린샷 2023-05-22 오전 11.31.35.png]]
난 잘 되는데? 왜 그런 걸까...
아마 컴파일러가 버전이 달라 알아서 처리해 주는듯 하다.

다음 예제이다.
```c
//  
// Created by Changjoon Lee on 2023/05/22.  
//  
  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	char str0[] = "korea";  
	char *str1 = "korea";  
	  
	printf("%s\n", str0);  
	printf("%s\n", str1);  
	
	str0[0] = 'c';
	printf("%s\n", str0);
	str1[1] = 'c';
	printf("%s\n", str1);	
	
	return 0;  
}
```

`str0`는 문자열까지 포함하여 스택에 올라가고, `str1`도 스택에 올라간다.
`str1`은 포인터이기에, 이것은 데이터에 저장돼 있는 문자열인 `"korea"`를 가리킨다.
전자는 문자열 변수 배열이고, 문자를 바꿀 수 있다.
후자는 상수문자열이고, 문자는 바꿔줄 수 없으며 읽기만 가능하다.

물론 출력은 동일하게 나온다.
하지만, 문자열을 바꾼 뒤 출력하면 에러가 나서 출력이 되지 않는다.
![[스크린샷 2023-05-22 오전 11.39.07.png]]

이제 포인터를 배워보자.
```c
//  
// Created by Changjoon Lee on 2023/05/22.  
//  
  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int num = 100;  
	int *pnum = &num;  
	  
	printf("%d, %d\n", num, *pnum);  
	  
	num = 999;  
	printf("%d, %d\n", num, *pnum);  
	  
	*pnum = 1200;  
	printf("%d, %d\n", num, *pnum);  
	  
	return 0;  
}
```

이를 실행하면 다음과 같은 결과가 나온다.
![[스크린샷 2023-05-22 오전 11.48.24.png]]

`pnum`은 `num`의 주소, 즉 `&num`를 담는 것으로 초기화되었다.
이렇게 포인터는 주소를 담는 변수이다.
여기서, 각 형의 주소인 포인터는 초기화 당시의 형을 계속 유지해야 한다는 것을 기억하자.
다음과 같이 바꿔서 실행해보면 어떨까?
```c
//  
// Created by Changjoon Lee on 2023/05/22.  
//  
  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int num = 100;  
	int *pnum = &num;  
	  
	printf("%ld, %p\n", pnum, pnum);  
	  
	num = 999;  
	printf("%ld, %p\n", &num, &num);  
	  
	*pnum = 1200;  
	printf("%d, %d\n", num, *pnum);  
	  
	return 0;  
}
```

결과는 다음과 같이 나온다.
![[스크린샷 2023-05-22 오전 11.55.09.png]]

`num`이라는 변수는 메모리 상에 올라가는데, `&num` 즉, 변수 값이 시작하는 지점의 주소값은 십진수로 표현하면 `6094730364`이고, 16진수로 표현하면 `0x16b46347c`이라는 것을 알 수 있다.

왜 이것을 쓰는가?
컴파일러가 프로그램이 실행될 때마다 변수값을 복사한다는 것은 상당한 부하를 가져오는 행위이다.
그래서 포인터를 통해 위치만 가리킬 뿐, 그 실제 변수는 그 자리에 그대로 있는 것임.

그럼 포인터의 용법을 한번 알아 보자.
`int* pnum`은 `int num`이나 `int arr[]`같은 것들의 시작 주소를 저장한다.
```c
//  
// Created by Changjoon Lee on 2023/05/22.  
//  
  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int kor[] = { 99, 100, 88, 78, 89 };  
	int *parr = kor;  
	  
	printf("%p\n", kor);  
	printf("%p\n", &kor[0]);  
	  
	for (int i = 0; i < sizeof(kor) / sizeof(int); ++i) {  
		printf("%d, ", parr[i]);  
	}  
	printf("\n");  
	  
	return 0;  
}
```

배열의 경우, 배열 내의 요소들이 물리적으로 연속된 주소를 가진다.
각 요소는 각각 4 바이트씩 공간을 차지하고 있다.
또, `kor`의 주소는 배열의 시작 지점의 주소를 가리킨다.
우리는 위 코드에 대한 결과를 통해 이를 확인할 수 있다.
![[스크린샷 2023-05-22 오후 12.26.36.png]]

참고로, `%p`는 16진수로 포인터를 나타내는 형식이다.

`kor[i]` 대신 포인터 변수인 `parr[i]`로 접근할 수 있다.
포인터 변수에 더하는 `i`는 주소를 `i`만큼 증가시킨다는 의미가 아니고 , `sizeof(int) * i`만큼 주소를 이동시킨다는 의미이다.
즉, `i`는 인덱스의 의미를 가진다.

다음과 같이 반복문을 짜면 포인터가 한 요소의 크기 단위로 이동하여 값을 읽는다.
```c
for (int i = 0; i < sizeof(kor) / sizeof(int); ++i) {  
	printf("%d", *(parr + i));
}
```

결과는 다음과 같다.
![[스크린샷 2023-05-22 오후 12.37.39.png]]

다른 예제는 포인터의 첫번째 용법을 알아보는 것이다.
배열을 함수에서 접근할 때 매개변수로 배열의 시작위치를 가리킬 때 사용한다.
```c
//  
// Created by Changjoon Lee on 2023/05/22.  
//  
  
#include <stdio.h>  
  
int Sum(int *sbjArr, int len)  
{  
	int total = 0;  
	for (int i = 0; i < len; ++i) {  
		total += sbjArr[i];  
		// 우항에 *(sbjArr + 1)라고 써도 동일한 표현이다.  
	}  
	return total;  
}  
  
int main(int argc, const char * argv[])  
{  
	int kor[] = { 99, 89, 77, 80, 80 };  
	int math[] = { 89, 88, 80, 90, 100, 78, 77 };  
	  
	int total = 0;  
	total = Sum(kor, sizeof(kor) / sizeof(int));  
	printf("kor의 총합은 %d입니다\n", total);  
	total = Sum(math, sizeof(math) / sizeof(int));  
	printf("math의 총합은 %d입니다\n", total);  
	  
	return 0;  
}
```

C#의 경우 배열의 참조변수가 객체를 가리키기에 객체 내에 배열의 길이정보까지 들어 있다.
따라서 배열의 참조변수만 인자로 넘겨주면 됐었다.

그러나 C에서는 배열의 이름은 시작위치만을 나타내고 배열의 끝은 어디까지인지 알 수 없다.
그러므로 C에서는 배열의 이름을 넘길 때 포인터 변수로 시작주소를 받고 배열의 길이정보도 함께 넘겨주어야 한다.

그리고 함수를 표현하는 다른 방법도 있다.
```c
int Sum(int sbjArr[], int len)
```

배열을 이렇게 메서드의 매개변수로서 선언할 수도 있다.
하지만 컴파일러는 위의 `int sbjArr[]`를 포인터인 `int *sbjArr`로 변환시킨다.

다음 예제는 초를 입력하고 시, 분, 초로 결과를 리턴하는 문제가 있다고 해보자.
```c
//  
// Created by Changjoon Lee on 2023/05/22.  
//  
  
#include <stdio.h>  
  
int GetHourMinSec(int totalsec)  
{  
	int hour = 10, min = 50, sec = 90;  
	return hour, min, sec;  
}  
  
int main(int argc, const char * argv[])  
{  
	int hour = 90, min = 8, sec = 29;  
	hour, min, sec = GetHourMinSec(10000);  
	printf("%d, %d, %d", hour, min, sec);  
	  
	return 0;  
}
```

`10000`를 넣어서 저 세 값을 한번에 리턴하도록 했다.
원래 이게 컴파일이 안된다.
(근데 컴파일러가 발전해서 다음과 같이 값이 출력되는 듯 하다.)
![[스크린샷 2023-05-23 오전 10.33.09.png]]


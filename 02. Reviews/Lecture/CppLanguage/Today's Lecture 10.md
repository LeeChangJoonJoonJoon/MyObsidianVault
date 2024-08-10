

2023-05-17

----
#C 

## 개요
어제까지 자료형에 대해 배워봤다.
오늘은 값을 입력하는 내용을 배워볼 것이다.

## 내용
다음 예제에 나온 `scanf()`는 표준입력(stdi)으로부터 데이터를 읽어와 변수에 할당하는 함수이다.
사용자로부터 값을 입력받을 때 주로 사용된다.
```c
//  
// Created by Changjoon Lee on 2023/05/17.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int rad;  
	double pi = 3.14;  
	  
	printf("반지름 입력 >> ");  
	scanf("%d", &rad);  
	  
	printf("반지름이 %d인 원의 둘레는 %.2lf, 넓이는 %.2lf입니다.", rad, rad * 2 * pi, rad * rad * pi);  
	return 0;  
}
```

다음과 같이 사용할 수 있다.
![[스크린샷 2023-05-17 오전 10.46.43.png]]

여기서 중요한 게 있다.
수업시간에는 `scanf` 대신 `scanf_s`를 썼는데(이게 마이크로소프트에서도 권장하는 바다), 컴파일러가 이를 읽어내질 못했다.
찾아보니, 내가 사용하는 유니버셜 플랫폼의 gcc-11의 `stdio.h`은 `scanf_s`를 품고 있지 않았다.

이제 형변환을 배워보고자 한다.
형변환에는 규칙이 있다.
1. 자동형변환 → 서로 다른 자료형끼리는 연산이 불가능하다. 그러므로 컴파일러는 자동으로 연산하는 자료형중에 한쪽으로 변환시킨다. 즉, 표현범위가 큰 쪽으로 자동으로 맞춰진다. 이를테면 float > int (서로 다른 형끼리 연산했을 때 큰 쪽으로 맞춰진다는 뜻이다.)
2. 강제 형변환 → `int a = 1;`로 선언된 데이터를 `(float)a`를 이용하여 강제 형변환.

하나의 비트는 0과 1 밖에 담지 못한다.
그래서, 비트를 8개 이어붙여서 하나의 바이트를 만들었다.
여기서 맨 앞의 비트를 부호 값을 저장하는 데 쓰고, 나머지는 수의 절대값을 나타내는 데에 썼다.
그럼, 하나의 바이트는 256가지의 수를 저장할 수 있는 것이다.
우린 이렇게 만들어진 8바이트 크기의 값인 `int`를 주로 쓰는 것이다.

정수의 경우에는 이와 같이 만들 수 있지만, 실수의 경우에는 다른 방식으로 저장한다.
복잡한 공식을 통해 저장한다는데, 그건 수학의 영역이라 여기선 다루지 않는다.

다음은 자동형변환에 대한 예제.
```c
//  
// Created by Changjoon Lee on 2023/05/17.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	char ch = 10; // 표현범위는 -128 ~ 127int i = 1000; // 표현범위는 -21억 ~ 21억  
	float f = 1.5f; // 표현범위는 -무한대 ~ +무한대  
	double d = ch * i * f + 1000;  
	  
	printf("결과 : %lf\n", d);  
	
	return 0;  
}
```

그런데, 만약 위의 `ch` 변수에 128 값을 담으면 어떻게 될까?
`ch`변수를 그대로 출력시키면 `-128`이 출력된다.
이를 '캐리(Carry)되었다'라고 말한다.

위의 예제를 실행하면 다음과 같이 출력된다.
표현번위가 큰 쪽으로 맞춰지는 것을 볼 수 있다.
![[스크린샷 2023-05-17 오전 11.11.21.png]]

다음은 캐리의 예제이다.
```c
//  
// Created by Changjoon Lee on 2023/05/17.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int i = 128;  
	char ch = i;  
	printf("int=%d, char=%d\n", i, ch);  
	  
	char c = 120;  
	int j = 10000;  
	printf("result=%d\n", c * j);  
	  
	return 0;  
}
```

이를 실행하면 다음과 같은 결과를 얻을 수 있다.
![[스크린샷 2023-05-17 오전 11.15.39.png]]

소수를 정수형에 담으면 어떻게 될까?
```c
//  
// Created by Changjoon Lee on 2023/05/17.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	double pi = 3.141592;  
	int i = pi;  
	printf("double=%d, int=%d\n", pi, i);  
	  
	return 0;  
}
```

다음은 결과이다.
컴파일러는 표현할 방법이 없기 때문에 소수점은 전부 날라간다.
![[스크린샷 2023-05-17 오전 11.19.47.png]]

이제 강제형변환에 대해 알아보자.
```c
//  
// Created by Changjoon Lee on 2023/05/17.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	double d1 = 3.4;  
	double d2 = 2.1;  
	int res1 = d1 * d2;  
	int res2 = (int)d1 * (int)d2;  
	printf("res1=%d, res2=%d\n", res1, res2);  
	  
	return 0;  
}
```

다음은 그 결과이다.
![[스크린샷 2023-05-17 오전 11.23.31.png]]

앞서 자동형변환 때처럼 소수점이 잘려나가는 것을 볼 수 있다.

다음은 나누기에서 자동형변환이다.
```c
//  
// Created by Changjoon Lee on 2023/05/17.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int i = 1;  
	double d = 2.0;  
	double result = i / d;  
	printf("result=%d\n", result);  
	  
	return 0;  
}
```

결과는 다음과 같다.
![[스크린샷 2023-05-17 오전 11.27.03.png]]

다음은 정수를 실수로 나누는 예제이다.
```c
//  
// Created by Changjoon Lee on 2023/05/17.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int i = 1;  
	double d = 2.0;  
	int result = i / d;  
	printf("result=%d\n", result);  
	  
	return 0;  
}
```

결과는 다음과 같다.
![[스크린샷 2023-05-17 오전 11.28.42.png]]

결과값을 실수로 저장하면 어떻게 될까?
```c
//  
// Created by Changjoon Lee on 2023/05/17.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int i1 = 1;  
	int i2 = 2;  
	double result = i1 / i2;  
	printf("%1f\n", result);  
	  
	return 0;  
}
```

결과는 다음과 같다.
![[스크린샷 2023-05-17 오전 11.31.04.png]]

정수랑 정수를 연산해도, 결과를 실수에 저장해서 생기는 일인 것이다.

강제형변환이 연산 중에 들어가면 어떻게 될까?
```c
//  
// Created by Changjoon Lee on 2023/05/17.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int i1 = 1;  
	int i2 = 2;  
	double result = (double)i1 / i2;  
	printf("%lf\n", result);  
	  
	return 0;  
}
```

다음은 그 결과이다.
![[스크린샷 2023-05-17 오전 11.33.27.png]]

```c
//  
// Created by Changjoon Lee on 2023/05/17.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int i1 = 1;  
	int i2 = 2;  
	double result = (double)(i1/i2);  
	printf("%lf\n", result);  
	  
	return 0;  
}
```

다음은 그 결과이다.
![[스크린샷 2023-05-17 오전 11.35.46.png]]

```c
//  
// Created by Changjoon Lee on 2023/05/17.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int kor1 = 98;  
	int kor2 = 79;  
	int kor3 = 100;  
	int total = kor1 + kor2 + kor3;  
	  
	double avg = total / 3;  
	printf("total=%d, avg=%d\n", total, avg);  
	  
	return 0;  
}
```

결과는 다음과 같다.
![[스크린샷 2023-05-17 오전 11.52.42.png]]

우리는 결과의 평균이 소수까지 나오도록 계산해 주고 싶다.
그러러면 다음과 같이 코드를 수정해 주면된다.
```c
//  
// Created by Changjoon Lee on 2023/05/17.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int kor1 = 98;  
	int kor2 = 79;  
	int kor3 = 100;  
	int total = kor1 + kor2 + kor3;  
	  
	double avg = total / 3;  
	printf("total=%d, avg=%d\n", total, avg);  
	  
	return 0;  
}
```
????????????????????????

이제 연산자에 대해 배워볼 것이다.
연산자에는 다음과 같은 종류가 있다.
1. 산술연산자 계열 → 부호연산자, 복합대입연산자 ... 우리가 다 아는 것들.
2. 관계연산자 계열 → 논리, 비트논리 연산자 등등
3. 조건연산자 → 삼항연산자 등
4. 기타 → 앞서 배웠던 `sizeof` 등...

다음 예제로 한번에 정리해 본다.
```c
//  
// Created by Changjoon Lee on 2023/05/17.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	printf("\n산술연산자\n");  
	printf("5+4=%d\n", 5 + 4);  
	printf("5-4=%d\n", 5 - 4);  
	printf("5*4=%d\n", 5 * 4);  
	printf("5/4=%d\n", 5 / 4);  
	printf("5%4=%d\n", 5 % 4);  
	  
	int val = 10;  
	printf("\n복합대입연산자\n");  
	printf("%d\n", val += 10);  
	printf("%d\n", val -= 10);  
	printf("%d\n", val *= 10);  
	printf("%d\n", val /= 10);  
	printf("%d\n", val %= 10);  
	  
	val = 1;  
	printf("\n증감연산자\n");  
	printf("전위증가 = %d\n", ++val); // 증가 먼저 하고 출력  
	printf("%d\n", val);  
	printf("후위증가 = %d\n", val++); // 출력 먼저 하고 증가  
	printf("%d\n", val);  
	  
	val = 100;  
	printf("\n부호연산자\n");  
	printf("%d\n", +val);  
	printf("%d\n", -val);  
	  
	return 0;  
}
```

결과는 다음과 같다.
![[스크린샷 2023-05-17 오후 12.15.38.png]]

이제 관계연산자를 알아보자.
```c
//  
// Created by Changjoon Lee on 2023/05/17.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int val1 = 10, val2 = 20, result = 0;  
	printf("관계연산자\n");  
	result = val1 > val2;  
	printf("%d\n", result);  
	result = val1 == val2;  
	printf("%d\n", result);  
	result = val1 <= val2;  
	printf("%d\n", result);  
	result = val1 == 10;  
	printf("%d\n", result);  
	
	val1 = 10, val2 = 20;  
	printf("\n논리연산자\n");  
	printf("%d\n", val1 > 10 || val2 == 20);  
	printf("%d\n", 1 && 1);  
	printf("%d\n", 1 || 1);  
	printf("%d\n", !1); // 참을 나타내는 '1'이다.  
	printf("%d\n", !0); // 참을 나타내는 '0'이다.  
	printf("%d\n", !- 10); // 0이 아닌 모든 숫자는 값이 있는 걸로 컴파일러가 인식을 해서 1(참)으로 해석한다.
	
	return 0;  
}
```

C언어 시절에는 참 거짓을 나타내는 boolean 형이 없었다.
그래서 int 형을 이용하여 결과를 저장하는데, 1은 참, 0은 거짓을 나타내는 것으로 약속했다.
이러다 보니 산술연산/관계논리연산이 혼재되어 사용하는 경우가 생겼다.
C++에서는 이 점을 보완하여 boolean 형이 등장했다.
Java, C#은 엄격하게 숫자와 참/거짓을 구분하여 사용한다.

위 코드를 실행하면 다음과 같은 결과가 나온다.
![[스크린샷 2023-05-17 오후 12.29.53.png]]

그리고 비트연산자.
```c
//  
// Created by Changjoon Lee on 2023/05/17.  
//  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	int val1 = 17, val2 = 33;  
	printf("%d\n", val1 & val2); // 참 and 참  
	printf("%d\n", val1 | val2); // 비트 간에 orprintf("%d\n", val1 ^ val2); // 비트 간에 xorprintf("%d\n", ~val1); // 비트를 반전  
	  
	return 0;  
}
```

실행을 하면 다음과 같은 결과가 나온다.
![[스크린샷 2023-05-17 오후 12.41.17.png]]

왜 이런 결과가 나오는 걸까?
비트를 통해 다음과 같이 17과 33을 저장한다.
`00010001`, `00100001` (맨 뒤부터 숫자를 써 간다는 것에 주의)
이를 더해보자.
이를 `&`로 연산하면 동일한 자리수에 and 연산이 일어난다.
둘 중 하나라도 `0`이 있으면 `0`을 출력하는 식으로 연산되는 것이다.
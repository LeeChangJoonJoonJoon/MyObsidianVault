

2023-05-24

----
#Cpp #CS #수업 #C #객체지향 #객체

## 개요
어제까지 포인터와 `struct`에 대해 배웠다.

## 내용
C언어에서 구조체 사용은 `struct Human human;`와 같이 `struct`키워드를 붙여줘야 했다.
다만 계속 쓰다 보니 `struct`를 생략해서 표현하기를 원했기에 C++부터는 `struct`의 생략이 가능해졌다.
C언어에서는 `typeof`를 사용하면 `struct`의 생략이 가능하다.

먼저 첫번째 방법을 소개한다.
```c
struct HUMAN  
{  
	char name[20];  
	int age;  
	float height;  
};  
typedef struct _Human Human;
```

그리고 두번째 방법은 다음과 같다.
```c
typedef struct _HUMAN  
{  
	char name[20];  
	int age;  
	float height;  
}Human;
```

이제 구조체 포인터에 대해 알아보자.
```c
//  
// Created by Changjoon Lee on 2023/05/24.  
//  
  
#include <stdio.h>  
#include <string.h>  
  
#define HUMAN_NUM 3  
  
typedef struct _HUMAN  
{  
	char name[20];  
	int age;  
	float height;  
}Human;  
  
int main(int argc, const char * argv[])  
{  
	Human human;  
	Human* pHuman = &human;  
	  
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

여기 구조체의 주소에 접근할 수 있는 방법이 두가지가 있다.
```c
(*pHuman).name;
pHuman->name;
```

이 둘 모두 가능하지만, 일반적으로 많이 쓰이는 방법은 후자이다.
다음과 같이 사용할 수 있다.
```c
//  
// Created by Changjoon Lee on 2023/05/24.  
//  
  
#include <stdio.h>  
#include <string.h>  
  
#define HUMAN_NUM 3  
  
typedef struct _HUMAN  
{  
	char name[20];  
	int age;  
	float height;  
}Human;  
  
int main(int argc, const char * argv[])  
{  
	Human human;  
	Human* pHuman = &human;  
	  
	printf("이름 >> ");  
	fgets(pHuman->name, strlen(pHuman->name), stdin);  
	human.name[strlen(pHuman->name) - 1] = '\0';  
	printf("나이 >> ");  
	scanf("%d\n", &pHuman->age);  
	printf("키 >> ");  
	scanf("%f", &pHuman->height);  
	  
	printf("*******************\n");  
	printf("이름 : %s\n", pHuman->name);  
	printf("나이 : %d\n", pHuman->age);  
	printf("키 : %.f\n", pHuman->height);  
	printf("*******************\n");  
	  
	return 0;  
}
```
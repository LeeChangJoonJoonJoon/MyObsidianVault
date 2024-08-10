

2023-05-25

----
#Cpp #CS #수업 #C #객체지향 #객체

## 개요
오늘은 구조체를 매개변수로 전달하는 것을 배워볼 것이다.

## 내용
```c
//  
// Created by Changjoon Lee on 2023/05/25.  
//  
  
#include <stdio.h>  
#include <stdlib.h>  
  
#define SOMEMACRONAME  
  
typedef struct _Human{  
	char name[20];  
	int age;  
	float height;  
}Human;  
  
void ShowHuman(Human human)  
{  
	printf("%s\n", human.name);  
	printf("%d\n", human.age);  
	printf("%.2f\n", human.height);  
  
}  
  
int main(int argc, const char * argv[])  
{  
	Human hong = { "홍길동", 25, 174.f };  
	ShowHuman(hong);  
	  
	return 0;  
}
```

여기서 `hong`의 값이 모두 구조체에 복사된다.
그러나 구조체의 크기가 커지면 프로그램에 부하를 가져올 수 있다.
그러므로 이 예제처럼 사용하는 것은 권장하지 않고, 포인터를 이용하는게 바람직하다.

포인터를 사용하여 `ShowHuman` 함수를 수정해 보면 다음과 같다.
```c
void ShowHuman(Human *p_human)  
{  
	printf("%s\n", p_human->name);  
	printf("%d\n", p_human->age);  
	printf("%.2f\n", p_human->height);  
}
```

이렇게 하면 주소만 복사된다.
객체화한 `hong`은 다음과 같이 넣어 준다.
```c
ShowHuman(&hong);
```

다음은 구조체의 정렬을 알아볼 것이다.
```c
  
typedef struct _St1{  
	char ch;  
	char ch2;  
	short s;  
	int i;  
	double d;  
}St1;  
  
typedef struct _St2{  
	char ch;  
	char ch2;  
	double d;  
	short s;  
	int i;  
}St2;
```

관찰을 해보면, 이 두 구조체는 변수는 똑같이 있지만 선언되는 순서가 다른 것을 알 수 있다.
이를 메인함수에서 `sizeof()`를 통해 출력해봤다.
근데 다음과 같이 서로 다른 값이 나오는 것을 확인할 수 있다.
![[스크린샷 2023-05-25 오전 10.22.09.png]]

내가 현재 쓰고 있는 맥 환경은 ARM64이다. 이 64비트 아키텍처는 메모리에 8바이트 단위로 정보를 저장한다.
`St1`은 `ch, ch2, s, i`는 각각 1 byte, 1 byte, 2 byte,  4 byte를 차지한다.
즉 8 byte가 꽉 차도록 배치되었다.
그런 다음 8 byte를 차지하는 `d`가 온 것이다.
하지만 `d`가 중간에 들어오면, `ch, ch2`를 저장하고 나머지 6 byte에 `d`를 넣을 수 없기에 빈 공간을 남겨놓고 다음 단위로 넘어가서 저장한다.

결론적으로, 전자는 16 byte를 차지하고, 후자는 24 byte를 차지하게 된다.

그럼 `pragma`를 이용하여 빈 공간이 없도록 정보를 메모리에 올리는 법을 배워보자.
```c
//  
// Created by Changjoon Lee on 2023/05/25.  
//  
  
#include <stdio.h>  
#include <stdlib.h>  
  
#define SOMEMACRONAME  
  
#pragma pack(push, 1)  
typedef struct _St1{  
	char ch;  
	char ch2;  
	short s;  
	int i;  
	double d;  
}St1;  
#pragma pack(pop);  
  
#pragma pack(push, 1)  
typedef struct _St2{  
	char ch;  
	char ch2;  
	double d;  
	short s;  
	int i;  
}St2;  
#pragma pack(pop);  
  
int main(int argc, const char * argv[])  
{  
	printf("St1의 크기 : %d\n", sizeof(St1));  
	printf("St2의 크기 : %d\n", sizeof(St2));  
	  
	return 0;  
}
```

정보의 빈 공간이 없도록 1 byte 단위로 공간을 구성하라는 의미이다.
이렇게 하면 `St1`과 `St2`의 메모리 공간의 크기가 동일하게 된다.
주로 이 구조체를 `char` 배열로 변환하여 소켓통신을 통해 상대방에게 전달할 때 이 처리를 서버, 클라이언트에게 해주지 않으면 서버와 클라이언트의 구조체 크기가 다를 경우 정상적인 데이터  송수신이 안된다.

이렇게 하여 둘 다 동일한 크기를 차지하도록 만들 수 있는 것이다.
![[스크린샷 2023-05-25 오전 10.34.51.png]]

이제 열거형을 배워보자.
```c
//  
// Created by Changjoon Lee on 2023/05/25.  
//  
  
#include <stdio.h>  
#include <stdlib.h>  
  
#define SOMEMACRONAME  
  
typedef enum _CNUM{  
	ONE = 1,  
	TWO,  
	THREE,  
	FOUR,  
	FIVE  
}CNUM;  
  
int main(int argc, const char * argv[])  
{  
	printf("ONE %d\n", ONE);  
	printf("TWO %d\n", TWO);  
	printf("THREE %d\n", THREE);  
	printf("FOUR %d\n", FOUR);  
	printf("FIVE %d\n", FIVE);  
	  
	CNUM cnum = ONE;  
	for (cnum = ONE; cnum < FIVE + 1; cnum++) {  
		printf("%d\n", cnum);  
	}  
  
	return 0;  
}
```

C언어에서의 `enum`형은 상수가 여러개 필요할 때 정의해서 사용한다.
컴파일러는 `enum`형의 값을 `int`로 처리한다.
값을 부여한 `enum`형 상수의 다음 상수는 자동으로 1이 증가한 값으로 부여된다.

프로그램에서는 숫자를 쓰기보다는 숫자를 상수로 변환해서 사용하는 것이 프로그램의 이해를 높여준다.
그래서, 첫번째 방법으로는 `#define`문을 쓰는 방법이 있다.
```c
//  
// Created by Changjoon Lee on 2023/05/24.  
//  
#include <stdio.h>  
#include <stdlib.h>  
//#include "osXgetch.h"  
  
#define ADD 1  
#define SUB 2  
#define MUL 3  
#define DIV 4  
#define EXIT 5  
  
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
		case ADD:  
			dNum0 = getDoubleNum();  
			dNum1 = getDoubleNum();  
			result = Add(dNum0, dNum1);  
			printResult(result);  
			break;  
		case SUB:  
			dNum0 = getDoubleNum();  
			dNum1 = getDoubleNum();  
			result = Sub(dNum0, dNum1);  
			printResult(result);  
			break;  
		case MUL:  
			dNum0 = getDoubleNum();  
			dNum1 = getDoubleNum();  
			result = Mul(dNum0, dNum1);  
			printResult(result);  
			break;  
		case DIV:  
			dNum0 = getDoubleNum();  
			dNum1 = getDoubleNum();  
			result = Div(dNum0, dNum1);  
			printResult(result);  
			break;  
		case EXIT:  
			isRun = 0;  
			break;  
	}  
	  
	//get_ch(); // 키보드 아무거나 입력될 때까지 대기  
}  
return 0;  
}
```

전에 했던 것과 똑같은 결과가 나오는 것을 확인할 수 있다.
혹은 여기에 `enum`형을 사용해서 `case`문에 대응시켜줄 수 있다.

다음은 전처리기.
```c
//  
// Created by Changjoon Lee on 2023/05/25.  
//  
  
#include <stdio.h>  
  
#define SOMEMACRONAME  
  
int main(int argc, const char * argv[])  
{  
	printf("HelloWorld\n");  
	  
	return 0;  
}
```

C언어에서는 소스코드로 작성하고 이것이 exe 파일로 만들어지는 절차가 있다.
소스코드 작성 → 컴파일(소스코드를 기계어로 전환)로 인해 \*.obj 라는 결과물이 생김 → 링크(관련 라이브러리-\*.lib, \*.dll-를 코드에 추가) → \*.exe(바이너리 파일) → 실행시 exe는 메모리에 올라가서 프로세스가 되고 CPU에 의해 동작된다.
이것이 우리가 빌드할 때에 생기는 일이다.

전처리기는 위의 `<stdio.h>` 파일을 모두 가져와서 포함시킨다.
다음과 같이 이 헤더파일의 내용을 복사해서 붙여 넣어도 같은 결과를 내놓는다.
```c
//  
// Created by Changjoon Lee on 2023/05/25.  
//  
  
/*  
* Copyright (c) 2000, 2005, 2007, 2009, 2010 Apple Inc. All rights reserved.  
*  
* @APPLE_LICENSE_HEADER_START@  
*  
* This file contains Original Code and/or Modifications of Original Code  
* as defined in and that are subject to the Apple Public Source License  
* Version 2.0 (the 'License'). You may not use this file except in  
* compliance with the License. Please obtain a copy of the License at  
* http://www.opensource.apple.com/apsl/ and read it before using this  
* file.  
*  
* The Original Code and all software distributed under the License are  
* distributed on an 'AS IS' basis, WITHOUT WARRANTY OF ANY KIND, EITHER  
* EXPRESS OR IMPLIED, AND APPLE HEREBY DISCLAIMS ALL SUCH WARRANTIES,  
* INCLUDING WITHOUT LIMITATION, ANY WARRANTIES OF MERCHANTABILITY,  
* FITNESS FOR A PARTICULAR PURPOSE, QUIET ENJOYMENT OR NON-INFRINGEMENT.  
* Please see the License for the specific language governing rights and  
* limitations under the License.  
*  
* @APPLE_LICENSE_HEADER_END@  
*/  
/*-  
* Copyright (c) 1990, 1993  
* The Regents of the University of California. All rights reserved.  
*  
* This code is derived from software contributed to Berkeley by  
* Chris Torek.  
*  
* Redistribution and use in source and binary forms, with or without  
* modification, are permitted provided that the following conditions  
* are met:  
* 1. Redistributions of source code must retain the above copyright  
* notice, this list of conditions and the following disclaimer.  
* 2. Redistributions in binary form must reproduce the above copyright  
* notice, this list of conditions and the following disclaimer in the  
* documentation and/or other materials provided with the distribution.  
* 3. All advertising materials mentioning features or use of this software  
* must display the following acknowledgement:  
* This product includes software developed by the University of  
* California, Berkeley and its contributors.  
* 4. Neither the name of the University nor the names of its contributors  
* may be used to endorse or promote products derived from this software  
* without specific prior written permission.  
*  
* THIS SOFTWARE IS PROVIDED BY THE REGENTS AND CONTRIBUTORS ``AS IS'' AND  
* ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE  
* IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE  
* ARE DISCLAIMED. IN NO EVENT SHALL THE REGENTS OR CONTRIBUTORS BE LIABLE  
* FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL  
* DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS  
* OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)  
* HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT  
* LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY  
* OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF  
* SUCH DAMAGE.  
*  
* @(#)stdio.h 8.5 (Berkeley) 4/29/95  
*/  
  
#ifndef _STDIO_H_  
#define _STDIO_H_  
  
#include <_stdio.h>  
  
__BEGIN_DECLS  
extern FILE *__stdinp;  
extern FILE *__stdoutp;  
extern FILE *__stderrp;  
__END_DECLS  
  
#define __SLBF 0x0001 /* line buffered */  
#define __SNBF 0x0002 /* unbuffered */  
#define __SRD 0x0004 /* OK to read */  
#define __SWR 0x0008 /* OK to write */  
/* RD and WR are never simultaneously asserted */  
#define __SRW 0x0010 /* open for reading & writing */  
#define __SEOF 0x0020 /* found EOF */  
#define __SERR 0x0040 /* found error */  
#define __SMBF 0x0080 /* _buf is from malloc */  
#define __SAPP 0x0100 /* fdopen()ed in append mode */  
#define __SSTR 0x0200 /* this is an sprintf/snprintf string */  
#define __SOPT 0x0400 /* do fseek() optimisation */  
#define __SNPT 0x0800 /* do not do fseek() optimisation */  
#define __SOFF 0x1000 /* set iff _offset is in fact correct */  
#define __SMOD 0x2000 /* true => fgetln modified _p text */  
#define __SALC 0x4000 /* allocate string space dynamically */  
#define __SIGN 0x8000 /* ignore this file in _fwalk */  
  
/*  
* The following three definitions are for ANSI C, which took them  
* from System V, which brilliantly took internal interface macros and  
* made them official arguments to setvbuf(), without renaming them.  
* Hence, these ugly _IOxxx names are *supposed* to appear in user code.  
*  
* Although numbered as their counterparts above, the implementation  
* does not rely on this.  
*/  
#define _IOFBF 0 /* setvbuf should set fully buffered */  
#define _IOLBF 1 /* setvbuf should set line buffered */  
#define _IONBF 2 /* setvbuf should set unbuffered */  
  
#define BUFSIZ 1024 /* size of buffer used by setbuf */  
#define EOF (-1)  
  
/* must be == _POSIX_STREAM_MAX <limits.h> */  
#define FOPEN_MAX 20 /* must be <= OPEN_MAX <sys/syslimits.h> */  
#define FILENAME_MAX 1024 /* must be <= PATH_MAX <sys/syslimits.h> */  
  
/* System V/ANSI C; this is the wrong way to do this, do *not* use these. */  
#ifndef _ANSI_SOURCE  
#define P_tmpdir "/var/tmp/"  
#endif  
#define L_tmpnam 1024 /* XXX must be == PATH_MAX */  
#define TMP_MAX 308915776  
  
#ifndef SEEK_SET  
#define SEEK_SET 0 /* set file offset to offset */  
#endif  
#ifndef SEEK_CUR  
#define SEEK_CUR 1 /* set file offset to current plus offset */  
#endif  
#ifndef SEEK_END  
#define SEEK_END 2 /* set file offset to EOF plus offset */  
#endif  
  
#define stdin __stdinp  
#define stdout __stdoutp  
#define stderr __stderrp  
  
#ifdef _DARWIN_UNLIMITED_STREAMS  
#if defined(__IPHONE_OS_VERSION_MIN_REQUIRED) && __IPHONE_OS_VERSION_MIN_REQUIRED < __IPHONE_3_2  
#error "_DARWIN_UNLIMITED_STREAMS specified, but -miphoneos-version-min version does not support it."  
#elif defined(__MAC_OS_X_VERSION_MIN_REQUIRED) && __MAC_OS_X_VERSION_MIN_REQUIRED < __MAC_10_6  
#error "_DARWIN_UNLIMITED_STREAMS specified, but -mmacosx-version-min version does not support it."  
#endif  
#endif  
  
/* ANSI-C */  
  
__BEGIN_DECLS  
void clearerr(FILE *);  
int fclose(FILE *);  
int feof(FILE *);  
int ferror(FILE *);  
int fflush(FILE *);  
int fgetc(FILE *);  
int fgetpos(FILE * __restrict, fpos_t *);  
char *fgets(char * __restrict, int, FILE *);  
#if defined(_DARWIN_UNLIMITED_STREAMS) || defined(_DARWIN_C_SOURCE)  
FILE *fopen(const char * __restrict __filename, const char * __restrict __mode) __DARWIN_ALIAS_STARTING(__MAC_10_6, __IPHONE_3_2, __DARWIN_EXTSN(fopen));  
#else /* !_DARWIN_UNLIMITED_STREAMS && !_DARWIN_C_SOURCE */  
FILE *fopen(const char * __restrict __filename, const char * __restrict __mode) __DARWIN_ALIAS_STARTING(__MAC_10_6, __IPHONE_2_0, __DARWIN_ALIAS(fopen));  
#endif /* (DARWIN_UNLIMITED_STREAMS || _DARWIN_C_SOURCE) */  
int fprintf(FILE * __restrict, const char * __restrict, ...) __printflike(2, 3);  
int fputc(int, FILE *);  
int fputs(const char * __restrict, FILE * __restrict) __DARWIN_ALIAS(fputs);  
size_t fread(void * __restrict __ptr, size_t __size, size_t __nitems, FILE * __restrict __stream);  
FILE *freopen(const char * __restrict, const char * __restrict,  
FILE * __restrict) __DARWIN_ALIAS(freopen);  
int fscanf(FILE * __restrict, const char * __restrict, ...) __scanflike(2, 3);  
int fseek(FILE *, long, int);  
int fsetpos(FILE *, const fpos_t *);  
long ftell(FILE *);  
size_t fwrite(const void * __restrict __ptr, size_t __size, size_t __nitems, FILE * __restrict __stream) __DARWIN_ALIAS(fwrite);  
int getc(FILE *);  
int getchar(void);  
  
#if !defined(_POSIX_C_SOURCE)  
__deprecated_msg("This function is provided for compatibility reasons only. Due to security concerns inherent in the design of gets(3), it is highly recommended that you use fgets(3) instead.")  
#endif  
char *gets(char *);  
  
void perror(const char *) __cold;  
int printf(const char * __restrict, ...) __printflike(1, 2);  
int putc(int, FILE *);  
int putchar(int);  
int puts(const char *);  
int remove(const char *);  
int rename (const char *__old, const char *__new);  
void rewind(FILE *);  
int scanf(const char * __restrict, ...) __scanflike(1, 2);  
void setbuf(FILE * __restrict, char * __restrict);  
int setvbuf(FILE * __restrict, char * __restrict, int, size_t);  
  
__swift_unavailable("Use snprintf instead.")  
#if !defined(_POSIX_C_SOURCE)  
__deprecated_msg("This function is provided for compatibility reasons only. Due to security concerns inherent in the design of sprintf(3), it is highly recommended that you use snprintf(3) instead.")  
#endif  
int sprintf(char * __restrict, const char * __restrict, ...) __printflike(2, 3);  
  
int sscanf(const char * __restrict, const char * __restrict, ...) __scanflike(2, 3);  
FILE *tmpfile(void);  
  
__swift_unavailable("Use mkstemp(3) instead.")  
#if !defined(_POSIX_C_SOURCE)  
__deprecated_msg("This function is provided for compatibility reasons only. Due to security concerns inherent in the design of tmpnam(3), it is highly recommended that you use mkstemp(3) instead.")  
#endif  
char *tmpnam(char *);  
  
int ungetc(int, FILE *);  
int vfprintf(FILE * __restrict, const char * __restrict, va_list) __printflike(2, 0);  
int vprintf(const char * __restrict, va_list) __printflike(1, 0);  
  
__swift_unavailable("Use vsnprintf instead.")  
#if !defined(_POSIX_C_SOURCE)  
__deprecated_msg("This function is provided for compatibility reasons only. Due to security concerns inherent in the design of sprintf(3), it is highly recommended that you use vsnprintf(3) instead.")  
#endif  
int vsprintf(char * __restrict, const char * __restrict, va_list) __printflike(2, 0);  
__END_DECLS  
  
  
  
/* Additional functionality provided by:  
* POSIX.1-1988  
*/  
  
#if __DARWIN_C_LEVEL >= 198808L  
#define L_ctermid 1024 /* size for ctermid(); PATH_MAX */  
  
__BEGIN_DECLS  
#include <_ctermid.h>  
  
#if defined(_DARWIN_UNLIMITED_STREAMS) || defined(_DARWIN_C_SOURCE)  
FILE *fdopen(int, const char *) __DARWIN_ALIAS_STARTING(__MAC_10_6, __IPHONE_3_2, __DARWIN_EXTSN(fdopen));  
#else /* !_DARWIN_UNLIMITED_STREAMS && !_DARWIN_C_SOURCE */  
FILE *fdopen(int, const char *) __DARWIN_ALIAS_STARTING(__MAC_10_6, __IPHONE_2_0, __DARWIN_ALIAS(fdopen));  
#endif /* (DARWIN_UNLIMITED_STREAMS || _DARWIN_C_SOURCE) */  
int fileno(FILE *);  
__END_DECLS  
#endif /* __DARWIN_C_LEVEL >= 198808L */  
  
  
/* Additional functionality provided by:  
* POSIX.2-1992 C Language Binding Option  
*/  
  
#if __DARWIN_C_LEVEL >= 199209L  
__BEGIN_DECLS  
int pclose(FILE *) __swift_unavailable("Use posix_spawn APIs or NSTask instead. (On iOS, process spawning is unavailable.)");  
#if defined(_DARWIN_UNLIMITED_STREAMS) || defined(_DARWIN_C_SOURCE)  
FILE *popen(const char *, const char *) __DARWIN_ALIAS_STARTING(__MAC_10_6, __IPHONE_3_2, __DARWIN_EXTSN(popen)) __swift_unavailable("Use posix_spawn APIs or NSTask instead. (On iOS, process spawning is unavailable.)");  
#else /* !_DARWIN_UNLIMITED_STREAMS && !_DARWIN_C_SOURCE */  
FILE *popen(const char *, const char *) __DARWIN_ALIAS_STARTING(__MAC_10_6, __IPHONE_2_0, __DARWIN_ALIAS(popen)) __swift_unavailable("Use posix_spawn APIs or NSTask instead. (On iOS, process spawning is unavailable.)");  
#endif /* (DARWIN_UNLIMITED_STREAMS || _DARWIN_C_SOURCE) */  
__END_DECLS  
#endif /* __DARWIN_C_LEVEL >= 199209L */  
  
/* Additional functionality provided by:  
* POSIX.1c-1995,  
* POSIX.1i-1995,  
* and the omnibus ISO/IEC 9945-1: 1996  
*/  
  
#if __DARWIN_C_LEVEL >= 199506L  
  
/* Functions internal to the implementation. */  
__BEGIN_DECLS  
int __srget(FILE *);  
int __svfscanf(FILE *, const char *, va_list) __scanflike(2, 0);  
int __swbuf(int, FILE *);  
__END_DECLS  
  
/*  
* The __sfoo macros are here so that we can  
* define function versions in the C library.  
*/  
#define __sgetc(p) (--(p)->_r < 0 ? __srget(p) : (int)(*(p)->_p++))  
#if defined(__GNUC__) && defined(__STDC__)  
__header_always_inline int __sputc(int _c, FILE *_p) {  
if (--_p->_w >= 0 || (_p->_w >= _p->_lbfsize && (char)_c != '\n'))  
return (*_p->_p++ = _c);  
else  
return (__swbuf(_c, _p));  
}  
#else  
/*  
* This has been tuned to generate reasonable code on the vax using pcc.  
*/  
#define __sputc(c, p) \  
(--(p)->_w < 0 ? \  
(p)->_w >= (p)->_lbfsize ? \  
(*(p)->_p = (c)), *(p)->_p != '\n' ? \  
(int)*(p)->_p++ : \  
__swbuf('\n', p) : \  
__swbuf((int)(c), p) : \  
(*(p)->_p = (c), (int)*(p)->_p++))  
#endif  
  
#define __sfeof(p) (((p)->_flags & __SEOF) != 0)  
#define __sferror(p) (((p)->_flags & __SERR) != 0)  
#define __sclearerr(p) ((void)((p)->_flags &= ~(__SERR|__SEOF)))  
#define __sfileno(p) ((p)->_file)  
  
__BEGIN_DECLS  
void flockfile(FILE *);  
int ftrylockfile(FILE *);  
void funlockfile(FILE *);  
int getc_unlocked(FILE *);  
int getchar_unlocked(void);  
int putc_unlocked(int, FILE *);  
int putchar_unlocked(int);  
  
/* Removed in Issue 6 */  
#if !defined(_POSIX_C_SOURCE) || _POSIX_C_SOURCE < 200112L  
int getw(FILE *);  
int putw(int, FILE *);  
#endif  
  
__swift_unavailable("Use mkstemp(3) instead.")  
#if !defined(_POSIX_C_SOURCE)  
__deprecated_msg("This function is provided for compatibility reasons only. Due to security concerns inherent in the design of tempnam(3), it is highly recommended that you use mkstemp(3) instead.")  
#endif  
char *tempnam(const char *__dir, const char *__prefix) __DARWIN_ALIAS(tempnam);  
__END_DECLS  
  
#ifndef lint  
#define getc_unlocked(fp) __sgetc(fp)  
#define putc_unlocked(x, fp) __sputc(x, fp)  
#endif /* lint */  
  
#define getchar_unlocked() getc_unlocked(stdin)  
#define putchar_unlocked(x) putc_unlocked(x, stdout)  
#endif /* __DARWIN_C_LEVEL >= 199506L */  
  
  
  
/* Additional functionality provided by:  
* POSIX.1-2001  
* ISO C99  
*/  
  
#if __DARWIN_C_LEVEL >= 200112L  
#include <sys/_types/_off_t.h>  
  
__BEGIN_DECLS  
int fseeko(FILE * __stream, off_t __offset, int __whence);  
off_t ftello(FILE * __stream);  
__END_DECLS  
#endif /* __DARWIN_C_LEVEL >= 200112L */  
  
#if __DARWIN_C_LEVEL >= 200112L || defined(_C99_SOURCE) || defined(__cplusplus)  
__BEGIN_DECLS  
int snprintf(char * __restrict __str, size_t __size, const char * __restrict __format, ...) __printflike(3, 4);  
int vfscanf(FILE * __restrict __stream, const char * __restrict __format, va_list) __scanflike(2, 0);  
int vscanf(const char * __restrict __format, va_list) __scanflike(1, 0);  
int vsnprintf(char * __restrict __str, size_t __size, const char * __restrict __format, va_list) __printflike(3, 0);  
int vsscanf(const char * __restrict __str, const char * __restrict __format, va_list) __scanflike(2, 0);  
__END_DECLS  
#endif /* __DARWIN_C_LEVEL >= 200112L || defined(_C99_SOURCE) || defined(__cplusplus) */  
  
  
  
/* Additional functionality provided by:  
* POSIX.1-2008  
*/  
  
#if __DARWIN_C_LEVEL >= 200809L  
#include <sys/_types/_ssize_t.h>  
  
__BEGIN_DECLS  
int dprintf(int, const char * __restrict, ...) __printflike(2, 3) __OSX_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_4_3);  
int vdprintf(int, const char * __restrict, va_list) __printflike(2, 0) __OSX_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_4_3);  
ssize_t getdelim(char ** __restrict __linep, size_t * __restrict __linecapp, int __delimiter, FILE * __restrict __stream) __OSX_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_4_3);  
ssize_t getline(char ** __restrict __linep, size_t * __restrict __linecapp, FILE * __restrict __stream) __OSX_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_4_3);  
FILE *fmemopen(void * __restrict __buf, size_t __size, const char * __restrict __mode) __API_AVAILABLE(macos(10.13), ios(11.0), tvos(11.0), watchos(4.0));  
FILE *open_memstream(char **__bufp, size_t *__sizep) __API_AVAILABLE(macos(10.13), ios(11.0), tvos(11.0), watchos(4.0));  
__END_DECLS  
#endif /* __DARWIN_C_LEVEL >= 200809L */  
  
  
  
/* Darwin extensions */  
  
#if __DARWIN_C_LEVEL >= __DARWIN_C_FULL  
__BEGIN_DECLS  
extern __const int sys_nerr; /* perror(3) external variables */  
extern __const char *__const sys_errlist[];  
  
int asprintf(char ** __restrict, const char * __restrict, ...) __printflike(2, 3);  
char *ctermid_r(char *);  
char *fgetln(FILE *, size_t *);  
__const char *fmtcheck(const char *, const char *) __attribute__((format_arg(2)));  
int fpurge(FILE *);  
void setbuffer(FILE *, char *, int);  
int setlinebuf(FILE *);  
int vasprintf(char ** __restrict, const char * __restrict, va_list) __printflike(2, 0);  
  
  
/*  
* Stdio function-access interface.  
*/  
FILE *funopen(const void *,  
int (* _Nullable)(void *, char *, int),  
int (* _Nullable)(void *, const char *, int),  
fpos_t (* _Nullable)(void *, fpos_t, int),  
int (* _Nullable)(void *));  
__END_DECLS  
#define fropen(cookie, fn) funopen(cookie, fn, 0, 0, 0)  
#define fwopen(cookie, fn) funopen(cookie, 0, fn, 0, 0)  
  
#define feof_unlocked(p) __sfeof(p)  
#define ferror_unlocked(p) __sferror(p)  
#define clearerr_unlocked(p) __sclearerr(p)  
#define fileno_unlocked(p) __sfileno(p)  
  
#endif /* __DARWIN_C_LEVEL >= __DARWIN_C_FULL */  
  
  
#ifdef _USE_EXTENDED_LOCALES_  
#include <xlocale/_stdio.h>  
#endif /* _USE_EXTENDED_LOCALES_ */  
  
#if defined (__GNUC__) && _FORTIFY_SOURCE > 0 && !defined (__cplusplus)  
/* Security checking functions. */  
#include <secure/_stdio.h>  
#endif  
  
#endif /* _STDIO_H_ */  
  
  
#define SOMEMACRONAME  
  
int main(int argc, const char * argv[])  
{  
	printf("HelloWorld\n");  
	  
	return 0;  
}
```

전처리기가 하는 두번째 일을 살펴보자.
```c
//  
// Created by Changjoon Lee on 2023/05/25.  
//  
  
#include <stdio.h>  
#include <stdlib.h>  
  
#define PI 3.14  
  
int main(int argc, const char * argv[])  
{  
	float rad, area;  
	printf("반지름 입력 >> ");  
	scanf("%f", &rad);  
	area = rad * rad * PI;  
	printf("반지름 %.2f인 원의 넓이는 %.2f입니다\n", rad, area);  
	  
	return 0;  
}
```

이를 실행하면 다음 결과가 나온다.
![[스크린샷 2023-05-25 오전 11.27.22.png]]

컴파일 이전에 값을 코드 내에 집어 넣어 준다.

이제 매크로에 대해 알아볼 것이다.
```c
//  
// Created by Changjoon Lee on 2023/05/25.  
//  
  
#include <stdio.h>  
#include <stdlib.h>  
  
#define SQRT(x) x*x  
  
double dsqrt(double x)  
{  
	return x * x;  
}  
  
int main(int argc, const char * argv[])  
{  
	printf("%d\n", SQRT(100));  
	printf("%d\n", SQRT(-20));  
	printf("%.2lf\n", SQRT(2.5));  
	  
	printf("%d\n", SQRT(4+5));  
	printf("%2lf\n", SQRT(4.2+5.3));  
	printf("%2lf\n", dsqrt(4+5));  
	  
	return 0;  
}
```

이를 실행하면 다음 결과 나온다.
![[스크린샷 2023-05-25 오전 11.35.58.png]]

C언어에서 간단한 함수의 기능은 일반 함수를 만들기 보단 종종 매크로 함수를 만들어서 사용한다.
여기서 `#define`문은 코드 내의 `SQRT(x)`를 `x*x`로 치환한다.
근데, 여기서 `SQRT(4+5)`의 결과가 `29`가 나왔다.
컴퓨터가 인식하는 과정을 보면 다음과 같다.
```c
printf("%d\n", SQRT(4+5)); // 4+5*4+5
```

즉, `5*4`가 먼저 연산되고 여기에 `5`와 `4`가 더해지는 것이다.
매크로 함수는 일반함수와 다르게 단순치환되기에 `x`에 수식이 들어가면 연산자 우선순위에 의해 의도하지 않은 결과가 나올 수도 있다.
***그러므로 매크로 함수의 매개변수 `x`는 반드시 `()`를 쳐서 `()`안의 연산이 먼저 진행되도록 해야 한다.***

매크로 함수를 수정한 모습은 다음과 같다.
```c
#define SQRT(x) ((x)*(x))
```

이는 프로세스를 끊고 함수를 호출하여 변수를 위한 공간을 할당하는 등의 부하를 줄여준다.
왜냐하면 매크로는 단순치환이기 때문이다!

이번에는 조건 컴팡일을 알아볼 것이다.
```c
//  
// Created by Changjoon Lee on 2023/05/25.  
//  
  
#include <stdio.h>  
  
#define _DEBUG_  
  
void DebugPrint(char *str)  
{  
#ifdef _DEBUG_  
	printf("%d\n", str);  
#endif  
}  
  
int main(int argc, const char * argv[])  
{  
	int num;  
	char str[20];  
	printf("숫자 입력 >> ");  
	scanf("%d", &num);  
	sprintf(str, "입력 숫자 = %d", num);  
	DebugPrint(str);  
	  
	return 0;  
}
```

여기 `#ifdef _DEBUG_`의 의미는 다음과 같다.
`_DEBUG_`가 `#define`되어 있지 않다면 그 아래에 있는 `_DEBUG_ ~#endif` 사이의 영역은 컴파일에 포함되지 않고 사라지게 된다.

우리는 이런 도구를 통해 개발할 때에는 디버깅을 해주고, 릴리즈 시에는 이를 다 지워버릴 수 있는 편의를 제공한다.

이제 대망의 파일 분할을 배워볼 것이다.
C언어는 순차적으로 컴파일을 하기에 `showMenu();`를 메인함수 뒤로 두면 컴파일 에러가 발생한다.
하지만 다음과 같이 함수의 원형정보를 올리면 이를 해결할 수 있다.
```c
//
// Created by Changjoon Lee on 2023/05/25.
//
#include <stdio.h>
#include <stdlib.h>
//#include "osXgetch.h"

enum {
    ADD = 1, SUB, MUL, DIV, EXIT
};

void showMenu();
int getSelectMenu();
double Add(double dNum0, double dNum1);
double Sub(double dNum0, double dNum1);
double Mul(double dNum0, double dNum1);
double Div(double dNum0, double dNum1);
double getDoubleNum();
void printResult(double result);
void cls();


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
            case ADD:
                dNum0 = getDoubleNum();
                dNum1 = getDoubleNum();
                result = Add(dNum0, dNum1);
                printResult(result);
                break;
            case SUB:
                dNum0 = getDoubleNum();
                dNum1 = getDoubleNum();
                result = Sub(dNum0, dNum1);
                printResult(result);
                break;
            case MUL:
                dNum0 = getDoubleNum();
                dNum1 = getDoubleNum();
                result = Mul(dNum0, dNum1);
                printResult(result);
                break;
            case DIV:
                dNum0 = getDoubleNum();
                dNum1 = getDoubleNum();
                result = Div(dNum0, dNum1);
                printResult(result);
                break;
            case EXIT:
                isRun = 0;
                break;
        }

        //get_ch(); // 키보드 아무거나 입력될 때까지 대기
    }
    return 0;
}

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

```

이렇게 코드를 짜면 코드가 난잡해진다.
이렇게 일일이 복사를 해줘야 하는 어려움이 있다.
그래서 나온 방법이, 헤더라는 텍스트 파일을 만들고 전처리문인 `#include`를 통해 .c파일에 포함시켜 주는 것이다.
```c
//  
// Created by Changjoon Lee on 2023/05/25.  
//  
  
#ifndef CPPMYPRACTICE_ARITHCALC_H  
#define CPPMYPRACTICE_ARITHCALC_H  
  
void showMenu();  
int getSelectMenu();  
double Add(double dNum0, double dNum1);  
double Sub(double dNum0, double dNum1);  
double Mul(double dNum0, double dNum1);  
double Div(double dNum0, double dNum1);  
double getDoubleNum();  
void printResult(double result);  
void cls();  
  
#endif //CPPMYPRACTICE_ARITHCALC_H
```


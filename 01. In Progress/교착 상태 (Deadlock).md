

2023-05-14

----
#Server #MVC #PANIM #Cpp #CS #게임서버프로그래밍교과서

## 개요
우리는 앞서 잠금(`lock`)과 잠금해제(`unlock`)의 개념에 대해 배웠다. 
어떤 스레드가 어느 한 객체에 접근할 때 다른 스레드로 하여금 접근하지 못하도록 하는게 잠금이고, 이를 해제하여 다른 스레드가 접근할 수 있도록 만드는게 잠금해제이다.
이때 고려해야 할 것이 교착상태(Deadlcok)이다.

## 내용
간단하게 설명하자면, 이는 서로 다른 두 스레드가 서로에 대해 잠금을 걸고 있는 상태를 말한다.
이를테면 다음과 같은 코드에서 교착상태가 발생한다.
```Cpp
#include <iostream> // string class 등을 담고 있는 헤더파일
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일
#include <thread>

int a = 0;
int b = 0;

void Thread1()
{
	lock(a); // ... 1
	{
		a++; 
		lock(b); 
		{
			b++; // ... 2
		}
	}
}

void Thread2()
{
	lock(b); // ... 3
	{
		b++;
		lock(a); // ... 4
		{
			a++;
		}
	}
}

int main(int argc, const char * argv[])
{
	Thread1();
	Thread2();
	return 0;
}
```

서로가 서로에 대해 잠금을 걸고 있다.
`Thread1()`은 `2`에서 `Thread2()`가 `3`에서 한 잠금을 해제하면 실행이 된다 하고,
`Thread2()`은 `4`에서 `Thread2()`가 `1`에서 한 잠금을 해제하면 실행이 된다 하는 상황이다.
결국 아무것도 실행되지 않는다.

다른 예제를 살펴보자.
먼저 헤더파일.
```Cpp
#include <mutex>  
  
class CriticalSection {  
	std::recursive_mutex m_critSec;  
public:  
	CriticalSection();  
	~CriticalSection();  
	  
	void Lock();  
	void Unlock();  
};  
  
class CriticalSectionLock {  
	CriticalSection* m_pCritSec;  
public:  
	CriticalSectionLock(CriticalSection& critSec);  
	~CriticalSectionLock();  
};  
  
CriticalSection::CriticalSection() {}  
  
CriticalSection::~CriticalSection() {}  
  
void CriticalSection::Lock() {  
	m_critSec.lock();  
}  
  
void CriticalSection::Unlock() {  
	m_critSec.unlock();  
}  
  
CriticalSectionLock::CriticalSectionLock(CriticalSection& critSec) {  
	m_pCritSec = &critSec;  
	m_pCritSec->Lock();  
}  
  
CriticalSectionLock::~CriticalSectionLock() {  
	m_pCritSec->Unlock();  
}
```

그리고 이를 포함하여 작동하는 코드.
```Cpp
#include <thread>  
#include <iostream>  
#include "my_Deadlock_Example.h"  
  
int a;  
CriticalSection a_mutex;  
int b;  
CriticalSection b_mutex;  
  
int main(int argc, const char * argv[])  
{  
	std::thread t1([]()  
	{  
		while (true)  
		{  
			CriticalSectionLock lock(a_mutex);  
			a++;  
			CriticalSection lock2(b_mutex);  
			b++;  
			std::cout << "t1 done. \n";  
		}  
	});  
		  
	std::thread t2([]()  
	{  
		while (true)  
		{  
			CriticalSectionLock lock(b_mutex);  
			b++;  
			CriticalSection lock2(a_mutex);  
			a++;  
			std::cout << "t2 done. \n";  
		}  
	});  
	// 스레드들의 일이 끝날 때까지 기다린다.  
	// 사실상 무한반복이므로 끝나지 않는댜.  
	t1.join();  
	t2.join();  
	  
	return 0;  
}
```


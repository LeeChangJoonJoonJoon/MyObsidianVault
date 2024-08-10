

2023-05-13

----
#Mac #Cpp #문제 #문제해결 #error #지선생 #ChatGPT #Bard

## 개요
[[임계 영역과 뮤텍스]]를 공부하던 중 다음 헤더파일에서 컴파일 에러가 떴다...
`CRITICAL_SECTION` 클래스를 못 읽어내는 듯 보였다.
```Cpp
class CriticalSection{  
	CRITICAL_SECTION m_critSec;  
public:  
	CriticalSection();  
	~CriticalSection();  
	  
	void Lock();  
	void Unlock();  
};  
  
class CriticalSectionLock{  
	CriticalSection* m_pCritSec;  
public:  
	CriticalSectionLock(CriticalSection& critSec);  
	~CriticalSectionLock();  
};  
  
CriticalSection::CriticalSection()  
{  
	InitializeCriticalSectionEx(&m_critSec, 0, 0);  
}  
  
CriticalSection::~CriticalSection() {  
	DeleteCriticalSection(&m_critSec);  
}  
  
void CriticalSection::Lock() {  
	EnterCriticalSection(&m_critSec);  
}  
  
void CriticalSection::Unlock() {  
	LeaveCriticalSection(&m_critSec);  
}  
  
CriticalSectionLock::CriticalSectionLock(CriticalSection& critSec)  
{  
	m_pCritSec = &critSec;  
	m_pCritSec -> Lock();  
}  
  
CriticalSectionLock::~CriticalSectionLock()  
{  
	m_pCritSec -> Unlock();  
}  
//  
// Created by Changjoon Lee on 2023/05/14.  
//  
  
#ifndef CPPMYPRACTICE_DEADLOCK_EXAMPLE_H  
#define CPPMYPRACTICE_DEADLOCK_EXAMPLE_H  
  
#endif //CPPMYPRACTICE_DEADLOCK_EXAMPLE_H
```

## 내용
알고보니 맥의 `gcc-11` 환경에서는 `CRITICAL_SECTION` 라이브러리를 지원하지 않았던 것...
지선생과 바드(Bard)에게 물어봐서 알게 되었던 것이다.
![[스크린샷 2023-05-14 오전 8.55.36.png|650]]
![[스크린샷 2023-05-14 오전 8.55.05.png]]

즉, `CRITICAL_SECTION`는 윈도우에만 해당되고, 이것을 대체키 위한 키워드는 `std::mutex`이라는 것.
지선생이 바꿔주고 바드가 인정한 코드는 다음과 같다.
```C++
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
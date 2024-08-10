

2024-07-12

----
#동적할당 #LowLevel #Cpp #메모리 


## 개요
다음 클래스의 `SetString(const char* pszParam)`을 보자.
```cpp
//  
// Created by Changjoon Lee on 2/4/24.  
//  
  
#ifndef STRINGCTRLSAMPLE_MYSTRING_H  
#define STRINGCTRLSAMPLE_MYSTRING_H  
  
  
class CMyString  
{  
public:  
    CMyString();  
    ~CMyString();  
  
private:  
    char* m_pszData;  
    int m_nLength;  
  
public:  
    int SetString(const char* pszParam = "");  
    const char* GetString();  
    void Release();  
};  
  
  
#endif //STRINGCTRLSAMPLE_MYSTRING_H
```

그리고 다음은 구현부.
```cpp
//  
// Created by Changjoon Lee on 2/4/24.  
//  
#include <bits/stdc++.h>  
#include "MyString.h"
  
CMyString::CMyString()  
//        : m_pszData(NULL), // 오래된 책들은 NULL을 쓴다.  
        : m_pszData(nullptr),  
          m_nLength(0)  
{  
  
}  
  
CMyString::~CMyString()  
{  
  
}  
  
int CMyString::SetString(const char* pszParam)  
{  
    Release();  
  
    if(!pszParam) return 0;  
  
    int nLength = strlen(pszParam);  
    if (!nLength) return 0;  
  
    m_pszData = new char[nLength + 1];  
    strcpy(m_pszData, pszParam);  
  
    m_nLength = nLength;  
    return nLength;  
}  
  
const char* CMyString::GetString()  
{  
    return m_pszData;  
}  
  
void CMyString::Release()  
{  
    if (m_pszData)  
        delete[] m_pszData;  
  
    m_pszData = nullptr;  
    m_nLength = 0;  
}
```

위 코드는 교과서 174p에 나온다.
생기는 의문들은 다음과 같다.
1. 멤버변수로 `m_pszData`는 포인터인데 왜 '할당'이란 행위를 할까?
	이미 할당된 다른 변수의 포인터를 그냥 대입하면 되는 거 아닌가?
2. 할당하고 해제할 때, 왜 배열에 대해서 하듯 할까?

## 내용
1. 물론, 다음과 같이 작성해도 당장 에러는 생기지 않는다. 
```cpp
m_pszData = (char*)pszParam;
```

하지만, 이렇게 작성하면 `m_pszData`를 해제하려고 했는데 그것에 대입한 포인터 `pszParam`까지 해제되는 문제가 생긴다. 
이 상황에서 다른 포인터 멤버변수들이 똑같은 행위를 하고 있었으면 댕글링 포인터 문제가 생긴다.
따라서 '데이터 무결성'을 지키기 위해, 별도로 동적할당을 통해 메모리공간을 확보하고, 거기에 인수의 값을 `strcpy()`로 복사해 넣은 것.

2. 원래 이렇게 한다는 듯이 다들 써 놓았다...
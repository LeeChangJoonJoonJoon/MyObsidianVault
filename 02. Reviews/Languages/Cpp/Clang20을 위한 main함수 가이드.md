

2023-12-30

----
#C #CLion #Cpp 

## 개요
책 <이것이 C++이다>에는 다음과 같이 메인함수가 쓰여 있다.
```cpp
int _tmain(int argc, _TCHAR* argv[])
{
	// do something...
	
	return 0;
}
```

하지만 내가 쓰는 CLion에선 Clang20 버전을 쓰고 있다.
위와 같이 쓰면 `_TCHAR`와 같은 변수들은 인식을 못한다.

## 내용
다음 두 방법 중 하나를 쓰면 된다.

물론 `void` 리턴으로 메인함수의 시그니처를 지정하는 건 안된다(C#에선 됐었던 것 같은데...).
```cpp
int main()
{
	// do something...
	
	return 0;
}
```

```cpp
int main(int argc, const char* argv[])
{
	// do something...
	
	return 0;
}
```


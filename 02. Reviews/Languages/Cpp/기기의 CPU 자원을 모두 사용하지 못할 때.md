

2023-05-06

----
#Server #MVC #PANIM #Cpp #CS #게임서버프로그래밍교과서

## 개요
서버를 구축하려면 먼저 스레드(`Thread`)와 멀티스레딩(`MultiThreading`)에 대해 알아야 한다. 


## 내용
### 프로그램과 프로세스
프로그램은 저장장치에 있고, 프로세스는 램에 존재한다.
각 프로세스는 독립된 메모리를 사용함.

프로세스 내에는 여러개의 스레드가 있음.
스레드마다 스택(Stack)을 가진다(스택, 힙에 대해서는 [[객체와 참조]]를 참고). 
참고로, 스택은 클래스가 저장돼 있는 곳이고 객체를 만들어서 힙(Heap)에 저장한다.
![[스크린샷 2023-05-06 오전 9.32.05.png|400]]

다음은 멀티스레딩의 예시이다.
```cpp
#include <vector>  
#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <chrono>  
  
using namespace std;  
const int MaxCount = 150000;  
  
bool IsPrimeNumber(int number)  
{  
	if (number == 1)  
	return false; // 1은 소수를 구하는 데에 있어서 제외해 버린다.  
	if (number == 2 || number == 3)  
	return true; // 2 혹은 3이면 소수로 간주.  
	  
	// 1, 2, 3 중 어느 것에도 해당되지 않을 경우 다음의 과정을 통해 소수인지를 구해낸다.  
	for (int i = 2; i < number - 1; i++)  
	{  
		if ((number % i) == 0)  
		return false;  
	}  
	  
	return true;  
}  
	  
void PrintNumbers(const vector<int>& primes)  
{  
	for (int v : primes)  
	{  
		cout << v << endl;  
	}  
}  
	  
int main()  
{  
	vector<int> primes;  
	auto t0 = chrono::system_clock::now();  
	  
	for (int i = 1; i <= MaxCount; i++)  
	{  
		if (IsPrimeNumber(i))  
		{  
			primes.push_back(i);  
		}  
	}  
  
	auto t1 = chrono::system_clock::now();  
	auto duration = chrono::duration_cast<chrono::milliseconds>(t1 - t0).count();  
	cout << "Took " << duration << " miliseconds." << endl;  
	  
	PrintNumbers(primes);  
}
```

근데 `MaxCount = 1500000000`로 놓고 돌려봤더니 출력까지 한세월이다.
저자는 이 프로그램이 단일 스레드를 통해 구동되고 있기 때문에 느리다고 지적한다.

그러면 어떤 방법이 있을까?

```cpp
#include <vector>  
#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <chrono>  
#include <thread>
#include <memory>
  
using namespace std;  

const int MaxCount = 150000;  
const int ThreadCount = 4;  
  
bool IsPrimeNumber(int number)  
{  
	if (number == 1)  
		return false; // 1은 소수를 구하는 데에 있어서 제외해 버린다.  
	if (number == 2 || number == 3)  
		return true; // 2 혹은 3이면 소수로 간주.  
	for (int i = 2; i < number - 1; i++)  
	{  
		if ((number % i) == 0)  
		return false;  
	}  
	  
	return true;  
}  
	  
void PrintNumbers(const vector<int>& primes)  
{  
	for (int v : primes)  
	{  
		cout << v << endl;  
	}  
}  
  
int main()  
{  
	int num = 1; // 이는 전역변수이다. 각 스레드는 여기서 값을 꺼내온다.  
	vector<int> primes;  
	  
	auto t0 = chrono::system_clock::now();  
	  
	vector<shared_ptr<thread> > threads; // 작동할 워커 스레드  
	  
	for (int i = 0; i < ThreadCount; ++i)  
	{  
		shared_ptr<thread> thread(new class thread([&](){  
		// 각 스레드의 메인 함수  
		// 값을 가져올 수 있으면 루프를 돈다.  
			while(true)  
			{  
				int n;  
				n = num;  
				num++;  
				  
				if (n > MaxCount)  
					break;  
				
				if (isPrimeNumber(n))  
					primes.push_back(n);  
			}  
		}));  
		// 스레드 객체를 일단 갖고 있는다.  
		threads.push_back(thread);  
	}  
	  
	// 모든 스레드가 일을 마칠 때까지 기다린다.  
	for (auto thread : threads)  
	{  
		thread->join();  
	}  
	  
	  
	auto t1 = chrono::system_clock::now();  
	auto duration = chrono::duration_cast<chrono::milliseconds>(t1 - t0).count();  
	cout << "Took " << duration << " miliseconds." << endl;  
	  
	//PrintNumbers(primes);  
}
```

이렇게 코드를 짤 수도 있다.
책에서 나온 설명으로는 부족해서, ChatGPT에게 물어보았다.
필자는 특히나 스레드를 할당하는 명령어가 생소했다.
![[스크린샷 2023-05-06 오후 7.30.00.png]]

정리하자면...
`vector<int> primes;`는 소수를 넣을 배열을 선언하는 것이다.
`vector<shared_ptr<thread> > threads;`는 스레드를 넣을 배열을 선언하는 것이다.

다음은 람다 함수이다.
```cpp
shared_ptr<thread> thread(new class thread([&](){
    // 각 스레드의 메인 함수
    // 값을 가져올 수 있으면 루프를 돈다.
    while(true)
    {
        int n;
        n = num;
        num++;
		
        if (n > MaxCount)
            break;
		
        if (isPrimeNumber)
        {
            primes.push_back(n);
        }
    }
}));

```

이 [[람다 함수]]는 `new class thread([&](){...})`에서 `new class thread`의 매개변수로서 실행되며 그 본체는 `{...}`내에 있는 `while{...}` 구문이다.
즉, `shared_ptr<thread> thread(new thread...`를 통해 스레드를 생성한 뒤, 이 스레드 내에서 람다 함수를 실행시키는 것이다.

`while` 구문을 통해 각 스레드는 변수 `n`을 선언하고 여기에 각 스레드가 접근한 `num`변수를 참조하여 가져다 넣는다.
`num` 변수의 값을 가져오는게 아니라 참조하고자 [[캡쳐 (Capture)]] 부분에 `&` 심볼을 표기했다.

다시 돌아와서, 스레드에 대해 논해보자.
위 코드는 실행시 다음과 같은 에러가 발생한다.
(`Undefined symbols for architecture arm64:`는 너무 겁 먹지 말자.
그냥 `arm64` 환경에서 문제가 생겼다는 것을 알려주는 것이고, `arm64` 환경이기 때문에 문제가 생겼음을 뜻하진 않는다.
동일한 코드를 윈도우에서 실행하면 `Undefined symbols for architecture x86_64:`라고 뱉어낸다.)
```gcc
/opt/homebrew/Cellar/gcc@11/11.3.0/bin/gcc-11 -o my_MultiThreading /Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/my_MultiThreading.cpp
Undefined symbols for architecture arm64:
  "__ZNSolsEPFRSoS_E", referenced from:
      __Z12PrintNumbersRKSt6vectorIiSaIiEE in cclbTL01.o
      _main in cclbTL01.o
  "__ZNSolsEi", referenced from:
      __Z12PrintNumbersRKSt6vectorIiSaIiEE in cclbTL01.o
  "__ZNSolsEx", referenced from:
      _main in cclbTL01.o
  "__ZNSt6chrono3_V212system_clock3nowEv", referenced from:
      _main in cclbTL01.o
  "__ZNSt6thread15_M_start_threadESt10unique_ptrINS_6_StateESt14default_deleteIS1_EEPFvvE", referenced from:
      __ZNSt6threadC1IZ4mainEUlvE_JEvEEOT_DpOT0_ in cclbTL01.o
  "__ZNSt6thread4joinEv", referenced from:
      _main in cclbTL01.o
  "__ZNSt6thread6_StateD2Ev", referenced from:
      __ZNSt6thread11_State_implINS_8_InvokerISt5tupleIJZ4mainEUlvE_EEEEED1Ev in cclbTL01.o
  "__ZNSt8ios_base4InitC1Ev", referenced from:
      __Z41__static_initialization_and_destruction_0ii in cclbTL01.o
  "__ZNSt8ios_base4InitD1Ev", referenced from:
      __Z41__static_initialization_and_destruction_0ii in cclbTL01.o
  "__ZSt17__throw_bad_allocv", referenced from:
      __ZN9__gnu_cxx13new_allocatorIiE8allocateEmPKv in cclbTL01.o
      __ZN9__gnu_cxx13new_allocatorISt10shared_ptrISt6threadEE8allocateEmPKv in cclbTL01.o
  "__ZSt20__throw_length_errorPKc", referenced from:
      __ZNKSt6vectorIiSaIiEE12_M_check_lenEmPKc in cclbTL01.o
      __ZNKSt6vectorISt10shared_ptrISt6threadESaIS2_EE12_M_check_lenEmPKc in cclbTL01.o
  "__ZSt28__throw_bad_array_new_lengthv", referenced from:
      __ZN9__gnu_cxx13new_allocatorIiE8allocateEmPKv in cclbTL01.o
      __ZN9__gnu_cxx13new_allocatorISt10shared_ptrISt6threadEE8allocateEmPKv in cclbTL01.o
  "__ZSt4cout", referenced from:
      __Z12PrintNumbersRKSt6vectorIiSaIiEE in cclbTL01.o
      _main in cclbTL01.o
  "__ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_", referenced from:
      __Z12PrintNumbersRKSt6vectorIiSaIiEE in cclbTL01.o
      _main in cclbTL01.o
  "__ZSt9terminatev", referenced from:
      __ZNSt6threadD1Ev in cclbTL01.o
  "__ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc", referenced from:
      _main in cclbTL01.o
  "__ZTINSt6thread6_StateE", referenced from:
      __ZTINSt6thread11_State_implINS_8_InvokerISt5tupleIJZ4mainEUlvE_EEEEEE in cclbTL01.o
  "__ZTVN10__cxxabiv117__class_type_infoE", referenced from:
      __ZTISt11_Mutex_baseILN9__gnu_cxx12_Lock_policyE2EE in cclbTL01.o
  NOTE: a missing vtable usually means the first non-inline virtual member function has no definition.
  "__ZTVN10__cxxabiv120__si_class_type_infoE", referenced from:
      __ZTISt15_Sp_counted_ptrIPSt6threadLN9__gnu_cxx12_Lock_policyE2EE in cclbTL01.o
      __ZTINSt6thread11_State_implINS_8_InvokerISt5tupleIJZ4mainEUlvE_EEEEEE in cclbTL01.o
      __ZTISt16_Sp_counted_baseILN9__gnu_cxx12_Lock_policyE2EE in cclbTL01.o
  NOTE: a missing vtable usually means the first non-inline virtual member function has no definition.
  "__ZTVNSt6thread6_StateE", referenced from:
      __ZNSt6thread6_StateC2Ev in cclbTL01.o
  NOTE: a missing vtable usually means the first non-inline virtual member function has no definition.
  "__ZdlPvm", referenced from:
      _main in cclbTL01.o
      __ZNSt6threadC1IZ4mainEUlvE_JEvEEOT_DpOT0_ in cclbTL01.o
      __ZN9__gnu_cxx13new_allocatorIiE10deallocateEPim in cclbTL01.o
      __ZN9__gnu_cxx13new_allocatorISt10shared_ptrISt6threadEE10deallocateEPS3_m in cclbTL01.o
      __ZNSt14__shared_countILN9__gnu_cxx12_Lock_policyE2EEC1IPSt6threadEET_ in cclbTL01.o
      __ZNSt15_Sp_counted_ptrIPSt6threadLN9__gnu_cxx12_Lock_policyE2EED0Ev in cclbTL01.o
      __ZNSt6thread11_State_implINS_8_InvokerISt5tupleIJZ4mainEUlvE_EEEEED0Ev in cclbTL01.o
      ...
  "__Znwm", referenced from:
      _main in cclbTL01.o
      __ZNSt6threadC1IZ4mainEUlvE_JEvEEOT_DpOT0_ in cclbTL01.o
      __ZNSt14__shared_countILN9__gnu_cxx12_Lock_policyE2EEC1IPSt6threadEET_ in cclbTL01.o
      __ZN9__gnu_cxx13new_allocatorIiE8allocateEmPKv in cclbTL01.o
      __ZN9__gnu_cxx13new_allocatorISt10shared_ptrISt6threadEE8allocateEmPKv in cclbTL01.o
     (maybe you meant: __ZnwmPv)
  "___cxa_begin_catch", referenced from:
      __ZNSt14__shared_countILN9__gnu_cxx12_Lock_policyE2EEC1IPSt6threadEET_ in cclbTL01.o
  "___cxa_end_catch", referenced from:
      __ZNSt14__shared_countILN9__gnu_cxx12_Lock_policyE2EEC1IPSt6threadEET_ in cclbTL01.o
  "___cxa_pure_virtual", referenced from:
      __ZTVSt16_Sp_counted_baseILN9__gnu_cxx12_Lock_policyE2EE in cclbTL01.o
  "___cxa_rethrow", referenced from:
      __ZNSt14__shared_countILN9__gnu_cxx12_Lock_policyE2EEC1IPSt6threadEET_ in cclbTL01.o
  "___gxx_personality_v0", referenced from:
      Dwarf Exception Unwind Info (__eh_frame) in cclbTL01.o
ld: symbol(s) not found for architecture arm64
collect2: error: ld returned 1 exit status

```

왜냐하면 여러 스레드가 공동으로 사용하고 있는 `num` 변수에 대해 [[데이터 레이스 (Data Race)]]가 발생하고 있기 때문이다.

구체적으로 위 코드의 어느 부분에서 이런 문제가 생겼는가?



2023-05-23

----
#Cpp #CS #지선생 #ChatGPT #라이브러리 #메모리 #객체 #객체지향 #make_unique #unique_ptr 

## 개요
[[팩토리 메서드 패턴(Factory Method Pattern)]]를 공부하던 중, 예제에서 `unique_ptr`과 `make_unique`를 만나게 되었다.
ChatGPT의 도움으로 이 두 키워드를 이해하게 되었다.

## 내용
### `unique_ptr`
먼저 예제를 보고 설명해 보겠다.
```cpp
#include <iostream>
#include <memory>

class MyClass {
public:
    MyClass() {
        std::cout << "MyClass constructor" << std::endl;
    }
	
    ~MyClass() {
        std::cout << "MyClass destructor" << std::endl;
    }
	
    void someFunction() {
        std::cout << "MyClass::someFunction()" << std::endl;
    }
};

int main() {
    std::unique_ptr<MyClass> ptr(new MyClass());  // Creating a unique_ptr
	
    if (ptr) {
        ptr->someFunction();  // Accessing the object's member function
    }
	
    ptr.reset();  // Deleting the object and resetting the unique_ptr
	
    return 0;
}

```

위 예제에서 `std::unique_ptr<MyClass>`의 형을 가진 `ptr(new MyClass())`는 클래스를 가리키는 포인터이다.
물론 그냥 다음과 같이 포인터 변수가 클래스를 가리키는 방법도 있다.
```cpp
#include <iostream> // string class 등을 담고 있는 헤더파일
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일

using namespace std;class MyClass{
	void myFunction()
	{
		std::cout << "MyClass Functions" << std::endl;		
	}
}

int main(int argc, const char * argv[])
{
	MyClass obj;
	MyClass* p_myClass = &obj;
	ptr->muFunction();
	
	return 0;
}
```

이 방법과 다른 점은, 이 `unique_ptr<Product> productA`와 같이 선언한 것은 이 포인터로 하여금 객체를 주시하고 있다가 더이상 그것이 사용되지 않으면 자동으로 삭제되도록 하여 메모리 누수를 방지한다는 점이다.
그리고 이 포인터는 그것이 가리키는 객체의 복사를 막아놓는다.
이를 베타적 소유권 시맨틱(Exclusive Management Semantic)이라고 한다.
또 이는 딥카피(Deep Copy)를 금지하는 대신, 이것의 이동, 즉 무브 시맨틱은 가능하게 만들어 두었다.

### `make_unique`
```cpp
#include <iostream>
#include <memory>

class MyClass {
public:
    MyClass() {
        std::cout << "MyClass constructor" << std::endl;
    }
	
    ~MyClass() {
        std::cout << "MyClass destructor" << std::endl;
    }
	
    void someFunction() {
        std::cout << "MyClass::someFunction()" << std::endl;
    }
};

int main() {
    auto ptr = std::make_unique<MyClass>();  // Creating a unique_ptr using make_unique
	
    if (ptr) {
        ptr->someFunction();  // Accessing the object's member function
    }
	
    // No need to explicitly delete or reset the unique_ptr
	
    return 0;
}

```

앞의 예제에서는 `std::unique_ptr<MyClass> ptr(new MyClass());`와 같이 객체를 메모리에 동적할당한 다음 이것을 포인터 변수로 형변환했다.
하지만 이 경우 `std::unique_ptr<MyClass>`를 직접 생성한다.
이는 그냥 앞의 `unique_ptr` 코드를 간소화한 것이라 생각하면 편하다.
[[팩토리 메서드 패턴(Factory Method Pattern)]]의 예제에 나오는 `make_unique`는 리턴형으로 `std::unique_ptr<ConcreteProduct>`을 내놓아야 하기에 클래스의 객체화(동적메모리 할당의 과정)와 이것의 리턴값으로의 대입을 생략하는 `std::make_unique<ConcreteProductA>`을 써준 것이다.
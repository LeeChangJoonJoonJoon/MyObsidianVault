

2023-05-23

----
#Cpp #PANIM #지선생 #ChatGPT #Bard #factory_class #class #복사생성자 #팩토리메서드패턴 #디자인패턴 #make_unique #unique_ptr 

## 개요
[[03. OracleConnectionPool.cpp]]을 만들던 중, 오라클에서 제공하는 OCI 내에 있는 `Environment` 클래스가 팩토리 클래스(Factory Class)라는 것을 알게 되었다.
이게 뭘까?

## 내용
다시 ChatGPT에게 물어보았다.
지선생이 알려준 팩토리 클래스의 정의는 다음과 같다.
![[스크린샷 2023-05-23 오후 2.45.21.png]]

이에 따르면, 팩토리 클래스는 컴파일 시에 클래스를 구체적으로 지정하지 않고도 다양한 유형의 객체를 생성할 수 있는 '디자인 패턴'이다. 
이는 객체 생성 로직을 캡슐화하여 클라이언트 코드가 구현의 세부사항을 할 필요 없이 공통 인터페이스를 통해 객체를 요청할 수 있게끔 해준다.

다음과 같은 예제와 함께 이해해 보자.
```cpp
#include <iostream>
#include <memory>

// Base class
class Product {
public:
    virtual void operation() = 0;
    virtual ~Product() {}
};

// Concrete product classes
class ConcreteProductA : public Product {
public:
    void operation() override {
        std::cout << "ConcreteProductA operation" << std::endl;
    }
};

class ConcreteProductB : public Product {
public:
    void operation() override {
        std::cout << "ConcreteProductB operation" << std::endl;
    }
};

// Factory class
class ProductFactory {
public:
    std::unique_ptr<Product> createProduct(char type) {
        switch (type) {
            case 'A':
                return std::make_unique<ConcreteProductA>();
            case 'B':
                return std::make_unique<ConcreteProductB>();
            default:
                return nullptr;
        }
    }
};

// Client code
int main() {
    ProductFactory factory;
    
    std::unique_ptr<Product> productA = factory.createProduct('A');
    if (productA)
        productA->operation();
    else
        std::cout << "Invalid product type!" << std::endl;
    
    std::unique_ptr<Product> productB = factory.createProduct('B');
    if (productB)
        productB->operation();
    else
        std::cout << "Invalid product type!" << std::endl;
    
    return 0;
}
```

이를 실행하면 다음과 같은 결과가 나온다.
```unix
$ cd "/Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/PANIM_DB/ProjectFiles/references/" && g++ -std=c++14 factoryClassEx.cpp -o factoryClassEx && "/Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/PANIM_DB
/ProjectFiles/references/"factoryClassEx
ConcreteProductA operation
ConcreteProductB operation
```

실행을 시작할 시점엔 `ProductFactory`가 객체화된다.
`Product`에 대한 스마트 포인터 `std::unique_ptr<Product>`형인 `productA`를 객체화한다.
여기에 앞서 객체화된 `factory` 내의 `createProduct()`로 직접 접근하는 방식으로 호출하여 그 리턴 값을 대입한다.

이때 `'A'`를 입력했으므로 `createProduct` 함수 속 `case 'A'`에 해당하여 `Product`의 상속을 받은 `ConcreteProductA`에 대한 스마트 포인터가 반환된다.
이 과정 속에서 `factory`라는 객체의 반환이 `ConcreteProductA` 객체의 포인터인 것을 관찰할 수 있다.
즉, 객체의 반환이 객체인 것이다.

그럴 경우 `if(productA)` 조건에 걸리게 되어 `productA->operation()`를 실행한다.
즉, `productA` 객체 속 멤버를 그것의 주소를 통해 접근한다.
이때 `productA`를 만드는 중에 가상 클래스`Product`를 `ConcreteProductA`로 구체화했으므로 이것의 멤버함수인 
```cpp
void operation() override {
	std::cout << "ConcreteProductA operation" << std::endl;
}
```
가 호출된다.

정리하면, 추상 클래스로부터 여러 구체화된 클래스를 지정해 놓고 경우에 따라 그 중 어떤 클래스로 구체화될지를 팩토리 클래스에서 ***교통정리*** 해준다.
이 중 어느 케이스에 해당하는지 판단하고 그 케이스에 진입하는 과정에서 추상클래스가 구체화된다.
이 구체화 과정이 곧 클래스의 객체화이기에 메인함수에 그 구체화된 클래스의 객체 속 멤버 함수를 가져다 쓸 수 있게 된다.
팩토리 클래스가 가상 클래스를 구체화, 객체화를 시키는 것이다.
이를 팩토리 메서드 패턴이라 한다.
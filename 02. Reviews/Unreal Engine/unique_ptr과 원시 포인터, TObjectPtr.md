

2023-10-15

----
#Cpp #Unreal #메모리 #unique_ptr 

## 개요
"언리얼 5의 `TObjectPtr`는 원시 포인터를 대체한다."
필자는 언리얼 5.3으로 만든 LyraStarterGame 예제에서 캐릭터의 입력을 처리하는 부분을 찾아보던 중 다음과 같은 코드를 봤다.
```cpp
void ALyraCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	PawnExtComponent->SetupPlayerInputComponent();
}
```

이 `PawnExtComponent`가 뭔지 찾아 올라가니 `ALyraCharacter` 클래스에서 다음과 같이 선언이 돼 있었다.
```cpp
private:

	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Lyra|Character", Meta = (AllowPrivateAccess = "true"))
	TObjectPtr<ULyraPawnExtensionComponent> PawnExtComponent;
```

이게 뭔지 알아보자.

## 내용
먼저 유니크 포인터에 대해 알아야 한다.
C++에서 일반적으로 쓰이는 유니크 포인터의 모습은 다음과 같다.
```cpp
#include <memory> 
void UniquePointerDemo() 
{ 
	std::unique_ptr<int> pInteger(new int(3)); // (1) new 연산자 사용 
	std::unique_ptr<int> pInteger2 = std::make_unique<int>(3); // (2) 함수 사용 
}
```

위 코드는 3의 값을 가지는 어떤 변수에 메모리를 할당한 뒤 그걸 가리키는 포인터를 `pInteger`로 정해주는 코드이다.
근데, 이 값타입 변수는 (마치 호출스택엔 잡히는데 함수명은 없는 람다함수처럼) 이름이 없다.
이 값에 접근할 수 있는 방법은 유니크 포인터를 통한 방법 뿐이다.
물론 이름이 있도록 다음과 같이 처리해 줄 순 있다.
```cpp
std::unique_ptr<int> pInteger(new int(3));

// 포인터를 사용하여 메모리에 접근
int* rawPtr = pInteger.get();
int value = *rawPtr; // 가능하지만 주의 필요
```

하지만 권장되는 방법은 아니기에 머릿속에서 지우길 바란다.
굳이 스마트 포인터를 사용하는 의미도 사라진다.

이렇게 스마트 포인터를 통해 할당하는 메모리 주소는 위처럼 안전하게 포인터로만 접근하도록 하고, (물론 값타입 변수에 그 포인터를 할당하도록 강제할 순 있으나) 자동으로 메모리에서 해제하는 등의 안전함을 가지고 있다.
이에 반해 원시 포인터는 STL이 아닌 정말 모든 메모리 관리가 개발자에게 달려 있는 포인터를 말한다.
![[Pasted image 20231015173332.png]]

그 원시 포인터를  `UObject`를 가리키는 용으로 대체하기 위한 게 언리얼에서는 `TObjectPtr<>`라는 의미.
꺾쇠 안에는 이 포인터로 가리킬 변수의 타입이 들어가게 된다.

이것 자체로 성능향상이나 안정성의 이점을 얻을 수 있는 것은 아니지만, 에디터빌드에서 실행시 **다이나믹 해상도**와 **액세스 트래킹**을 추가하면서 생겨났다.

다이나믹 해상도는, ???
액세스 트래킹은 객체가 언제 사용되고 있는지 실제로 감지하는 기능을 말한다.
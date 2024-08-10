

2023-12-30

----
#객체 #객체지향 #기능 #라이브러리 #Unreal #Json #Cpp #메모리

## 개요
지금까지는 `BeginPlay`함수가 호출되는 타이밍에 HTTP 요청을 했다.
하지만 나는 레벨이 모두 렌더되기 전에 요청을 하고 응답을 받은 뒤 자료구조로 그것을 저장하고 있고 싶다.
그리고 레벨 상에 객체들, 이를테면 위젯 객체가 만들어지고 이미 받아져 있는 자료구조를 참조만 하면 되도록 설계하면 성능상의 효율을 기할 수 있다.

## 내용
다음은 언리얼 공식 문서에 등장하는 게임플레이 사이클이다.
![[Pasted image 20231230202149.png]]

이렇게 봐서는 게임의 구조를 어떻게 짜야 하는지 모르겠다.
다만, 엔진 소스 내의 SubSystem이라는 클래스의 헤더에 다음과 같은 설명을 발견할 수 있었다.
```cpp
/** Subsystems are auto instanced classes that share the lifetime of certain engine constructs  
 * *  Currently supported Subsystem lifetimes are:  
 *     Engine     -> inherit UEngineSubsystem 
 *     Editor     -> inherit UEditorSubsystem 
 *     GameInstance -> inherit UGameInstanceSubsystem 
 *     World      -> inherit UWorldSubsystem 
 *     LocalPlayer     -> inherit ULocalPlayerSubsystem 
 * 
 * 
 *  Normal Example: 
 *     class UMySystem : public UGameInstanceSubsystem 
 *  Which can be accessed by: 
 *     UGameInstance* GameInstance = ...; 
 *     UMySystem* MySystem = GameInstance->GetSubsystem<UMySystem>(); 
 * 
 *  or the following if you need protection from a null GameInstance 
 *     UGameInstance* GameInstance = ...; 
 *     UMyGameSubsystem* MySubsystem = UGameInstance::GetSubsystem<MyGameSubsystem>(GameInstance); 
 * 
 * 
 *  You can get also define interfaces that can have multiple implementations. 
 *  Interface Example : 
 *      MySystemInterface 
 *    With 2 concrete derivative classes: 
 *      MyA : public MySystemInterface 
 *      MyB : public MySystemInterface 
 * 
 *  Which can be accessed by: 
 *     UGameInstance* GameInstance = ...; 
 *     const TArray<UMyGameSubsystem*>& MySubsystems = GameInstance->GetSubsystemArray<MyGameSubsystem>(); 
 * */
```

요약 + 첨언하면 다음과 같다.
- `UGameInstance` 생성 이후, `UGameSubSystem` 또한 생성됨.
- `UGameInstance` 초기화시 `UGameSubSystem`에서 `Initialize()` 함수 호출.
- `UGameInstance` 종료시에는 `Deinitialize()`호출.
- 바로 그 순간에 `UGameSubSystem`에 대한 참조가 없으면 GC가 물어간다.

다음 링크에 개시된 이미지 속 코드를 보자.
https://forums.unrealengine.com/t/game-instance-subsystem-doesnt-work/491579

`UGameInstance`에 의해 객체가 관리되는 `UGameSubSystem`에서 HTTP 요청을 보내고 받는다.
![353503-sub.png](https://d3kjluh73b9h9o.cloudfront.net/original/4X/e/1/e/e1e72e8e70055f55d99330be2257e369c7b2d6ef.png)

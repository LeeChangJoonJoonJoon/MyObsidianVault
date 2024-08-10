

2023-10-10

----
#Cpp #Unreal #수업

## 개요
Tick은 깊게 들어가면 그것이 절대시간을 의미하지 않음을 알 수 있다. 
Tick이 발동하지 않는 오브젝트들도 있고, 오브젝트나 액터마다 Tick이 발동할 때 다른 간격이 적용되기도 한다.
이를테면 우리가 액터에는 `Tick()`을 쓰는데 애님 인스턴스에는 `NativeUpdateAnimation()`이 따로 있는 것을 봤듯이 말이다.

## 내용
왜 우리는 이걸 고민해 봐야 하는가?
언리얼 엔진은 그 기본 단에서 내부적으로 어느 한 오브젝트의 틱이 완료되지 않으면 다른 것들의 틱이 실행되지 않도록 하거나 동시에 실행되도록 강제하는 등의 Tick Group 혹은 Tick Dependency를 논리기반으로 삼고 있다.
이를 커스텀하는 키워드 중 하나가 바로 우리가 클래스 생성 마법사로 만든 액터 클래스들의 생성자에서 본 `PrimaryActorTick.bCanEverTick`이다.

언리얼 공식 문서에 따르면 다음과 같이 다른 옵션들도 존재한다.
![[Pasted image 20231010215643.png]]

엔진 소스에는 다음과 같이 `enum` 형으로 지정돼 있다.
```cpp
/** Determines which ticking group a tick function belongs to. */ 
UENUM(BlueprintType) 
enum ETickingGroup 
{ 
/** Any item that needs to be executed before physics simulation starts. */ 
TG_PrePhysics UMETA(DisplayName="Pre Physics"), 
/** Special tick group that starts physics simulation. */ 
TG_StartPhysics UMETA(Hidden, DisplayName="Start Physics"), 
/** Any item that can be run in parallel with our physics simulation work. */ 
TG_DuringPhysics UMETA(DisplayName="During Physics"), 
/** Special tick group that ends physics simulation. */ 
TG_EndPhysics UMETA(Hidden, DisplayName="End Physics"), 
/** Any item that needs rigid body and cloth simulation to be complete before being executed. */ TG_PostPhysics UMETA(DisplayName="Post Physics"), 
/** Any item that needs the update work to be done before being ticked. */ 
TG_PostUpdateWork UMETA(DisplayName="Post Update Work"), 
/** Catchall for anything demoted to the end. */ 
TG_LastDemotable UMETA(Hidden, DisplayName = "Last Demotable"), 
/** Special tick group that is not actually a tick group. After every tick group this is repeatedly re-run until there are no more newly spawned items to run. */ 
TG_NewlySpawned UMETA(Hidden, DisplayName="Newly Spawned"), TG_MAX, 
};
```

그러면 게임 로직 내에서 다음과 같이 지정해 줄 수 있다.
```cpp
PrimaryComponentTick.TickGroup = TG_PrePhysics;
```

이런 옵션들은 특히나 서버-클라이언트 연동에 필수적인 것들이다.
다른 클라이언트들과의 월드를 동기화하는 작업에 쓰이기 때문이다.

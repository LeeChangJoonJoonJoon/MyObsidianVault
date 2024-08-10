

2024-02-10

----
#Unreal #InputSystem 

## 개요
현재 회사에서 진행하는 프로젝트에서 카메라의 움직임을 다루는 기능을 개발해야 한다.
카메라의 움직임은 두가지 모드를 가진다.

첫번째로는 위에서 아래를 내려다보는 스타크래프트 같은 시야를 구현하는 것이다.
 * 마우스 완쪽 버튼은 피킹과 UI 클릭을 위해서만 존재한다. 드래그 같은 건 없다.
 * 그리고 수직 혹은 수평 방향의 이동은 edge panning/scrolling(스타크래프트 방식의 화면 이동)읕 통해 조작.
 * 이 때 edge 부분으로 마우스가 밀어내는 속도가 어떤지에 따라 가속도의 크기가 다르다.
 * 한 지점(이걸 어떻게 설정할지도 고민인데)을 중심으로 각도를 이동하는 건 마우스 오른쪽 버튼의 드래그를 통해 조작한다.
 * 각도를 움직일 때의 중심점은 아마 화면 정 중앙에서 쏘는 레이에 히트된 물체를 기준으로 할 듯.
 * 마우스 휠은 확대/축소의 역할을 한다.

두번째로는 1인칭 시야에서 하나의 층을 돌아다니는 모드이다.
 * 개별 기기를 보는 시야로 들어왔을 때는 층 내에서 WASD를 이용해서 수평 이동을 할 수 있다.
 * 마우스 오른쪽 버튼을 통해 각도를 조정하는 경우 위 아래로는 제한을 걸어둔다. 카메라가 다른 층이나 바닥 밑 허공으로 가지 못하도록.

이 두가지 모드를 자유자재로 왔다갔다 해야 한다.
각 모드별로 조작법도 다르고, 카메라의 위치도 다르다.
기존 UE4를 사용하는 이득우의 교과서에선 카메라의 위치가 다른 모드를 구현할 수 있도록 돼 있었으나, 우리의 경우 조작법까지 달라야 한다. 전통적인 키바인딩 방식으로는 한계가 있는 것이다.

## 내용
먼저 `Enhanced Input` 플러그인을 활성화한 뒤, C++ 모듈에 `"EnhancedInput"`을 추가한다.
그리고 `Project Settings`의 `Input`항목에 다음과 같이 설정돼 있는지 확인한다.
![[Pasted image 20240210173403.png]]

다음 두가지를 추가한다.
![[Pasted image 20240210175156.png]]
![[Pasted image 20240210175211.png]]

우리는 여기서 WASD 키로 2차원 이동을 구현할 것이기에 `IA_Move`의 `Value Type`을 `Axis2D (Vector2D)`로 설정.
![[Pasted image 20240210175633.png]]

`IMC_Default`에서 실행될 입력을 다음과 같이 `IA_Move`로 설정해 준다.
![[Pasted image 20240210175401.png]]

그리고 키를 추가할 수 있다.
![[Pasted image 20240210175430.png]]

`Axis2D (Vector2D)`를 설정했을 때 설정 내에서의 기본 값은 X 축 방향으로 +하는 것이다.
각 키별로 `Modifiers`를 추가하여 방향을 돌리고(`Swizzle Input Axis Values`) 크기가 커지는 방향을 뒤집는(`Negate`) 등의 커스텀을 해줄 수 있다.
![[Pasted image 20240211182308.png]]

앞으로 가는 방향에 해당되는 W 키는 `Modifiers` 설정에 아무것도 없다.
뒤로 가는 방향에 해당되는 S 키는 `Negate`, 오른쪽으로 가는 방향에 해당되는 D 키는 원래 정방향을 돌려(`Swizzle`)주었고, A 키는 돌려(`Swizzle`)주고 뒤집어(`Negate`) 주었다.

이제 아래와 같이 `UDataAsset` 클래스를 상속받은 녀석을 만들어 준다.
`UDataAsset` 클래스는 에디터 상에서 파일 형태로 정보를 저장하는 데에 쓰인다.
![[Pasted image 20240211183632.png]]
![[Pasted image 20240211183539.png]]

미리 만든 `UDataAsset` 클래스는 다음과 같다. 
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
#pragma once  
  
#include "MyLab_v3.h"  
#include "EnhancedInputComponent.h"  
#include "ML_EnhancedInputLocPlayerSubsys.h"  
#include "Engine/DataAsset.h"  
#include "ML_PlayerInputActions.generated.h"  
  
/**  
* 
*/  
UCLASS(BlueprintType)  
class MYLAB_V3_API UML_PlayerInputActions : public UDataAsset  
{  
    GENERATED_BODY()  
  
public:  
    UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category="Default")  
    UInputMappingContext* MappingContext_default;  
    UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category="Default")  
    int32 MapPriority_default;  
    UPROPERTY(EditDefaultsOnly, BlueprintReadWrite, Category="Default")  
    UInputAction* Move;  
  
    // 여기 저장되는 데이터들은 플레이 타임에 꺼내서 사용하는 용도이다.  
};  
  
namespace EPlayerInputActions  
{  
    template<class T, class FuncType>  
    void BindInput_TriggerOnly(UEnhancedInputComponent* _Input, const UInputAction* _Action, T* _Obj, FuncType _TriggerFunc)  
    {       if (nullptr == _TriggerFunc) return;  
       _Input->BindAction(_Action, ETriggerEvent::Triggered, _Obj, _TriggerFunc);   
    }  
}
```

cpp로 만든 변수를 에디터의 디테일 창에서 값을 넣어준다.
![[Pasted image 20240211183842.png]]

우리의 최종 목표는 `namespace EPlayerInputActions` 내에 정의된 함수를 호출하는 것이다.
다음 `APlayerController` 클래스에서 이 `UDataAsset` 클래스를 참조하도록 한다.
또, `APawn` 클래스 내에서 `SetupPlayerInputComponent()`가 호출되기 전에 호출되는 `APlayerController` 클래스를 상속받은 클래스의 `AML_PlayerController::SetupInputComponent()` 함수 내에서 이 데이터 에셋을 이용한 입력 로직의 초기화 작업을 수행해 준다.
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
#pragma once  
  
#include "MyLab_v3.h"  
#include "ML_EnhancedInputLocPlayerSubsys.h"  
#include "GameFramework/PlayerController.h"  
#include "ML_PlayerController.generated.h"  
  
/**  
 * */  
UCLASS()  
class MYLAB_V3_API AML_PlayerController : public APlayerController  
{  
    GENERATED_BODY()  
  
public:  
    AML_PlayerController();  
    UFUNCTION()  
    void AddInputMapping(const UInputMappingContext* InputMapping, const int32 MappingPriority = 0) const;  
    UFUNCTION()  
    void RemoveInputMapping(const UInputMappingContext* InputMapping) const;  
  
    // AddInputMapping 함수를 통해   
UFUNCTION()  
    void SetInputDefault(const bool Enabled = true) const;  
    UFUNCTION()  
    FORCEINLINE UDataAsset* GetInputActionAsset() const { return PlayerActionsAsset; };  
  
protected:  
    virtual void SetupInputComponent() override;  
  
    UPROPERTY(BlueprintReadWrite, EditDefaultsOnly, Category="Player Settings")  
    UDataAsset* PlayerActionsAsset;  
};
```

이 중 `SetupInputComponent()` 함수의 구현부를 보면 다음과 같다.
```cpp
void AML_PlayerController::SetupInputComponent()  
{  
    Super::SetupInputComponent();  
  
    const auto InputSubsystem = ULocalPlayer::GetSubsystem<UML_EnhancedInputLocPlayerSubsys>(GetLocalPlayer());  
    if (!InputSubsystem) return;  
  
    // EnhancedInput의 매핑을 모두 해제하는 함수를 Subsystem에서 호출.  
    InputSubsystem->ClearAllMappings();  
    SetInputDefault();  
}
```

`UEnhancedInputLocalPlayerSubsystem`를 상속 받은 `UML_EnhancedInputLocPlayerSubsys` 클래스에 접근헤서 `ClearAllMappings()` 함수를 호출.
이는 `InputMappingContext`를 모두 삭제하여 초기화하는 부분이다.
사용자가 원하는 `InputMappingContext`와 `InputAction`를 설정하는 작업의 시작은 `SetInputDefault()` 함수부터 시작된다.
다음 `AML_PlayerController` 클래스의 구현부는 모두 `AML_PlayerController` 클래스가 폰 클래스에 빙의하기 전에 일어나는 일이라는 것을 생각하며 코드를 보자.
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
  
#include "ML_PlayerController.h"  
#include "ML_EnhancedInputLocPlayerSubsys.h"  
#include "ML_PlayerInputActions.h"  
  
AML_PlayerController::AML_PlayerController()  
{  
    static ConstructorHelpers::FObjectFinder<UML_PlayerInputActions> DA_INPUT(  
       TEXT("/Game/_01_RTS/Datas/DA_DefaultPlayerInput.DA_DefaultPlayerInput"));  
    if (!DA_INPUT.Succeeded()) return;  
    PlayerActionsAsset = DA_INPUT.Object;  
}  
  
void AML_PlayerController::AddInputMapping(const UInputMappingContext* InputMapping, const int32 MappingPriority) const  
{  
    const auto InputSubsystem = ULocalPlayer::GetSubsystem<UML_EnhancedInputLocPlayerSubsys>(GetLocalPlayer());  
    if (!InputSubsystem) return;  
  
    ensure(InputMapping);  
    // 이미 InputMapping를 MappingContext로 가지고 있으면 리턴.  
    if (InputSubsystem->HasMappingContext(InputMapping)) return;  
  
    InputSubsystem->AddMappingContext(InputMapping, MappingPriority);  
}  
  
void AML_PlayerController::RemoveInputMapping(const UInputMappingContext* InputMapping) const  
{  
    const auto InputSubsystem = ULocalPlayer::GetSubsystem<UML_EnhancedInputLocPlayerSubsys>(GetLocalPlayer());  
    if (!InputSubsystem) return;  
    ensure(InputMapping);  
    InputSubsystem->RemoveMappingContext(InputMapping);  
}  
  
void AML_PlayerController::SetInputDefault(const bool Enabled) const  
{  
    // ensureMsgf(PlayerActionsAsset, TEXT("PlayerActionsAsset is NULL!!!"));  
  
    // 에디터에서 연결해준 DataAsset을 가져온다.  
    const auto PlayerActions = Cast<UML_PlayerInputActions>(PlayerActionsAsset);  
    if (!PlayerActions) return;  
    ensure(PlayerActions->MappingContext_default);  
  
    // bool 변수 값에 따라 어떤 함수를 호출할지 결정.  
    if (Enabled) AddInputMapping(PlayerActions->MappingContext_default, PlayerActions->MapPriority_default);  
    else RemoveInputMapping(PlayerActions->MappingContext_default);  
}  
  
void AML_PlayerController::SetupInputComponent()  
{  
    Super::SetupInputComponent();  
  
    const auto InputSubsystem = ULocalPlayer::GetSubsystem<UML_EnhancedInputLocPlayerSubsys>(GetLocalPlayer());  
    if (!InputSubsystem) return;  
  
    // EnhancedInput의 매핑을 모두 해제하는 함수를 Subsystem에서 호출.  
    InputSubsystem->ClearAllMappings();  
    SetInputDefault();  
}
```

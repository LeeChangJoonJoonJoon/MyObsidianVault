

2023-12-09

----
#delegate #객체지향 #람다함수 #문제해결 #Cpp #Unreal 

## 개요
지난 삼성물산 POC에서 가장 문제가 됐던 것은 클래스 커플링과 스레드 스케쥴링이었다.
클래스 커플링 같은 경우, 위젯이 아닌 클래스가 다른 위젯의 생성을 관장하는 일이 생기는 것이었고, 
원래 이는 API 요청에 대한 응답과 JSON 파싱이 끝난 뒤 위젯의 생성을 진행하기 위한 것(즉 스레드 스케쥴링을 위한 것)이었으나 클래스 하나에 모든 기능을 넣는 것만 못하게 됐다.
이를 해결하기 위해 블루프린트의 `Event Dispatcher`를 사용했으나, 변수를 전달할 수 없다는 치명적인 문제가 있었다.
블루프린트만을 이용해 개발을 해 온 동료는 급기야 `Delay` 노드를 이용하여 감으로 스레드가 종료되는 시점을 때려 맞췄다... (구글링해 보면 실제로 많은 블루프린트 개발자들이 이렇게 한다는 게 충격이었다.)
요청하는 API가 수십 건을 넘는 시스템에서 응답 받는 스레드 종료시점을 '시간을 재어서' 때려맞추는 게 얼마나 위험한 짓인가.

## 내용
이득우 선생과 달리, 여기선 델리게이트 매크로 선언을 `GameModeBase` 클래스에 한다.
현재 나에게 필요한 것은 HTTP 요청-응답을 실행하는 액터와 위젯의 통신이므로 메인 위젯의 생성을 담당하는 `GameModeBase`에 이를 선언하는 게 더더욱 적절하게 보인다.
.cpp 파일에선 아무런 처리 없이 헤더에 Delegate 매크로만 적어 줬다.
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
#pragma once  
  
#include "MyLab_v2.h"  
#include "GameFramework/GameModeBase.h"  
#include "Chapter_05GameModeBase.generated.h"  
  
/**  
 * */  
DECLARE_DELEGATE(FStandardDelegateSignature)  
DECLARE_DELEGATE_OneParam(FParamDelegateSignature, FLinearColor)  
  
UCLASS()  
class MYLAB_V2_API AChapter_05GameModeBase : public AGameModeBase  
{  
    GENERATED_BODY()  
  
public:  
    FStandardDelegateSignature MyStandardDelegate;  
    FParamDelegateSignature MyParameterDelegate;  
};
```

`DECLARE_DELEGATE_OneParama`는 델리게이트를 통해 하나의 변수를 전달한다.
(이러면 인터페이스 쓸 필요가 없잖아?)
`DECLARE_DELEGATE`로 바인딩된 이벤트를 일으키고 델리게이트 호출 시점을 결정하는 클래스는 다음과 같다.
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
#pragma once  
  
#include "MyLab_v2.h"  
#include "GameFramework/Actor.h"  
#include "MyTriggerVolume.generated.h"  
  
UCLASS()  
class MYLAB_V2_API AMyTriggerVolume : public AActor  
{  
    GENERATED_BODY()  
      
public:   
    // Sets default values for this actor's properties  
    AMyTriggerVolume();  
  
protected:  
    // Called when the game starts or when spawned  
    virtual void BeginPlay() override;  
  
public:   
    // Called every frame  
    virtual void Tick(float DeltaTime) override;  
  
    UPROPERTY()  
    UBoxComponent* TriggerZone;  
    UFUNCTION()  
    virtual void NotifyActorBeginOverlap(AActor* OtherActor) override;  
    UFUNCTION()  
    virtual void NotifyActorEndOverlap(AActor* OtherActor) override;  
};
```

```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
  
#include "MyTriggerVolume.h"  
#include "Chapter_05GameModeBase.h"  
  
// Sets default values  
AMyTriggerVolume::AMyTriggerVolume()  
{  
    // Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.  
    PrimaryActorTick.bCanEverTick = true;  
      
    TriggerZone = CreateDefaultSubobject<UBoxComponent>(TEXT("TriggerZone"));  
    TriggerZone->SetBoxExtent(FVector(200, 200, 100));  
}  
  
// Called when the game starts or when spawned  
void AMyTriggerVolume::BeginPlay()  
{  
    Super::BeginPlay();  
      
}  
  
// Called every frame  
void AMyTriggerVolume::Tick(float DeltaTime)  
{  
    Super::Tick(DeltaTime);  
  
}  
  
// Super:: 안써도 된다!  
void AMyTriggerVolume::NotifyActorBeginOverlap(AActor* OtherActor)  
{  
    const auto Message = FString::Printf(TEXT("%s entered me"), *(OtherActor->GetName()));  
    GEngine->AddOnScreenDebugMessage(-1, 1, FColor::Red, Message);  
  
    if (const UWorld* TheWorld = GetWorld())  
    {  
       AGameModeBase* GameMode = UGameplayStatics::GetGameMode(TheWorld);  
  
       if (const AChapter_05GameModeBase* MyGameMode = Cast<AChapter_05GameModeBase>(GameMode))  
       {  
          MyGameMode->MyStandardDelegate.ExecuteIfBound();  
  
          auto Color = FLinearColor(1, 0, 0, 1);  
          MyGameMode->MyParameterDelegate.ExecuteIfBound(Color);  
       }  
    }  
}  
  
void AMyTriggerVolume::NotifyActorEndOverlap(AActor* OtherActor)  
{  
    const auto Message = FString::Printf(TEXT("%s left me"), *(OtherActor->GetName()));  
    GEngine->AddOnScreenDebugMessage(-1, 1, FColor::Red, Message);  
}
```

그리고 이 델리게이트의 호출을 '듣는' 클래스는 다음과 같다.
```cpp
// Fill out your copyright notice in the Description page of Project Settings.  
  
#pragma once  
  
#include "MyLab_v2.h"  
#include "GameFramework/Actor.h"  
#include "DelegateListener.generated.h"  
  
UCLASS()  
class MYLAB_V2_API ADelegateListener : public AActor  
{  
    GENERATED_BODY()  
      
public:   
    // Sets default values for this actor's properties  
    ADelegateListener();  
  
protected:  
    // Called when the game starts or when spawned  
    virtual void BeginPlay() override;  
  
public:   
    // Called every frame  
    virtual void Tick(float DeltaTime) override;  
    virtual void EndPlay(const EEndPlayReason::Type EndPlayReason) override;  
  
    UFUNCTION()  
    void EnableLight();  
    UPROPERTY()  
    UPointLightComponent* PointLight;  
};
```

```cpp
// Fill out your copyright notice in the Description page of Project Settings.


#include "DelegateListener.h"
#include "Chapter_05GameModeBase.h"

// Sets default values
ADelegateListener::ADelegateListener()
{
 	// Set this actor to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	PointLight = CreateDefaultSubobject<UPointLightComponent>(TEXT("PointLight"));
	RootComponent = PointLight;
	PointLight->SetVisibility(false);
	PointLight->SetLightColor(FLinearColor::Blue);
}

// Called when the game starts or when spawned
void ADelegateListener::BeginPlay()
{
	Super::BeginPlay();

	if (const UWorld* TheWorld = GetWorld())
	{
		AGameModeBase* GameMode = UGameplayStatics::GetGameMode(TheWorld);

		if (AChapter_05GameModeBase* MyGameMode = Cast<AChapter_05GameModeBase>(GameMode))
		{
			// UFUNCTION()에 바인딩할 때는 BindUObject, C++ 함수에는 BindRaw, static 함수에는 BindStatic.
			// 바인딩된 함수를 가진 인스턴스가 메모리에서 해제될 때 수동으로 언바인딩을 해주지 않으려면 BindUObject 쓰는 게 마음 편하다. 
			MyGameMode->MyStandardDelegate.BindUObject(this, &ADelegateListener::EnableLight);
		}
	}
}

// Called every frame
void ADelegateListener::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}

void ADelegateListener::EndPlay(const EEndPlayReason::Type EndPlayReason)
{
	Super::EndPlay(EndPlayReason);
	if (const UWorld* TheWorld = GetWorld())
	{
		AGameModeBase* GameMode = UGameplayStatics::GetGameMode(TheWorld);

		if (const AChapter_05GameModeBase* MyGameMode = Cast<AChapter_05GameModeBase>(GameMode))
		{
			// MyGameMode->MyStandardDelegate.
		}
	}
}

void ADelegateListener::EnableLight()
{
	PointLight->SetVisibility(true);
}
```


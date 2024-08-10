

2023-07-25

----


## 개요
Unreal에서 여러개의 동일한 컴포넌트로 구성된 액터에서 원하는 컴포넌트를 검색하는 방법을 찾아보던 중, 이터레이터를 통해 데이터 집합을 만들 수 있다는 내용을 찾았다.
현재 내가 다루고 있는 액터의 컴포넌트들의 이름은 `182`와 같이 각 자리가 0~9의 수로 이루어져 있다.
내가 원하는 것은 앞의 두자리로 컴포넌트를 검색하고 이것이 `SetVisibility(true)`가 처리된 상태인지 확인하는 것이다.

## 내용
인터넷에서 가져온 예제는 다음과 같다.
```cpp
for (auto It = FruitMap.CreateConstIterator(); It; ++It)
{
    FPlatformMisc::LocalPrint
    (
        *FString::Printf(TEXT("(%d, \"%s\")\n"), It.Key(), *It.Value())
    );
}
```

여기선 아주 간단하게 이터레이터의 `Key`와 `Value`를 출력(`LocalPrint`)하는 것을 수행한다.

나의 경우, 다음과 같은 코드에 적용해야 한다.
먼저 헤더파일.
```cpp
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "AnonymousBat_2.h"
#include "GameFramework/Character.h"
#include "AB_SoundCube_2.h"
#include "AB_Character.generated.h"

/// <summary>
/// Todo: 라인트레이스가 일정정도 먼 곳부터 인식을 하게 만들어야 플레이어가 액터 안에 들어오도록 스폰되는 일을 막을 수 있다.
///	Todo: 그리고 이걸 구현하는 과정에서 라인트레이스가 눈에 보이면 좋을 것 같다!
/// </summary>

UCLASS()
class ANONYMOUSBAT_2_API AAB_Character : public ACharacter
{
	GENERATED_BODY()

public:
	// Sets default values for this character's properties
	AAB_Character();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	// Called to bind functionality to input
	virtual void SetupPlayerInputComponent(class UInputComponent* _pPlayerInputComponent) override;

private:
	// Set up parameters for getting the player viewport
	FVector PlayerViewPtLoc;
	FRotator PlayerViewPtRot;
	float Reach;
	float Reach_Max;

	FVector SweepStartPt;
	FVector SweepEndPt;
	TArray<FHitResult> HitResults;

	void PushSoundCube();
	void PrePushSoundCube();
	TArray<FHitResult> SweepInRange();
	bool IsGrounded(const UPrimitiveComponent* _pCubeComponent);

	bool bIsEKeyDown;

public:
	UPROPERTY()
		AAB_SoundCube_2* pAB_SoundCube;
	UPROPERTY()
		UMaterialInstanceDynamic* SoundCubeMatInstDynamic;
	UPrimitiveComponent* pCubeComponent_Hit;
	TArray<FString> CubeNames_Hit;

};

```

그리고 다음은 이터레이터를 사용하려는 소스이다.
```cpp

// Fill out your copyright notice in the Description page of Project Settings.

///
/// Todo: 스윕을 썼을 때 가장 밑에 있는 녀석부터 visible이 되도록 -> 원래는 정육면체였던 안보이는 메시 컴포넌트가 사용자가 뭘 들고 있느냐에 따라 형태를 달리 하여 visible이 되도록 -> 
/// Todo: 
/// 

#include "AB_Character.h"

// Sets default values
AAB_Character::AAB_Character()
{
	// Set this character to call Tick() every frame.  You can turn this off to improve performance if you don't need it.
	PrimaryActorTick.bCanEverTick = true;

	Reach = 500.0f;
	bIsEKeyDown = false;

	//static ConstructorHelpers::FObjectFinder<UStaticMesh> CubeMeshFinder_Default(TEXT("/Script/Engine.StaticMesh'/Game/_03_BuildingSoundBlock/Meshes/AB_Cube_2.AB_Cube_2'"));
	//static ConstructorHelpers::FObjectFinder<UStaticMesh> CubeMeshFinder_1(TEXT("/Script/Engine.StaticMesh'/Game/_03_BuildingSoundBlock/Meshes/Actor01_3.Actor01_3'"));
	//static ConstructorHelpers::FObjectFinder<UStaticMesh> CubeMeshFinder_2(TEXT("/Script/Engine.StaticMesh'/Game/_03_BuildingSoundBlock/Meshes/Actor02_3.Actor02_3'"));
	//static ConstructorHelpers::FObjectFinder<UStaticMesh> CubeMeshFinder_3(TEXT("/Script/Engine.StaticMesh'/Game/_03_BuildingSoundBlock/Meshes/Actor03_3.Actor03_3'"));
}

// Called when the game starts or when spawned
void AAB_Character::BeginPlay()
{
	Super::BeginPlay();

}

// Called every frame
void AAB_Character::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

}

// Called to bind functionality to input
void AAB_Character::SetupPlayerInputComponent(UInputComponent* _pPlayerInputComponent)
{
	Super::SetupPlayerInputComponent(_pPlayerInputComponent);

	_pPlayerInputComponent->BindAction("PreHoldCube", EInputEvent::IE_Pressed, this, &AAB_Character::PrePushSoundCube);
	_pPlayerInputComponent->BindAction("HoldCube", EInputEvent::IE_Released, this, &AAB_Character::PushSoundCube);
}

/// <summary>
/// E 키를 누르면 스윕이 켜지고, E키를 떼면 스윕이 꺼짐과 동시에 마지막으로 닿은 녀석들이 나타난다.
/// 스윕이 켜진 상태에서 닿은 녀석들은 머티리얼이 투명한 것으로 변경된다.
/// 그리고 더이상 닿지 않게 되면 원래 안보이는 상태로 변경된다.
/// </summary>
void AAB_Character::PrePushSoundCube()
{
	bIsEKeyDown = true;

	HitResults = SweepInRange();
	if (!HitResults.IsEmpty())
	{
		pAB_SoundCube = Cast<AAB_SoundCube_2>(HitResults.GetData()->GetActor());
		AB2CHECK(nullptr != pAB_SoundCube);

		if (pAB_SoundCube)
		{
			for (auto& hitResult : HitResults)
			{
				hitResult.GetComponent()->SetVisibility(true);
				AB2LOG(Warning, TEXT("Push Sweeping has hit: %s"), *(hitResult.GetComponent()->GetName()));
			}
		}
	}
}

void AAB_Character::PushSoundCube()
{
	bIsEKeyDown = false;

	HitResults = SweepInRange();
	if (!HitResults.IsEmpty())
	{
		pAB_SoundCube = Cast<AAB_SoundCube_2>(HitResults.GetData()->GetActor());
		AB2CHECK(nullptr != pAB_SoundCube);

		if (pAB_SoundCube)
		{
			for (auto& hitResult : HitResults)
			{
				pCubeComponent_Hit = hitResult.GetComponent();
				if (IsGrounded(pCubeComponent_Hit))
				{
					hitResult.GetComponent()->SetVisibility(true);
					hitResult.GetComponent()->SetCollisionObjectType(ECollisionChannel::ECC_WorldStatic);

				}

				AB2LOG(Warning, TEXT("Prepush Sweeping has hit: %s"), *(hitResult.GetComponent()->GetName()));
			}
		}
	}
}

TArray<FHitResult> AAB_Character::SweepInRange()
{
	Controller->GetPlayerViewPoint(OUT PlayerViewPtLoc, OUT PlayerViewPtRot);

	SweepStartPt = PlayerViewPtLoc + 100.0f * PlayerViewPtRot.Vector();
	SweepEndPt = PlayerViewPtLoc + PlayerViewPtRot.Vector() * 500.0f;

	//DrawDebugLine(GetWorld(), SweepStartPt, SweepEndPt, FColor::Red, true, 0.0f);

	const FCollisionQueryParams TraceParams(FName(TEXT("")), true, GetOwner());
	GetWorld()->SweepMultiByChannel(HitResults, SweepStartPt, SweepEndPt, FQuat::Identity, ECC_Visibility, FCollisionShape::MakeSphere(0.5f), TraceParams);

	return HitResults;
}

bool AAB_Character::IsGrounded(const UPrimitiveComponent* _pCubeComponent)
{
	_pCubeComponent->GetName().ParseIntoArray(CubeNames_Hit, TEXT(" ")); // 스윕에 맞은 컴포넌트를 가져오는 것임!

	if (CubeNames_Hit[2] != "0")
	{
		UStaticMeshComponent* pCube_Actor;
		for (auto It = pAB_SoundCube->GetComponents().CreateConstIterator(); It; ++It)
		{
			/*code here*/
		}
	}
	return true;
}
```

여기서 알고리즘 두뇌를 써야 한다.
우리가 원하는 기능은, 최대 10층 높이까지 쌓을 수 있는 큐브 구조물에서 어느 한 큐브가 자신보다 낮은 층에 하나라도 없으면 생성될 수 없도록 하는 것이다.
그러면 필요한 재료는 뭘까?

캐릭터 스윕에 닿은 큐브의 x, y 좌표가 필요하다.
이 좌표가 일치하는 다른 층의 큐브 중 스윕에 닿은 큐브보다 z 좌표가 작은 녀석들 중 하나라도 Visible 상태가 아니면 스윕에 닿은 큐브를 `SetVisibility(false)`로 설정해 놓는다.

그래서 `bool AAB_Character::IsGrounded(const UPrimitiveComponent* _pCubeComponent)`를 구현한 내용은 다음과 같다.
```cpp
bool AAB_Character::IsGrounded(const UPrimitiveComponent* _pCubeComponent)
{
	_pCubeComponent->GetName().ParseIntoArray(CubeNames_Hit, TEXT(" "));

	if (CubeNames_Hit[2] != "0")
	{
		for (auto It = pAB_SoundCube->GetComponents().CreateConstIterator(); It; ++It) // 스태틱메시컴포넌트의 이름을 가져와야 함!
		{
			if (const UStaticMeshComponent* pCube_Actor = Cast<UStaticMeshComponent>(*It))
			{
				pCube_Actor->GetName().ParseIntoArray(CubeNames_Actor, TEXT(" "));
				if (CubeNames_Hit[0] == CubeNames_Actor[0] && CubeNames_Hit[1] == CubeNames_Actor[1])
				{
					if (FCString::Atoi(*CubeNames_Hit[2]) > FCString::Atoi(*CubeNames_Actor[2]) && Cast<UStaticMeshComponent>(*It)->IsVisible() == false)
					{
						AB2LOG(Warning, TEXT("'Cause %s is empty, %s IS NOT GROUNDED!!!"), *pCube_Actor->GetName(), *_pCubeComponent->GetName());
						return false;
					}
				}
			}
		}
	}
	else
		return true;
	return true;
}
```

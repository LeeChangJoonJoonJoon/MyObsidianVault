

2023-09-12

----
#Cpp #Unreal #메모리 #디자인패턴 #팩토리메서드패턴 #객체지향 #객체 #compiler

## 개요
> 언리얼에서는 런타임에 빠른 타입 체킹과 클래스 검색을 위해, 컴파일 타임에서 클래스와 타입 등의 메타 정보를 생성한다. 이러한 메타 정보는 `UClass` 라는 언리얼 클래스에 보관된다. `UClass` 에는 클래스의 계층 구조나 멤버 변수/함수에 대한 정보가 들어있다. 이러한 `UClass` 를 이용하면, 단순히 타입을 검색하는 것을 넘어 런타임에서 인스턴스의 멤버 변수 값을 변경하거나 멤버 함수를 호출할 수도 있다. (Java, C# 에서는 이런 기능을 리플렉션 (Reflection) 이라고 부른다)
> 출처: https://koreanfoodie.me/913

## 내용
 <<인생 언리얼 교과서>> 제2권의 37페이지에선, 우리가 C++에서 생성자를 쓸 때와는 다르게 작동한다고 설명했다. 
 이 생성자에서 초기화한 멤버들은 우리가 런타임 시에 보는 객체에 곧바로 적용되지 않는다.
 우리가 클래스를 만들고 컴파일을 한 뒤 에디터를 실행하면 거기에 우리가 틀지은 클래스의 모든 특성을 가진 CDO가 생성된다. 
 이는 일종의 클래스의 프로토타입으로서, 실제 플레이타임에 나타나는 인스턴스들은 이 CDO에서 복사되어져 생성되는 것이다.

이것은 전반적인 설명이고, 조금더 미시적으로 관찰을 해보면...
우리가 `Ctrl + Shift + B`를 누르면 이 때 메모리에 클래스가 로드된다.
여기서 우리가 언리얼 로그를 열어 확인할 수 있듯 DLL 파일이 업데이트되고, 이 DLL 파일 내에서 클래스를 읽어들이는 작업이 실행된다.

이 '로드'의 의미는 객체화가 아니다. 
객체화의 이전 단계로서 객체화를 위해 클래스의 각종 정보를 메모리에 적재하는 것이다.

이 적재된 클래스가 객체화되어서 곧바로 언리얼의 게임 오브젝트가 되는 게 아니다.
언리얼 엔진은 이 클래스로부터 CDO를 만들 따름이다.

이렇게 만들어진 CDO는 런타임(*에디터를 실행하는 시점*) 시에 일종의 템플릿으로 쓰인다.
이 템플릿으로부터 다시 객체들이 복사되어져 나온다.
CDO는 객체를 생성하는 객체인 셈.
CDO로부터 생성된 객체가 바로 우리가 플레이 타임(*에디터를 켜고 `Alt + P`를 누른 때*)에 보는 오브젝트들이다.
이 오브젝트들은 각자의 상황에 맞는 값을 가지되, 큰 틀에서는 CDO의 형식을 가진다.

언리얼에선 어떤 자식 클래스가 부모 클래스로부터 상속을 받을 때, 자식 클래스의 CDO는 부모 클래스의 CDO를 복사한다.
그런 다음 하위 클래스의 자체적인 기본 값으로 덮어쓰는 것이다.

우리가 다음 클래스 마법사를 통해 만든 `UCLASS()` 매크로가 붙은 클래스의 .cpp 파일 내의 생성자는 곧바로 플레이타임의 오브젝트에 적용되는 것이 아니다.
![[Pasted image 20230920153420.png]]

```cpp
ACP_Character_Anim::ACP_Character_Anim()  
{  
    // Set this pawn to call Tick() every frame.  You can turn this off to improve performance if you don't need it.  
    PrimaryActorTick.bCanEverTick = true;  

	const float AxisSpeed = 250.f;

    GetCapsuleComponent()->SetRelativeRotation(FRotator(-90.f, 0.f, 0.f));  
    GetCapsuleComponent()->SetCapsuleHalfHeight(400.f, true);  
    GetCapsuleComponent()->SetCapsuleRadius(50.f, true);  
    GetArrowComponent()->SetRelativeRotation_Direct(FRotator(90.f, 0.f, 0.f));  
  
    pSpringArm = CreateDefaultSubobject<USpringArmComponent>(TEXT("SPRINGARM"));  
    pCamera = CreateDefaultSubobject<UCameraComponent>(TEXT("CAMERA"));  
  
    pSpringArm->SetupAttachment(GetCapsuleComponent());  
    pSpringArm->TargetArmLength = 1000.f;  
    pSpringArm->SetRelativeRotation(FRotator(75.f, 0.f, 0.f));  
  
    pCamera->SetupAttachment(pSpringArm);  
  
    static ConstructorHelpers::FObjectFinder<USkeletalMesh> Plane(TEXT("/Script/Engine.SkeletalMesh'/Game/_01_BasicSettings/SkeletalMeshes/SK_PLANE/SK_West_UAV_MQ1.SK_West_UAV_MQ1'"));  
    if (Plane.Succeeded())  
    {  
       CPCHECK(Plane.Succeeded());  
       GetMesh()->SetSkeletalMesh(Plane.Object);  
       GetMesh()->SetRelativeLocationAndRotation(FVector(155.f, 0.f, 0.f), FRotator(90.f, 0.f, 0.f));  
    }  
}
```

이는 CDO에 적용되는 생성자이다.
우리가 이 생성자 내에 여러 변수들을 초기화했으나, 실제로는 CDO를 만들기 위해 이 변수의 값을 전부 `0`, `NULL` 등의 기본 값으로 채운다.
그리고 비로소 CDO로부터 오브젝트들을 복사를 통해 만들어 낼 때에 이 변수들에 구체적인 값들을 집어넣는다.

소프트웨어 아키텍처의 측면에서 이를 논해 보면, CDO는 '팩토리'와 같은 것이다 (팩토리 메서드 패턴을 모르겠으면 이 두 단락은 넘어가도 좋음).
[[팩토리 메서드 패턴(Factory Method Pattern)]]에서 논한 바에 따르면, 팩토리 객체는 다른 클래스의 객체화를 통제한다.
그러므로 CDO는 클래스 템플릿으로서 실제로 런타임에 어떤 형태의 객체가 나올지를 결정하는 근간이 된다.
이 CDO에 초기화된 변수에 여러 값을 다르게 대입하여 실제 런타임에 언리얼 오브젝트를 만들어내는 것이다.

하지만 엄밀하게 말하면 CDO가 팩토리는 아니다.
팩토리 메서드 패턴은 클래스의 상속이 전제되기 때문이다.
여기 CDO는 상속의 방법으로가 아니라 복사의 방법으로, 동일한 템플릿에서 객체들을 만들어낼 따름이다.

우리가 언리얼 클래스를 컴파일하면 `UClass` 인스턴스가 생성되며, 이 `UClass`는 자신에게 해당되는 CDO를 생성한다.
여기서 CDO의 모든 멤버변수는 초기값(0, NULL 등등)으로 설정된다.
플레이 타임에 진입하게 되면, `SpawnActor`, `CreateNewObject`등의 명령어로 인해 게임 오브젝트의 객체화가 요구될 때 엔진은 `UClass`를 찾아가서 그것이 가지고 있는 CDO를 복사하여 게임 오브젝트를 생성한다.
이 `UClass`가, 원래 CDO에서 0이나 NULL 값으로 초기값이 설정된 멤버들에 구체적인 값을 넣어준다.
우리가 만든 클래스의 경우 `AxisSpeed의` 값을 250.f로 설정해주는 것임.
이상은 `UClass`가 CDO에 대해 어떤 관계를 맺고 어떤 역할을 하는지에 대한 설명이었다.

<참고자료> 
https://velog.io/@starkshn/%EC%96%B8%EB%A6%AC%EC%96%BC-CDO%EB%9E%80
https://koreanfoodie.me/902
https://docs.unrealengine.com/4.27/ko/ProgrammingAndScripting/GameplayArchitecture/Classes/
https://docs.unrealengine.com/4.27/ko/ProductionPipelines/DevelopmentSetup/ManagingGameCode/CppClassWizard/
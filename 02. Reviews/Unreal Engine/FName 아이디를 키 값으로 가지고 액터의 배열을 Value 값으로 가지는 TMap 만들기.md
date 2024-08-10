

2023-10-28

----
#Cpp #Unreal #자료구조 #unique_ptr 

## 개요
난 원래 다음과 같은 자료형을 만들고자 하였다.
```cpp
USTRUCT(BlueprintType)  
struct FAbsorptionChillerHeater  
{  
    GENERATED_BODY()  
  
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=Hierarchy)  
    TMap<FName, TArray<AActor*>> coolingWaterSupplyLineGroup;  
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=Hierarchy)  
    TMap<FName, TArray<AActor*>> coolingWaterReturnLineGroup;  
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=Hierarchy)  
    TMap<FName, TArray<AActor*>> chilledWaterSupplyLineGroup;  
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=Hierarchy)  
    TMap<FName, TArray<AActor*>> chilledWaterReturnLineGroup;  
};
```

일단 컴파일이 되지 않았다.
`Error: The type 'TArray<AActor*>' can not be used as a value in a TMap`

거꾸로, 액터 포인터를 키값으로 주고 그것이 실데이터상 가지는 아이디 값인 `FName` 타입의 스트링을 Value로 두도록 만들어 봤다.
왜냐하면, 현실에선 하나인 `StaticMeshActor`가 레벨 상에선 두개인 경우가 존재했기 때문이다.
```cpp
USTRUCT(BlueprintType)  
struct FAbsorptionChillerHeater  
{  
    GENERATED_BODY()  
  
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=Hierarchy)  
    TMap<TSoftObjectPtr<AActor>, FName> coolingWaterSupplyLineGroup;  
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=Hierarchy)  
    TMap<TSoftObjectPtr<AActor>, FName> coolingWaterReturnLineGroup;  
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=Hierarchy)  
    TMap<TSoftObjectPtr<AActor>, FName> chilledWaterSupplyLineGroup;  
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=Hierarchy)  
    TMap<TSoftObjectPtr<AActor>, FName> chilledWaterReturnLineGroup;  
};
```

이렇게 하면 다음과 같이 블루프린트에서 액터의 포인터를 키값으로 넣어줄 수 있다.
![[Pasted image 20231028201341.png]]

다만 여전히 이상적인 것은 아이디 값이 키 값이고 Value가 액터의 배열인 상황이다.
이걸 만들어 보는 걸 목표로 한다.

## 내용
ChatGPT에 물어보니 다음과 같이 코딩하는 방법을 추천했다.
```cpp
USTRUCT(BlueprintType)  
struct FActorSet  
{  
    GENERATED_BODY()  
  
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=Hierarchy)  
    TSet<TSoftObjectPtr<AActor>> Set;  
};  
  
USTRUCT(BlueprintType)  
struct FAbsorptionChillerHeater  
{  
    GENERATED_BODY()  
  
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=Hierarchy)  
    TMap<FName, FActorSet> coolingWaterSupplyLineGroup;  
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=Hierarchy)  
    TMap<FName, FActorSet> coolingWaterReturnLineGroup;  
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=Hierarchy)  
    TMap<FName, FActorSet> chilledWaterSupplyLineGroup;  
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=Hierarchy)  
    TMap<FName, FActorSet> chilledWaterReturnLineGroup;  
};
```

`TSet`, `TArray`는 동적할당을 사용하는데, `UPROPERTY()`를 사용하는 컬렉션 요소 타입 변수는 동적할당이 안된다.
그래서 컴파일이 안됐던 것.
그럼 위와 같이 코딩했을 때 블루프린트상에선 다음과 같이 값을 넣어줄 수 있다.
![[Pasted image 20231028225723.png]]


2023-11-10

----
#Json #Cpp #Unreal 

## 개요
기존 코드에서 `TSharedPtr<FJsonObject> FJsonObject`를 `Deserialize`에 넣는 방식으로는 `FJsonObject`가 `NULL`로 `out`되었는데, 이제 4개의 `JsonValue`로 역직렬화가 성공했다.
이제 구체적으로 모든 `ngsi-ld`의 모든 JSON 형식에 맞는 파싱 클래스를 만들어야 한다.

## 내용
`ngsi-ld`의 모든 JSON 형식에서 규칙을 찾아야 한다.
우리가 찾으려는 정적 데이터들의 공통된 특징은, 그것이 모두 문자 배열로 담겨 있다는 것이다. 
모든 경로에 해당하는 필드들을 타고 내려간 다음, 그 맨 마지막 값이 문자 배열인 경우 멈추고 그것을 따로 저장하는 식의 로직을 짜면 되는 것.

동료가 만들어준 간단한 웹서비스를 통해 알아보면, 단건조회는 중괄호로 묶여 있고 하나의 기기의 정보만 나온다. 
잘 보면, 그것의 타입을 컴파일러는 `Object`라고 인식한다.
![[Pasted image 20231111142838.png]]

전체조회는 대괄호로 묶이고 같은 종류의 네개의 기기가 나온다.
이 경우 크기가 4인 `Array`로 인식되고, 각각의 엘리먼트는 `Object`.
![[Pasted image 20231111143017.png]]

다음과 같은 방법으로 `FJsonValue`를 `FJsonObject`로 변환 후 그것의 `TMap` 멤버변수의 내부 값을 들여다 보면 다음과 같다.
![[Pasted image 20231111191648.png]]

우리가 요청한 JSON 데이터의 Key-Value가 보인다.
우리가 원하는 정보는 전부 정적인 데이터이고, 배열 안에 장비명이 담겨 있는 식으로 구성돼 있다.
여기에 착안하여, 배열 타입이 나올 때까지 JSON 오브젝트를 타고 들어가는 식의 코드를 짜 보았다.
```cpp
void AML_JsonParser::OnResponseReceived(FHttpRequestPtr _Request, FHttpResponsePtr _Response, bool _bWasSuccessful)  
{  
    if (200 != _Response->GetResponseCode()) return;  
    const FString ResponseBody = _Response->GetContentAsString();  
  
    const TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(ResponseBody);  
    JasonParser(Reader, EEndPtType::STR_ARR);  
}  
  
void AML_JsonParser::JasonParser(const TSharedRef<TJsonReader<>>& _Reader, EEndPtType _EndPtType)  
{  
    FStaticData StaticData;  
  
    TSharedPtr<FJsonValue> JasonValue;  
    if (!FJsonSerializer::Deserialize(_Reader, JasonValue)) return;  
  
    const auto& JasonArr_temp = JasonValue->AsArray();  
    for (auto& JasonVal : JasonArr_temp)  
    {  
       // 다음과 같이 하면 첫번째 층위의 필드에 접근할 수 있다.  
       const auto& JasonMap_temp = JasonVal->AsObject()->Values;  
       ParserInParser(JasonMap_temp, _EndPtType);  
    }  
}  
  
void AML_JsonParser::ParserInParser(const TMap<FString, TSharedPtr<FJsonValue>>& _JsonMap, EEndPtType _EndPtType)  
{  
    for (auto& JasonElem : _JsonMap)  
    {  
       EJson EndPtType;  
         
       switch (_EndPtType)  
       {  
       case EEndPtType::STR:  
          EndPtType = EJson::String;  
          break;  
       case EEndPtType::STR_ARR:  
          EndPtType = EJson::Array;  
          break;  
       case EEndPtType::NUM:  
          EndPtType = EJson::Number;  
          break;  
       default: ;  
       }  
  
       const auto& JasonVal_temp = JasonElem.Value;  
       if (EndPtType == JasonVal_temp->Type) // 우리의 경우, JasonElem.Value->Type이 Array인 경우를 목적지로 삼는다.  
       {  
          // 배열과 그것의 키값을 감싸는 중괄호 오브젝트의 키값은 유니크하다.  
       }  
       else if (EJson::Object == JasonVal_temp->Type)  
          ParserInParser(JasonVal_temp->AsObject()->Values, _EndPtType);  
    }  
}
```

흡수식 냉온수기 하나에 대한 오브젝트를 타고 내려간 다음엔, `ParserInParser`의 재귀 사이클 내로 진입한다.
각 필드를 검사하는데, 그 필드의 값이, 찾고자 하는 정보의 타입과 일치하면 변수와 바인딩하는 작업에 들어가고,그게 아니라 다시 오브젝트인 경우 한번더 그 내부의 필드를 검사하도록 `ParserInParser` 자신을 호출한다.

흡수식 냉온수기가 4개, 하나의 흡수식 냉온수기에 대해 배열이 아홉개로, 배열은 총 36개가 있다.
따라서 `if (EndPtType == JasonVal_temp->Type)`에 해당하는 경우는 총 36개인 것.
로그를 찍어 확인해 보자.
![[Pasted image 20231114213212.png]]

잘 나온다.

각 배열에 이름을 붙여줘야 하므로 그 배열에 해당하는 유니크한 값을 찾는 로직을 짰다.
![[Pasted image 20231114223330.png]]

이를테면 다음과 같은 배열을 찾았다 해보면, 그 위에 있는, 전체 오브젝트를 통틀어 유니크한 이름을 찾아내는 방식.
![[Pasted image 20231114224234.png]]

이제 이것들을 구조체에 담는 작업이 필요.
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

위와 같이 하나의 필드 하나 당 변수 하나를 할당하여 값을 집어넣는 방식을 여러번 시도해 봤으나, 같은 코드를 여러 변수에 대해 각각 여러번 써 줘야 한다는 단점이 있었다.
구조체 내의 멤버변수는 (베열과 달리) 하나하나 써줘야 특정할 수 있기 때문이다.
그래서 생각해 낸 것이, 파싱한 필드네임과 필드의 배열 값들의 아이디를 `TMap` 형태로 만드는 것이었다.
다만 키값은 중복될 수 있어야 한다.
왜냐하면, 하나의 필드네임은 여러개의 기기를 배열로 가지고 있기 때문이다.
이는 다음 코드로 해결할 수 있다.
```cpp  
USTRUCT(BlueprintType)  
struct FStaticData  
{  
    GENERATED_BODY()  
  
    FString Id;  
    TMultiMap<FString, FString> Data;  
};
```

`TMultiMap`은 키값이 중복될 수 있는 맵 형태이다.
이를 가시화하고자 다음과 같이 태그를 가진 액터를 스폰하는 코드를 짰다.
파싱해주는 오브젝트에서 인터페이스 호출.
```cpp  
void AML_JsonParser::ParserToObject_Implementation(FStaticData& _StaticData)  
{  
    Execute_ParserToObject(UGameplayStatics::GetActorOfClass(GetWorld(), AML_ObjectGenerator::StaticClass()),  
                           _StaticData);  
}
```

그리고 스폰 객체.
```cpp
void AML_ObjectGenerator::ParserToObject_Implementation(FStaticData& _StaticData)  
{  
    const auto World = GetWorld();  
    for (auto& ObjData : _StaticData.Data)  
    {  
       const FActorSpawnParameters SpawnParams;  
       const auto Actor_temp = World->SpawnActor<AML_ObjectGenerated>(  
          AML_ObjectGenerated::StaticClass(),  
          FTransform(FRotator::ZeroRotator, FVector(1000.f, 1000.f, 100.f)),  
          SpawnParams  
       );  
  
       Actor_temp->Tags.Add(FName(*ObjData.Value));  
       const FString str_temp = _StaticData.Id + TEXT(":") + ObjData.Key;  
       Actor_temp->Tags.Add(FName(*str_temp));  
    }  
}
```

그리고 확인해 보면,
![[Pasted image 20231119231347.png]]

다만, 여기서 전체 액터의 검색 기능을 구현하기 위해서는 이 모든 액터들이 스폰되고 나서 전체액터를 구조체로 담는 행위를 해야 한다.
이를 위해 [[비동기 호출]]함수의 바인딩을 구현해볼 것이다.
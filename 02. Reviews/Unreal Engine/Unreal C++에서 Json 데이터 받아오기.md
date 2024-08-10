

2023-11-04

----


## 개요
다음과 같이 HTTP 요청을 보내는 함수를 작성하던 중, `SetVerb`가 뭔지 모르겠어서 검색해 보았다.
```cpp
void AML_JsonParser::HttpCall(const FString& _InURL, const FString& _InVerb)  
{  
    const TSharedRef<IHttpRequest> Request = Http->CreateRequest();  
    Request->OnProcessRequestComplete().BindUObject(this, &AML_JsonParser::OnResponseReceived);  
  
    Request->SetURL(_InURL);  
    Request->SetVerb(_InVerb); //   
Request->SetHeader("Content-Type", TEXT("application/json"));  
  
    TSharedRef<FJsonObject> RequestObj = MakeShared<FJsonObject>();  
      
}
```

서버에 보낼 요청의 종류를 결정하는 데에 쓰인다고 한다.
![[스크린샷 2023-11-04 오후 11.11.57.png]]

## 내용

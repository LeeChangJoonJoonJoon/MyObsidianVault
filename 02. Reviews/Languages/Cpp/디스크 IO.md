

2024-09-04

----
#Cpp #diskIo #파일입출력 

## 개요
CPU가 디스크에 저장된 파일을 메모리로 가져올 때 그 정보는 순차적으로 쌓인다. 
디스크의 저장 위치를 포인터처럼 찝어서 바로 접근하는 방법은 없는 건가?

## 내용
언리얼에서 파일 전체를 문자열로 읽어오는 과정은 다음과 같다. 
```cpp
TRACE_CPUPROFILER_EVENT_SCOPE_STR(TEXT("FileIO"));  

TUniquePtr<FArchive> Reader(IFileManager::Get().CreateFileReader(*EachFilePath, FILEREAD_AllowWrite));  
if (!Reader)  
{       continue;  
}  
FScopedLoadingState ScopedLoadingState(*Reader->GetArchiveName());  

int64 Size = Reader->TotalSize();  
if (!Size)  
{       StringData.Empty();  
   return;  
}  
if (Reader->Tell() != 0)  
{       UE_LOG(LogStreaming, Warning, TEXT("Archive '%s' has already been read from."), *Reader->GetArchiveName());  
   return;  
}  
// typedef unsigned char        uint8  
// uint8는 아주 유명한 캐릭터입니다.  
uint8* Ch = (uint8*)FMemory::Malloc(Size);  
Reader->Serialize(Ch, Size);  
bool Success = Reader->Close();  

///////////////////// do something with Ch data /////////////////////   
/////////////////////////////////////////////////////////////////////  

// handle SHA verify of the file   if (EnumHasAnyFlags(FFileHelper::EHashOptions::None, FFileHelper::EHashOptions::EnableVerify) &&  
   (EnumHasAnyFlags(FFileHelper::EHashOptions::None, FFileHelper::EHashOptions::ErrorMissingHash) || FSHA1::GetFileSHAHash(*Reader->GetArchiveName(), nullptr)))  
{  
   // kick off SHA verify task. this frees the buffer on close       FBufferReaderWithSHA Ar(Ch, Size, true, *Reader->GetArchiveName(), false, true);   }   else   {  
   // free manually since not running SHA task  
   FMemory::Free(Ch);  
}
```

`uint8`은 맥의 경우 `unsigned char`이고, 윈도우의 경우 `wchar_t`이다. 
이 경우 디스크에 위치한 파일에 한번만 접근하는 건가?
만약 다음과 같이 그때마다 `fgets`로 파일의 일부를 가져오도록 짠다면 CPU와 디스크가 신호를 주고 받는 횟수가 늘어나서 비효율적이지 않을까?
```cpp
if (FILE* fp = fopen("../../../../UE_PROJECT/nflowinunreal/Dambreak_opti/Dambreak_opti__0.csv", "r"))  
{  
    std::vector<std::string> lines;  
    char buffer[256];  
    while (fgets(buffer, sizeof(buffer), fp) != NULL)  
    {        
	    lines.push_back(std::string(buffer));  
    }    
    
    fclose(fp);  
}
```


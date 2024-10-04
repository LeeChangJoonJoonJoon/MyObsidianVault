

2024-09-22

----
#최적화 #Cpp #CS 

## 개요
`DoThreadWork()` 함수에서 하나의 분할 받은 `.csv` 파일을, 스레드 풀에 있는 스레드 갯수만큼 분할한 부분을 파싱하는 로직은 다음과 같다. 
```cpp
int32 Pid; // 맨 앞에 있는 수가 PID라는 전제로 아래 로직을 짰음.  
  
uint8 Buffer[16]; // 파싱하기 전에 컴마 단위로 끊은 문자열을 저장할 버퍼.  
  
for (int32 k = 0; k < 16; k++)  
    Buffer[k] = '\0';  
  
int32 ColIdx = 0; // 컬럼 인덱스. 하나의 컴마 단위를 캐스팅하여 PosVec4_2dArr에 저장한 뒤 증가하며, 개행이 될 때마다 0으로 초기화.  
int32 BuffIdx = 0; // 버퍼의 몇번째에 문자를 저장할지 결정하는 인덱스. ex) 1.234 -> 1, ., 2, 3, 4 각각을 버퍼의 인덱스로 접근해서 저장.  
double Value; // 버퍼에 저장된 문자열을 double로 변환한 값.  
  
for (int64 j = BeginIdxOfCharRound; j < EndIdxOfCharRound; j++)  
{  
    switch (Ch.Get()[j])  
    {  
    case '\n': // 개행이 나오면, 한 줄에 대한 처리가 끝난 것임.  
       {  
          for (int32 k = 0; k < BuffIdx + 1; k++)  
             Buffer[k] = '\0';  
  
          ColIdx = 0;  
          BuffIdx = 0;  
          break;  
       }  
    case ',':  
       {  
          if (!ColIdx) // 만약 맨 처음 col을 읽고 있다면~~~  
          {  
             Pid = std::atoi(reinterpret_cast<const char*>(Buffer));  
             // 만약 현재 우리가 받은 dambreak 파일과 같이 첫번째 컬럼이 PID라면 이렇게 처리.  
          }  
          else  
          {  
             Value = std::strtod(reinterpret_cast<const char*>(Buffer), nullptr);  
             switch (ColIdx)  
             {             
             case 3:  
                {  
                   PosVec4_2dArr[FrameNum][Pid].X = Value;  
                   break;  
                }  
             case 4:  
                {  
                   PosVec4_2dArr[FrameNum][Pid].Y = Value;  
                   break;  
                }  
             case 5:  
                {  
                   PosVec4_2dArr[FrameNum][Pid].Z = Value;  
                   break;  
                }  
             case 1:  
                {  
                   PosVec4_2dArr[FrameNum][Pid].W = Value;  
                   break;  
                }  
             case 6:  
                {  
                   VelVec4_2dArr[FrameNum][Pid].X = Value;  
                   break;  
                }  
             case 7:  
                {  
                   VelVec4_2dArr[FrameNum][Pid].Y = Value;  
                   break;  
                }  
             case 8:  
                {  
                   VelVec4_2dArr[FrameNum][Pid].Z = Value;  
                   break;  
                }  
             case 2:  
                {  
                   VelVec4_2dArr[FrameNum][Pid].W = Value;  
                   break;  
                }  
             default:  
                break;  
             }  
          }  
  
          for (int32 k = 0; k < BuffIdx + 1; k++)  
             Buffer[k] = '\0';  
  
          BuffIdx = 0;  
          ColIdx++;  
          break;  
       }  
    case '.':  
       {  
          Buffer[BuffIdx] = '.';  
          BuffIdx++;  
          break;  
       }  
    default:  
       {  
          Buffer[BuffIdx] = Ch.Get()[j];  
          BuffIdx++;  
          break;  
       }  
    }  
}
```

## 내용
여기서 `reinterpret_cast<const char*>`의 속도를 개선해 보고자 한다. 
https://80000coding.oopy.io/cf627735-2ff8-4d04-8849-9e356269e57e

우선, 위 링크에 따르면 이 `reinterpret_cast<const char*>` 키워드는 컴파일러가 해당 메모리 위치에 저장된 바이트를 읽을 때 어떤 형식으로 읽을지를 변환해 주는 것이라 했다. 
즉, 런타임에서 해당 키워드가 쓰인 타이밍이 됐을 때 컴파일러가 해당 메모리를 다른 타입으로 읽는다는 것. 
따라서 메모리 상에는 어떤 수가 저장돼 있을 뿐, 어떤 타입으로 해석하기 나름.

그래서 궁금한 것. 
1. `reinterpret_cast<const char*>`를 쓰면 메모리 위치의 변경이 일어나는가?
2. 메모리 내에서 저장되는 형식은 어떻게 바뀌나?

직접 메모리 프로파일링을 해보기로.
... 했는데 스택 메모리에 저장돼 있어서 그런지 memory view에 뜨지 않았다... ㅅㅂ

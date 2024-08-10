

2023-10-19

----
#compiler #Unreal #Xcode #Mac #Cpp 

## 개요
현재 문제는 다음과 같다.
언리얼은 VSCode를 인식하고 컴파일, `Refresh Visual Studio Code Project`도 정상적으로 된다.
근데 VSCode가 헤더파일을 찾질 못한다.
![[Pasted image 20231019201332.png]]

그리고 빌드를 누르면 다음과 같이 에러가 나온다.
![[Pasted image 20231019201407.png]]

에러 내용은 대략 다음과 같다.
![[Pasted image 20231019201447.png]]

## 내용
우선 헤더파일을 찾지 못하는 문제부터 고쳐보자.
조금 옛날 게시물이긴 하지만, 다음 링크에서 힌트를 찾아 본다.
https://forums.unrealengine.com/t/vscode-on-mac-produce-intellisense-errors-due-to-missing-include/414423/4

다음을 보면, `“/Users/Shared/UnrealEngine/UE_4.20/Engine/**”`라고 엔진 소스의 위치를 써 넣으라고 나온다.
![[Pasted image 20231019201923.png]]

나의 경우 엔진 파일의 경로는 다음과 같다.
`/Users/Shared/Epic Games/UE_5.3/Engine`

그러니 나는 `/Users/Shared/Epic Games/UE_5.3/Engine/**`라고 넣어주면 된다.
(그래도 일단 실패...)
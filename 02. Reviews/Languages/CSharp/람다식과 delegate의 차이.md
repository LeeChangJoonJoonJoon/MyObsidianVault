

2023-05-06

----
#CSharp #람다함수 #닷넷정복 #delegate

## 개요
[[람다 함수]]를 공부하던 중...
프로토타입을 받아 와서 더 많은 메서드를 덧붙인다는 점에서 둘은 동일해 보이는데, 굳이 구분하여 사용하는 것이 이해가 되지 않았다.
따라서 둘의 차이를 짚고 넘어가야 할 필요가 있어 보인다.

## 내용
'닷넷 정복'엔 delegate의 예시로 다음 코드를 보여준다.
```C#
using System;

namespace MyApp // Note: actual namespace depends on the project name.
{
	delegate void Dele(int a);
	
    internal class Program
    {
		public static void Method1(int a){ Console.WriteLine("Method1 " + a); }
		public static void Method2(int a){ Console.WriteLine("Method2 " + a); }
		
        static void Main(string[] args)
        {
	        Dele d;
	        d = Method1;
	        d(12); // Method1 12
	        d = Method2;
	        d(34); // Method2 34
        }
    }
}
```

이를 도식으로 나타내면 다음과 같다.
![[스크린샷 2023-05-06 오후 10.21.17.png|500]]

즉, 객체화되기 전의 `Dele()` 메서드는 리턴 값과 매개변수의 형을 지정해주고, 이 형식에 맞도록 구체적인 메서드를 `internal class Program{...}` 내에서 정해주는 방식인 것이다.

람다식과의 차이는 무엇이란 말인가?

`delegate`만 썼을 때는 `internal class Program{...}` 내에서 구체적인 함수의 내용을 적어 주어야했다.
반면 람다식은 메인함수 내에서 이를 적어준다.
이는 구체화할 함수의 내용이 간단할 수록 유용하다.

'닷넷 정복'은 둘 사이의 차이를 다음과 같이 설명하고 있다.
> `delegate` 키워드, 인수의 타입, `return`문 등 형식을 맞추기 위한 번거로운 것을 제거하여 익명 메서드를 압축한 것이 람다식이다. 람다는 조건을 점검하는 코드를 조건식 형태의 데이터처럼 다루어 쿼리문을 함축적으로 표현한다. 익명 메서드에 적용되는 규칙이 람다 표현식에도 대부분 그대로 적용된다. 인수의 개수나 타입 일치성을 점검하는 규칙은 거의 동일하며 자신을 포함한 메서드의 인수나 지역변수를 식내에서 참조할 수 있는 변수캡처 기능도 똑같이 적용된다.

결론. 표현 방식의 차이이다.
어떤 걸 구체적으로 추가할지에 따라 각각의 유용성이 달라진다.
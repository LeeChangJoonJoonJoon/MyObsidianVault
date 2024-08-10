

2023-05-04

----
#Server   

## 개요
어제 방을 만드는 것까지 구현해 보았다.
이제 Exit를 누르면 방에서 나가는 것을 구현해 보자.
또, 플레이어의 스폰을 무작위로 배치하고 플레이어끼리 그 동작을 동기화해 볼 것이다. 

## 내용
Scene에서 나가는 Exit 버튼에 함수를 할당해야 한다.

생성된 플레이어가 로컬이라는 것을 알려주는 스크립트를 하나 생성해 줄 것이다. 
`camlookat`을 Player Text Parent에 넣어줘야 한다.
이는 플레이어의 ID가 카메라에 잘 보이도록 하기 위함이다.

서버를 통해 다른 사람에게도 자신의 모습을 쏴 줘야 하고
스폰된 것도 자신 클라이언트의 씬에 등장시켜줘야 한다.

우리가 만든 것을 서버에 보내자.
그러기 위해선 프로토콜을 추가해야 한다.
```C#
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
using System.Threading.Tasks;  
  
namespace MetaBotServer  
{  
public enum PROTOCOL : short  
{  
BEGIN = 0,  
  
REG_MEMBER_REQ = 1, // 회원가입 요청  
REG_MEMBER_ACK = 2, // 회원가입 응답  
  
LOGIN_REQ = 3, // 로그인 요청  
LOGIN_ACK = 4, // 로그인 응답  
  
MAKE_ROOM_REQ = 5, // 방 생성 요청  
MAKE_ROOM_ACK = 6, // 방 생성 응답  
  
ROOM_LIST_REQ = 7, // 방 리스트 요청  
ROOM_LIST_ACK = 8, // 방 리스트 응답  
  
JOIN_ROOM_REQ = 9, // 방 참여 요청  
JOIN_ROOM_ACK = 10, // 방 참여 응답  
  
SPAWN_PLAYER_REQ = 11, // 게임룸에 플레이어 생성  
SPAWN_PLAYER_ACK = 12, // 게임룸에 플레이어 생성 응답  
  
TRANSFORM_PLAYER_REQ = 13, // 플레이어 위치/회전 정보 전달  
TRANSFORM_PLAYER_ACK = 14, // 플레이어 위치/회전 정보 응답  
  
SHOT_PLAYER_REQ = 15, // 플레이어 총 발사 전달  
SHOT_PLAYER_ACK = 16, // 플레이어 총 발사 응답  
  
CHAT_MSG_REQ = 17, // 게임룸내에서 채팅 전송  
CHAT_MSG_ACK = 18, // 게임룸내에서 채팅 응답  
  
END  
}  
}
```

유저들이 게임룸 안에서 패킷을 주고 받도록 설계할 것이다.


2023-05-08

----
#Server  #OracleDB #Unity #CSharp #수업 #MVC 

## 개요
오늘은 카메라가 움직이고 플레이어가 움직이는 모습을 서버를 통해 연동하는 것을 구현해 보고자 한다.
Exit 처리는 그냥 씬만 바뀌게 할 뿐, MVC에서 실제로 이벤트가 생기는 건 아니다.
로그를 띄워주는 역할도 만들어야 한다.

## 내용
내가 움직이는 플레이어의 위치와 회전 값을 다른 사용자에게 보내준다.
우리는 서버가 모든 사용자에게 뿌리도록 하는 MMORPG와 같이 만들고자 한다.
![[스크린샷 2023-05-08 오전 10.55.15.png|300]]

반면, 배틀넷(Battle Net)에서 하는 것과 같이 방을 만들고 사용자끼리 직접 접속하는 방식도 있다. 
여기서 서버는 초대와 각종 로그(방 정보, 전적)만 관리한다.
그리고 다른 사용자끼리 접속하도록 서버역할을 하는 방장(Host)를 만들수도 있고, 아니면 모든 사용자가 서로 연결되게 만들 수도 있다.
![[스크린샷 2023-05-08 오전 10.56.30.png|400]]

다음 함수를 통해 플레이어의 `Transform` 정보를 담은 패킷을 보내준다.
```C#
SendReqTransformPlayer();
```

다만 매 60 프레임 마다 패킷을 보내는 것은 서버에 부하를 너무 많이 준다.
따라서 그 정도로 패킷을 여러번 주진 않고 다음을 이용하여 부드럽게 이동하도록 만들어 준다.
(일종의 보간하는 역할을 한다고 생각하면 좋다.)
```C#
controller.SimpleMove(moveDir * moveSpeed);
```

다음은 이전 프레임의 정보를 저장하는 변수이다.
이는 플레이어의 `Transform` 변화가 일정 크기 이상일 때만 패킷을 전송하도록 만들기 위함이다.
```C#
// 서버로 비교/전송할 Transform 정보  
private Vector3 prePosition = Vector3.zero; // 이전 프레임의 위치 정보  
private Vector3 preRotator = Vector3.zero; // 이전 프레임의 회전 정보
```

다음은 현재 프레임의 정보를 저장하는 변수이다.
이는 서버로 비교/전송할 `Transform` 정보에 해당한다.
```C#
private Vector3 nowPosition = Vector3.zero; // 현재 프레임의 위치 정보  
private Vector3 nowRotation = Vector3.zero; // 현재 프레임의 회전 정보  
private float nowForward = 0f; // 현재 애니메이션 값  
private float nowStrafe = 0f; // 현재 애니메이션 값  
private Vector3 nowMoveDir = Vector3.zero; // 현재 controller.SimpleMove로 이동하는 벡터값
```

다음 조건일 때 패킷을 보내지 않는다는 것을 표현하고자 한다.
```C#
Vector3.Distance(nowPosition, prePosition) < 0.01f && (Mathf.Abs(nowRotator.y - preRotator.y) < 3f)
```

이 조건에 해당되지 않을 경우, 다음 플레이어 정보를 패킷에 담아 서버에 보낸다.
```C#
CPacket msg = CPacket.create((short)PROTOCOL.TRANSFORM_PLAYER_REQ);  
  
msg.push(playerInfo.USER_ID);  
// controller.SimpleMove로 이동해야 하는 값  
msg.push(nowMoveDir.x);  
msg.push(nowMoveDir.y);  
msg.push(nowMoveDir.z);  
// transform.position  
msg.push(nowPosition.x);  
msg.push(nowPosition.y);  
msg.push(nowPosition.z);  
// transform.eulerAngles  
msg.push(nowRotator.x);  
msg.push(nowRotator.y);  
msg.push(nowRotator.z);  
// animator val.  
msg.push(nowForward);  
msg.push(nowStrafe);  
  
CNetworkManager.Instance.send(msg);
```

이 패킷을 받기 위해선 서버에 있는 `CGameUser`에 가서 다음을 추가해 준다.
```C#
public void send(CPacket msg)  
{  
this.token.send(msg);  
}
```

여기서 문제 발생.
```C#
CPacket ack = CPacket.create((short)PROTOCOL.TRANSFORM_PLAYER_ACK);  
  
// 여기서 패킷의 데이터를 꺼낸다.  
string uid = msg.pop_string();  
float moveX = msg.pop_Float();  
float moveY = msg.pop_Float();  
float moveZ = msg.pop_Float();  
float posX = msg.pop_Float();  
float posY = msg.pop_Float();  
float posZ = msg.pop_Float();  
float eulerX = msg.pop_Float();  
float eulerY = msg.pop_Float();  
float eulerZ = msg.pop_Float();  
float forward = msg.pop_Float();  
float strafe = msg.pop_Float();  
  
SDebug.WriteLog($"uid: {uid}, mx: {moveX}, my: {moveY}, mz: {moveZ}, " +  
$"posX: {posX}, posY: {posY}, posZ: {posZ}" +  
$"eulX: {eulerX}, eulY: {eulerY}, eulZ: {eulerZ}" +  
$"forward: {forward}, strafe: {strafe}");  
ack.push(uid);  
ack.push(moveX);  
ack.push(moveY);  
ack.push(moveZ);  
ack.push(posX);  
ack.push(posY);  
ack.push(posZ);  
ack.push(eulerX);  
ack.push(eulerY);  
ack.push(eulerZ);  
ack.push(forward);  
ack.push(strafe);
```

이와 같이 패킷을 분해해서 패킷을 보내는 데에 있어서 장애가 생겼다.
`CPacket`을 받는 `pop_strig()` 메서드는 패킷의 정보를 빼내는 역할을 한다.
이 빼내는 과정에서 [[FreeNet의 패킷 구조에서 비롯되는 에러]]가 생긴 것이다.
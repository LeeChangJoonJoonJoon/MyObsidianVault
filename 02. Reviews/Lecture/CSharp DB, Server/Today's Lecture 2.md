

2023-05-03

----
#CSharp #Server #OracleDB

## 내용
[[Today's Lecture 1]]에서는 지금까지 한 내용을 대략적으로 정리해 두었고, 진짜 오늘 배운 내용은 여기부터 시작하겠다.

게임룸을 위한 클래스들을 만들었다.
전체적인 구조를 종합하여 설명해 보면, `CGameRoomManager`는 맨 위에서 `CGameRoom`을 관리하고, 다시 `CGameRoom은 CPlayer`를 관리한다. 
그리고 `CPlayer`에는 `CGameUser`와 `CUserToken`이 일대일대응된다.
이 셋은 서로 상호참조 한다.

`CUserToken`은 Low Level의 패킷을 처리하는 역할을 한다.
`CGameUser`는 패킷의 로직 분리를 담당한다.
`CPlayer`는 게임룸 내에서의 클라이언트에 해당된다.

이를 도식화하면 다음과 같이 그려진다.
![[스크린샷 2023-05-03 오전 11.26.35.png]]

이 세 구성요소가 하나의 클라이언트를 구성한다.
물론, CUserToken 내에서 모든 기능들을 구현해도 작동은 된다.
하지만 코드를 관리하는 데에 있어 용이함을 위해 따로 작성하여 조합하는 방식을 택했다.

위와 같이 도식화한 구조를 클래스로 만들어 주었다.
```C#
namespace _09_MetaBotServer  
{  
// 게임 룸에 진입한 게임 상에서의 클라이언트  
public class CPlayer  
{  
public CGameRoom GAME_ROOM { get; set; } // 소속 룸을 의미  
public CGameUser GAME_USER { get; set; } // 연결된 CGameUser 객체를 의미  
public string USER_ID { get; set; } // Player의 정보를 의미한다.  
  
public CPlayer(CGameRoom gameRoom, CGameUser gameUser, string user_id)  
{  
this.GAME_ROOM = gameRoom;  
this.GAME_USER = gameUser;  
this.USER_ID = user_id;  
this.GAME_USER.PLAYER = this;  
}  
}  
}
```

`CGameUser`는 이미 만들어 둔 상태이다.
여기에 조금 수정을 가했다.
```C#
using FreeNet;  
using MetaBotServer;  
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
using System.Threading.Tasks;  
  
namespace _09_MetaBotServer  
{  
// 패킷의 비즈니스 로직 분기를 위한 클라이언트  
public class CGameUser : IPeer  
{  
public CPlayer PLAYER { get; set; } // 게임룸 내에서의 클라이언트관련 정보를 담당.  
CProcessPacket procPacket; // 하나의 클래스로만 패킷을 처리하면 모듈화하기 힘들다.  
CUserToken token; // 패킷 처리를 해주는 곳.  
  
public delegate void Remove_user(CGameUser user);  
public Remove_user remove_user_callback { get; set; }  
  
public CGameUser(CProcessPacket procPacket, CUserToken token)  
{  
this.procPacket = procPacket;  
this.token = token;  
this.token.set_peer(this);  
}  
  
public void disconnect()  
{  
this.token.socket.Disconnect(false);  
}  
  
public void on_message(Const<byte[]> buffer)  
{  
CPacket msg = new CPacket(buffer.Value, this);  
  
process_user_operation(msg);  
}  
  
public void on_removed()  
{  
if (remove_user_callback != null)  
remove_user_callback(this);  
}  
  
public void process_user_operation(CPacket msg)  
{  
PROTOCOL protocol = (PROTOCOL)msg.pop_protocol_id();  
  
switch(protocol)  
{  
case PROTOCOL.REG_MEMBER_REQ:  
procPacket.Process_REG_MEMBER_REQ(msg);  
break;  
case PROTOCOL.LOGIN_REQ:  
procPacket.Process_LOGIN_REQ(msg);  
break;  
case PROTOCOL.ROOM_LIST_REQ:  
procPacket.Process_ROOM_LIST_REQ(msg);  
break;  
case PROTOCOL.MAKE_ROOM_REQ:  
procPacket.Process_MAKE_ROOM_REQ(msg);  
break;  
}  
}  
  
public void send(CPacket msg)  
{  
this.token.send(msg);  
}  
}  
}
```

`CUserToken`은 예전에 만들어 두었다.
```C#
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
using System.Net.Sockets;  
  
namespace FreeNet  
{  
public class CUserToken  
{  
public Socket socket { get; set; }  
  
public SocketAsyncEventArgs receive_event_args { get; private set; }  
public SocketAsyncEventArgs send_event_args { get; private set; }  
  
// 바이트를 패킷 형식으로 해석해주는 해석기.  
CMessageResolver message_resolver;  
  
// session객체. 어플리케이션 딴에서 구현하여 사용.  
IPeer peer;  
  
// 전송할 패킷을 보관해놓는 큐. 1-Send로 처리하기 위한 큐이다.  
Queue<CPacket> sending_queue;  
// sending_queue lock처리에 사용되는 객체.  
private object cs_sending_queue;  
  
public CUserToken()  
{  
this.cs_sending_queue = new object();  
  
this.message_resolver = new CMessageResolver();  
this.peer = null;  
this.sending_queue = new Queue<CPacket>();  
}  
  
public void set_peer(IPeer peer)  
{  
this.peer = peer;  
}  
  
public void set_event_args(SocketAsyncEventArgs receive_event_args, SocketAsyncEventArgs send_event_args)  
{  
this.receive_event_args = receive_event_args;  
this.send_event_args = send_event_args;  
}  
  
/// <summary>  
/// 이 매소드에서 직접 바이트 데이터를 해석해도 되지만 Message resolver클래스를 따로 둔 이유는  
/// 추후에 확장성을 고려하여 다른 resolver를 구현할 때 CUserToken클래스의 코드 수정을 최소화 하기 위함이다.  
/// </summary>  
/// <param name="buffer"></param>  
/// <param name="offset"></param>  
/// <param name="transfered"></param>  
public void on_receive(byte[] buffer, int offset, int transfered)  
{  
this.message_resolver.on_receive(buffer, offset, transfered, on_message);  
}  
  
void on_message(Const<byte[]> buffer)  
{  
if (this.peer != null)  
{  
this.peer.on_message(buffer);  
}  
}  
  
public void on_removed()  
{  
this.sending_queue.Clear();  
  
if (this.peer != null)  
{  
this.peer.on_removed();  
}  
}  
  
/// <summary>  
/// 패킷을 전송한다.  
/// 큐가 비어 있을 경우에는 큐에 추가한 뒤 바로 SendAsync매소드를 호출하고,  
/// 데이터가 들어있을 경우에는 새로 추가만 한다.  
///  
/// 큐잉된 패킷의 전송 시점 :/// 현재 진행중인 SendAsync가 완료되었을 때 큐를 검사하여 나머지 패킷을 전송한다.  
/// </summary>  
/// <param name="msg"></param>  
public void send(CPacket msg)  
{  
CPacket clone = new CPacket();  
msg.copy_to(clone);  
  
lock (this.cs_sending_queue)  
{  
// 큐가 비어 있다면 큐에 추가하고 바로 비동기 전송 매소드를 호출한다.  
if (this.sending_queue.Count <= 0)  
{  
this.sending_queue.Enqueue(clone);  
start_send();  
return;  
}  
  
// 큐에 무언가가 들어 있다면 아직 이전 전송이 완료되지 않은 상태이므로 큐에 추가만 하고 리턴한다.  
// 현재 수행중인 SendAsync가 완료된 이후에 큐를 검사하여 데이터가 있으면 SendAsync를 호출하여 전송해줄 것이다.  
Console.WriteLine("Queue is not empty. Copy and Enqueue a msg. protocol id : " + msg.protocol_id);  
this.sending_queue.Enqueue(clone);  
}  
}  
  
/// <summary>  
/// 비동기 전송을 시작한다.  
/// </summary>  
void start_send()  
{  
lock (this.cs_sending_queue)  
{  
// 전송이 아직 완료된 상태가 아니므로 데이터만 가져오고 큐에서 제거하진 않는다.  
CPacket msg = this.sending_queue.Peek();  
  
// 헤더에 패킷 사이즈를 기록한다.  
msg.record_size();  
  
// 이번에 보낼 패킷 사이즈 만큼 버퍼 크기를 설정하고  
this.send_event_args.SetBuffer(this.send_event_args.Offset, msg.position);  
// 패킷 내용을 SocketAsyncEventArgs버퍼에 복사한다.  
Array.Copy(msg.buffer, 0, this.send_event_args.Buffer, this.send_event_args.Offset, msg.position);  
  
// 비동기 전송 시작.  
bool pending = this.socket.SendAsync(this.send_event_args);  
if (!pending)  
{  
process_send(this.send_event_args);  
}  
}  
}  
  
static int sent_count = 0;  
static object cs_count = new object();  
/// <summary>  
/// 비동기 전송 완료시 호출되는 콜백 매소드.  
/// </summary>  
/// <param name="e"></param>  
public void process_send(SocketAsyncEventArgs e)  
{  
if (e.BytesTransferred <= 0 || e.SocketError != SocketError.Success)  
{  
//Console.WriteLine(string.Format("Failed to send. error {0}, transferred {1}", e.SocketError, e.BytesTransferred));  
return;  
}  
  
lock (this.cs_sending_queue)  
{  
// count가 0이하일 경우는 없겠지만...  
if (this.sending_queue.Count <= 0)  
{  
throw new Exception("Sending queue count is less than zero!");  
}  
  
//todo:재전송 로직 다시 검토~~ 테스트 안해봤음.  
// 패킷 하나를 다 못보낸 경우는??  
int size = this.sending_queue.Peek().position;  
if (e.BytesTransferred != size)  
{  
string error = string.Format("Need to send more! transferred {0}, packet size {1}", e.BytesTransferred, size);  
Console.WriteLine(error);  
return;  
}  
  
  
//System.Threading.Interlocked.Increment(ref sent_count);  
lock (cs_count)  
{  
++sent_count;  
//if (sent_count % 20000 == 0)  
{  
Console.WriteLine(string.Format("process send : {0}, transferred {1}, sent count {2}",  
e.SocketError, e.BytesTransferred, sent_count));  
}  
}  
  
//Console.WriteLine(string.Format("process send : {0}, transferred {1}, sent count {2}",  
// e.SocketError, e.BytesTransferred, sent_count));  
  
// 전송 완료된 패킷을 큐에서 제거한다.  
//CPacket packet = this.sending_queue.Dequeue();  
//CPacket.destroy(packet);  
this.sending_queue.Dequeue();  
  
// 아직 전송하지 않은 대기중인 패킷이 있다면 다시한번 전송을 요청한다.  
if (this.sending_queue.Count > 0)  
{  
start_send();  
}  
}  
}  
  
//void send_directly(CPacket msg)  
//{  
// msg.record_size();  
// this.send_event_args.SetBuffer(this.send_event_args.Offset, msg.position);  
// Array.Copy(msg.buffer, 0, this.send_event_args.Buffer, this.send_event_args.Offset, msg.position);  
// bool pending = this.socket.SendAsync(this.send_event_args);  
// if (!pending)  
// {  
// process_send(this.send_event_args);  
// }  
//}  
  
public void disconnect()  
{  
// close the socket associated with the client  
try  
{  
this.socket.Shutdown(SocketShutdown.Send);  
}  
// throws if client process has already closed  
catch (Exception) { }  
this.socket.Close();  
}  
  
public void start_keepalive()  
{  
System.Threading.Timer keepalive = new System.Threading.Timer((object e) =>  
{  
CPacket msg = CPacket.create(0);  
msg.push(0);  
send(msg);  
}, null, 0, 3000);  
}  
}  
}
```


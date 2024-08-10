

2023-05-03

----
#CSharp #Server #OracleDB

## 개요
오늘은 지금까지 해오던 대로 오라클과 서버를 유니티에 연결하여 게임을 할 수 있게 만드는 것을 배울 것이다. 
여기서 명심할 것은, MetaServer 파일은 서버 역할을 하고, 유니티 스크립트는 클라이언트 역할을 한다는 것이다. 
오늘은 같은 방에 있는 사용자끼리 패킷을 주고 받는 것을 구현할 것이다.

## 지금까지 배운 내용
다음 코드는 지금까지 만든 클래스들이다.

```C#
using FreeNet;  
using NetworkDebug;  
using System;  
using System.Collections.Generic;  
using System.Data.SqlClient;  
using System.Linq;  
using System.Text;  
using System.Threading.Tasks;  
  
namespace _09_MetaBotServer  
{  
internal class Program  
{  
static CProcessPacket procPacket = new CProcessPacket();  
static List<CGameUser> userlist = new List<CGameUser>();  
  
static void Main(string[] args)  
{  
CPacketBufferManager.initialize(2000);  
  
CNetworkService service = new CNetworkService();  
service.session_created_callback += on_session_created;  
service.initialize();  
service.listen("0.0.0.0", 7979, 10000);  
  
while (true)  
{  
string input = Console.ReadLine();  
if (input.Equals("exit"))  
break;  
  
System.Threading.Thread.Sleep(1000);  
}  
  
SDebug.WriteLog("Server End");  
}  
  
static void on_session_created(CUserToken token)  
{  
CGameUser user = new CGameUser(procPacket, token);  
user.remove_user_callback += on_remove_user;  
  
SDebug.WriteLog($"{user} Client Connected");  
  
lock (userlist)  
{  
userlist.Add(user);  
}  
}  
  
static void on_remove_user(CGameUser user)  
{  
SDebug.WriteLog($"{user} Client Remove");  
  
lock (userlist)  
{  
userlist.Remove(user);  
}  
}  
}  
}
```

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
public class CGameUser : IPeer  
{  
CProcessPacket procPacket;  
CUserToken token;  
  
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

```C#
using _02_DBManager;  
using FreeNet;  
using MetaBotServer;  
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
using System.Threading.Tasks;  
  
namespace _09_MetaBotServer  
{  
public class CProcessPacket  
{  
GameRoomDao gameDao = new GameRoomDao();  
MemberDao memDao = new MemberDao();  
  
public void Process_REG_MEMBER_REQ(CPacket msg)  
{  
// DB에 적용  
MemberVo vo = new MemberVo();  
vo.MEMBER_ID = msg.pop_string();  
vo.MEMBER_PASS = msg.pop_string();  
int row = memDao.InsertMember(vo);  
  
// Unity Client에 응답  
CPacket ack = CPacket.create((short)PROTOCOL.REG_MEMBER_ACK);  
if (row == 1)  
ack.push((byte)1); // success  
else  
ack.push((byte)0); // fail  
  
msg.owner.send(ack);  
}  
  
public void Process_LOGIN_REQ(CPacket msg)  
{  
string id = msg.pop_string();  
string pass = msg.pop_string();  
MemberVo vo = memDao.GetMemberData(id, pass);  
  
// Unity Client에 결과를 응답  
CPacket ack = CPacket.create((short)PROTOCOL.LOGIN_ACK);  
ack.push(id);  
if(id.Equals(vo.MEMBER_ID))  
ack.push((byte)1); // success  
else  
ack.push((byte)0); // fail  
msg.owner.send(ack);  
}  
  
public void Process_ROOM_LIST_REQ(CPacket msg)  
{  
IList<GameRoomVo> voList = gameDao.GetGameRoomList();  
  
CPacket ack = CPacket.create((short)(PROTOCOL.ROOM_LIST_ACK));  
ack.push_int16((short)voList.Count);  
foreach(var vo in voList)  
{  
ack.push(vo.GAMEROOM_ID);  
ack.push(vo.GAMEROOM_NAME);  
}  
  
msg.owner.send(ack);  
}  
  
public void Process_MAKE_ROOM_REQ(CPacket msg)  
{  
string user_id = msg.pop_string();  
string gameroom_id = msg.pop_string();  
string gameroom_name = msg.pop_string();  
  
GameRoomVo vo = new GameRoomVo();  
vo.GAMEROOM_ID = gameroom_id;  
vo.GAMEROOM_NAME = gameroom_name;  
int row = gameDao.InsertGameRoom(vo);  
  
CPacket ack = CPacket.create((short)PROTOCOL.MAKE_ROOM_ACK);  
ack.push(user_id);  
ack.push(gameroom_id);  
if (row == 1)  
ack.push((byte)1);  
else  
ack.push((byte)0);  
  
msg.owner.send(ack);  
}  
}  
}
```

다음은 프로토콜 즉, 약속 체계이다.
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
  
END  
}  
}
```

```C#
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
using System.Threading.Tasks;  
  
namespace NetworkDebug  
{  
public static class SDebug  
{  
static bool isDebug { get; set; }  
  
static SDebug()  
{  
isDebug = true;  
}  
  
public static void WriteLog(string message)  
{  
if (isDebug)  
{  
#if UNITY_STANDALONE_WIN  
Debug.Log(message);  
#else  
Console.WriteLine(message);  
#endif  
}  
}  
}  
}
```

다음은 강의 내내 사용하는 `FreeNet`라이브러리이다.
```C#
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
using System.Net;  
using System.Net.Sockets;  
using System.Threading;  
  
namespace FreeNet  
{  
class CListener  
{  
// 비동기 Accept를 위한 EventArgs.SocketAsyncEventArgs accept_args;  
  
Socket listen_socket;  
  
// Accept처리의 순서를 제어하기 위한 이벤트 변수.  
AutoResetEvent flow_control_event;  
  
// 새로운 클라이언트가 접속했을 때 호출되는 콜백.  
public delegate void NewclientHandler(Socket client_socket, object token);  
public NewclientHandler callback_on_newclient;  
  
public CListener()  
{  
this.callback_on_newclient = null;  
}  
  
public void start(string host, int port, int backlog)  
{  
this.listen_socket = new Socket(AddressFamily.InterNetwork,  
SocketType.Stream, ProtocolType.Tcp);  
  
IPAddress address;  
if (host == "0.0.0.0")  
{  
address = IPAddress.Any;  
}  
else  
{  
address = IPAddress.Parse(host);  
}  
IPEndPoint endpoint = new IPEndPoint(address, port);  
  
try  
{  
listen_socket.Bind(endpoint);  
listen_socket.Listen(backlog);  
  
this.accept_args = new SocketAsyncEventArgs();  
this.accept_args.Completed += new EventHandler<SocketAsyncEventArgs>(on_accept_completed);  
  
Thread listen_thread = new Thread(do_listen);  
listen_thread.Start();  
}  
catch (Exception e)  
{  
//Console.WriteLine(e.Message);  
}  
}  
  
/// <summary>  
/// 루프를 돌며 클라이언트를 받아들입니다.  
/// 하나의 접속 처리가 완료된 후 다음 accept를 수행하기 위해서 event객체를 통해 흐름을 제어하도록 구현되어 있습니다.  
/// </summary>  
void do_listen()  
{  
this.flow_control_event = new AutoResetEvent(false);  
  
while (true)  
{  
// SocketAsyncEventArgs를 재사용 하기 위해서 null로 만들어 준다.  
this.accept_args.AcceptSocket = null;  
  
bool pending = true;  
try  
{  
// 비동기 accept를 호출하여 클라이언트의 접속을 받아들입니다.  
// 비동기 매소드 이지만 동기적으로 수행이 완료될 경우도 있으니  
// 리턴값을 확인하여 분기시켜야 합니다.  
pending = listen_socket.AcceptAsync(this.accept_args);  
}  
catch (Exception e)  
{  
//Console.WriteLine(e.Message);  
continue;  
}  
  
// 즉시 완료 되면 이벤트가 발생하지 않으므로 리턴값이 false일 경우 콜백 매소드를 직접 호출해 줍니다.  
// pending상태라면 비동기 요청이 들어간 상태이므로 콜백 매소드를 기다리면 됩니다.  
// http://msdn.microsoft.com/ko-kr/library/system.net.sockets.socket.acceptasync%28v=vs.110%29.aspx  
if (!pending)  
{  
on_accept_completed(null, this.accept_args);  
}  
  
// 클라이언트 접속 처리가 완료되면 이벤트 객체의 신호를 전달받아 다시 루프를 수행하도록 합니다.  
this.flow_control_event.WaitOne();  
  
// *팁 : 반드시 WaitOne -> Set 순서로 호출 되야 하는 것은 아닙니다.  
// Accept작업이 굉장히 빨리 끝나서 Set -> WaitOne 순서로 호출된다고 하더라도  
// 다음 Accept 호출 까지 문제 없이 이루어 집니다.  
// WaitOne매소드가 호출될 때 이벤트 객체가 이미 signalled 상태라면 스레드를 대기 하지 않고 계속 진행하기 때문입니다.  
}  
}  
  
/// <summary>  
/// AcceptAsync의 콜백 매소드  
/// </summary>  
/// <param name="sender"></param>  
/// <param name="e">AcceptAsync 매소드 호출시 사용된 EventArgs</param>void on_accept_completed(object sender, SocketAsyncEventArgs e)  
{  
if (e.SocketError == SocketError.Success)  
{  
// 새로 생긴 소켓을 보관해 놓은뒤~  
Socket client_socket = e.AcceptSocket;  
  
// 다음 연결을 받아들인다.  
this.flow_control_event.Set();  
  
// 이 클래스에서는 accept까지의 역할만 수행하고 클라이언트의 접속 이후의 처리는  
// 외부로 넘기기 위해서 콜백 매소드를 호출해 주도록 합니다.  
// 이유는 소켓 처리부와 컨텐츠 구현부를 분리하기 위함입니다.  
// 컨텐츠 구현부분은 자주 바뀔 가능성이 있지만, 소켓 Accept부분은 상대적으로 변경이 적은 부분이기 때문에  
// 양쪽을 분리시켜주는것이 좋습니다.  
// 또한 클래스 설계 방침에 따라 Listen에 관련된 코드만 존재하도록 하기 위한 이유도 있습니다.  
if (this.callback_on_newclient != null)  
{  
this.callback_on_newclient(client_socket, e.UserToken);  
}  
  
return;  
}  
else  
{  
//todo:Accept 실패 처리.  
//Console.WriteLine("Failed to accept client.");  
}  
  
// 다음 연결을 받아들인다.  
this.flow_control_event.Set();  
}  
  
//int connected_count = 0;  
//void on_new_client(Socket client_socket)  
//{  
// //Interlocked.Increment(ref this.connected_count);  
  
// //Console.WriteLine(string.Format("[{0}] A client connected. handle {1}, count {2}",  
// // Thread.CurrentThread.ManagedThreadId, client_socket.Handle,  
// // this.connected_count));  
// //return;  
  
// // 연결이 성립되면 패킷 헤더 읽을 준비를 한다.  
// //StateObject state = new StateObject(client_socket);  
// //state.remain_size_to_read = CPacket.HEADER_SIZE;  
// //client_socket.BeginReceive(state.buffer, 0, CPacket.HEADER_SIZE, 0,  
// // new AsyncCallback(on_recv_header), state);  
//}  
  
//void on_recv_header(IAsyncResult iar)  
//{  
//StateObject state = (StateObject)iar.AsyncState;  
  
//int read_size = state.workSocket.EndReceive(iar);  
  
//// 헤더가 짤려서 올 경우 못읽은 만큼 다시 읽어온다.  
//if (state.remain_size_to_read > read_size)  
//{  
// state.remain_size_to_read -= read_size;  
// state.workSocket.BeginReceive(state.buffer, read_size, state.remain_size_to_read, SocketFlags.None,  
// new AsyncCallback(on_recv_header), state);  
// return;  
//}  
  
////Console.WriteLine("read size " + remain_size);  
//if (read_size <= 0)  
//{  
// Console.WriteLine(string.Format("[{0}] [on_disconnect] client socket {1}",  
// System.Threading.Thread.CurrentThread.ManagedThreadId,  
// state.workSocket.Handle));  
// state.workSocket.Close();  
// return;  
//}  
//else  
//{  
// // 바디 사이즈 파싱.  
// Int16 body_size = BitConverter.ToInt16(state.buffer, 0);  
  
// if (body_size <= 0 || body_size > 10240)  
// {  
// state.workSocket.Close();  
// return;  
// }  
  
// // 프로토콜 id 파싱.  
// short protocol_id = BitConverter.ToInt16(state.buffer, 2);  
// state.set_protocol(protocol_id);  
  
// state.body_size = body_size;  
// state.remain_size_to_read = state.body_size;  
  
// // 바디 읽기.  
// Array.Clear(state.buffer, 0, state.buffer.Length);  
// state.workSocket.BeginReceive(state.buffer, 0, body_size, 0,  
// new AsyncCallback(on_recv), state);  
//}  
//}  
  
//void on_recv(IAsyncResult iar)  
//{  
//StateObject state = (StateObject)iar.AsyncState;  
  
//int read_size = state.workSocket.EndReceive(iar);  
//if (state.remain_size_to_read > read_size)  
//{  
// state.remain_size_to_read -= read_size;  
// state.workSocket.BeginReceive(state.buffer, read_size, state.remain_size_to_read, SocketFlags.None,  
// new AsyncCallback(on_recv), state);  
// return;  
//}  
  
//if (read_size <= 0)  
//{  
// Console.WriteLine(string.Format("[{0}] [on_disconnect] client socket {1}",  
// System.Threading.Thread.CurrentThread.ManagedThreadId,  
// state.workSocket.Handle));  
// state.workSocket.Close();  
//}  
//else  
//{  
// try  
// {  
// Interlocked.Increment(ref recv_body_count);  
  
// // 하나의 패킷 길이만큼만 잘라서 peer에 전달.  
// byte[] clone_buffer = new byte[state.body_size];  
// System.Buffer.BlockCopy(state.buffer, 0, clone_buffer, 0, state.body_size);  
  
// // Peer객체에 전달하고 다음 메시지를 받을 수 있도록 다시 헤더를 읽을 준비를 한다.  
// CPacket msg = new CPacket(state.protocol_id, clone_buffer);  
// state.peer.on_recv(msg);  
  
// // 이 버퍼는 다음 패킷을 받을 때 사용할 것이기 때문에 깔끔하게 클리어 해준다.  
// Array.Clear(state.buffer, 0, state.buffer.Length);  
  
// // 헤더 읽기.  
// state.remain_size_to_read = CPacket.HEADER_SIZE;  
// state.workSocket.BeginReceive(state.buffer, 0, CPacket.HEADER_SIZE, 0,  
// new AsyncCallback(on_recv_header), state);  
// }  
// catch (Exception e)  
// {  
// Console.WriteLine(e.Message + "\n" + e.StackTrace);  
  
// //byte[] clone_buffer = new byte[state.body_size];  
// //System.Buffer.BlockCopy(state.buffer, 0, clone_buffer, 0, state.body_size);  
  
// //CPacket msg = new CPacket(clone_buffer);  
// //Int32 data = msg.pop_int32();  
  
// //Console.WriteLine("int32 " + data + ", buffer : " + clone_buffer.Length);  
// }  
//}  
//}  
}  
}
```

```C#
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
  
namespace FreeNet  
{  
class Defines  
{  
public static readonly short HEADERSIZE = 2;  
}  
  
/// <summary>  
/// [header][body] 구조를 갖는 데이터를 파싱하는 클래스.  
/// - header : 데이터 사이즈. Defines.HEADERSIZE에 정의된 타입만큼의 크기를 갖는다.  
/// 2바이트일 경우 Int16, 4바이트는 Int32로 처리하면 된다.  
/// 본문의 크기가 Int16.Max값을 넘지 않는다면 2바이트로 처리하는것이 좋을것 같다.  
/// - body : 메시지 본문.  
/// </summary>  
class CMessageResolver  
{  
public delegate void CompletedMessageCallback(Const<byte[]> buffer);  
  
// 메시지 사이즈.  
int message_size;  
  
// 진행중인 버퍼.  
byte[] message_buffer = new byte[1024];  
  
// 현재 진행중인 버퍼의 인덱스를 가리키는 변수.  
// 패킷 하나를 완성한 뒤에는 0으로 초기화 시켜줘야 한다.  
int current_position;  
  
// 읽어와야 할 목표 위치.  
int position_to_read;  
  
// 남은 사이즈.  
int remain_bytes;  
  
public CMessageResolver()  
{  
this.message_size = 0;  
this.current_position = 0;  
this.position_to_read = 0;  
this.remain_bytes = 0;  
}  
  
/// <summary>  
/// 목표지점으로 설정된 위치까지의 바이트를 원본 버퍼로부터 복사한다.  
/// 데이터가 모자랄 경우 현재 남은 바이트 까지만 복사한다.  
/// </summary>  
/// <param name="buffer"></param>  
/// <param name="offset"></param>  
/// <param name="transffered"></param>  
/// <param name="size_to_read"></param>  
/// <returns>다 읽었으면 true, 데이터가 모자라서 못 읽었으면 false를 리턴한다.</returns>  
bool read_until(byte[] buffer, ref int src_position, int offset, int transffered)  
{  
if (this.current_position >= offset + transffered)  
{  
// 들어온 데이터 만큼 다 읽은 상태이므로 더이상 읽을 데이터가 없다.  
return false;  
}  
  
// 읽어와야 할 바이트.  
// 데이터가 분리되어 올 경우 이전에 읽어놓은 값을 빼줘서 부족한 만큼 읽어올 수 있도록 계산해 준다.  
int copy_size = this.position_to_read - this.current_position;  
  
// 앗! 남은 데이터가 더 적다면 가능한 만큼만 복사한다.  
if (this.remain_bytes < copy_size)  
{  
copy_size = this.remain_bytes;  
}  
  
// 버퍼에 복사.  
Array.Copy(buffer, src_position, this.message_buffer, this.current_position, copy_size);  
  
  
// 원본 버퍼 포지션 이동.  
src_position += copy_size;  
  
// 타겟 버퍼 포지션도 이동.  
this.current_position += copy_size;  
  
// 남은 바이트 수.  
this.remain_bytes -= copy_size;  
  
// 목표지점에 도달 못했으면 falseif (this.current_position < this.position_to_read)  
{  
return false;  
}  
  
return true;  
}  
  
/// <summary>  
/// 소켓 버퍼로부터 데이터를 수신할 때 마다 호출된다.  
/// 데이터가 남아 있을 때 까지 계속 패킷을 만들어 callback을 호출 해 준다.  
/// 하나의 패킷을 완성하지 못했다면 버퍼에 보관해 놓은 뒤 다음 수신을 기다린다.  
/// </summary>  
/// <param name="buffer"></param>  
/// <param name="offset"></param>  
/// <param name="transffered"></param>  
public void on_receive(byte[] buffer, int offset, int transffered, CompletedMessageCallback callback)  
{  
// 이번 receive로 읽어오게 될 바이트 수.  
this.remain_bytes = transffered;  
  
// 원본 버퍼의 포지션값.  
// 패킷이 여러개 뭉쳐 올 경우 원본 버퍼의 포지션은 계속 앞으로 가야 하는데 그 처리를 위한 변수이다.  
int src_position = offset;  
  
// 남은 데이터가 있다면 계속 반복한다.  
while (this.remain_bytes > 0)  
{  
bool completed = false;  
  
// 헤더만큼 못읽은 경우 헤더를 먼저 읽는다.  
if (this.current_position < Defines.HEADERSIZE)  
{  
// 목표 지점 설정(헤더 위치까지 도달하도록 설정).  
this.position_to_read = Defines.HEADERSIZE;  
  
completed = read_until(buffer, ref src_position, offset, transffered);  
if (!completed)  
{  
// 아직 다 못읽었으므로 다음 receive를 기다린다.  
return;  
}  
  
// 헤더 하나를 온전히 읽어왔으므로 메시지 사이즈를 구한다.  
this.message_size = get_body_size();  
  
// 다음 목표 지점(헤더 + 메시지 사이즈).  
this.position_to_read = this.message_size + Defines.HEADERSIZE;  
}  
  
// 메시지를 읽는다.  
completed = read_until(buffer, ref src_position, offset, transffered);  
  
if (completed)  
{  
// 패킷 하나를 완성 했다.  
callback(new Const<byte[]>(this.message_buffer));  
  
clear_buffer();  
}  
}  
}  
  
int get_body_size()  
{  
// 헤더 타입의 바이트만큼을 읽어와 메시지 사이즈를 리턴한다.  
  
Type type = Defines.HEADERSIZE.GetType();  
if (type.Equals(typeof(Int16)))  
{  
return BitConverter.ToInt16(this.message_buffer, 0);  
}  
  
return BitConverter.ToInt32(this.message_buffer, 0);  
}  
  
void clear_buffer()  
{  
Array.Clear(this.message_buffer, 0, this.message_buffer.Length);  
  
this.current_position = 0;  
this.message_size = 0;  
  
}  
}  
}
```

```C#
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
using System.Threading;  
using System.Net;  
using System.Net.Sockets;  
  
namespace FreeNet  
{  
public class CNetworkService  
{  
int connected_count;  
CListener client_listener;  
SocketAsyncEventArgsPool receive_event_args_pool;  
SocketAsyncEventArgsPool send_event_args_pool;  
BufferManager buffer_manager;  
  
public delegate void SessionHandler(CUserToken token);  
public SessionHandler session_created_callback { get; set; }  
  
// configs.  
int max_connections;  
int buffer_size;  
readonly int pre_alloc_count = 2; // read, write  
  
public CNetworkService()  
{  
this.connected_count = 0;  
this.session_created_callback = null;  
}  
  
// Initializes the server by preallocating reusable buffers and  
// context objects. These objects do not need to be preallocated  
// or reused, but it is done this way to illustrate how the API can  
// easily be used to create reusable objects to increase server performance.  
//  
public void initialize()  
{  
this.max_connections = 10000;  
this.buffer_size = 1024;  
  
this.buffer_manager = new BufferManager(this.max_connections * this.buffer_size * this.pre_alloc_count, this.buffer_size);  
this.receive_event_args_pool = new SocketAsyncEventArgsPool(this.max_connections);  
this.send_event_args_pool = new SocketAsyncEventArgsPool(this.max_connections);  
  
// Allocates one large byte buffer which all I/O operations use a piece of. This gaurds  
// against memory fragmentation  
this.buffer_manager.InitBuffer();  
  
// preallocate pool of SocketAsyncEventArgs objects  
SocketAsyncEventArgs arg;  
  
for (int i = 0; i < this.max_connections; i++)  
{  
// 동일한 소켓에 대고 send, receive를 하므로  
// user token은 세션별로 하나씩만 만들어 놓고  
// receive, send EventArgs에서 동일한 token을 참조하도록 구성한다.  
CUserToken token = new CUserToken();  
  
// receive pool  
{  
//Pre-allocate a set of reusable SocketAsyncEventArgs  
arg = new SocketAsyncEventArgs();  
arg.Completed += new EventHandler<SocketAsyncEventArgs>(receive_completed);  
arg.UserToken = token;  
  
// assign a byte buffer from the buffer pool to the SocketAsyncEventArg object  
this.buffer_manager.SetBuffer(arg);  
  
// add SocketAsyncEventArg to the pool  
this.receive_event_args_pool.Push(arg);  
}  
  
  
// send pool  
{  
//Pre-allocate a set of reusable SocketAsyncEventArgs  
arg = new SocketAsyncEventArgs();  
arg.Completed += new EventHandler<SocketAsyncEventArgs>(send_completed);  
arg.UserToken = token;  
  
// assign a byte buffer from the buffer pool to the SocketAsyncEventArg object  
this.buffer_manager.SetBuffer(arg);  
  
// add SocketAsyncEventArg to the pool  
this.send_event_args_pool.Push(arg);  
}  
}  
}  
  
public void listen(string host, int port, int backlog)  
{  
this.client_listener = new CListener();  
this.client_listener.callback_on_newclient += on_new_client;  
this.client_listener.start(host, port, backlog);  
}  
  
/// <summary>  
/// todo:검토중...  
/// 원격 서버에 접속 성공 했을 때 호출됩니다.  
/// </summary>  
/// <param name="socket"></param>  
public void on_connect_completed(Socket socket, CUserToken token)  
{  
// SocketAsyncEventArgsPool에서 빼오지 않고 그때 그때 할당해서 사용한다.  
// 풀은 서버에서 클라이언트와의 통신용으로만 쓰려고 만든것이기 때문이다.  
// 클라이언트 입장에서 서버와 통신을 할 때는 접속한 서버당 두개의 EventArgs만 있으면 되기 때문에 그냥 new해서 쓴다.  
// 서버간 연결에서도 마찬가지이다.  
// 풀링처리를 하려면 c->s로 가는 별도의 풀을 만들어서 써야 한다.  
SocketAsyncEventArgs receive_event_arg = new SocketAsyncEventArgs();  
receive_event_arg.Completed += new EventHandler<SocketAsyncEventArgs>(receive_completed);  
receive_event_arg.UserToken = token;  
receive_event_arg.SetBuffer(new byte[1024], 0, 1024);  
  
SocketAsyncEventArgs send_event_arg = new SocketAsyncEventArgs();  
send_event_arg.Completed += new EventHandler<SocketAsyncEventArgs>(send_completed);  
send_event_arg.UserToken = token;  
send_event_arg.SetBuffer(new byte[1024], 0, 1024);  
  
begin_receive(socket, receive_event_arg, send_event_arg);  
}  
  
/// <summary>  
/// 새로운 클라이언트가 접속 성공 했을 때 호출됩니다.  
/// AcceptAsync의 콜백 매소드에서 호출되며 여러 스레드에서 동시에 호출될 수 있기 때문에 공유자원에 접근할 때는 주의해야 합니다.  
/// </summary>  
/// <param name="client_socket"></param>  
void on_new_client(Socket client_socket, object token)  
{  
//todo:  
// peer list처리.  
  
Interlocked.Increment(ref this.connected_count);  
  
Console.WriteLine(string.Format("[{0}] A client connected. handle {1}, count {2}",  
Thread.CurrentThread.ManagedThreadId, client_socket.Handle,  
this.connected_count));  
  
// 플에서 하나 꺼내와 사용한다.  
SocketAsyncEventArgs receive_args = this.receive_event_args_pool.Pop();  
SocketAsyncEventArgs send_args = this.send_event_args_pool.Pop();  
  
CUserToken user_token = null;  
if (this.session_created_callback != null)  
{  
user_token = receive_args.UserToken as CUserToken;  
this.session_created_callback(user_token);  
}  
  
begin_receive(client_socket, receive_args, send_args);  
//user_token.start_keepalive();  
}  
  
void begin_receive(Socket socket, SocketAsyncEventArgs receive_args, SocketAsyncEventArgs send_args)  
{  
// receive_args, send_args 아무곳에서나 꺼내와도 된다. 둘다 동일한 CUserToken을 물고 있다.  
CUserToken token = receive_args.UserToken as CUserToken;  
token.set_event_args(receive_args, send_args);  
// 생성된 클라이언트 소켓을 보관해 놓고 통신할 때 사용한다.  
token.socket = socket;  
  
bool pending = socket.ReceiveAsync(receive_args);  
if (!pending)  
{  
process_receive(receive_args);  
}  
}  
  
// This method is called whenever a receive or send operation is completed on a socket  
//  
// <param name="e">SocketAsyncEventArg associated with the completed receive operation</param>  
void receive_completed(object sender, SocketAsyncEventArgs e)  
{  
if (e.LastOperation == SocketAsyncOperation.Receive)  
{  
process_receive(e);  
return;  
}  
  
throw new ArgumentException("The last operation completed on the socket was not a receive.");  
}  
  
// This method is called whenever a receive or send operation is completed on a socket  
//  
// <param name="e">SocketAsyncEventArg associated with the completed send operation</param>  
void send_completed(object sender, SocketAsyncEventArgs e)  
{  
CUserToken token = e.UserToken as CUserToken;  
token.process_send(e);  
}  
  
// This method is invoked when an asynchronous receive operation completes.  
// If the remote host closed the connection, then the socket is closed.  
//  
private void process_receive(SocketAsyncEventArgs e)  
{  
// check if the remote host closed the connection  
CUserToken token = e.UserToken as CUserToken;  
if (e.BytesTransferred > 0 && e.SocketError == SocketError.Success)  
{  
token.on_receive(e.Buffer, e.Offset, e.BytesTransferred);  
  
bool pending = token.socket.ReceiveAsync(e);  
if (!pending)  
{  
// Oh! stack overflow??  
process_receive(e);  
}  
}  
else  
{  
Console.WriteLine(string.Format("error {0}, transferred {1}", e.SocketError, e.BytesTransferred));  
close_clientsocket(token);  
}  
}  
  
public void close_clientsocket(CUserToken token)  
{  
token.on_removed();  
  
// Free the SocketAsyncEventArg so they can be reused by another client  
// 버퍼는 반환할 필요가 없다. SocketAsyncEventArg가 버퍼를 물고 있기 때문에  
// 이것을 재사용 할 때 물고 있는 버퍼를 그대로 사용하면 되기 때문이다.  
if (this.receive_event_args_pool != null)  
{  
this.receive_event_args_pool.Push(token.receive_event_args);  
}  
  
if (this.send_event_args_pool != null)  
{  
this.send_event_args_pool.Push(token.send_event_args);  
}  
}  
}  
}
```

```C#
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
  
namespace FreeNet  
{  
public struct Const<T>  
{  
public T Value { get; private set; }  
  
public Const(T value)  
: this()  
{  
this.Value = value;  
}  
}  
}
```

```C#
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
  
namespace FreeNet  
{  
/// <summary>  
/// byte[] 버퍼를 참조로 보관하여 pop_xxx 매소드 호출 순서대로 데이터 변환을 수행한다.  
/// </summary>  
public class CPacket  
{  
public IPeer owner { get; private set; }  
public byte[] buffer { get; private set; }  
public int position { get; private set; }  
  
public Int16 protocol_id { get; private set; }  
  
public static CPacket create(Int16 protocol_id)  
{  
//CPacket packet = new CPacket();  
CPacket packet = CPacketBufferManager.pop();  
packet.set_protocol(protocol_id);  
return packet;  
}  
  
public static void destroy(CPacket packet)  
{  
CPacketBufferManager.push(packet);  
}  
  
public CPacket(byte[] buffer, IPeer owner)  
{  
// 참조로만 보관하여 작업한다.  
// 복사가 필요하면 별도로 구현해야 한다.  
this.buffer = buffer;  
  
// 헤더는 읽을필요 없으니 그 이후부터 시작한다.  
this.position = Defines.HEADERSIZE;  
  
this.owner = owner;  
}  
  
public CPacket()  
{  
this.buffer = new byte[1024];  
}  
  
public Int16 pop_protocol_id()  
{  
return pop_int16();  
}  
  
public void copy_to(CPacket target)  
{  
target.set_protocol(this.protocol_id);  
target.overwrite(this.buffer, this.position);  
}  
  
public void overwrite(byte[] source, int position)  
{  
Array.Copy(source, this.buffer, source.Length);  
this.position = position;  
}  
  
public byte pop_byte()  
{  
byte data = (byte)BitConverter.ToInt16(this.buffer, this.position);  
this.position += sizeof(byte);  
return data;  
}  
  
public Int16 pop_int16()  
{  
Int16 data = BitConverter.ToInt16(this.buffer, this.position);  
this.position += sizeof(Int16);  
return data;  
}  
  
public Int32 pop_int32()  
{  
Int32 data = BitConverter.ToInt32(this.buffer, this.position);  
this.position += sizeof(Int32);  
return data;  
}  
  
public string pop_string()  
{  
// 문자열 길이는 최대 2바이트 까지. 0 ~ 32767  
Int16 len = BitConverter.ToInt16(this.buffer, this.position);  
this.position += sizeof(Int16);  
  
// 인코딩은 utf8로 통일한다.  
string data = System.Text.Encoding.UTF8.GetString(this.buffer, this.position, len);  
this.position += len;  
  
return data;  
}  
  
  
  
public void set_protocol(Int16 protocol_id)  
{  
this.protocol_id = protocol_id;  
//this.buffer = new byte[1024];  
  
// 헤더는 나중에 넣을것이므로 데이터 부터 넣을 수 있도록 위치를 점프시켜놓는다.  
this.position = Defines.HEADERSIZE;  
  
push_int16(protocol_id);  
}  
  
public void record_size()  
{  
Int16 body_size = (Int16)(this.position - Defines.HEADERSIZE);  
byte[] header = BitConverter.GetBytes(body_size);  
header.CopyTo(this.buffer, 0);  
}  
  
public void push_int16(Int16 data)  
{  
byte[] temp_buffer = BitConverter.GetBytes(data);  
temp_buffer.CopyTo(this.buffer, this.position);  
this.position += temp_buffer.Length;  
}  
  
public void push(byte data)  
{  
byte[] temp_buffer = BitConverter.GetBytes(data);  
temp_buffer.CopyTo(this.buffer, this.position);  
this.position += sizeof(byte);  
}  
  
public void push(Int16 data)  
{  
byte[] temp_buffer = BitConverter.GetBytes(data);  
temp_buffer.CopyTo(this.buffer, this.position);  
this.position += temp_buffer.Length;  
}  
  
public void push(Int32 data)  
{  
byte[] temp_buffer = BitConverter.GetBytes(data);  
temp_buffer.CopyTo(this.buffer, this.position);  
this.position += temp_buffer.Length;  
}  
  
public void push(string data)  
{  
byte[] temp_buffer = Encoding.UTF8.GetBytes(data);  
  
Int16 len = (Int16)temp_buffer.Length;  
byte[] len_buffer = BitConverter.GetBytes(len);  
len_buffer.CopyTo(this.buffer, this.position);  
this.position += sizeof(Int16);  
  
temp_buffer.CopyTo(this.buffer, this.position);  
this.position += temp_buffer.Length;  
}  
  
public void push(float data)  
{  
byte[] temp_buffer = BitConverter.GetBytes(data);  
temp_buffer.CopyTo(this.buffer, this.position);  
this.position += temp_buffer.Length;  
}  
  
public float pop_Float()  
{  
float data = BitConverter.ToSingle(this.buffer, this.position);  
this.position += sizeof(float);  
return data;  
}  
}  
}
```

```C#
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
  
namespace FreeNet  
{  
public class CPacketBufferManager  
{  
static object cs_buffer = new object();  
static Stack<CPacket> pool;  
static int pool_capacity;  
  
public static void initialize(int capacity)  
{  
pool = new Stack<CPacket>();  
pool_capacity = capacity;  
allocate();  
}  
  
static void allocate()  
{  
for (int i = 0; i < pool_capacity; ++i)  
{  
pool.Push(new CPacket());  
}  
}  
  
public static CPacket pop()  
{  
lock (cs_buffer)  
{  
if (pool.Count <= 0)  
{  
Console.WriteLine("reallocate.");  
allocate();  
}  
  
return pool.Pop();  
}  
}  
  
public static void push(CPacket packet)  
{  
lock(cs_buffer)  
{  
pool.Push(packet);  
}  
}  
}  
}
```

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

```C#
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
using System.Net.Sockets;  
  
namespace FreeNet  
{  
/// <summary>  
/// 서버와 클라이언트에서 공통으로 사용하는 세션 객체.  
/// 서버일 경우 :/// 하나의 클라이언트 객체를 나타낸다.  
/// 이 인터페이스를 구현한 객체를 CNetworkService클래스의 session_created_callback호출시 생성하여 리턴시켜 준다.  
/// 객체를 풀링할지 여부는 사용자가 원하는대로 구현한다.  
///  
/// 클라이언트일 경우 :/// 접속한 서버 객체를 나타낸다.  
///  
/// 이 클래스의 모든 매소드는 thread unsafe하므로 공유자원에 접근할 때는 동기화처리가 필요하다.  
/// </summary>  
public interface IPeer  
{  
/// <summary>  
/// 소켓 버퍼로부터 데이터를 수신하여 패킷 하나를 완성했을 때 호출 된다.  
/// 호출 흐름 : .Net Socket ReceiveAsync -> CUserToken.on_receive -> CPeer.on_message///  
/// 패킷 순서에 대해서(TCP)  
/// 이 매소드는 .Net Socket의 스레드풀에 의해 작동되어 호출되므로 어느 스레드에서 호출될지 알 수 없다.  
/// 하지만 하나의 CPeer객체에 대해서는 이 매소드가 완료된 이후 다음 패킷이 들어오도록 구현되어 있으므로  
/// 클라이언트가 보낸 패킷 순서는 보장이 된다.  
///  
/// 주의할점  
/// 이 매소드에서 다른 CPeer객체를 참조하거나 공유자원에 접근할 때는 동기화 처리를 해줘야 한다.  
/// 매소드가 리턴되면 buffer는 비워지며 다음 패킷을 담을 준비를 하기 때문에 매소드를 리턴하기 전에 사용할 데이터를 모두 빼내야 한다.  
/// buffer참조를 다른 객체에 넘긴 후 매소드가 리턴된 이후에 사용하게 해서는 안된다. 이런 경우에는 참조대신 복사하여 처리해야 한다.  
/// </summary>  
/// <param name="buffer">  
/// Socket버퍼로부터 복사된 CUserToken의 버퍼를 참조한다.  
/// </param>  
void on_message(Const<byte[]> buffer);  
  
  
/// <summary>  
/// 원격 연결이 끊겼을 때 호출 된다.  
/// 이 매소드가 호출된 이후부터는 데이터 전송이 불가능하다.  
/// </summary>  
void on_removed();  
  
  
void send(CPacket msg);  
  
  
void disconnect();  
  
  
void process_user_operation(CPacket msg);  
}  
}
```

```C#
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
using System.Net;  
  
namespace FreeNet  
{  
}
```

```C#
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
using System.Net.Sockets;  
  
namespace FreeNet  
{  
// Represents a collection of reusable SocketAsyncEventArgs objects.  
class SocketAsyncEventArgsPool  
{  
Stack<SocketAsyncEventArgs> m_pool;  
  
// Initializes the object pool to the specified size  
//  
// The "capacity" parameter is the maximum number of  
// SocketAsyncEventArgs objects the pool can hold  
public SocketAsyncEventArgsPool(int capacity)  
{  
m_pool = new Stack<SocketAsyncEventArgs>(capacity);  
}  
  
// Add a SocketAsyncEventArg instance to the pool  
//  
//The "item" parameter is the SocketAsyncEventArgs instance  
// to add to the pool  
public void Push(SocketAsyncEventArgs item)  
{  
if (item == null) { throw new ArgumentNullException("Items added to a SocketAsyncEventArgsPool cannot be null"); }  
lock (m_pool)  
{  
m_pool.Push(item);  
}  
}  
  
// Removes a SocketAsyncEventArgs instance from the pool  
// and returns the object removed from the pool  
public SocketAsyncEventArgs Pop()  
{  
lock (m_pool)  
{  
return m_pool.Pop();  
}  
}  
  
// The number of SocketAsyncEventArgs instances in the pool  
public int Count  
{  
get { return m_pool.Count; }  
}  
}  
}
```
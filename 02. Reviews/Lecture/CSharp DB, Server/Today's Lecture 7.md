

2023-05-11

----
#Server #수업 #CSharp 

## 개요
어제까지 우리는 다음을 배웠다.
`CPacketBufferManager`에서 패킷을 저장하고 `pop()`을 통해 패킷을 꺼내어서 쓸 수 있게 한다.
이 안에는 `pool`이 있는데, 여기서 패킷을 담아 두었다가 필요하면 꺼내고 다 쓰면 돌려주는 구조이다.

`CNetworkService`는 세션이 연결되면 `on_session_created`를 생성해 준다.
최대 클라이언트의 개수를 정해주고, 그 클라이언트가 데이터를 주고 받을 때 정보의 양, 즉 `buffer`의 크기를 1024로 정해 주었다.
그리고 한 클라이언트 당 `send`와 `receive`를 위한 버퍼가 각각 하나 필요하기에 두개씩 할당해 줬다.

`SocketAsyncEventArgsPool` 클래스에서는 비동기 호출을 위해 버퍼를 관리해 준다.

다시 `CNetworkManager`에서는, `SocketAsyncEventArgs` 클래스를 인스턴스화하여 닷넷에 전달해 준다.
통신이 완료됐을 때 `receive_complete` 메서드를 호출해 주는 코드는 다음과 같다.
(`send_completed`도 마찬가지)
```C#
arg.Completed += new EventHandler<SocketAsyncEventArgs>(receive_completed);
```

그리고 이 메서드는 누구에 대해 비동기가 걸려 있는지 정보를 담고 있는 `token`객체를 가지고 있어야 한다.
```C#
arg.UserToken = token;
```

`SetBuffer()`는 실제 데이터가 담기게 될 버퍼의 시작 지점과 버퍼의 크기를 지정해 주는 역할을 하는 메서드이다.

## 내용
`CListener`에서는 `on_new_client`를 호출하고 `start()` 메서드를 통해 연결을 시작한다.
실제로 물리적인 통신에 이르기까지 기기 내에서 프레임워크를 거치는 과정 중에, 어플리케이션과 'Transport Layer'를 연결해주는 게 소켓이다.

소켓에는 (클라이언트가) 정보를 어디로 보낼지 정해야 한다.
```C#
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
```

우리가 보는 사이트의 서버 주소를 알 수 있는 방법이 있다.
![[스크린샷 2023-05-11 오전 10.35.31.png]]

여기 나온 `216.58.200.227`가 바로 서버의 ip 주소이다.

ip를 통해 다른 호스트의 주소(`address`)를 알아낸 뒤에, 그 호스트 내에서 정확히 어떤 프로그램으로 통신을 보낼 것인지 정해주는 `port` 값을 정해준다.
우리가 앞서 지정해 준 포트 값은 임의로 지정한 것이었다.

다음과 같이 신호가 꺼져 있는 상태의 객체를 만든다.
```C#
this.flow_control_event = new AutoResetEvent(false);
```

그리고 만약 비동기 호출의 요청을 진행 중일 때에 클라이언트가 접속을 하면, 서버에서는 닷넷의 비동기를 거치지 않고 바로 연결한다.
```C#
pending = listen_socket.AcceptAsync(this.accept_args);
```

이 `pending` 변수는,
참일 경우 `ReceiveAsync`를 호출하는 중에 있다는 것임.
거짓일 경우 닷넷에 비동기 수신 등록 중이라는 것임.

`WaitOne()` 메서드를 통해서, 신호를 받은 상태에서 스레드를 멈춰 있게 한다.
만약 이렇게 설계하지 않으면, 신호를 받고 있지도 않은데 계속 스레드가 닷넷에 정보를 주고 있는 상태가 되어서 비효율이 발생한다.
```C#
this.flow_control_event.WaitOne();
```

`Set()` 메서드를 통해서는 다시 작동하게 한다.
```C#
this.flow_control_event.Set();
```

`e.SocketError == SocketError.Success`가 참이면 연결에 성공했다는 뜻이다.

`client_socket`는 ???
`send_event_args`???
`receive_event_args`???

다음 메서드의 객체 `e`를 통해, 수신이 되었을 때 닷넷에 의해 처리된 결과 값이 전달된다.
```C#
void receive_completed(object sender, SocketAsyncEventArgs e)  
{  
	if (e.LastOperation == SocketAsyncOperation.Receive)  
	{  
		process_receive(e);  
		return;  
	}  
	  
	throw new ArgumentException("The last operation completed on the socket was not a receive.");  
}
```

그리고 `process_receive()` 메서드를 보자.
```C#
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
```

`e.BytesTransferred > 0 && e.SocketError == SocketError.Success`를 통해 0 바이트 이상의 데이터가 전송되었으며, `e.SocketError`가 성공이 떠야 연결이 성공한 것이다. 
만약 여기에 해당되지 않는 경우 연결이 종료된 상황인 것이다.

만약 해당이 된다면, `token`에 있는 `on_receive()` 메서드를 호출해 준다.

`CUserToken` 클래스를 보자.
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
이 메서드는 무슨 역할을 하는가?
TCP는 정확한 전송을 위해서 흐름 제어를 한다. 
이는 통신망의 상황에 따라 패킷의 크기를 유동적으로 송수신하는 것이다. 
그러므로 애플리케이션 계층에서 보낸 패킷이 상황에 따라 분리되거나 합쳐져서 가게 된다. 
그러므로 정확한 처리를 위해서는 프로토콜 설계에 따른 패킷의 구분이 필요하다. 
즉, 프로토콜 설계에 따라 정확하게 패킷을 만들어주는 역할을 하는 것임.

우리가 예전에 윈폼을 통해서 구현했던 서버는, 개행 문자를 통해 패킷을 분할하였고, 이 개행 문자가 있는지를 서버가 한 바이트씩 검사했다.
이런 비효율적인 방식은 클라이언트가 사용할 때는 그렇게 큰 문제가 되지 않지만, 서버의 경우 부하가 걸리기 때문에 좋은 방법이 아니다.

우리가 설계한 프로토콜은 다음과 같다.

| Header                     | Body(명령 - Int16 2byte)                 | Body(명령에 따른 데이터)                 |
| -------------------------- | ---------------------------------------- | ---------------------------------------- |
| Body Size | ........................................ | ..................................... |

큐에 무언가가 들어 있다면 아직 이전 전송이 완료되지 않은 상태이므로 큐에 추가만 하고 리턴한다.
현재 수행중인 `SendAsymc`가 완료된 이후에 큐를 검사하여 데이터가 있으면 `SendAsync`를 호출하여 전송한다.
(여기서, `queue`의 자료구조에서...
집어넣는 과정을 `Enqueue`라고 하고, 
꺼내는 과정을 `Dequeue`라고 한다.
반면 `stack`은...
집어넣는 과정을 `Push`라고 하고,
꺼내는 과정을 `Pop`이라 한다.
반면 여기엔 `Peek`라는 용어도 나온다.
이는 데이터를 꺼내지 않고 데이터를 확인만 하는 것을 일컫는다.)

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

이 `CMessageResolver`는 수신한 데이터를 온전한 하나의 패킷으로 만들어주는 역할을 한다.
`position_to_read`는 헤더의 마치는 지점까지의 크기를 말한다.
패킷을 완성할 때까지 이 객체의 역할이 끝나고, 그 뒤의 영역을 읽는 과정에선 `CPacket`이 작동하게 된다.

`CPacket`에선 프로토콜에서 미리 지정한 `enum`형을 받아서 반환한다.
헤더 뒤에 따라오는 2바이트의 `short`형 정보는 이 프로토콜을 해석하는 방법을 알려주는 것이다.
헤더는 데이터의 길이를 알려주는데, 이 2바이트의 길이를 뺀 값, 즉 `record_size()`에서 도출된 값을 가져가는 것이다.
이 `record_size()` 함수는 헤더에 이 길이를 저장한다.

`CUserToken`에서는, 다음 조건문을 통해 정상적인 전송인지 여부를 판별한다.
```C#
if (e.BytesTransferred <= 0 || e.SocketError != SocketError.Success)
```

데이터가 0이하이면 전송이 정상적이지 않다는 뜻이다.

데이터가 전송이 된 후에는, 아직도 큐에 데이터가 남아 있다.
다음은 전송된 데이터의 크기를 나타낸다.
```C#
int size = this.sending_queue.Peek().position;
```

그리고 `e.ByteTransffered`는 실제 전송된 데이터의 크기를 말한다.
`sent_count`를 통해 전송되었다는 정보를 남긴다.


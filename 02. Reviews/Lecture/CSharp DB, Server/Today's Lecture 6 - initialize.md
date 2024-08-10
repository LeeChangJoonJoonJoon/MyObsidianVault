

2023-05-10

----
#Server #initialize 

## 개요
오늘은 지금까지 만든 코드를 리뷰해 보는 시간을 가질 것이다.
다음 코드를 보자
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

여기 코드에서는 `BufferManager`가 객체화 되어서 쓰이고 있다.
그래서 `BufferManager.cs`로 가 보면...
```C#
using System;  
using System.Collections.Generic;  
using System.Text;  
using System.Net.Sockets;  
using System.Threading;  
  
namespace FreeNet  
{  
/// <summary>  
/// This class creates a single large buffer which can be divided up and assigned to SocketAsyncEventArgs objects for use  
/// with each socket I/O operation. This enables bufffers to be easily reused and gaurds against fragmenting heap memory.  
///  
/// The operations exposed on the BufferManager class are not thread safe.  
/// </summary>  
internal class BufferManager  
{  
  
int m_numBytes; // the total number of bytes controlled by the buffer pool  
byte[] m_buffer; // the underlying byte array maintained by the Buffer Manager  
Stack<int> m_freeIndexPool; //  
int m_currentIndex;  
int m_bufferSize;  
  
public BufferManager(int totalBytes, int bufferSize)  
{  
m_numBytes = totalBytes;  
m_currentIndex = 0;  
m_bufferSize = bufferSize;  
m_freeIndexPool = new Stack<int>();  
}  
  
/// <summary>  
/// Allocates buffer space used by the buffer pool  
/// </summary>  
public void InitBuffer()  
{  
// create one big large buffer and divide that out to each SocketAsyncEventArg object  
m_buffer = new byte[m_numBytes];  
}  
  
/// <summary>  
/// Assigns a buffer from the buffer pool to the specified SocketAsyncEventArgs object  
/// </summary>  
/// <returns>true if the buffer was successfully set, else false</returns>  
public bool SetBuffer(SocketAsyncEventArgs args)  
{  
if (m_freeIndexPool.Count > 0)  
{  
args.SetBuffer(m_buffer, m_freeIndexPool.Pop(), m_bufferSize);  
}  
else  
{  
if ((m_numBytes - m_bufferSize) < m_currentIndex)  
{  
return false;  
}  
args.SetBuffer(m_buffer, m_currentIndex, m_bufferSize);  
m_currentIndex += m_bufferSize;  
}  
return true;  
}  
  
/// <summary>  
/// Removes the buffer from a SocketAsyncEventArg object. This frees the buffer back to the  
/// buffer pool  
/// </summary>  
public void FreeBuffer(SocketAsyncEventArgs args)  
{  
m_freeIndexPool.Push(args.Offset);  
args.SetBuffer(null, 0, 0);  
}  
}  
}
```

## 내용
먼저 `CPakcetBufferManager`를 살펴보자.
```C#
static Stack<CPacket> pool;  
```

이렇게 선언된 변수가 있다.
`Stack`은, 가장 나중에 저장된 것부터 꺼내올 수 있는 제너릭 자료형이다.
`Queue`와 반대되는 순서를 가진다.

여기에 패킷을 담는 데에 사용하다.
다만 여기의 경우는 스택을 쓰든 큐를 쓰든 무방하다.
`pool`이 바로 이 스택으로 만들어진다.

다음과 같이 서버에서 패킷 객체가 담긴 `pool`을 만들어서 쓴다.
이 경우에는 `pool_capacity`를 2000개로 정했다.
```C#
static void allocate()  
{  
	for (int i = 0; i < pool_capacity; ++i)  
	{  
		pool.Push(new CPacket());  
	}  
}  
```

이 `pool`의 용도는, 항상 새로운 객체를 생성하고 사용이 끝나면 파괴하는 것은 성능이 낮아지는 결과를 가져오기에, 객체를 일정 수만큼 생성하여 재사용하고 다시 `pool`에 담는 과정을 통해 최적화를 하기 위한 것이다.

다음은 
```C#
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
```

`cs_buffer`를 명시하여 하나의 스레드만 접근할 수 있도록 하고,
`return pool.Pop();`을 통해 `pool`에서 객체를 꺼내어서 쓰는데,
`pool.Count <= 0`인 상황, 즉 접속이 많아서 2000개의 패킷 객체를 모두 소진했을 때 `if`문에 진입한다.

다음 메서드를 통해 다 사용한 `CPacket` 객체를 다시 `pool`에 반납한다.
```C#
public static void push(CPacket packet)  
{  
	lock(cs_buffer)  
	{  
		pool.Push(packet);  
	}  
}  
```

다음은 `CNetworkService`를 다룰 차례.
```C#
this.client_listener.callback_on_newclient += on_new_client;
```

여기서 `delegate`를 이용하여 접속이 될 때 `on_new_client` 메서드를 추가한다.
`on_connect_completed` 메서드는 `token`이라는 인수를 받아 오는데, 이는 송수신자의 연결에 대한 정보가 담겨 있는 변수이다.
즉, 누구와 지금 송수신을 하고 있는지에 대한 정보를 담고 있는 것이다.

최대 접속 수를 10000개로 잡아 줬다.
```C#
this.max_connections = 10000;
```

이제 `buffer`에 대해 생각해 보자.
버퍼란, 통신에 사용할 공간을 의미한다.

여기선 `BufferManager`라는 클래스 내에서 버퍼가 선언된다.
여기서 할당한 버퍼의 `byte[]`의 크기는 10000 * 1024 * 2(주고/받기 때문에)이다.
이 전체 크기를 쪼개어서 송수신을 위한 공간으로 사용한다.

이 쪼개진 것을 꺼내서 쓸 때에는 비동기 방식을 쓰기 위해 다음 두 객체를 사용한다.
```C#
this.receive_event_args_pool = new SocketAsyncEventArgsPool(this.max_connections);  
this.send_event_args_pool = new SocketAsyncEventArgsPool(this.max_connections);
```

전자는 수신에 관계된 이벤트 풀이고 후자는 송신에 관계된 이벤트 풀이다.
비동기 방식은, 이벤트가 생기는지 고지해 주는 객체가 필요하기 때문에 위와 같이 만들어 준 것이다.

`CNetworkService`에 가서 다음을 보자.
```C#
arg = new SocketAsyncEventArgs();  
arg.Completed += new EventHandler<SocketAsyncEventArgs>(receive_completed);  
arg.UserToken = token;  
```

`SocketAsyncEventArgs()`가 만들어서 `args`에 넘겨주고, 이 넘겨주는 시점에 닷넷의 로우레벨 기능을 통해 버퍼 사이즈를 잘라준다.
단지 여기서는 버퍼를 잘라두는 게 아닌, 어디를 자를지만 정해놓는 역할을 한다.
또한, 비동기 호출을 통해 `receive_completed`를 발동.

```C#
args.SetBuffer(m_buffer, m_currentIndex, m_bufferSize);  
m_currentIndex += m_bufferSize;  
```

그리고 위 코드를 통해 `BufferManager`는 버퍼의 처음 위치정보 `m_currentIndex`와 버퍼의 크기 `m_bufferSize`를 넣어 버퍼의 크기를 할당해 준다.

앞서 우리는 버퍼의 총 크기가 10000이고 그 안에 잘릴 `m_buffer`의 크기가 1024라고 했다.
따라서 다음과 같이 각각의 버퍼가 바이트 단위로 잘릴 것이다
0 ~ 1023 \*
1024 ~ 2047 \*
2048 ~ 3071 \*
.
.
.

그리고 위에서 나온 `*` 지점이 바로 `args`가 지정해둔 지점인 것이다.
`SetBufferManager()`가 한번 작동할 때마다 위와 같이 한번의 덩어리를 생성해 낸다.

정리하면, 10000명의 클라이언트와의 비동기 통신을 위해서는 다음의 세가지 정보가 필요하다.
이 세가지 정보는 `SocketAsyncEventArgs` 객체에 담겨 전달된다.
1. 통신이 완료되었을 때 호출될 이벤트 핸들러 메서드 (`receive_completed`와 `send_completed` 메서드)
2. 메모리 공간을 얼만큼 쪼개어서 사용할 것인지 (`BufferManager` 공간의 시작과 끝 위치)
3. 누구와 송수신하고 있는지에 대한 클라이언트 연결 정보 (`CUserToken` 객체)

10000개의 클라이언트와의 비동기 송수신 작업을 하기 위해 각각 수신 `args`, 수신 `args` 객체를 저장한다.
또, `send_event_args_pool`에 송신 `args` 객체를 저장한다.



2023-05-11

----
#Server #수업 #CSharp #socket #listen 

## 개요

`SocketAsyncEventArgsPool`
```cs
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

```cs
using _02_DBManager;  
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Net.Http.Headers;  
using System.Text;  
using System.Threading.Tasks;  
  
namespace _09_MetaBotServer  
	{  
	// 게임룸들을 등록 관리하는 클래스  
	public class CGameRoomManager  
	{  
		GameRoomDao roomDao = new GameRoomDao();  
		  
		List<CGameRoom> roomList = new List<CGameRoom>();  
		  
		public CGameRoomManager()  
		{  
			// 시작하자마자 DB에 있는 게임룸 정보를 가져와 CGameRoom객체를 생성해  
			// 리스트에 저장한다.  
			  
			IList<GameRoomVo> voList = roomDao.GetGameRoomList();  
			  
			foreach(var vo in voList)  
			{  
				CGameRoom gameRoom = new CGameRoom(vo.GAMEROOM_ID, vo.GAMEROOM_NAME);  
				roomList.Add(gameRoom);  
			}  
		}  
		  
		public void AddRoomList(CGameRoom gameRoom)  
		{  
			roomList.Add(gameRoom);  
		}  
		  
		public void RemoveRoomList(CGameRoom gameRoom)  
		{  
			roomList.Remove(gameRoom);  
		}  
		  
		// 게임방에 진입하면 CPlayer객체를 만들어서 해당 GameRoom에 등록시킨다  
		public void AddRoomCreatePlayer(string gameroom_id, string gameroom_name,  
		CGameUser gameUser, string user_id)  
		{  
			foreach(var room in roomList)  
			{  
			// 게임룸id, name이 일치하는 room객체를 찾아서  
			// 해당 room에 CPlayer객체를 등록한다.  
				if(room.GAMEROOM_ID.Equals(gameroom_id) &&  
				room.GAMEROOM_NAME.Equals(gameroom_name))  
				{  
					CPlayer player = new CPlayer(room, gameUser, user_id);  
					gameUser.PLAYER = player;  
					room.AddPlayerList(player);  
				}  
			}  
		}  
	}  
}
```

```cs
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
using System.Threading.Tasks;  
  
namespace _09_MetaBotServer  
{  
	// 게임룸에 진입한 게임상에서의 클라이언트  
	public class CPlayer  
	{  
		public CGameRoom GAME_ROOM { get; set; } // 소속룸  
		public CGameUser GAME_USER { get; set; } // 연결된 CGameUser 객체  
		public string USER_ID { get;set; } // Player의 정보  
		  
		public byte spawnIdx { get; set; } // Player가 생성된 초기위치의 index  
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

```cs
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
		public CGameRoomManager GAMEROOM_MGR { get; set; }  
		  
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
			  
			// DB에 새로운 GameRoom등록  
			GameRoomVo vo = new GameRoomVo();  
			vo.GAMEROOM_ID = gameroom_id;  
			vo.GAMEROOM_NAME = gameroom_name;  
			int row = gameDao.InsertGameRoom(vo);  
			  
			CPacket ack = CPacket.create((short)PROTOCOL.MAKE_ROOM_ACK);  
			ack.push(user_id);  
			ack.push(gameroom_id);  
			if (row == 1) // 성공  
			{  
				// DB에는 새로 GameRoom이 등록되었고  
				// GameRoomManager에는 초기의 DB방 정보만 있어서  
				// 불일치한다.  
				// 새로 생성된 GameRoom정보를 일치하게 하기 위해서  
				// CGameRoomManager의 List에 새로 생성된 CGameRoom개체를 등록  
				GAMEROOM_MGR.AddRoomList(new CGameRoom(gameroom_id, gameroom_name));  
				  
				ack.push((byte)1);  
			}  
			else  
				ack.push((byte)0);  
			  
			msg.owner.send(ack);  
		}  
		  
		public void Process_JOIN_ROOM_REQ(CPacket msg)  
		{  
			string user_id = msg.pop_string();  
			string gameroom_id = msg.pop_string();  
			string gameroom_name = msg.pop_string();  
			  
			int row = memDao.UpdateMemberJoin(user_id, gameroom_id);  
			  
			CPacket ack = CPacket.create((short)PROTOCOL.JOIN_ROOM_ACK);  
			ack.push(user_id);  
			ack.push(gameroom_id);  
			if (row == 1)  
			{  
				GAMEROOM_MGR.AddRoomCreatePlayer(gameroom_id, gameroom_name,  
				(CGameUser)msg.owner, user_id);  
				  
				ack.push((byte)1);  
			}  
			else  
				ack.push((byte)0);  
			  
			msg.owner.send(ack);  
		}  
		  
		public void Process_SPAWN_PLAYER_REQ(CPacket msg)  
		{  
			string user_id = msg.pop_string();  
			byte idx = msg.pop_byte();  
			  
			// 현재 패킷 -> CGameUser -> CPlayerCPlayer player = ((CGameUser)msg.owner).PLAYER;  
			player.spawnIdx = idx;  
			  
			// CPlayer가 소속된 CGameRoomCGameRoom gameRoom = player.GAME_ROOM;  
			CPacket ack = gameRoom.GetSpawnedPlayers();  
			  
			// GameRoom에 소속된 모든 CPlayer한테 전송  
			gameRoom.BroadcastPacket(ack);  
		}  
	}  
}
```

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
		static CGameRoomManager gameRoomMgr = new CGameRoomManager();  
		  
		static void Main(string[] args)  
		{  
			procPacket.GAMEROOM_MGR = gameRoomMgr;  
			  
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

## 내용
이제 클라이언트 쪽에서 서버와 연동되는 코드를 살펴보자.

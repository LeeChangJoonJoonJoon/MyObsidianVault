

2023-05-09

----
#Server #Client #Unity 

## 개요
이제 배틀필드 내에서 로그를 띄우는 기능을 구현할 차례.

## 내용
`CUIFieldManager.cs`에 다음 두 메서드를 추가해 준다.
```C#
public void WriteEventLog(string log)  
{  
	logInfo.text += ("\n" + log);  
}  
  
public void SetConnInfo(short count)  
{  
	connInfo.text = String.Format("{0,4:D4}", count);  
}
```

그리고 `LeavePlayer()` 메서드에 플레이어가 방을 떠났다는 정보를 출력해 주고자 `WriteEventLog()`를 다음과 같이 추가해 준다.
```C#
public void LeavePlayer(string uid)  
{  
	foreach(var go in CGameManager.Instance.playerList)  
	{  
		if(go.GetComponent<CPlayerInfo>().USER_ID == uid)  
		{  
			CGameManager.Instance.playerList.Remove(go);  
			  
			Destroy(go);  
			  
			if (uid == CProcessPacket.Instance.USER_ID)  
				CProcessPacket.Instance.ChangeLobbyScene();  
			  
			if (CUIFieldManager.Instance != null)  
			{  
				string log = $"<color=#ff0000>{uid} has left room</color>";  
				CUIFieldManager.Instance.WriteEventLog(log);  
				  
				SetMinusConnInfo();  
			}  
			  
			break;  
		}  
	}  
}
```

로그를 출력해주는 프로토콜을 서버와 클라이언트의 `protocol.cs`에 추가해 준다.
```C#
WRITE_LOG_REQ = 21, // 게임 중에 발생한 로그 전달 요청  
WRITE_LOG_ACK = 22, // 게임 중에 발생한 로그 전달 응답
```


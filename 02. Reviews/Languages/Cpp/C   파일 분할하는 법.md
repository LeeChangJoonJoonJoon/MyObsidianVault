

2023-05-15

----
#Server #OracleDB #Manager #Unreal #MVC #Cpp #PANIM #ChatGPT #지선생 

## 개요
[[Oracle DB를 C++의 DB Manager와 연동하기]]에서 오라클 데이터베이스를 연동하는 코드를 만들기에 앞서 필요한 내용이다.
C++에서 어떻게 상속을 하고 정리할지에 대한 것이다.

## 내용
C#에서는 특정 `namespace{...}` 내에 있으면 별개의 파일이어도 전부 그 네임스페이스 내에 있는 클래스 혹은 함수였지만, 여기선 C#의 인터페이스가 함수의 이름만 정해주듯 헤더파일에서 한 네임스페이스 내에 속한 클래스들의 이름만 적어주면 된다.

지선생께서 주신 예제는 다음과 같다.

일단 상속 관계를 알려주는 네임스페이스의 헤더파일을 다음과 같이 만들어 준다.
```cpp
// MyNamespace.h

namespace MyNamespace {
    class MyClass1;
    class MyClass2;
}
```

이 헤더 파일 내 클래스의 본체는 .cpp 파일에서 정의해 준다.
```cpp
// MyNamespace.cpp

#include <iostream>
#include "MyNamespace.h"

namespace MyNamespace {
    class MyClass1 {
    public:
        void memberFunction1();
    };
	
    class MyClass2 {
    public:
        void memberFunction2();
    };
	
    void MyClass1::memberFunction1() {
        std::cout << "This is MyClass1's member function." << std::endl;
    }
	
    void MyClass2::memberFunction2() {
        std::cout << "This is MyClass2's member function." << std::endl;
    }
}
```

이 네임스페이스를 프로그램 실행시 사용하려면 다음과 같이 하면 된다.
```cpp
// main.cpp

#include "MyNamespace.h"

int main() {
    MyNamespace::MyClass1 object1;
    MyNamespace::MyClass2 object2;
	
    object1.memberFunction1();
    object2.memberFunction2();
	
    return 0;
}
```

이제 적용해 보자.
실제 구현한 방법은 다음과 같다.
```cpp
//  
// Created by Changjoon Lee on 2023/05/17.  
//  
  
#include "WhenMyCompilerMessedUp.h"  
  
#ifndef DBMANAGER_H  
#define DBMANAGER_H  
  
namespace DBManager {  
	class OracleConnectionPool {  
	public:  
		OracleConnectionPool();  
		static OracleConnectionPool* staticOracleConnectionPool;  
		static OracleConnectionPool* Instance(const std::string& ip = "localhost", int port = 1521, const std::string& service_name = "xe", const std::string& id = "metabot", const std::string& pass = "metabot");  
		OracleConnection* GetConnection();  
		void ReturnOracleConnection(OracleConnection* conn);  
		void CloseAll();  
	private:  
		std::string ip;  
		int port;  
		std::string service_name;  
		std::string id;  
		std::string password;  
		std::string OracleConnInfo;  
		std::stack<OracleConnection*> connStack;  
		  
		OracleConnectionPool(const std::string& ip, int port, const std::string& service_name, const std::string& id, const std::string& password);  
	};  
}  
  
#endif // DBMANAGER_H
```

그리고 다음은 위 정의된 클래스들을 정의하는 .cpp 파일이다.
```cpp
//  
// Created by Changjoon Lee on 2023/05/15.  
//  
  
#include "OracleConnectionPool.h"  
  
namespace DBManager {  
	OracleConnectionPool* OracleConnectionPool::staticOracleConnectionPool = nullptr;  
	  
	OracleConnectionPool::OracleConnectionPool(const std::string& ip, int port, const std::string& service_name, const std::string& id, const std::string& password)  
	: ip(ip), port(port), service_name(service_name), id(id), password(password) {  
		this->OracleConnInfo = "(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=" + ip + ")(PORT=" + std::to_string(port) + ")))"  
	"(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=" + service_name + ")));User Id=" + id + ";Password=" + password;  
	}  
	  
	OracleConnectionPool* OracleConnectionPool::Instance(const std::string& ip, int port, const std::string& service_name, const std::string& id, const std::string& pass) {  
		if (staticOracleConnectionPool == nullptr) {  
			staticOracleConnectionPool = new OracleConnectionPool(ip, port, service_name, id, pass);  
		}  
		return staticOracleConnectionPool;  
	}  
	  
	OracleConnection* OracleConnectionPool::GetConnection() {  
		OracleConnection* conn = nullptr;  
		if (!connStack.empty()) {  
			conn = connStack.top();  
			connStack.pop();  
			if (conn->KeepAlive) {  
				return conn;  
			}  
		}  
		conn = new OracleConnection();  
		conn->KeepAlive = true;  
		  
		try {  
			oracle::occi::Environment* env = oracle::occi::Environment::createEnvironment(oracle::occi::Environment::DEFAULT);  
			conn->connection = env->createConnection(id, password, OracleConnInfo);  
			delete env;  
			  
			if (conn->connection->isConnected()) {  
				DBDebug::WriteLog("Oracle Server 연결 성공");  
			}  
		}  
		catch (const std::exception& ex) {  
			delete conn;  
			conn = nullptr;  
			DBDebug::WriteLog("DB Exception: " + std::string(ex.what()));  
		}  
			return conn;  
	}  
	  
	void OracleConnectionPool::ReturnOracleConnection(OracleConnection* conn) {  
		if (conn && conn->KeepAlive) {  
			connStack.push(conn);  
		}  
		else {  
			if (conn && conn->connection) {  
				delete conn->connection;  
			}  
			delete conn;  
		}  
	}  
	  
	void OracleConnectionPool::CloseAll() {  
		while (!connStack.empty()) {  
			OracleConnection* conn = connStack.top();  
			connStack.pop();  
			if (conn && conn->connection) {  
				conn->connection->terminateConnection();  
				delete conn->connection;  
			}  
			delete conn;  
		}  
	}  
}
```

이 멤버들을 메인함수에서 실행시킬 때는 다음과 같이 해주면 된다.
![[스크린샷 2023-05-18 오전 6.35.22.png]]

정리된 파일들을 보면 다음과 같다.
![[스크린샷 2023-05-15 오후 4.30.41.png|300]]

이 클래스들은 향후 .cpp 파일 내에서 구체적으로 정의를 해줄 것이고, 이것의 객체화 및 작동은 `MainController`에서 할 것이다.
이 `MainConroller` 내에서는 앞에서 한번 `#include DBManager`를 해주고, 인스턴스화할 객체의 소속을 밝혀주면 된다.

이제 오라클에 연결하는 로직을 짜야 한다.

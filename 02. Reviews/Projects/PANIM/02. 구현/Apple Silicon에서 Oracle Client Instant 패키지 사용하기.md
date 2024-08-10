

2023-05-16

----
#PANIM #Cpp #Mac #OracleDB #CLion #UNIX #터미널 #ChatGPT #지선생 #Rosetta2 #Bard 

## 개요
오라클과 연동된 cpp 프로젝트를 만들던 중, 오라클이 지원하는 인스턴스는 인텔 기반 맥 밖에 지원하지 않는다는 사실을 알게 됐다.
![[스크린샷 2023-05-16 오전 7.41.42.png]]

애플 실리콘 맥에서 `#include <oci.h>`를 쓰려면 어떻게 해야 할까?

## 내용
`#include <oci.h>`를 포함하고 있는 프로젝트는 Rosetta2 환경를 통해 생성되어야 한다.
터미널 앱을 다음과 같이 `Rosetta를 사용하여 열기`를 클릭한 뒤 연다.
![[스크린샷 2023-05-16 오전 7.43.55.png|300]]

여기서 원하는 위치에 .cpp 파일을 생성해주면 된다.

하지만 앞서 [[C++ 파일 분할하는 법]]를 통해 이미 만든 프로젝트들이 있다.
이를 Rosetta2 기반의 프로젝트로 변환하려면 어떻게 해야 할까?

지선생에게 물어봤다.
![[스크린샷 2023-05-16 오전 8.48.56.png]]

잠깐, 그러면 에뮬레이션되고 있는 .cpp 프로젝트는 그렇지 않은 프로젝트와 연결될 수 있는가?
그것도 지선생에게 물어봤다.
![[스크린샷 2023-05-16 오전 8.50.42.png]]

된다고 한다. 
다만, 라이브러리나 구문에 있어서 운영체제를 타는 것은 조심하라는 원론적인 이야기.
지난번에 [[교착 상태 (Deadlock)]]에서 `CRITICAL_SECTION`이 윈도우 기반 API여서 GDG와 같은 대체 API 쓰는 것을 떠올리면 될 것이다.

앞서 실행한 로제타2 기반의 터미널 앱에서 에뮬레이션을 실행하고 싶은 디렉토리로 이동한다.
```unix
cd /Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/PANIM_DB
```

그리고, 지선생이 말씀하신 대로 다음 명령을 실행하자.
이 명령은 기존에 `arm64` 기반으로 만들어진 프로젝트를 `x86`기반으로 빌드해준다.
```unix
g++ -o MainController_x86 /Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/PANIM_DB/MainController.cpp
```

그러면 다음과 같이 새로운 파일이 생성된 것을 볼 수 있다.
![[스크린샷 2023-05-16 오후 2.53.03.png|300]]

실험 삼아 간단한 프로그램을 `x86` 환경에서 실행해 보자.

원래 `arm64` 환경에서 만든 코드는 다음과 같다.
```cpp
#include <cmath> // 기본적인 연산자들을 담고 있는 헤더파일  
#include <iostream> // string class 등을 담고 있는 헤더파일  
#include <stdio.h>  
  
int main(int argc, const char * argv[])  
{  
	printf("jsdhgfsdjkhgfskug");  
	return 0;  
}
```

실행하는 명령은 다음과 같이 해당 파일을 터미널에 옮기고 리턴해 주면 된다.
```unix
/Users/changjoonlee/GitHub/Project_PANIM_GitHub/CppMyPractice/_x86_Compiled/MainController_x86
```

그러면 다음과 같이 결과가 나온다.
![[스크린샷 2023-05-16 오후 3.06.14.png]]

앞서 작성한 문자열 `jsdhgfsdjkhgfskug`이 출력되는 것을 확인했다.

이제 진짜로 오라클 API를 활용해서 프로젝트를 만들어 보자.
바드(Bard)에게 물어본 결과 다음과 같이 하면 된다고 한다.
![[스크린샷 2023-05-16 오후 5.35.15.png]]

버전 19.8의 Base Package 안에 있는 다음 파일을 앞에다 넣고, 뒤에다가는 `usr` 디렉토리의 위치를 적어주면 된다.
![[스크린샷 2023-05-16 오후 5.36.48.png]]

내가 실행한 명령어는 다음과 같다.
(참고로, `dsenableroot`와 `su - ` 명령어를 통해 관리자 권한으로 터미널을 실행해야 한다.
그러지 않으면 `permission denied`라는 에러 메시지가 뜬다.)
![[스크린샷 2023-05-16 오후 5.38.47.png]]

터미널에 `open /usr/local/lib`를 리턴하면 다음과 같이 파인더가 열리고 파일이 생긴 것을 확인할 수 있다.
![[스크린샷 2023-05-16 오후 5.40.51.png]]

이제 `#include <oci.h>`를 쓸 수 있...는가 싶더니 컴파일러가 인식을 못했다.
답답해서 다음과 같이 `OracleConnectionPool.h`에 직접 경로를 한땀한땀 써줬다...
```cpp
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/nzerror.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/ldap.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/nzt.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/occi.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/occiAQ.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/occiCommon.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/occiControl.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/occiData.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/occiObjects.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/oci.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/oci1.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/oci8dp.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/ociap.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/ociapr.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/ocidef.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/ocidem.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/ocidfn.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/ociextp.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/ocikpr.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/ociver.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/ocixmldb.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/ocixstream.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/odci.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/orastruc.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/oratypes.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/oraxml.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/oraxml.hpp>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/oraxmlcg.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/oraxsd.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/oraxsd.hpp>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/ori.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/orid.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/orl.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/oro.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/ort.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/xa.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/xml.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/xml.hpp>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/xmlctx.hpp>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/xmldav.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/xmldf.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/xmlerr.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/xmlev.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/xmlotn.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/xmlotn.hpp>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/xmlproc.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/xmlsch.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/xmlsoap.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/xmlsoap.hpp>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/xmlsoapc.hpp>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/xmlurl.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/xmlxptr.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/xmlxsl.h>
#include </Users/changjoonlee/GitHub/instantclient_19_8-2/sdk/include/xmlxvm.h>
```

그랬더니 컴파일러가 다음 코드를 제대로 읽어내는 듯 보였다.
```cpp
//  
// Created by Changjoon Lee on 2023/05/15.  
  
#include <iostream>  
#include "OracleConnectionPool.h"  
#include "DBManager.h"  
  
using namespace std;  
using namespace oracle::occi;  
  
namespace DBManager {  
	class OracleConnectionPool {  
	private:  
		char ip;  
		int port;  
		char service_name;  
		char id;  
		char password;  
		char OracleConnInfo;  
	
	public:  
		int Hello() {  
		try {  
			// Initialize the OCI environment  
			oracle::occi::Environment* env = oracle::occi::Environment::createEnvironment();  
			  
			// Create a connection  
			std::string connectionString = "dbname=XE;sid=XE;port=1521";  
			std::string username = "your_username";  
			std::string password = "your_password";  
			oracle::occi::Connection* conn = env->createConnection(username, password, connectionString);  
			  
			// Create a statement  
			oracle::occi::Statement* stmt = conn->createStatement("SELECT 1 FROM dual");  
			  
			// Execute the statement  
			oracle::occi::ResultSet* rs = stmt->executeQuery();  
			if (rs->next()) {  
				int result = rs->getInt(1);  
				std::cout << "Result: " << result << std::endl;  
			}  
			  
			// Clean up resources  
			conn->terminateStatement(stmt);  
			env->terminateConnection(conn);  
			oracle::occi::Environment::terminateEnvironment(env);  
			} catch (oracle::occi::SQLException& ex) {  
				std::cout << "Oracle Exception: " << ex.getMessage() << std::endl;  
			} catch (std::exception& ex) {  
				std::cout << "Exception: " << ex.what() << std::endl;  
			}  
			  
			return 0;  
		}  
	};  
}
```

꼬박 이틀에 걸려서 컴파일러가 오라클 관련 키워드를 인식하도록 만들어 줬다.
처음엔 `oci.h`를 쓰려 했으나, 이와 관련한 자료가 거의 없을 뿐더러 있는 것들 조차도 너무 오래된 자료여서 `occi.h`로 중간에 바꿨다...

2023-05-18에서 왔습니다 당신은 지금 IDE 때문에 고생하고 계십니다.
CLion 쓰지 말고 그냥 VSCode 쓰세요.
그러면 [[03. OracleConnectionPool.cpp]]에 잘 나와 있는 것처럼 컴파일러가 정상적으로 `#include <iostream>`를 인식하게 만들 수 있다.
아, VSCode의 includePath 설정은 다음과 같이 했다.
![[화면 기록 2023-05-18 오후 12.11.51.mov]]

이렇게 하면, 앞에서처럼 직접 파일 경로를 복사해서 `#include`문 뒤에 덕지덕지 붙이는 행동은 하지 않아도 된다...
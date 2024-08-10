

2023-05-03

----
#OracleDB #error #Unity #문제해결 #문제 #MVC #수업 

## 개요
우리는 지금 수업 시간에서 완결된 형태의 'MVC'(Model View Controller) 패턴을 만들고자 한다.
그 과정에서 예시로 테이블을 생성해 보았다. 
하지만...

나는 다음 쿼리를 작성하고 실행시켰는데 테이블을 찾을 수 없었다.
```sql
--SYSTEM 계정으로 처리하는 부분 START
DROP USER metabot CASCADE;

CREATE USER metabot IDENTIFIED BY metabot DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp PROFILE DEFAULT;

GRANT CONNECT, RESOURCE TO metabot ;
GRANT CREATE VIEW, CREATE SYNONYM TO metabot;

ALTER USER metabot ACCOUNT UNLOCK;

conn metabot/metabot;
--SYSTEM 계정으로 처리하는 부분 END
--metabot START
--metabot 접속했으므로 metabot 사용자의 테이블이 생성
DROP SEQUENCE log_sequence;

DROP TABLE log;
DROP TABLE member;
DROP TABLE gameroom;

CREATE SEQUENCE log_sequence
  START WITH 6
  INCREMENT BY 1
  NOMAXVALUE
  NOMINVALUE
  NOCYCLE
  CACHE 20;

CREATE TABLE gameroom (
  gameroom_id   VARCHAR2(100),
  gameroom_name VARCHAR2(100)
);
CREATE TABLE member (
  member_id     VARCHAR2(100),
  member_pass   VARCHAR2(100),
  member_name   VARCHAR2(100),
  member_age    VARCHAR2(3),
  member_job    VARCHAR2(100),
  member_phone  VARCHAR2(40),
  gameroom_id   VARCHAR2(100)
);
CREATE TABLE log (
  log_seq   NUMBER,
  log_time  DATE default SYSDATE,
  log_ip    VARCHAR2(20),
  log_port  VARCHAR2(5),
  log_info  VARCHAR2(1000),
  member_id VARCHAR2(100)
);

INSERT INTO gameroom VALUES('WORLD_01', 'ABC');
INSERT INTO gameroom VALUES('WORLD_02', 'Korea');
INSERT INTO gameroom VALUES('WORLD_03', 'Japan');
INSERT INTO gameroom VALUES('WORLD_04', 'China');
INSERT INTO gameroom VALUES('WORLD_05', 'Zepeto');
COMMIT;

INSERT INTO member VALUES('asdf', '1234', '홍길동', '24', '활빈당', '010-1111-1111', 'WORLD_01');
INSERT INTO member VALUES('qwer', '1234', '임꺽정', '34', '의적', '010-2222-2222', 'WORLD_02');
INSERT INTO member VALUES('zxcv', '1234', '장길산', '44', '광대', '010-3333-3333', 'WORLD_03');
INSERT INTO member VALUES('aaaa', '1234', '차돌바위', '21', '나무꾼', '010-4444-5555', 'WORLD_04');
INSERT INTO member VALUES('bbbb', '1234', '일지매', '31', '댄서', '010-5555-4444', 'WORLD_05');
INSERT INTO member VALUES('cccc', '1234', 'Albert', '41', 'Developer', '010-6666-7777', 'WORLD_05');
INSERT INTO member VALUES('dddd', '1234', 'Alex', '55', 'Knight', '010-7777-6666', 'WORLD_04');
INSERT INTO member VALUES('eeee', '1234', 'James', '25', 'Writer', '010-8888-9999', 'WORLD_03');
INSERT INTO member VALUES('ffff', '1234', 'Henry', '26', 'Teacher', '010-9999-8888', 'WORLD_02');
INSERT INTO member VALUES('gggg', '1234', 'Brown', '29', 'Boxer', '010-0000-1111', 'WORLD_01');
COMMIT;

ALTER SESSION SET nls_date_format='YYYY-MM-DD:HH24:MI:SS';
INSERT INTO log VALUES(1, '2023-04-20:11:59:24', '192.168.10.21', '9876', 'WORLD_02 Enter', 'asdf');
INSERT INTO log VALUES(2, '2023-04-20:13:29:45', '192.168.10.25', '9111', 'WORLD_02 Enter', 'aaaa');
INSERT INTO log VALUES(3, '2023-04-20:15:32:11', '192.168.10.11', '10980', 'WORLD_03 Enter', 'cccc');   
INSERT INTO log VALUES(4, '2023-04-21:10:10:20', '192.168.10.25', '9111', 'aaaa killed asdf in WORLD_02', 'aaaa');
INSERT INTO log VALUES(5, '2023-04-21:12:59:14', '192.168.10.25', '9111', 'WORLD_02 Leave', 'aaaa');
COMMIT;

-- PRIMARY KEY 설정
ALTER TABLE gameroom
 ADD CONSTRAINT gameroom_gameroom_id_pk PRIMARY KEY(gameroom_id);

ALTER TABLE member
 ADD CONSTRAINT member_member_id_pk PRIMARY KEY(member_id);

ALTER TABLE log
 ADD CONSTRAINT log_log_seq_pk PRIMARY KEY(log_seq);

-- FOREIGN KEY 설정
ALTER TABLE member
 ADD CONSTRAINT member_gameroom_id_fk FOREIGN KEY(gameroom_id) 
 REFERENCES gameroom(gameroom_id);

ALTER TABLE log 
 ADD CONSTRAINT member_member_id_fk FOREIGN KEY(member_id)
 REFERENCES member(member_id);


-- private OracleConnectionPool(string ip, int port, string service_name, string id, string password)
-- {
--     this.ip = ip;
--     this.port = port;
--     this.service_name = service_name;
--     this.id = id;
--     this.password = password;

--     this.OracleConnInfo = $"Data Source=(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST={ip})(PORT={port})))" +
--         $"(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME={service_name}))); User Id = {id}; Password = {password};";
-- }
```

몇번이고 쿼리문을 다시 실행시켰으나 마찬가지였다.

## 내용
스크립트 출력에는 다음과 같이 나왔다. 
```sql
User METABOT이(가) 삭제되었습니다.


User METABOT이(가) 생성되었습니다.


Grant을(를) 성공했습니다.


Grant을(를) 성공했습니다.


User METABOT이(가) 변경되었습니다.

CONNECT 스크립트 명령으로 생성된 접속이 해제되었습니다.
```

아마 오라클이 `CONN metabot/metabot`이하는 못읽어내는 듯 보인다. 
그리고 `CONNECT 스크립트 명령으로 생성된 접속이 해제되었습니다.`는 이를 암시하는 듯 보인다.

`metabot` 접속의 속성을 클릭해서 이것 저것 눌러 보던 중 다음 사실을 알게 되었다. 
![[스크린샷 2023-05-05 오후 6.02.28.png]]

왜 접속이 안돼 있는 것 처럼 나와 있는 것일까?
그래서 `CONN metabot/metabot`이라고 쿼리가 작성돼 있으니 사용자 이름과 비밀번호에 그대로 적어 넣었다.
테스트했더니 성공.
![[스크린샷 2023-05-05 오후 6.06.31.png]]

또한, 이렇게 성공한 접속 내에서 `DROP SEQUENCE log_sequence;` 이하만 남기고 윗부분을 지운 채로 쿼리를 실행시켜 보았다. 
![[스크린샷 2023-05-05 오후 7.20.19.png]]

성공.

----
위의 설명은 내가 헤맨 과정을 쓴 것이다. 
정리하자면, 우리가 수업 시간에 다룬 쿼리 부분 중 다음 부분은 시스템 계정으로 새로운 계정을 만들고 권한을 부여하는 과정에 해당된다.
```sql
--SYSTEM 계정으로 처리하는 부분 START
DROP USER metabot CASCADE;

CREATE USER metabot IDENTIFIED BY metabot DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp PROFILE DEFAULT;

GRANT CONNECT, RESOURCE TO metabot ;
GRANT CREATE VIEW, CREATE SYNONYM TO metabot;

ALTER USER metabot ACCOUNT UNLOCK;

conn metabot/metabot;
--SYSTEM 계정으로 처리하는 부분 END
--metabot START
--metabot 접속했으므로 metabot 사용자의 테이블이 생성
```

그리고 그 이하의 `CREATE`는 테이블을 만드는 것이고, `INSERT`는 만들어진 빈 테이블에 내용을 집어넣는 과정이다.

----

그리고 `INSERT INTO gameroom VALUES('WORLD_01', 'ABC');` 이하가 실행되지 않는 것 같았다.
이것에 대한 문제 해결은 [[ORA-01950 테이블스페이스 'USERS'에 대한 권한이 없습니다]]에서 다루어 보고자 한다.
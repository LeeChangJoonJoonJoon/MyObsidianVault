

2023-05-03

----
#OracleDB #ERD

## 개요
지금 수업 시간에는 유니티 UI를 이용하여서 사용자를 등록하고, 방을 만들고 방 안에 회원 정보를 기입하는 DB를 만들고 있었다. 
이를 ERD로 표현해 보면 다음과 같다.
![[스크린샷 2023-05-03 오전 8.34.37.png|500]]

## 내용
앞선 수업에서 나는 선생님과 함께 다음 쿼리문을 작성하였다.
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

간단히 쿼리에 대해 설명을 하자면, 처음 사용자는 SYSTEM 계정으로 접속이 된 상태이다.
이를 다음과 같이 확인할 수 있다.
![[스크린샷 2023-05-03 오후 2.57.29.png]]

접속명은 aiameta이고, 사용자 이름은 디폴트 값으로 `system`, 비밀번호는 오라클 데이터베이스를 도커에 띄울 때에 다음과 같이 입력한 바 있다.
```unix
docker run --restart unless-stopped --name oracle -e ORACLE_PASSWORD=meta -p 1521:1521 -d gvenzl/oracle-xe
```

그래서 처음 `system`계정으로 접속된 상태에서 위 쿼리문을 실행시키면, 
![[스크린샷 2023-05-03 오후 3.01.40.png]]

이처럼 `User METABOT`이 생성되었다는 출력을 받게 된다.
그리고 `Grant` 쿼리를 이용하여 `system` 계정에서 `METABOT`에게 DB에 대한 권한을 부여했다.
이 `METABOT`으로 접속이 됐으니 다음과 같이 확인할 수 있다.
![[스크린샷 2023-05-03 오후 3.20.19.png]]

다만 여기서 문제가 생겼다.
`CREATE` 구문들이 실행이 안됐는지, 테이블을 찾을 수 없었다.
이 문제와 관련해서는 [[Oracle 테이블 생성 안되는 문제]]에서 다루기로 한다.
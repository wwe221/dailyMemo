# 2021

### 개발

#### JPA

##### ##To## Mapping

```java
// ManyToOne
// DasApi.java
@ManyToOne
@JoinColumn(name = "gateway_id", referencedColumnName = "id", insertable = false, updatable = false)
// DasApi table의 gateway_id 와 Gateway table의 id 를 기준으로 조인한다.
private DasGateway gateway;
```

```java
// OneToMany
// Controller.java
@LazyCollection(LazyCollectionOption.FALSE)
@OrderBy("ordinal")
@OneToMany(cascade = CascadeType.ALL)
@JoinColumn(name = "ctl_id") // Controller 의 id 와 Das table 의 ctl_id 를 기준으로 조인한다.
private List<Das> dasList = new ArrayList<Das>();
```

#####  Json , Array 매핑

```java
@TypeDef(name = "json", typeClass = JsonType.class) // postgre Json to Java String
    @Type(type="json")
    @Column(name = "recipe")
    private String recipe = "[[0,0],[1,0]]";

@TypeDef(name = "list-array", typeClass  = com.vladmihalcea.hibernate.type.array.DoubleArrayType.class) // postgre Number Array to Java Array
	@Type(type = "list-array")
    @Column(name = "heat_reaction", columnDefinition = "double[]")
    private Double[] heatReaction = {0d,0d};
```



##### Enum Type 추가

###### Java 

```java
@Column(name = "var_type")
@Enumerated(EnumType.STRING)
private VarType dataType;

public enum VarType {CV, MV, DV, NA}
```

###### postgreSql 

```sql
create type var_type as enum ('CV', 'MV', 'DV', 'NA');
alter type var_type owner to hpcb;
-- custom dataType 추가시 cast 도 create 해줘야 String 으로 사용가능.
CREATE CAST (character varying AS var_type) WITH INOUT AS IMPLICIT;
```

### 기타

#### Oracle DB 에디션 변경

(downgrade Enterprise Edition -> Standard Edition2)

###### 버전 선택

만약 web service 말고 다른 서비스들도 DB 를 조회하고 있다면 같은 버전을 사용하는 것을 권장한다. 

12c 에서 19c 로 버전을 업그레이드 한다면 12c 를 지원하는 Oracle provier 가 19c 에서는 작동을 하지 않기 때문에 새로운 버전을 다운로드하고 옵션을 수정해야하는 일이 생긴다.

Oracle 

expdp를 통한 데이터베이스 전체 Dump

system 대신 user를 이용해 특정 스키마만 dump 할 수도 있다



expdp로 dump하기 위해선 디렉토리가 db에 저장되어 있어야한다.

sysdba로 로그인후 다음과 같이 디렉토리를 관리할 수 있다.

**1** 디렉토리 조회

SQL> **select \* from dba_directories ;** 

 **2** 디렉토리 생성

SQL> **create or replace directory** *DATA_PUMP_DIR2* **as** '*/data2/dpdump*' ;

 **3** 디렉토리 삭제

SQL> **drop directory** *DATA_PUMP_DIR2* ;

 **4** 디렉토리 권한 부여

SQL> **grant read,write on directory** *DATA_PUMP_DIR2* **to** *yncc* ;



###### expdp 예시

```powershell
expdp system@ORCL directory=DATA_PUMP_DIR2 dumpfile=dump.dmp logfile=dump.log
```



###### impdp 예시

```powershell
 impdp system@ORCL directory=DATA_PUMP_DIR2 dumpfile=test1.dmp logfile=test1.log tables=hoya.test1
```



###### 오라클 리스너 확인 

```powershell 
lsnrctl service
```



###### DB LINK 확인 

기존 DB 를 dumping 하고나서 DB Link도 같이 복사가 됐는지 확인한다.

```sql
  SELECT *
  FROM DBA_DB_LINKS;
```



###### DB LINK 생성

```sql
CREATE DATABASE LINK TESTDB
CONNECT TO SUSER
IDENTIFIED BY "1234"
USING '(DESCRIPTION =
			(ADDRESS = (PROTOCOL = TCP)(HOST = 100.211.111.107)(PORT = 1521))
		    (CONNECT_DATA =
				(SERVER = DEDICATED)
				(SERVICE_NAME = SEP)
             )
        )';
```



####  Window11 remove 'Show More Options' in Explorer

```powershell
reg add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /f /ve
```


# 2021

## 개발

### Java

#### Stream

스트림은 데이터의 흐름이다. 배열 또는 컬렉션 인스턴스에 함수 여러 개를 조합하여 원하는 형태로 가공된 결과를 얻을 수 있다. 또 람다를 이용해서 코드를 간결하게 표현할 수 있다. 즉, 배열과 컬렉션을 함수형으로 처리하는 것.

또 간단하게 병렬처리(*multi-threading*)가 가능하다. 하나의 작업을 둘 이상의 작업으로 나눠서 동시에 진행하는 것을 병렬 처리(*parallel processing*)라고 한다. 쓰레드를 이용해 많은 요소들을 빠르게 처리할 수 있다.
*병렬 처리를 작업할때 collect 의 사용은 적합하지 않다. (stream 의 collection을 합치는 작업의 부담이 크다)*

##### examples

`Map<String, Long>` 에서 빈도 높은 단어 top 10

```java
List<String> topTen = freq.keySet()
    .stream()
    .sorted(Comparator.comparing(freq::get).reversed())
    .limit(10)
    .collect(Collectors.toList());
```

총 무게를 구한다.

```java
int sum = widgets.stream()
  .filter(w -> w.getColor() == RED)
  .mapToInt(w -> w.getWeight())
  .sum();
```

###### collect 사용

```java
// List<People>에서 사람들의 이름만 뽑아 리스트로 수집한다
List<String> list = people.stream()
    .map(Person::getName)
    .collect(Collectors.toList());

// List<People>에서 이름만 뽑아 TreeSet 으로 수집한다
Set<String> set = people.stream()
    .map(Person::getName)
    .collect(Collectors.toCollection(TreeSet::new));

// 리스트의 원소들을 콤마로 구분된 하나의 String으로 수집한다.
String joined = things.stream()
    .map(Object::toString)
    .collect(Collectors.joining(", "));

// 모든 직원 급여의 총합을 구한다
int total = employees.stream()
    .collect(Collectors.summingInt(Employee::getSalary));

// 부서별 직원 목록을 만든다
Map<Department, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// 부서별 급여 합계를 구한다
Map<Department, Integer> totalByDept = employees.stream()
    .collect(
        Collectors.groupingBy(
            Employee::getDepartment,
            Collectors.summingInt(Employee::getSalary)
        )
    );

// PASS한 학생과 FAIL한 학생 리스트를 따로 수집한다
Map<Boolean, List<Student>> passingFailing = students.stream()
    .collect(Collectors.partitioningBy(s -> s.getGrade() >= PASS_THRESHOLD));
```

###### flatMap 사용

중첩 구조를 제거하여 단일 컬렉션으로 만들어준다. (flattening)

```java
List<String> words = List.of("Cat", "Dog");
List<String> uniq = words.stream()
    .map(word -> word.split(""))
    .flatMap(Arrays::stream)
    .distinct()
    .collect(Collectors.toList());
// 결과는 ["C", "a", "t", "D", "o", "g"]

List<Integer> flat = Stream.of(10, 20, 30)
    .flatMap(x -> Stream.of(x, x + 1, x + 2))
    .collect(Collectors.toList());
// 결과는 [10, 11, 12, 20, 21, 22, 30, 31, 32]
```

###### iterate 사용

```java
// 출력
Stream.iterate(0, n -> n + 1)
    .limit(10)
    .forEach(n -> System.out.printf("%d ", n));
// 0 1 2 3 4 5 6 7 8 9

//피보나치 수열
Stream.iterate(new int[]{0, 1}, n -> new int[]{ n[1], n[0] + n[1]})
    .limit(20)
    .forEach(n -> System.out.printf("%d ", n[1]));
// 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597 2584 4181 6765
```

###### 조건걸기

filter 외에, takeWhile , dropWhile 을 사용해 조건을 걸어 필터링할 수 있다.

***filter***

```java
// 모든 항목을 루프하여 조건에 맞는 아이템을 수집한다.
List<Integer> numbers = Stream.of(11, 16, 30, -8, 7, 4, 100)
    .filter(n -> n > 10)
    .collect(Collectors.toList());
// 11, 16, 30, 100
```

***takeWhile***

```java
//순서대로 아이템을 수집하다가 Predicate가 처음으로 false가 나오면 멈춘다(short circuit).
List<Integer> numbers = Stream.of(11, 16, 30, -8, 7, 4, 100)
    .takeWhile(n -> n > 10)
    .collect(Collectors.toList());
// 11, 16, 30
```

***dropWhile***

```java
// Predicate가 처음으로 false를 리턴한 이후로 모두 수집한다. 그 이전은 모두 버린다.
List<Integer> numbers = Stream.of(11, 16, 30, -8, 7, 4, 100, -10)
    .dropWhile(n -> n > 10)
    .collect(Collectors.toList());
// -8, 7, 4, 100, -10
```



#### Spring Boot

###### CommandLineRunner

Applicaiton 구동 시 함께 시작하는 Thread 생성

```java
// ApplicationConfig.java
@Configuration
public class ApplicationConfig {    
    /*    @Bean(name="executor")
    public TaskExecutor taskExecutor() {
        return new SimpleAsyncTaskExecutor();
    }	*/
    @Bean
    public CommandLineRunner schedulingRunner(TaskExecutor executor) {       
        return args -> {
            executor.execute(new MyThread());
        };
    }
}
// MyThread.java
public class MyThread implements Runnable {
    @Override
    public void run() {
        // Thread's work
    }
}
```



#### JPA

###### ##To## Mapping 

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

######  Json , Array 매핑

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



###### Enum Type 추가

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

### JS

###### Declare Date.format 

```javascript
Date.prototype.format = function (f) {
    if (!this.valueOf()) return " ";
    var weekName = ["일요일", "월요일", "화요일", "수요일", "목요일", "금요일", "토요일"];
    var d = this;
    return f.replace(/(yyyy|yy|MM|dd|E|hh|mm|ss|a\/p)/gi, function ($1) {
        switch ($1) {
            case "yyyy":
                return d.getFullYear();
            case "yy":
                return (d.getFullYear() % 1000).zf(2);
            case "MM":
                return (d.getMonth() + 1).zf(2);
            case "dd":
                return d.getDate().zf(2);
            case "E":
                return weekName[d.getDay()];
            case "HH":
                return d.getHours().zf(2);
            case "hh":
                return ((h = d.getHours() % 12) ? h : 12).zf(2);
            case "mm":
                return d.getMinutes().zf(2);
            case "ss":
                return d.getSeconds().zf(2);
            case "a/p":
                return d.getHours() < 12 ? "오전" : "오후";
            default:
                return $1;
        }
    });
};
String.prototype.string = function (len) {
    var s = '', i = 0;
    while (i++ < len) {
        s += this;
    }
    return s;
};
String.prototype.zf = function (len) {
    return "0".string(len - this.length) + this;
};
Number.prototype.zf = function (len) {
    return this.toString().zf(len);
};
```

###### 마우스 클릭 event

```javascript
$("#target").on("mousedown", "td", function (e) {
    if (e.which === 1) {
        //왼쪽 클릭
    } else if (e.which === 3) { 
        //오른쪽 클릭
    }
});
```



###### 우클릭 방지

```javascript
$(document).bind("contextmenu", function(e) {
    e.preventDefault();
});
```



##### kendo

###### Grid 

template of  "databound everyRow"

```javascript
dataBound: function () {
    var grid = this;
    this.tbody.find('tr').each(function () {
        var dataItem = grid.dataItem(this);
        
    });
```



### SQL



###### 하나의 copcode 에서 각 column 들 중  0이 아닌 최신 값 조회



Table  [PRIACE_BASE_INFO]

| COPCODE | BASEPRICE | FUNDFACTOR | TAXFACTOR | RESERVEBASEPRICERATE | TIME             |
| :-----: | --------- | ---------- | --------- | -------------------- | ---------------- |
| 3사업장 | 0         | 0          | 0         | 0                    | 2021-11-08 16:11 |
| 3사업장 | 8190      | 0.037      | 0.1       | 0                    | 1900-01-01 1:01  |
| 3사업장 | 7177      | 0          | 0.1       | 2                    | 2021-11-08 1:00  |



Query

```sql
select copcode
     , basePrice
     , fundFactor
     , NVL(reserveBasePriceRate, 0) AS reserveBasePriceRate
from (
    select COPCODE AS copcode
    , FIRST_VALUE(case when BASEPRICE > 0 then BASEPRICE else NULL end IGNORE NULLS)
	    OVER (order by time desc range between unbounded preceding and unbounded following) AS basePrice
    , FIRST_VALUE(case when fundFactor > 0 then fundFactor else NULL end IGNORE NULLS)
	    OVER (order by time desc range between unbounded preceding and unbounded following) AS fundFactor
    , FIRST_VALUE(case when reserveBasePriceRate > 0 then reserveBasePriceRate else null end IGNORE NULLS)
    	OVER (order by time desc range between unbounded preceding and unbounded following) AS reserveBasePriceRate
    , row_number() over (order by time desc)	AS idx
    from PRICE_BASE_INFO
    where COPCODE = #copcode#
    and NVL(time, to_date('1900-01-01')) <= sysdate
    order by time desc
)
where idx = 1
```

Result

| COPCODE | BASEPRICE | FUNDFACTOR | RESERVEBASEPRICERATE |
| ------- | --------- | ---------- | -------------------- |
| 3사업장 | 7177      | 0.037      | 2                    |



## 기타

#### Oracle DB 에디션 변경

(downgrade Enterprise Edition -> Standard Edition2)

###### 버전 선택

만약 web service 말고 다른 서비스들도 DB 를 조회하고 있다면 같은 버전을 사용하는 것을 권장한다. 

12c 에서 19c 로 버전을 업그레이드 한다면 12c 를 지원하는 Oracle provier 가 19c 에서는 작동을 하지 않기 때문에 새로운 버전을 다운로드하고 옵션을 수정해야하는 일이 생긴다.



```sql
--12c 이후로 기본적으로 db user ID 를 생성할때 C## 을 앞에 붙여야하는 rule이 적용된다.
--userID C## rule 제거
alter session set "_ORACLE_SCRIPT"=true;
```



##### expdp

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


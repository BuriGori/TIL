# 10장 객체지향 쿼리 언어

## 10.1 객체지향 쿼리 소개


EntityManager.find() 메소드를 사용하면 식별자로 엔티티 하나를 조회할 수 있다.
- 식별자로 조회 EntityManager.find()
- 객체 그래프 탐색

하지만, ORM을 사용하면 엔티티 객체를 대상으로 검색하는 방법이 필요하다.
- SQL
    1. 데이터베이스 테이블을 대상으로 하는 데이터 중심의 쿼리
- JPQL
    1. 엔티티 객체를 대상으로 하는 객체지향 쿼리이다.
    2. SQL을 추상화해서 특정 DB의 SQL에 의존하지 않는다.

||JPA가 제공하는 쿼리||
|:---:|:---:|:---:|
|공식|JPQL||
||Criteria 쿼리||
||네이티브 SQL||
|비공식|QueryDSL||
||JDBC 직접 사용||
||MyBatis같은 SQL 매퍼 프레임 워크 사용||

## 10.1.1 JPQL 소개
JPQL(Java Persistence Query Language)은 엔티티 객체를 조회하는 객체지향 쿼리다.
1. JPQL은 SQL을 추상화해서 특정 데이트베이스에 의존하지 않는다.
    - 데이터베이스 방언만 변경하면 JPQL을 수정하지 않아도 자연스럽게 데이터베이스를 변경할 수 있다.
2. JPQL은 SQL보다 간결하다.
    - 엔티티 직접 조회, 묵시적 조인, 다형성 지원으로 SQL보다 코드가 간결하다.

### JPQL 사용 예
``` Java
// 쿼리 생성
String jpql = "select m from Member as m where m.username = 'kim'";
List<Member> resultList =
        em.createQuery(jpql, Member.class).getResultList();
```
getResultList() 메소드를 실행하면 JPA는 JPQL을 SQL로 변환해서 데이터베이스를 조회한다.  
그리고 조회한 결과로 Member 엔티티를 생성해서 반환한다.

### 실행된 SQL
``` SQL
select
    member.id as id,
    member.age as age,
    member.team_id as team,
    member.name as name
from
    Member mbmer
where
    member.name = 'kim'
```
-------
## 10.1.2 Criteria 소개
Criteria는 JPQL을 생성하는 빌더 클래스다.  

### 장점
1. 프로그래밍 코드로 JPQL을 작성할 수 있다.
    - ex) query.select(m).where(...)
2. 컴파일 시점에 오류를 발견할 수 있다.
    - 문자기반 쿼리는 런타임 시점에 발견 가능 -> 문제
3. IDE를 사용하면 코드 자동완성을 지원한다.
4. 동적 쿼리를 작성하기 편하다.

### 단점
1. 모든 장점을 상쇄할 정도로 복잡하고 장황하다.
2. 사용하기 불편하다.
3. 코드를 읽기 어렵다.

### Criteria 쿼리 예
``` JAVA
// Criteria 사용 준비
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> query = cb.creatQuery(Member.class);
//루크 클래스(조회를 시작할 클래스)
Root<Member> m = query.from(Member.class);
//쿼리 생성
CriteriaQuery<Member> cq =
            query.select(m).where(cb.equal(m.get("username"), "kim));
List<Member> resultList = em.createQuery(cq).getResultList();
```

### 메타모델 API
자바가 제공하는 어노테이션 프로세서 기능을 사용하면 어노테이션을 분석해서 클래스를 생성할 수 있다.  
JPA는 이 기능을 사용해서 엔티티 클래스로부터 Criteria 전용 클래스를 생성하는데, 이를 **메타 모델**이라 한다.

    ex) Member 엔티티 클래스로부터 Member_ 라는 Criteria 전용 클래스 생성
    결과) m.get("username")	-> m.get(Member_.username)
------
## 10.1.3 QueryDSL 소개
QueryDSL 도 Criteria 처럼 JPQL 빌더 역할을 한다. JPA 표준은 아니다.  

### 장점
1. 코드 기반이면서 단순하고 사용하기 쉽다.
2. 작성한 코드도 JPQL과 비슷해서 한눈에 들어온다.
3. 스프링 데이터 프로젝트에서 지원할 정도로 기대되는 프로젝트다.

### QueryDSL 예
``` java
//준비
JPAQuery query = new JPAQuery(em);
QMember member = QMember.member;
//쿼리, 결과조회
List<Member> members =
        query.from(member)
        .where(member.username.eq("kim"))
        .list(member);
```
QueryDSL도 어노테이션 프로세서를 사용해서 쿼리 전용 클래스를 만들어야 한다.  
QMember는 Member 엔티티 클래스를 기반으로 생성한 QueryDSL 쿼리 전용 클래스.

-------
## 10.1.4 네이티브 SQL 소개
JPA는 SQL을 직접 사용할 수 있는 기능을 지원하는데 이것을 네이티브 SQL 이라 한다.  
SQL은 지원하지만 JPQL이 지원하지 않는 기능을 사용할 때는 네이티브 SQL을 사용하면 된다.  

### 네이티브 SQL의 단점
- 특정 데이터베이스에 의존하는 SQL 작성
    - 데이터베이스 변경 시 네이티브 SQL도 수정해줘야 한다.

### 네이티브 SQL 예
``` java
String sql = "SELECT ID, AGE, ITEM_ID, NAME FROM MEMBER WHERE NAME = 'kim'";
List<Member> resultList =
    em.createNativeQuery(sql, Member.class).getResultList();
```
------
## 10.1.5 JDBC 직접 사용,MyBatis같은 SQL 매퍼 프레임 워크 사용
JPA는 JDBC 커
넥션을 얻는 API를 제공하지 않기 때문에 JPA 구현체가 제공하는 방법을 사용해야 한다.  
### 하이버네이트 JDBC 흭득
```java
Session session = entityManager.unwrap(Session.class);
session.doWork(new Work() {
    @Override
    public void execute(Connection connection) throws SQLExcetion {
        // work ...
    }
});
```

JPA를 우회해서 데이터베이스에 접근하기 때문에 JPA가 전혀 인식하지 못한다.  
JDBC나 마이바티스를 사용하는 것은 모두 JPA를 우회해서 데이터베이스에 접근한다.  
그럼 영속성 컨텍스트와 데이터베이스를 불일치 상태로 만들어 데이터 무결성을 훼손하는 경우가 발생할 수 있다.  
따라서 JPA와 함께 사용하려면 영속성 컨텍스트를 적절한 시점에 강제로 플러시를 해줘야 한다.

    이런 이슈를 해결하기 위해 SQL을 실행하기 직전에 영속성 컨텍스트를 수동으로 플러시해서 데이터베이스와 영속성 컨텍스트를 동기화해준다.

## 10.2 JQPL

어떤 방법을 사용하든 JPQL(Java Persistence Query Language)에서 모든 것이 시작한다.

1. JPQL은 객체지향 쿼리 언어이다. 테이블 대상이 아닌 엔티티 객체를 대상.
2. JPQL은 SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않는다.
3. JPQL은 결국 SQL로 변환된다.

### 10.2.1 기본 문법과 쿼리 API

1. SQL과 비슷하게 SELECT, UPDATE, DELETE 문을 사용할 수 있다.
    - UPDATE, DELETE 문은 벌크 연산이라고 한다.
2. INSERT 문은 없다. -> EntityManager.persist() 메소드 사용.

JPQL문법
```java
select_문 :: =
    select_절
    from_절
    [where_절]
    [groupby_절]
    [having_절]
    [orderby_절]
update_문 :: = update_절 [where_절]
delete_문 :: = delete_절 [where_절]
```

### SELECT 문
```java
SELECT m FROM Member AS m where m.username = 'Hello'
```

특징
1. 대소문자 구분
    - 엔티티와 속성은 대소문자를 구분한다.
    - SELECT, FROM, AS 같은 JPQL 키워드는 대소문자를 구분하지 않는다.
2. 엔티티 이름
    - JPQL에서 사용한 Member는 클래스 명이 아니라 엔티티명이다.
    - 엔티티명은 @Entity(name="xxx")로 지정할 수 있다.
    - 지정하지 않으면 클래스명이 기본값.
    - 기본값인 클래스명을 엔티티명으로 사용하는 것을 추천
3. 별칭은 필수
    - JPQL은 별칭을 필수로 사용해야 한다.
    - 'AS'는 생략 가능. (=> Member m 처럼 사용해도 된다.)
```java
SELECT username FROM Member m // username -> m.username으로 바꿔줘야 한다.
```

### TypeQuery, Query
1. TypeQuery
    - 반환 타입이 명확한 경우
    - em.createQuery() 메소드의 두 번째 파라미터에 반환할 타입을 지정해준다.
    ```java
    TypeQuery<Member> query =
    em.createQuery("SELECT m FROM Member m", Member.class);

    List<Member> resultList = query.getResultList();
    for (Member member : resultList) {
        System.out.println("member = " + member);
    }
    ```
2. Query
    - 반환 타입이 명확하지 않은 경우
    - SELECT 절의 조회 대상이 하나면 Object를, 둘 이상이면 Object[]를 반환
    ```java
    String query =em.createQuery("select m.username, m.age from Member m");
    List resultList = query.getResultList();

    for( Object o : resultlist){
        Object[] result = (Object[]) o; //결과가 둘 이상이면 Object[]를 반환
        System.out.println("username = "+ result[0]);
        System.out.println("age = "+result[1]);
    }
    ```

### 결과 조회
1. query.getResultList()
    - 만약 결과가 없으면 빈 컬렉션 반환
2. query.getSingleResult()
    - 결과가 정확히 하나일 때 사용
    - 결과가 없으면 예외 발생 : NoResultException
    - 결과가 1개보다 많으면 예외 발생 : NonUniqueResultException

### 10.2.2 파라미터 바인딩
    JDBC 는 위치 기준 파라미터 바인딩만 지원하지만, JPQL은 위치 기준 파라미터 바인딩 + 이름 기준 파라미터 바인딩 지원한다.

1. 이름 기준 파라미터 (메소드 체인 방식)
    - 파라미터 이름 앞에 :를 사용한다.
    ```java
    String usernameParam = "User1";
    List<Member> members = em.createQuery("SELECT m FROM Member m where m.username = :username", Member.class)
    
    query.setParameter("username", usernameParam)
    List<Member> resultList = query.getResultList();
    ```
2. 위치 기준 파라미터(메소드 체인 방식)
    - "?" 다음에 위치값을 주면 된다. 위치값은 1부터 시작한다.
    ```java
    List<Member> members = em.createQuery("SELECT m FROM Member m where m.username = ?1", Member.class)
    
    query.setParameter(1, usernameParam)
    List<Member> resultList = query.getResultList();
    ``` 
    - 위치기준 파라미터 방식보다는 이름 기준 파라미터 바인딩 방식을 사용하는 것이 더 명확하다.

다음과 같이 파라미터 바인딩 방식을 사용하지 않고 JPQL을 수정해서 직접 문자를 더해 만들어 넣으면 SQL 인젝션 공격을 당할 수 있다.
```java
"select m from Member m where m.username = '" + usernameParam + "'"
```
또한 파라미터 바인딩 방식을 사용 시 성능 이슈도 존재한다.
 - 파라미터의 값이 달라도 같은 쿼리로 인식해서 JPQ는 JPQL을 SQL로 파싱한 결과를 재사용할 수 있다.
 - 데이터베이스도 내부에서 실행한 SQL을 파싱해서 사용하는데 같은 쿼리는 파싱한 결과를 재사용할 수 있다.  

결과적으로 애플리케이션과 데이터베이스 모두 해당 쿼리의 파싱 결과를 재사용할 수 있어서 전체 성능이 향상된다. =>파라미터 바인딩 방식은 선택이 아니라 필수!

### 10.2.3 프로젝션
SELECT 절에 조회할 대상을 지정하는 것을 프로젝션(Projection)이라 한다.  
프로젝션 대상  
- 엔티티
- 임베디드 타입
- 스칼라 타입(숫자, 문자 등 기본 데이터)  

(중복제거)DISTINCT, 통계쿼리 등이 가능하다.

### 여러 값 조회
    프로젝션에 여러 값을 선택하면 TypeQuery를 사용할 수 없고 대신 Query를 사용해야 한다.
    조회한 엔티티는 영속성 컨텍스트에서 관리된다.(위에 엔티티 조회도 동일)

```java
List<Object[]> resultList =
    em.createQuery("SELECT o.member, o.product, o.orderAmount FROM Order o")
    .getResultList();
for (Object[] row : resultList) {
    Member member = (Member) row[0];        //엔티티
    Product product = (Product) row[1];     //엔티티
    int orderAmount = (Integer) row[2];     //스칼라
}
```
### NEW 명령어
실제 개발에서 Object[]를 직접 사용하지 않고 DTO 형태의 의미있는 객체로 변환해서 사용.  
샘플 UserDTO
```java
public class UserDTO {
    private String username;
    private int age;
    public UserDTO(String username, int age) {
        this.username = username;
        this.age = age;
    }
    // ...
}
```
NEW 명령어를 사용하면 반환받을 클래스를 지정해줄 수 있는데 이 클래스의 생성자에 JPQL 조회 결과를 넘겨줄 수 있다.  
NEW 명령어를 통해 지정해준 클래스로 TypeQuery를 사용할 수 있어 지루한 객체 변환 작업을 줄일 수 있다.
```java
TypeQuery<UserDTO> query =
    em.createQuery("SELECT new jpabook.jpql.UserDTO(m.username, m.age)
                    FROM Member m", UserDTO.class);
List<UserDTO> resultList = query.getResultList();
```
NEW 명령어 주의사항
1. 패키지 명을 포함한 전체 클래스 명을 입력해야 한다.
2. 순서와 타입이 일치하는 생성자가 필요하다.


### 10.2.4 페이징 API
- 페이징 처리용 SQL은 지루하고 반복적이다.  
- 그리고 더 큰 문제는 데이터베이스마다 페이징 처리 SQL 문법이 다르다는 점이다.
- JPA는 페이징을 두 API로 추상화
    1. setFirstResult(int startPosition) : 조회 시작 위치(0부터 시작)
    2. setMaxResults(int maxResult) : 조회할 데이터 수
```java
TypeQuery<Member> query =
    em.createQuery("SELECT m FROM Member m ORDER BY m.username DESC",
    Member.class);
// 11번째부터 20건의 데이터 조회, 11~30
query.setFirstResult(10);
query.setMaxResult(20);
query.getResultList();
```
    데이터베이스별 페이징 처리는 책의 예제로 나와있다.

### 10.2.7 페치 조인
SQL에서 이야기하는 조인의 종류는 아니고, JPQL에서 성능 최적화를 위해 제공하는 기능이다.
연관된 엔티티나 컬렉션을 한 번에 같이 조회하는 기능, join fetch 명령어로 사용할 수 있다.

```java
페치 조인 ::= [ LEFT [OUTER] | INNER ] JOIN FETCH 조인경로
```

### 엔티티 페치 조인
회원 엔티티를 조회하면서 연관된 팀 엔티티도 함께 조회
```java
select m
from Member m join fetch m.team
```
```sql
//실행되는 SQL
SELECT
    M.*, T.*
FROM MEMBER T
INNER JOIN TEAM T ON M.TEAM_ID = T.ID
```
일반적인 JPQL 조인과 다르게 페치 조인은 별칭을 사용할 수 없다.

JPQL을 사용하는 페치 조인
```java
String jpql = "select m from Member m join fetch m.team";
List<Member> members = em.createQuery(jpql, Member.class)
  .getResultList();
for (Member member : members) {
  //페치조인으로 회원과 팀을 함께 조회 -> 지연로딩 발생 안 함.
  System.out.println("username = " + memrber.getUserName() + ", " +
  "teamname = " + member.getTeam().name());
  ...
  // 출력결과
  username = 회원 1, teamname = 팀A
  username = 회원 2, teamname = 팀A
  username = 회원 3, teamname = 팀B
}
```
회원과 팀을 지연 로딩 설정
- 회원 조회 시 페치 조인을 사용해서 팀을 함께 조회
- 연관된 팀 엔티티는 프록시가 아닌 실제 엔티티.
- 지연 로딩이 일어나지 않는다.
- 실제 엔티티이므로 회원 엔티티가 준영속 상태가 되어도 팀 조회 가능

컬렉션 페치 조인
```java
select t
from Team t join fetch t.members
where t.name = '팀A'
```
```sql
SELECT
    T.*, M.*
FROM TEAM T
INNER JOIN MEMBER M ON T.ID = M.TEAM_ID
WHERE T.NAME = '팀A'
```
팀을 조회하면서 페치 조인을 사용해서 연관된 회원 컬렉션(t.members)도 함께 조회한다. -> 지연 로딩이 발생하지 않는다.

컬렉션 페치 조인 사용
```java
String jpql = "select t from Team t join fetch t.members where t.name = `팀A`";
List<Team> teams = em.createQuery(jpql, Team.class).getResultList();
for(Team team: teams){
    System.out.println("teamname = " + team.getName() + ", team = " + team);
    for(Member member : team.getMembers()){
        System.out.println("->username = " + member.getUsername() + ", member = " + member);
    }
}
//출력결과
teamname = 팀A, team = Team@0x100
->username = 회원1, member = Member@0X200
->username = 회원2, member = Member@0X300
teamname = 팀A, team = Team@0x100
->username = 회원1, member = Member@0X200
->username = 회원2, member = Member@0X300
```

중복제거(DISTINCT)를 사용하게 되면
``` sql
select distinct t //팀엔티티의 중복을 제거하라는 것
from Team t join fetch t.members
where t.name = '팀A'

//출력결과
teamname = 팀A, team = Team@0x100
->username = 회원1, member = Member@0X200
->username = 회원2, member = Member@0X3
```
    일반조인의 경우는 연관간계까지 고려하지 않기 때문에 SQL문이 반복되거나
    더 실행되는 결과를 만들 수 있음

### 페치 조인의 특징과 한계
특징
1. 페치 조인을 사용하면 SQL 한 번으로 연관된 엔티티들을 함께 조회할 수 있어서 SQL 호출 횟수를 줄여 성능을 최적화할 수 있다.
2. 글로벌 로딩 전략보다 우선한다.
    - 예를 들어 글로벌 로딩 전략을 지연 로딩으로 설정해도 JPQL에서 페치 조인을 사용하면 페치 조인을 적용해서 함께 조회한다.
    - ex) OneToMany(fetch = FetchType.LAZY) // 글로벌 로딩 전략
```
최적화를 위해 글로벌 로딩 전략을 즉시 로딩으로 설정하면 애플리케이션 전체에서 항상 즉시 로딩이 일어난다.
일부는 빠를 수 있지만 전체로 보면 사용하지 않는 엔티티를 자주 로딩하므로 오히려 성능에 악영향을 미칠 수 있다.
따라서 글로벌 전략은 될 수 있으면 지연 로딩을 사용하고, 최적화가 필요하면 페치 조인을 적용하는 것이 효과적이다.
```
3. 연관된 엔티티를 쿼리 시점에 조회하므로 준영속 상태에서도 객체 그래프를 탐색할 수 있다.

한계
1. 별칭을 줄 수 없다.
    - 따라서 SELECT, WHERE 절, 서브 쿼리에 페치 조인 대상을 사용할 수 없다.
2. 둘 이상의 컬렉션을 페치할 수 없다.
3. 컬렉션 페치 조인 시 페이징 API를 사용할 수 없다.
    - 컬렉션(일대다)이 아닌 단일 값 연관 필드(일대일, 다대일)들은 페치 조인을 사용해도 페이징 API 사용 가능
    - 하이버네이트에서 컬렉션 페치 조인하고 페이징 API 를 사용하면 경고 로그 남기며 메모리에서 페이징 처리 => 데이터가 많으면 성능 이슈, 메모리 초과 예외 발생 가능해서 위험

-------
### 10.2.15 Named 쿼리 : 정적 쿼리
- 어플리케이션 로딩 시점에 JPQL 문법을 체크하고 미리 파싱해준다.
- 오류를 빨리 확인할 수 있고, 사용하는 시점에는 파싱된 결과를 재사용.
- 성능상 이점을 가져올 수 있다.
- @NamedQuery 어노테이션이나 XML에 작성 가능

Named 쿼리를 어노테이션에 정의
```java
// 정의
@Entity
@NamedQuery(
  name = "Member.findByUsername",
  query = "select m from Member m where m.username = :username")
public class Member {
    ...
}
// 사용
List<Member> resultList = em.createNamedQuery("Member.findByUsername"),
            Member.class)
  .setParameter("username", "회원1")
  .getResultList();
```

2개 이상의 Named 쿼리 정의
@NamedQueries 어노테이션 사용
```java
@Entity
@NamedQueries({
    @NamedQuery(
        name = "Member.findByUsername",
        query = "select m from Member m where m.username = :username"),
    @NamedQuery(
        name = "Member.count"
        query = "select count(m) from Member m")
})
public class Member { ... }
```

    참고 사이트(중간 조건식, 서브쿼리, 전체내용 등등)
https://sun-22.tistory.com/78?category=363030 //조건식, 서브쿼리  
https://ugo04.tistory.com/122 //경로표현식
https://ykh6242.tistory.com/91?category=981344 // 전체 내용 이해
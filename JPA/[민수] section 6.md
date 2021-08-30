# 6장 다양한 연관관계 맵핑

## **-5장에 대한 복습-**

    다양한 연관관계 맵핑에 대해서 들어가기 앞서 연관관계의 기초인 5장에 내용을 정리하고 넘어가야 한다.

- 다중성
- 단방향, 양방향
- 연관관계의 주인

### **1.다중성**
    다중성을 판단하기 어려울 때는 반대의 경우를 생각해보면 편하다.
    (일대다의 반대는 다대일과 같은 경우, 일대일의 반대는 일대일이다.)
    But, 다대다 관계는 실무에서 거의 사용하지 않는다.

### **2.단방향, 양방향**
- 테이블의 경우  
    외래 키 하나로 조인을 사용해 양방향으로 퀄가 가능하므로 사실상 방향성이라는 개념이 없음  

- 객체의 경우  
    참조용 필드를 가지고 있는 객체만 연관된 객체를 조회할 수 있다. 그렇기에 한 쪽만 참조하는 것을 단방향 관계라 하고 양쪽이 서로 참조하는 것을 양방향 관계라 한다.

### **3.연관관계의 주인**
    데이터 베이스는 외래 키 하나로 두 테이블이 연관관계를 맺기에 연관관계를 관리하는 포인트는 1개이다.
    반면에 엔티티를 양방향으로 맵핑하면 (A -> B, B -> A)2곳에서 서로를 참조하기에 관리 포인트는 2개가 된다.!!

- JPA의 경우  
두 객체중 하나의 객체를 정해서 데이터 베이스를 관리하는 연관관계의 주인으로 만든다. 보통 외래키를 가진 테이블과 매핑한 엔티티가 외래키를 관리하는게 효육적이므로 연관관계의 주인으로 만든다.


|연관관계의 주인|연관관계의 주인이 아님|
|:---:|:---:|
|mappedby 속성을 사용하지 않음|mappedby속성을 사용함(주인 필드 이름을 값으로 입력해야함)|
|외래 키를 수정 및 읽기 가능|외래 키를 수정할 수 없고 읽기만 가능|

## **-다양한 연관관계-**

    다중성과 단뱡향,양방향을 고려한 모든 연관관계이다.(참고로 왼쪽을 연관관계의 주인으로 정했다.)
|---|연관관계의 주인|방향|
|:---:|:---:|:---:|
|다대일|다|단방향, 양방향|
|일대다|일|단방향, 양방향|
|일대일|일|주 테이블 단방향, 양방향|
|일대일|일|대상 테이블 단방향, 양방향|
|다대다|다|단방향, 양방향|


## **1.다대일**
- JPA에서 가장 많이 사용하고 꼭 알아야 되는 다중성이다.  
- 데이터베이스 테이블의 일(1), 다(N) 관계에서는 외래 키는 항상 다쪽에 있다.
### **다대일 단방향[N:1]**
--------

``` java
//회원 엔티티
@Entity
@NoArgsConstructor
@ToString(exclude = "team")
@Getter @Setter
public class Member {

    @Id
    @Column(name = "MEMBER_ID")
    private String id;

    private String username;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team; //팀의 참조를 보관

    //getter, setter..
}
```
``` java
//팀 엔티티
@Entity
@NoArgsConstructor
@Getter @Setter
public class Team {

    @Id
    @Column(name = "TEAM_ID")
    private String id;

    private String name;

    //getter, setter ..
}
```
![image](https://user-images.githubusercontent.com/68093714/131366634-d7656aad-fca6-410e-98d3-110d4961130a.png)

회원은 Member.team으로 팀 엔티티를 참조할 수 있다. 반대로 팀에는 회원을 참조하는 필드가 없기 때문에 다대일 단방향 연관관계이다. 특히
``` java
@ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team; //팀의 참조를 보관
```
@JoinColumn(name = "TEAM_ID")를 사용해서 Member.team필드를 TEAM_ID 외래 이와 매핑했기 때문에 Member.team 필드로 회원 테이블의 TEAM_ID를 관리한다.
<br/><br/><br/>

### **다대일 양방향[N:1, 1:N]**
--------
``` java
//회원 엔티티
@Entity
@NoArgsConstructor
@ToString(exclude = "team")
@Getter @Setter
public class Member {

    @Id
    @Column(name = "MEMBER_ID")
    private String id;

    private String username;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team; //팀의 참조를 보관

    //회원의 편의 메소드
    public void setTeam(Team team){
        
        this.team = team;
        //무한루프에 빠지지 않도록 체크합니다.
        if(!team.getMembers().contains(this)){
            team.getMembers().add(this);
        }
    }
    //getter, setter..
}
```
``` java
//팀 엔티티
@Entity
@NoArgsConstructor
@Getter @Setter
public class Team {

    @Id
    @Column(name = "TEAM_ID")
    private String id;

    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

    //팀의 편의 메소드
    public addMember(Member member){
        this.members.add(member);
        if(member.getTeam() != this){ // 무한루프에 빠지지 않도록 체크함
            member.setTeam(this);
        }
    }
}
```
![image](https://user-images.githubusercontent.com/68093714/131366580-a24c1195-d794-4c13-8212-7b3972025da4.png)

- 양방향은 외래 키가 있는 쪽이 연관관계의 주인이다.

    JPA는 외래 키를 관리할 때 연관관계의 주인만 사용한다. 주인이 아닌 Team.members는 조회를 위한 JPQL이나 객체 그래프를 탐색할 때 사용한다.

- 양방향 연관관계는 항상 서로를 참조해야 한다.

    서로를 참조하게 하려면 연관관계 편의 메소드를 작성하는 것이 좋은데 현재 setTeam(), addMember() 메소드가 이런 메소드들이다. 편의 메소드는 한곳에만 작성하거나 양쪽 다 작성할 수 있는데, 양쪽에 다 작성하면 무한 루프에 빠지므로 주의해야 한다. 예제 코드는 편의 메소들를 양쪽에 다 작성해서 둘 중 하나만 호출하면 된다.
## **2.일대다**
- 일 쪽이 연관관계의 주인이 되는 경우이다.
- 표준스펙에서 지원은 하지만 실무에서 이 모델을 권장하지 않는다.
### **일대다 단방향[1:N]**
-----
``` java
//팀 엔티티
@Entity
@Getter @Setter
public class Team {

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private String id;

    private String name;

    @OneToMany
    @JoinColumn(name = "TEAM_ID") //MEMBER 테이블의 TEAM_ID (FK)
    private List<Member> members = new ArrayList<>();
    //getter, setter..
}
```
``` java
//회원 엔티티
@Entity
@Setter
@Getter
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private String id;

    private String username;
    //getter, setter..
}
```
![image](https://user-images.githubusercontent.com/68093714/131366919-405fddfa-c50c-4dbd-b5a3-6dd710aeadfb.png)

    일대다 단방향 관계에서는 특이한 부분이 있는데, 테이블에서는 Member에 TEAM_ID라는 외래 키를 관리하지만
    객체에서는 Team에 members로 참조 필드가 있다는 것이 특징이다.
일대다 단방향 관계를 매핑할 때는 @JoinColumn을 명시해야한다. 그렇지 않으면 JPA는 연결 테이블을 중간에 두고 연관관계를 관리하는 조인 테이블 전략을 기본으로 사용해서 매핑한다. 조인 테이블은 7.4절에서 만나요~
- 일대다 단방향 매핑의 단점  
매핑한 객체가 관리하는 외래 키가 다른 테이블에 있다는 점이다! 왜냐하면 본인 테이블에 있다면 한번의 SQL문(INSERT)으로 끝낼 수 있지만, 다른 테이블에 있다면 추가적인 SQL문(UPDATE)을 실행해줘야한다.
- 일대다 단방향 매핑 보다는 다대일 양방향 매핑을 사용하자
위에서 나온 단점처럼 다른 테이블에 있는 외래 키를 관리하는 것과 같이 성능, 관리면에서 부담스러운 문제이다. 상황에 따라 다르지만 다대일 양방향 매핑을 권장한다.
<br/>

                    **일대다 단방향 매핑의 단점이 생기는 이유**
        객체와 테이블 연관관계에서 비는 부분을 SQL문으로 채워야 한다는 것이 핵심!!
<br/><br/><br/>

### **일대다 양방향[1:N, N:1]**
-----
``` java
//팀 엔티티
@Entity
@Getter @Setter
public class Team {

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private String id;

    private String name;

    @OneToMany
    @JoinColumn(name = "TEAM_ID") //MEMBER 테이블의 TEAM_ID (FK)
    private List<Member> members = new ArrayList<>();
    //getter, setter..
}
```
``` java
//회원 엔티티
@Entity
@Setter
@Getter
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private String id;

    private String username;
    
    @ManyToOne
    @JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)
    private Team team;
    
    //getter, setter..
}
```
![image](https://user-images.githubusercontent.com/68093714/131370705-03344507-53fb-49f7-b5de-ddde16e1d6ea.png)

    일대다 양방향 맵핑은 존재하지 않는다.(대부분의 코딩처럼 가능은 하다 like 위에 있는 코드)
    왜냐하면 연관관계의 주인은 항상 '다' 에게 있으므로 @ManyToOne에는 mappedBy 속성이 없기 때문이다.

- 위 코드 처럼 회원 엔티티에 TEAM_ID를 매핑했다. 이 방법의 단점으로는 다음과 같다.
    1. 같은 외래 키를 관리하므로 문제가 발생한다.
    2. @JoinColumn에 설정으로 읽기만 가능하게 만든 형태이기 때문에 단방향 매핑이 갖는 단점을 그대로 유지한다.
    3. 딴생각말고 다대일 양방향 매핑을 사용하자

## **3.일대일**


## **4.다대다**
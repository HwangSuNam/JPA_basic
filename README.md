# 1. JPA 시작하기
### 1. 주의
- 엔티티 매니저 팩토리는 하나만 생성해서 어플리케이션 전체에서  공유
- 엔티티 매니저는 쓰레드간 공유하면 안 됨(사용하고 버려야 한다.)
- JPA의 모든 데이터 변경은 트랜잭션 안에서 실행

### 2. JPQL
- 엔티티 객체를 대상으로 검색(객체 지향 쿼리 언어)

# 2. JPA 소개
### 1. JPA의 패러다임 불일치 해결
- 상속
- 연관관계
- 객체 그래프 탐색(신뢰할 수 있는 엔티티, 계층)
- 비교하기

### 2. JPA 성능 최적화 기능
- 1차 캐시와 동일성 보장
- 트랜잭션을 지원하는 쓰기 지연(insert랑 update, write behind)
- 지연 로딩

# 3. 영속성 관리 - 내부 동작 방식
### 1. 엔티티 생명주기
- 비영속(new/transient) : 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
- 영속(managed) : 영속성 컨텍스트에 관리되는 상태
- 준영속(detached) : 영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제(removed) : 삭제된 상태

### 2. 영속성 컨텍스트의 이점
- 1차 캐시
	- insert하기 전, 캐시에서 조회(1차 캐시에서 조회)
	- 동일한 건을 find 2번 하면 1번째 만 DB에서 조회(데이터베이스에서 조회)
- 영속 엔티티 동일성 보장
- 엔티티 등록, 트랜잭션을 지원하는 쓰기 지연
	- 영속성 컨텍스트에는 1차 캐시 영역 말고도 쓰기 지연 SQL 저장소가 존재하며 여기에 버퍼처럼 쌓임
	- 이후, commit() 순간, flush() 수행
- 엔티티 수정, 변경 감지(더티 체킹)
	- 1차 캐시에 ID(pk)에 해당하는 엔티티(setter로 바뀔 수 있음) 뿐만 아니라 스냅샷(최초 상태)을 떠둔다.
	- commit(flush()) 순간에 엔티티와 스냅샷을 비교하여 update 쿼리를 만들어 쓰기 지연 저장소에 저장한다.
	- 이후, 영속성 컨텍스트의 flush()를 수행하면서 update 및 commit이 수행됨
	- 엔티티 삭제(em.remove()) : 트랜잭션 커밋 시점에 delete()가 수행됨
- 지연 로딩

### 3. 플러시
- DB 트랜잭션이 발생하면 수행됨
	- 변경 감지(더티 체킹) 수행
	- 수정된 엔티티 쓰기 지연 SQL 저장소에 등록
	- 쓰기 지연 SQL 저장소의 쿼리를 데이터 베이스에 전송(등록, 수정, 삭제 쿼리)
- **1차 캐시를 지우는 것이 아니라 쓰기 지연 SQL저장소만 반영되는 과정**
	- 영속성 컨텍스트를 비우는 것이 아니다!

### 4. 플러시 하는 방법
- em.flush() - 수동
- 트랜잭션 커밋 - 자동
- JPQL 쿼리 실행 - 자동
	- JPQL을 실행하기 전에 앞선 persist() 관련 쿼리에 대한 반영을 해야만 하니까


### 5. 준영속(detached) 상태로 만드는 법
- 영속성 컨텍스트에서 다시 분리, 삭제하는 것. 그래서 더티 체킹 쓰기 지연 같은 것을 이용 못 함
- em.detach()
- em.clear() : 전체를 지워버리는 것
- em.close() : 영속성 컨텍스트를 종료하는 것

# 4. 엔티티 맵핑
###  1. @Entity
- 기본 생성자 필수(파라미터가 없는 public or protected)
- final 사용 금지
- name 프로퍼티는 jpa에서 사용하는 이름을 변경하는 것

### 2. @Table
- 사용하는 테이블 명 변경
- uniqueConstraints, indexes 등을 여기서 설정할 수 있음

### 3. DDL 생성 기능
- @Column(unique = true, length = 10)은 DDL을 자동 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않는다.

### 4. 맵핑 정리
- @Column : 컬럼 매핑
	- length : 길이
	- updatable = false : 어플리케이션 상에서 업데이트 못함
	- nullable = false : null 못 들어감
	- unique = true :유니크 제약 조건, 잘 안쓴다.. 제약조건 이름이 랜덤이라 못 씀
		- @Table(uniqueConstraints = "unique~~")라고 쓴다. 그러면 제약 조건 명이 정해짐
	- columnDefinition : 개발자가 직접 컬럼 정보를 줄 수 있다.
		- columnDefinition = "varchar(100)  default 'EMPTY'
	- precision, scale : BigDecimal 또는 BigInteger 타입에서 사용, presicsion은 소수점을 포함한 전체 자릿수, scale은 소수의 자릿수
- @Temporal(TemporalType.xxx) : 날짜 타입 매핑, TIMESTAMP...
	- 최신 버전은 LocalDate, LocalDateTime을 쓰면 사용할 필요 없다.
- @Enumerated(EnumType.STRING) : ORDINAL은 숫자로 들어가기 때문에 절대 사용하지 않는다.
- @Lob : 길이 제한 없음(BLOB, CLOB 매핑)
	- 타입이 문자(String, char[]..)이면 CLOB(byte..) 나머지는 BLOB
- @Transient : 특정 필드를 컬럼에 매핑하지 않음(매핑 무시)

### 5. 기본키 맵핑
- 직접 할당은 @Id만 사용
- 자동 생성은 @GeneratedValue도 사용
	- strategy = GenerationType.AUTO는 방언에 맞춰서 TABLE, SEQUENCE, IDENTITY 선택됨
		- IDENTITY : 기본키를 데이터베이스에 위임, MySQL의 auto_increment와 같은..
			- id값을 알 수 있는 시점은 insert하는 순간이다.
			- ** 그러므로 이 케이스만 특이하게 persist()하는 순간 insert가 수행된다.
		- SEQUENCE : 문자면 안되고 숫자 타입만 됨, Long인 랩핑 클래스를 사용해야 함
			- 테이블마다 시퀀스를 따로 관리하고 싶으면 @SequenceGenerator를 사용한다.
			- persist() 순간에 시퀀스로부터 id를 얻는 쿼리가 수행된다.
			- 한 쓰레드 안에서 시퀀스를 여러개 사용하면 네트워크 통신이 많이 일어나서 성능상 문제가 생긴다.
				- allocationSize = 50를 사용
				- 처음 두번 call next가 일어나는데, 첫 번째는 더미(0->1), 두번 째(1 -> 51)을 실행하여 app이 메모리로 가지고 있으면서 set한다.
		- TABLE : auto_increment, 시퀀스를 사용하지 않고 키 생성 전용 테이블을 만들어서 시퀀스 흉내를 냄, 성능 별로 안 좋음
			- SEQUENCE와 마찬가지로 allocationSize로 성능 최적화를 할 수 있다.

# 5. 연관관계 맵핑 기초
### 1. 객체와 테이블의 연관관계의 차이
- 테이블은 외래 키 조인으로 연관된 테이블을 찾는다.
- 객체는 참조를 사용해서 연관된 객체를 찾는다.
- 테이블과 객체 사이에는 이런 큰 간격이 있다.
=> 객체의 참조와 DB의 외래키를 맵핑해서 연관관계를 정리할 수 있다.

### 2. 객체와 테이블이 관계를 맺는 차이
- 객체는 연관관계 2개(회원 -> 팀, 팀 -> 회원), 테이블은 연관관계 1개(회원 <-> 팀)
- 객체의 양방향 관계는 서로 다른 단방향 관계 2개
- *** 연관관계의 주인은 외래키를 관리해야하고 FK를 IUD할 수 있는 객체이다.(주인의 반대편은 읽기밖에 안됨)

### 3. 연관관계 주인
- 외래키가 있는 곳(다 쪽)을 주인으로 정해라
- 주인이 아닌 쪽에 add만 하면 PK 값은 NULL이 됨
	- 양방향 맵핑시 가장 많이 하는 실수, 연관관계의 주인에 값을 입력하지 않음
- mappedBy의 name으로 설정되어 있는 것이 연관관계 주인

### 4. 연관관계 메소드는 어느 쪽에 설계해도 상관없다.
	- 개발이 편한 쪽으로 하자
 
### 5. 순수한 객체 관계를 고려하면 항상 양 쪽 다 값을 입력해야 한다.

### 6. 실제 설계할 때는
- 우선, 단방향 매핑만으로 한다.
	- 양방향으로 해서 좋을게 없음..
	- * JPQL에서 역방향으로 탐색할 일이 많음
		- 이 때,(실제 개발할 때) 필요성을 느끼면 추가하면 됨(테이블에 영향을 주지 않으니까)

** 연관관계도 잘 끊어내는 것도 중요
- 원래 맞는 것은 Member -> Order 이다. Member에서 Order를 찾아가는 것은 너무 오버 스펙 설계이다. 그러므로 단방향이 원래 올바르다.

# 6. 다양한 연관관계 맵핑
### 1. 다대일
- 다 쪽에 연관관계 설정
- 단방향 : 다 쪽에만 프로퍼티 설정하면 됨
- 양방향 : 일 쪽에서 프로퍼티 설정(mapped by)
- 단방향이든 양방향이든 테이블의 변화는 없다.

### 2. 일대다 단방향(실무에서는 거의 안 씀)
- 일이 연관관계 주인, 그런데 테이블은 다 쪽에 외래키가 있음
	- 객체와 테이블의 차이 때문에 반대편 테이블의 외래 키를 관리하는 특이한 구조
- 일대다,  team.getMembers().add(member);함으로 써 팀 테이블에 데이터를 넣는게 아닌게 된다.
	- ** 그러므로 UPDATE 쿼리가 하나 더 실행된다.
	- 이렇게 되면 실무에서 운영이 힘들어진다.
	- 다대일, 필요하면 다대다로 설계한다.
- @JoinColumn은 꼭 넣어야 함, 안 넣으면 조인 테이블이 생김

### 3. 일대다 양방향(실무에서는 거의 안 씀)
- 공식적인 스펙이 아님
- 일대다 단방향 설정 후, 다 쪽에 @ManyToOne, @JoinColumn(name = "team_id", insertable = false, udpatable = false) 선언
	- 읽기 전용 필드를 사용해서 양방향처럼 사용하는 방법임
- 이렇게 쓰지말고 다대일 양방향 사용하자

### 4. 일대일
- unique 제약조건 설정
- 다대일 단방향 매핑과 거의 비슷
- 대상 테이블에 외래키 단방향 불가능(일대다처럼 외래키를 다른 테이블에서 관리할 수 없음)
- 대상 테이블에 외래키 양방향은 사실상, 일대일 주테이블에 외래키 양방향과 동일한 맵핑방법
- ** 주 테이블에 외래키 : 개발자들이 선호 그러나 값이 없으면 null이 들어갈 수도 있음
- ** 대상 테이블에 외래키 : DBA가 선호
	- 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때, 테이블 구조 유지
	- 프록시의 한계로 지연 로딩으로 설정해도 즉시 로딩으로 성능 문제 있을 수 있음, 이해가 잘 안가네..

### 5. 다대다
- 실무에서 사용 못 함
- 주문 시간, 수량 같은 데이터가 들어올 수 있음
- 쿼리도 이상하게 수행될 수 있음
- 다대다 한계 극복 : 연결 테이블용 엔티티 추가(연결 테이블을 엔티티로 승격)
	- @ManyToMany -> @ManyToOne, @OneToMany

###  6. @JoinColumn
- name : 매핑할 외래키 이름
- referencedColumnName : 외래 키가 참조하는 대상 테이블의 컬럼명
- foreignKey(DDL) : 외래키 제약 조건을 직접 지정, 이 속성은 테이블 생성할 때문 사용한다.

# 7. 상속관계 맵핑
### 1. 상속관계 전략
- 조인 전략 : PK는 각 테이블에 FK로 상속 후, DTYPE을 이용해 종류 결정
	- @Inheritance(strategy = InheritanceType.JOINED)
	- 하위 테이블을 persist만 해도 알아서 상위 테이블에 insert까지 수행함(총 2회 insert)
	- 상위 테이블은 @DiscriminatorColumn, 하위 테이블은 @DiscriminatorValue("XX")을 선언해서 Item 테이블만 보아도 어떤 종류의 하위 아이템인지 판단할 수 있도록 하자
- 단일 테이블 전략 : 한 테이블에 다 때려 넣기
	- 기본값임, @Inheritance(strategy = InheritanceType.SINGLE_TABLE)
	- 조인 전략과 마찬가지로 상위 테이블은 @DiscriminatorColumn, 하위 테이블은 @DiscriminatorValue("XX") 사용
	- @DiscriminatorColumn을 선언 안해도 DTYPE이 강제로 생성되긴 함
** 조인 전략 <-> 단일 테이블 전략 바꾸는데 strategy만 바꾸면 되므로 상당히 편리하다

- 구현 클래스마다 테이블 생성 전략(쓰면 안되는 전략)
	- InheritanceType.TABLE_PER_CLASS
	- @DiscriminatorColumn 선언할 필요 없음
	- 부모 타입으로 조회 em.find(Item.class, id);의 경우, 모든 하위 테이블을 union하여 상당히 비효율적으로 쿼리가 실행됨

###  2. 반복되는 프로퍼티
- 공통으로 사용하는 매핑 정보를 모으는 역할
- base 클래스 만들고 @MappedSuperclass를 선언, 그러면 이 클래스를 상속받음으로써 적용할 수 있음
- 직접 생성해서 사용할 일이 없으므로 추상 클래스로 만들어야 한다.
- createdBy, lastModfiedDate 등은 나중에 스프링 데이터 jpa에서 세션을 이용해 어노테이션을 적용하여 자동화하여 set할 수 있음
** 참고로 @Entity 클래스는 @Entity로 선언된 엔티티 클래스나 @MappedSuperclass를 선언한 클래스만 상속 가능

# 8. 프록시와 연관관계 관리
###  1. 프록시
- em.find() vs em.getReference()
- em.getReference() : 데이터베이스 조회를 미루는 가짜(프록시) 엔티티 객체 조회
	- find는 진짜 객체를 리턴하는 반면, getReference는 proxy 객체(빈 껍데기)를 리턴
		- 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 진짜 객체 반환
	- 호출하는 시점에 데이터베이스에 쿼리 실행 안 함
	- getId()는 파라미터로 넣었기 때문에 쿼리를 수행 안 한거고 다른 정보를 가져올 때는 그 시점에 쿼리 실행
- 프록시 특징
	- 실제 클래스를 상속받아서 만들어짐
	- 실제 클래스와 겉 모양이 같다.
	- Entity target 프로퍼티가 내부 필드로 있고 실제 객체를 참조한다.
		- ** 프록시의 getName()을 호출하면 target.getName()이 호출되면서 실제 객체의 메소드가 호출 됨
		- ** 만약 target이 참조하는 객체가 없으면 영속성 컨텍스트를 통해서 초기화 요청, 이후 DB 쿼리 실행하고 target의 참조가 생김
	- 사용하는 입장에서는 진짜 객체인지 프록시 객체인지 구분하지 않고 사용하면 됨
	- 프록시 객체는 처음 사용할 때, 한 번만 초기화
	- *** 타입 비교는 프록시가 넘어올 수도 있으므로 ==이 아니라 intanceof(상속까지 체크)로 해야 한다.
	- ** 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 em.getReference()를 호출해도 실제 엔티티 반환
		- ** 이미 영속성 컨텍스트에 리얼 엔티티가 있다면 em.getReference()를 호출해도 jpa는 한 트랜잭션 안에서 같은 id로 컨텍스트로부터 가져와야하기 때문에 같은 객체이다(==이 true)
		- ** 그 반대도 마찬가지, 이미 영속성 컨텍스트에 프록시 엔티티가 있다면 em.find()를 호출해도 프록시로 객체를 반환한다.
		- 마찬가지로 같은 id로 프록시를 2번 가져오면 같은 프록시 객체를 반환한다.
	- 영속성 컨텍스트의 도움을 받을 수 없는 detached 상태일 때, 프록시를 초기화하면 문제 발생(실무에서 굉장히 많이 발생함)
		- detache(), close() 등 호출 후, getName() 호출

- 프록시 인스턴스의 초기화 여부 확인
	- emf.getPersistenceUnitUtil.isLoaded(Object entity);
- 프록시 강제 초기화
	- getName() 쓰지 않고 Hibernate.initialize(entity);
	- 하이버네이트가 제공해주는 것이고 JPA 표준은 없음

###  2. 즉시 로딩과 지연 로딩
- 지연 로딩(lazy) : proxy를 이용하여 셋팅하고 실제 프로퍼티를 get하면 쿼리 실행하고 프록시 초기화됨
- 즉시 로딩(eager) : proxy를 이용하지 않고 실제 엔티티를 이용, 바로 가져오기 때문
- ** 실무에서는 모든 연관관계에 지연 로딩을 사용해야 한다!
	- 즉시 로딩은 예상치 못한 SQL 발생
	- 즉시 로딩은 JPQL에서 N+1 문제를 일으킨다.
		- ex) 전체 select 쿼리 조회, select m from Member m -> m의 데이터 개수 만큼 team 쿼리 수행
		- * 해결 방법 : fetch join(동적으로 가져오는 방법), 엔티티 그래프(어노테이션 이용), 배치 사이즈??
	- @ManyToOne, @OneToOne은 디폴트가 EAGER -> 반드시 LAZY로 설정
	
###  3. 영속성 전이(CASCADE)
- 특정 엔티티를 영속 상태로 만들 때, 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때
- CascadeType.ALL
- 연관관계 매핑하는 것과 아무 관련이 없음
- 소유자가 하나일 때, 라이프 사이클이 똑같을 때 만 사용한다.
	- 이럴 경우 절대 사용하면 안됨 Parent -> Childre <- Team
		- 만약 Team을 이용해서 remove하면 Parent 입장에서는 곤란한 경우가 발생할 수 있다.

###  4. 고아 객체
- 고아 객체 제거 : 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제
- orpahnRemoval = true
	- ex) parent.getChild().remove(0); => delete from child where id = ?
- 참조하는 곳이 하나일 때 사용해야 함
- **특정 엔티티가 개인 소유할 때 사용
- 부모가 지워지면 CascadeType.REMOVE처럼 동작하여 자식들 모두가 지워진다.

###  5. CASCADE + 고아 객체(CascadeType.ALL + orpahnRemoval = true)
- 두 옵션을 모두 활성화하면 부모 엔티티를 통해서 자식의 생명주기를 관리할 수 있음
- 도메인 주도 설계(DDD)의 Aggregate Root 개념을 구현할 때, 유용
	- Parent가 Aggregate Root, Child는 Aggregate Root가 관리하는 애
	- DAO, Repository가 필요없게 되는 상황이 되니까

# 9. 값 타입
###  1. JPA의 데이터 타입 분류
- 엔티티 타입
	- 안에 데이터 프로퍼티가 변해도 식별자로 지속해서 추적 가능
- 값 타입
	- int, Integer, String처럼 단순히 값으로 사용하는 자바 기본 타입이나 객체
	- 식별자가 없고 변경시 추적 불가능

###  2. 값 타입
- 기본 값 타입
	- 자바 기본 타입 : int, double
	- 래퍼 클래스 : Integer, Long
	- String
- 임베디드 타입 : 복합 값 타입
- 컬렉션 값 타입 : List..

###  3. 기본 값 타입
- 생명 주기를 엔티티의 의존
- 값 타입은 공유하면 안되고 공유 자체도 안됨
	- 래프 클래스나 String은 공유는 가능하지만 변경은 안 됨

###  4. 임베디드 타입
- 새로운 값 타입을 직접 정의할 수 있음
- 주로 기본 값 타입을 모아서 만들어서 복합 값 타입이라고도 함
- 값 타입 정의하는 곳에 @Embeddable, 값 타입을 사용하는 곳에 @Embedded
- ** 기본 생성자 필수!
- 재사용 가능하고 응집도 높음, 응집도가 높으므로 isWork()처럼 의미있는 메소드를 만들 수 있음
- 임베디드 타입은 엔티티를 가지고 있게 설계할 수도 있음
	- 일반 엔티티가 foreinKey를 가지고 있는 것과 마찬가지 상황임
- @Column 같은 것도 다 됨
- 한 엔티티 내에서 같은 값 타입을 사용한다면?
	- 컬럼명이 중복되는 문제 발생
	- @AttributeOverrides, @AttributeOverride을 이용해서 재정의하여 해결
		- ex) @Embedded
    			@AttributeOverrides({
            			@AttributeOverride(name = "city", column = @Column(name = "WORK_CITY")),
           			@AttributeOverride(name = "street", column = @Column(name = "WORK_STREET")),
            			@AttributeOverride(name = "zipcode", column = @Column(name = "WORK_ZIPCODE"))
    			})

###  5. 값 타입과 불변 객체
- 값 타입의 실제 인스턴스인 값을 공유하는 것은 위험
- 임베디트 타입은 사이트 이펙트 발생 => member1.getHomeAddress().setCity("newCity");
	- member1, member2는 각각 HomeAddress 객체를 생성하여 set하는 방식을 써야 한다.
- 하지만 코딩상으로 막을 수 있는 방법이 없다. == 객체의 공유 참조는 피할 수 없다.
- ** 값 타입은 불변 객체(Immutable object)로 설계해야 함
	- 생성자로만 값을 설정하고 수정자를 만들지 않으면 됨(getter만 남겨두고 setter는 모두 삭제)
	- 참고로, Integer, String은 자바가 제공하는 대표적인 불변 객체

###  6. 값 타입 비교
- 기본(primitive) 값 타입은 값만 같으면 되지만 객체 타입은 참조값을 비교하기 때문에 아님
- 값 타입은 기본적으로 ==을 사용하면 되지만 임베디드 타입같은 값 타입은 equals를 재정의해서 사용해야 한다.

# 10. 값 타입 컬렉션
###  1. 값 타입 컬렉션
- 값 타입을 하나 이상 저장할 때, 사용
- @ElementCollection, @CollectionTable 사용
	- ex) @ElementCollection
    		@CollectionTable(name = "FAVORITE_FOOD",
            	joinColumns = @JoinColumn(name = "MEMBER_ID")
    		)
	)
- * 일대다 형식으로 풀어내야 함
	- 별도의 테이블 필요
- 값 타입이므로 엔티티 사이클에 종속되므로 primitive 값과 동일하다고 봐도 무방함
	- CascadeType.ALL + OrphanRomoval = TRUE와 동일하다
- 지연 로딩(LAZY) 사용
- 값 수정
	- Set<String> : remove() 후, add()해야 한다.
	- List<Address> : remove(new Address("~~","~~","~~"))로 지우고 add(new Address("~~","~~","~~"))하면 된다.
		- equal(), hashcode()를 반드시 구현해야하고
- 번외 : ** 이전에 배웠던 사이트 이펙트로 인해 값 타입 수정은 immutable하게 수정해야 한다.
	- ex) findMember.setHomeAddress(new Address("~","~","~"))

###  2. 값 타입 컬렉션의 제약사항
- 값 타입은 엔티티와 다르게 식별자 개념이 없다.
- 값은 변경하면 추적이 어렵다.
- ** 값 타입 칼렉션에 변경 사항이 발생하면, 주인 엔티티와 연관된 모든 데이터를 삭제하고 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다.
- 값 타입 컬렉션을 매핍하는 테이블은 모든 컬럼을 묶어서 기본 키를 구성해야 한다.(개발자가 직접)
	- null 입력x, 중복 저장x
- 추적할 필요도 없고 진짜 단순할 때만 사용한다.

** => 값 타입 컬렉션을 쓰지말고 일대다로 풀자!
- 1. Address를 한 번 감싼 AddressEntity를 엔티티로 만들고 @Id, @GeneratedValue 선언하여 PK값을 주자
	- Address 엔티티가 복합PK가 아니라 단일 식별자를 가지게 되고 외래키를 가진 상황이 된다.
- 2. 주인 엔티티인 Member는 일대다 단방향으로 맵핑한다.
	- @OneToMany(cascade = CascadType.ALL, OrphanRomoval = true)
	  @JoinColumn(name = "member_id")
- 3. 일대다이기 때문에 FK를 일 쪽에서 관리하기 때문에 update 발생하는 것은 어쩔 수 없다.

# 11. 객체지향 쿼리 언어
###  1. JPQL 소개
- 나이가 18살 이상인 회원을 모두 가져오고 싶다면 사용
- 테이블이 아닌 엔티티 객체를 개상으로 검색하는 객체 지향 쿼리
- SQL과 문법 유사
- SQL을 추상화해서 특정 데이터베이스 SQL에 의존X
- ** 동적 쿼리를 작성하기 힘들다.
	- Criterial를 사용 => 못쓴다, 복잡할 수록 쓰기 힘듦(CriterialBuilder cb = em.getCritetiaBuilder();...)
	- QueryDSL을 사용한다.
		- 오픈 소스 라이브러리
		- 컴파일 시점에 문법 오류를 찾을 수 있음(TypeSafe)
		- 동적 쿼리 작성이 단순하고 쉬움, 실무 사용 권장

- 참고로 네이티브 SQL의 경우, JPA가 제공하는 SQL을 직접 사용하는 기능.
	- JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능
	- CONNECT BY 처럼 특정 db만 사용하는 SQL힌트
	- em.createNativeQuery()를 사용해서 순수 SQL 작성하면 됨
		- JPA 기술을 쓰기 때문에 바로 직전에 flush 호출됨
- JDBC를 직접 사용하면 사용하기 전에 반드시 flush해야 함
- 실무에서는 JPQL + QueryDSL 조합으로 거의 다 해결된다.

###  2. JPQL
- 테이블이 아닌 엔티티 객체를 개상으로 검색하는 객체 지향 쿼리
- SQL을 추상화해서 특정 데이터베이스 SQL에 의존X
- 결국 SQL로 변환돼서 실행된다.
- 한 방에 여러 개를 UPDATE, DELETE하려면 JPQL 사용(벌크 연산)
- 엔티티 속성은 대소문자 구분하며 키워드는 대소문자 구분없음
- 엔티티 이름 사용, 테이블 이름 아님
- * 별칭 필수 Member m과 같은(as는 생략)
- TypeQuery는 반환 타입이 명확할 때, 사용
	- ex) TypeQuery<Member> query = em.creatQuery("select ~", Member.class);
- Query는 반환 타입이 명확하지 않을 때, 사용
	- ex) Query query = em.creatQuery("select m.username, m.age~");
- getResultList() : 하나 이상의 값이 반환될 때
	- * 결과가 없으면 빈 리스트 반환
- getSingleResult() : 결과가 정확히 하나만 나와야 함
	- 결과가 없거나 둘 이상이면 Exception 발생
	- try catch로 잡아야 함... Spring data jpa는 마찬가지로 try catch로 잡고 Optional로 반환함
- 파라미터 바인딩
	- ex) Member result = em.createQuery("select m from Member m where m.username = :username", Member.class)
		.setParameter("username", "member1")
		.getSingleResult();

###  3. 프로젝션(SELECT)
- SELECT 절에 조회할 대상을 지정하는 것
- 프로젝션 대상 : 엔티티, 임베디드 타입, 스칼라 타입(숫자, 문자등 기본 데이터 타입, 관계형 데이터베이스에서 SELECT하는 대상)
- 엔티티 프로젝션 : SELECT m FROM Member m
	- List로 받으면 대상들이 전부 영속성 컨텍스트에서 관리됨
- 엔티티 프로젝션 : SELECT m.team FROM Member m
	- 복잡한 조인 쿼리가 실행됨(묵시적 조인), 이렇게 쓰면 안되고 SELECT t FROM Member m join m.team t으로 사용해야 함
		- 이렇게 해야 쿼리가 예측이 됨
- 임베디드 타입 프로젝션 : SELECT m.address FROM Member m
- 스칼라 타입 프로젝션 : SELECT m.username, m.age FROM Member m
	- TypeQuery로 못 받음
	- 여러 값을 조회할 경우
		- 1. Query 타입 Query query로 받는다. Object로 받기 때문에 Object[]로 캐스팅 후, 가져올 수 있다.
				- List resultList = em.createQuery(~~~).getResultList();
				- Object o = resultList.get(0);
				- Object[] result = (Object[]) o;
				- result[0], result[1]
		- 2. TypeQuery 이용
				- List<Object[]> resultList = em.createQuery(~~~).getResultList();
				- Object[] object = resultList.get(0);
				- result[0], result[1]
		- 3. new 명령어로 조회(제일 깔끔)
				- 단순 값을 DTO로 바로 조회
				- List<MemberDTO> result = em.createQuery("SELECT new jpql.MemberDTO(m.username, m.age) FROM Member m", MemberDTO.class).getListResult();
				- 패키지가 길어지면 지저분해지는 단점이 있긴 함, queryDSL에서는 극복됨
- distinct로 중복 제거 가능

###  4. 페이징 API
- order by를 해야 함
- setFirstResult : 조회 시작 위치(0부터 시작)
- setMaxResults : 조회할 데이터 수

###  5. 조인
- 내부 조인 : SELECT m FROM Member m [INNER] JOIN m.team t
- 외부 조인 : SELECT m FROM Member m LEFT [OUTER] JOIN m.team t
- 세타 조인(막조인) : SELECT count(m) FROM Member m, Team t WHERE m.username = t.name
- ON 절
	- 연관관계 없는 엔티티 외부 조인 가능(과거엔 내부 조인만 가능, 이게 위에 세타 조인인가 봄)
	- 조인 대상 필터링
		- (JPQL)SELECT m, t FROM Member m LEFT JOIN m.team t on t.name = 'A'  === (SQL) SELECT ~~ FROM Member m LEFT JOIN Team t ON m.TEAM_ID = i.id and t.name ='A'
	- 연관관계 없는 엔티티 외부 조인
		- (JPQL)SELECT m, t FROM Member m LEFT JOIN Team t on m.username = t.name  === (SQL) SELECT ~~ FROM Member m LEFT JOIN Team t ON m.username = t.name

###  6. 서브쿼리
- ex) select m from Member m where m.age > (select avg(m2.age) from Member m2)
- ex) 한 건이라도 주문한 고객 : select m from Member m where (select count(o) from Order o where m = o.member) > 0
- 서브 쿼리 지원 함수
	- [NOT] EXISTS (subquery) : 서브쿼리에 결과가 존재하면 참
	- ex) 팀A 소속인 회원 : select m from Member m where exists (select t from m.team t where t.name = ‘팀A') 
		- ALL, ANY, SOME (subquery)
		- ex) 전체 상품 각각의 재고보다 주문량이 많은 주문들 : select o from Order o where o.orderAmount > ALL (select p.stockAmount from Product p)
		- ex) 어떤 팀이든 팀에 소속된 회원 : select m from Member m where m.team = ANY (select t from Team t)
	- [NOT] IN (subquery) : 서브쿼리의 결과중 하나라도 같은 것이 있으면 참

- JPA는 WHERE, HAVING 절에서만 서브쿼리 가능하나 하이버네이트에서는 SELECT 절도 가능
- ** FROM 절의 서브쿼리는 현재 JPQL에서 불가능
	- 조인으로 풀 수 있으면 풀어서 해결
	- 정 안되면 애플리케이션에서 한 번 가져온 다음 조작하는 방법으로(2 번 정도 쿼리 수행) 또는 nativeSQL


###  7. JPQL 타입 표현
- 문자 : '~~', '을 표현 하려면 ''이렇게 2개 널으면 됨
- 숫자 : 10L(Long), 10D(Double), 10F(Float)
- Boolean : TRUE, FALSE
- ENUM : jpabook.MemberType.Admin(패키지명 포함)
	- ex) select m.username, 'HELLO', true ftom Member m where m.type = jpql.MemberType.USER;
		- 이렇게 하드코딩할 때나 길지 파라미터 바인딩하면 패키지명도 사실 그리 길지 않음
- 엔티티 타입 : Type(m) = Member (상속 관계에서 사용)
	- ex) em.createQuery("select i from Item i where type(i) = Book", Item.class)


###  8. 조건식(CASE 등등)
- 기본 case 식
	- ex) elect case when m.age <= 10 then '학생요금' when m.age >= 60 then '학생요금' else '일반요금' end from Member m
- coalesce : 하나씩 조회해서 null이 아니면 반환
	- ex) select coalesce(m.username, '이름 없는 회원') from Member m;
- nullif : 두 값이 같으면 null 반환, 다르면 첫번째 값 반환
	- ex) select nullif(m.username, '관리자') from Member m;

###  9. JPQL 함수
- 표준 함수
	- CONCAT 또는 ||, SUBSTRING, TRIM. LOWER, UPPER, LENGTH, LOCATE, ABS, SQRT, MOD
	- SIZE, INDEX(JPA용도)
		- ex) select size(t.members) from team t;
		- INDEX는 거의 안 씀(컬렉션 값 타입에서 @OrderColumn)
- 사용자 정의 함수
	- 하이버네이트는 사용전 방언에 추가해야 한다
		- 사용하는 DB방언을 상속 받은 클래스를 생성하고 사용자 정의 함수를 등록한다.
		- 등록하는 방식은 부모 클래스를 보고 구현되어 있는 것을 참고한다.
		- ex) select function('group_conct', m.name) from Member m
			- 인젝트 랭귀지를 빼버리면 select group_conct(m.name) from Member m 가능

# 12. 객체지향 쿼리 언어2
###  1. 경로 표현식
- 점을 찍이서 객체 그래프를 탐색하는 것
- 묵시적 조인은 내부 조인만 가능
- 상태 필드 : 단순히 값을 저장하기 위한 필드
	- 경로 탐색의 끝, 탐색 불가능
	- ex) m.username
- 연관 필드 : 연관 관계를 위한 필드
	- 단일 값 연관 필드 : @ManyToOne, @OneToOne, 대상이 엔티티
		- 묵시적 내부 조인(inner join) 발생, 탐색 가능
		- ex) m.team.members~~~
	- 컬렉션 값 연관 필드 : @OneToMany, @ManyToMany, 대상이 컬렉션
		- 묵시적 내부 조인 발생, 탐색 불가능
		- 점 찍어도 자동 완성 안됨
		- ex) m.orders
		- select t.members.(X) from Team t; -> select m.(O) from Team t join t.members m;
- ** 실무에서는 묵시적 조인하지 마라. 명시적으로 해야한다.
- 예제
	- select o.member.team from Order o -> 성공하나 묵시적 조인이 2번 일어남
	- select t.members from Team -> 성공하나 컬렉션이라 더 들어갈 순 없음. 조인은 1번 일어남
	- select t.members.username from Team t -> 실패, 컬렉션이라
	- select m.username ftom Team t join t.members m -> 성공, 컬렉션이지만 명시적 조인으로 가져와서 별칭을 이용해 접근 가능

###  2. fetch join(무지무지 중요함)
- [LEFT [OUTER] | INNER]join fetch 조인 경로
- SQL 조인 종류 X
- JPQL에서 성능 최적화를 위해 제공하는 기능
- 연관된 엔티티나 컬렉션을 SQL 한 번에 함께 조회하는 기능
	- 지연 로딩으로 설정해도 무시하고 먹힘
- ex) select t from Team t join fetch t.members where t.name = ‘팀A'  === SELECT T.*, M.* FROM TEAM T INNER JOIN MEMBER M ON T.ID=M.TEAM_ID WHERE T.NAME = '팀A' 
- 문제 상황1. 다대일
	- System.out.println("member = " + member.getUsername() + ", " + member.getTeam().getName());
	- 루프를 돌면서 다음과 같이 수행됨// 회원1, 팀A(SQL), // 회원2, 팀A(1차캐시), // 회원3, 팀B(SQL)
	- 프록시이기 때문에 N+1 문제가 발생, 즉시로딩이든 지연로딩이든 발생함
	- fetch join으로 해결해야 한다.
- 문제 상황2. 일대다
	- select t from Team t join fetch t.members을 실행
	- 팀A에 회원1, 회원2, 팀B에 회원3가 있으므로 DB SQL의 결과는 3개가 나온다.
	- 팀A에 대한 결과가 2개이지만 ID가 똑같으므로영속성 컨텍스트에는 팀A(공유)가 하나만 저장된다.
		- 어쨌든 DB 결과는 2개이므로 각각 fetch join해서 총 3개의 결과(팀A 2개, 팀B 1개)가 나온다.
- JPQL의 DISTINCT 2가지 기능 제공
	- SQL에 DISTINCT를 추가
		- distinct는 완전히 똑같아야 중복 제거가 된다.
	- 애플리케이션에서 엔티티 중복 제거
		- distinct를 추가하면 JPA가 한 번 더 걸러줌으로써 ID만 똑같아도 중복 제거가 된다.
- fetch join과 일반 조인의 차이
	- 일반 조인 실행시, 연관된 엔티티를 함께 조회하지 않음
	- fetch join은 연관된 엔티티도 함께 퍼올린다.(즉시 로딩)
		- 객체 그래프를 SQL 한 번에 조회하는 개념


###  3. fetch join의 한계
- 페치 조인 대상에는 별칭을 줄 수 없고 그러므로 별칭을 이용해 where 필터를 할 수 없다.
	- ex) select t from Team t join fetch t.members as m where m.age > 10
	- JPA는 team.members 하면 members 다 갈 수 있어야 한다.
	- ** 이렇게 하면 정합성이 깨지고 영속성 컨텍스트가 혼란스러워한다. 팀A를 각각 호출하면서 하나는 10개만 가져오고 하나는 전체 다 가져오는 경우..
	- JPA가 의도한 설계는 Team의 Members가 다 나와야 한다. 조건을 걸어서 몇 개만 나오게 하면 안된다.
		- Team에서 Member를 가져오는게 아니라 처음부터 Member에서 5개를 가져오는 쿼리를 수행하는 게 맞다.
- 둘 이상의 컬렉션은 fetch join할 수 없다.
	- team -> members, orders
	- 일대다에서는 데이터가 뻥튀기되기 때문에 예상하지 못한다.
- 컬렉션을 페치 조인하면 페이징 API(setFirstResult, setMaxResults)를 사용할 수 없다.
	- 일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인해도 페이징 가능
	- 하이버네이트는 경고 로그를 남기고 메모리에서 페이징(매우 위험)
- fetch join 대신 @BatchSize(size = 100) 과 같이 @OneToMany 쪽에 설정하면 in 쿼리를 한 번 더 실행함으로써 즉시 다 가져올 수 있다.
	- 글로벌 세팅 : hibernate.default_batch_fetch_size, value =100
- 정리
	- 모든 것을 페치 조인으로 해결할 수 는 없음
	- 페치 조인은 객체 그래프를 유지할 때 사용하면 효과적(검색, member.team)
	- 여러 테이블을 조인해서 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야한다면 페치조인 보다는 일반 조일을 사용하고 필요한 데이터들만 조회해서 DTO로 반환하는 것이 효과적

###  4. 다형성 쿼리
- TYPE : 조회 대상을 특정 자식으로 한정
	- ex) select i from Item i where type(i) in (Book, Movie)
- TREAT : 자바의 타입 캐스팅과 유사
	- 상속 구조에서 부모타입을 특정 자식 타입으로 다룰 때 사용
	- ex) select i from Item i where treat(i as Book).author = 'kim'

###  5. 엔티티 직접 사용
- JPQL에서 엔티티를 직접 사용하면 SQL에서 해당 엔티티의 기본 키값을 사용
	- ex) select count(m.id), select count(m) === select count(m.id) as cnt
- 엔티티 직접 사용, 기본 키 값
	- ex) select m from Member m when m 또는 m.id = :member;
	- ~~~.setParameter의 값을 member 객체 또는 member의 id(String)으로 할 수 있음
- 엔티티 직접 사용, 외래 키 값
	- ex) select m from Member m when m.team 또는 m.team.id = :team
	- ~~~.setParameter의 값을 team 객체 또는 team의 id(String)으로 할 수 있음


###  5. Named 쿼리 - 어노테이션
- 미리 정의해서 이름을 부여해두고 사용하는 JQPL
- 정적 쿼리
- 어노테이션 또는 XML에 정의
- 장점
	- 1. 어플리케이션 로딩 시점에 SQL로 파싱해서 캐시하고 있음
	- 2. 로딩 시점에 쿼리 검증
- spring data jpa에서는.. 아래 처럼 사용한다. 이것도 결국 nameQuery
	- @Query("select u from User u where u.emaiAddress = ?1")
	- User findByUser(String emailAddress)

###  6. 벌크 연산
- 2 개 이상의 update, delete
- INSERT(insert into ..select)도 지원한다.
- 더티 체킹으로 실행하려면 너무 많은 SQL을 실행
- createQuery 마지막에 executeUpdate()를 사용한다.
- ** 주의 : 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리
	- 먼저, 벌크 연산을 먼저 실행 또는 벌크 연산 수행 후(자동 fulsh됨), 영속성 컨텍스트 초기화(영속성 컨텍스트는 계속 남아있기 때문)
		- clear() 후, 다시 find해서 가져와야 한다.
- spring data jpa에서는.. @Modifying @Query(~~~)가 같은 원리이다.

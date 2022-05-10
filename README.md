# jpa-main-point
JPA를 이용한 웹 애플리케이션 개발 중 의문 사항, 공부 정리

<h3 style="font-weight:bold;">영속성 컨텍스트</h3>
<p>
엔티티를 영구 저장하는 환경 <br>
애플리케이션과 데이터베이스 사이에서 객체를 보관하는 가상의 데이터베이스 같은 역할을 한다. <br>
엔티티 매니저를 통해 엔티티를 저장하거나 조회하면 엔티티 매니저는 영속성 컨텍스트에 엔티티를 보관하고 관리한다. <br>
출처: https://velog.io/@neptunes032/JPA-%EC%98%81%EC%86%8D%EC%84%B1-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8%EB%9E%80  <br> 
</p>
<p>
사용법: <br>
@PersistenceContext // 스프링이 엔티티매니저를 주입 시켜줌 <br> 
private EntityManager em; <br>
-> 스프링 jpa에서 @Autowired로 대체 가능 <br>
따라서 의존관계 주입하는 방법과 동일하게 사용 가능 <br>
class 레벨에 @RequiredArgsConstructor <br>
private final EntityManager em; <br> 
</p>

<h3 style="font-weight:bold;">Dirty Checking(변경 감지)</h3>
<p>엔티티를 변경할 때 사용하는 방법, transaction할 때 변경된 값들을 감지 후 자동으로 update 한다.</p>
<ol>
  <li>transaction이 있는 서비스 계층에 식별자, 변경할 data 전달</li>
  <li>식별자를 통해 엔티티를 조회해서 영속성 컨텍스트로 등록 후 값을 변경(이때 setter를 지양하고 엔티티의 메서드 이용)</li>
  <li>transaction 커밋 시점에 변경 감지 실행</li>
</ol>

<h3 style="font-weight:bold;">프록시 기초</h3>
<p>1. em.find() : 데이터베이스를 통해 실제 엔티티 조회</p>
<p>2. em.getReference() : 데이터베이스 조회를 미루는 가짜 엔티티 객체(프록시) 조회 (하이버네이트가 프록시를 만들어줌)</p>
<ol><b>프록시</b>
  <li>실제 클래스를 상속 받아서 만들어짐</li>
  <li>클래스와 겉모양이 같다</li>
  <li>사용하는 입장에는 진짜 객체인지 프록시 객체인지 구분할 필요가 없다</li>
  <li> ex. <br>
    <p>Entity : id, name, getId(), getName()</p>
    <p>Proxy : getId(), getName(), Entity target(엔티티 참조 필드)</p>
  </li>
</ol> 
<ol><b>프록시 객체의 초기화(가져오는 과정)</b>
  <li>클라이언트의 조회(ex. getName())요청</li>
  <li>프록시는 실제 Entity가 없으면 영속성 컨텍스트에 실제 Entity의 초기화를 요청</li>
  <li>영속성 컨텍스트(같은 transaction 범위 내)에 없으면 DB에서 조회</li>
  <li>실제 Entity를 생성</li>
  <li>프록시에서 실제 Entity에 조회 요청</li>
</ol>
<ol><b>프록시의 특징</b>
  <li>프록시 객체는 처음 사용할 때 한 번만 초기화되는 것</li>
  <li>프록시는 초기화시 프록시 객체가 실제 엔티티로 바뀌는 것이 아니라 프록시 객체로 부터 실제 엔티티에 접근이 가능하게 되는 것!</li>
  <li>프록시 객체는 원본 엔티티를 상속 받으므로 타입 체크시 == 비교는 실패한다(상속 관계라면 == 비교 false) -> instance of를 사용해야 한다</li>
  <li>영속성 컨텍스트에 찾는 엔티티가 이미 있으면 em.getReference() 해도 프록시가 아닌 실제 엔티티가 조회된다 <br> -> jpa는 동일성을 보장하기 때문에 영속성 컨텍스트에 있는 객체를 꺼내오면 모두 같다<br>
getReference()를 통해 같은 id로 프록시로만 조회해도 모두 같음
  </li>
  <li>영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태라면 프록시 초기화하면 문제 발생한다
  <br>
    -> detach(), clear(), close() 등을 사용하여 준영속 상태라면 초기화할 수 없음
    (프록시의 초기화는 영속성 컨텍스트를 통해 DB에 쿼리를 보낸는 것이기 때문에)
  </li>
</ol>
<p>프록시는 즉시 로딩과 지연 로딩을 이해하기 위한 기초라고 생각, 실제로 getReference() 사용 거의 안한다</p>

<h3 style="font-weight:bold;">즉시 로딩과 지연 로딩</h3>
<ul><b>지연 로딩</b>
  <li>
    객체 Member 와 Team이 연관관계일 때, Member만 조회해도 되는데 Team까지 조회되는 경우는 효율적이지 않다.<br>
  -> 지연 로딩(LAZY)으로 해결 가능, 처음 Member를 조회할 때가 아니라 실제로 Team의 필드를 가져올 때 team 조회 쿼리를 발생시켜 조회한다
  </li>
</ul>
<ul><b>즉시 로딩</b>
  <li>
    만약 Member와 Team을 항상 같이 조회한다면 즉시 로딩(EAGER)을 사용해서 한번에 조회할 수 있다.<br>
    *but, 실무에서는 가급적 지연로딩을 사용해야 한다<br>
    - 즉시 로딩 적용시 예상치 못한 SQL이 발생할 수 있기 때문에
    -> find() 이용시 모든 연관 관계 테이블을 join 해서 가져옴
    - 즉시 로딩은 JPQL에서 N+1 문제를 일으킨다
    -> Member를 createQuery()를 이용해 JPQL로 조회시 Team이 EAGER로 돼있다면 JPQL에 의해
    Member만 조회 쿼리(1)를 보낸 직후 결과 row 수만큼 Team 조회 쿼리(N)을 발생시킨다
    - @ManyToOne, @OneToOne(~ToOne)은 기본 설정이 즉시 로딩이다
    -> 실무에서는 신경써서 LAZY로 설정해야 한다
  </li>
</ul>
<ol>* N+1 문제 해결 방법
  <li>우선 모든 연관관계에서 지연 로딩으로 설정(@~ToOne 에서는 직접 설정 필요)</li>
  <li>JPQL에서 fetch join을 사용해서 필요한 테이블만 join해서 한 번에 가져오기</li>
  <li>@EntityGraph 를 이용하는 방법도 있다</li>
  <li>결론: 모든 연관관계에서 지연 로딩 사용하고, N+1 문제는 JPQL fetch join으로 해결하기</li>
</ol>

<h3 style="font-weight:bold;">영속성 전이(CASCADE)</h3>
<ul>
  <li>특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때 사용</li>
  <li>영속성 전이는 연관관계 매핑하는 것과 아무 관련 없다</li>
  <li>주의점: 자식 엔티티를 관리하는 부모 엔티티가 한 개이고, 생성/ 삭제 등의 생명 주기가 완전히 일치할 때 사용해야 한다. 부모와 관련없이 자식 엔티티만 따로 생성하거나 삭제해야한다면 사용 X</li>
  <li>결론: 설계 처음에는 cascade를 사용하지 않고 설계한 후 라이프 사이클이 완전 동일한 객체라면 cascade를 설정해서 리팩토링하는 설계가 좋다</li>
</ul>

<h3 style="font-weight:bold;">고아 객체</h3>
<ul>
  <li>고아 객체 제거: 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제</li>
  <li>orphanRemoval = true 으로 설정</li>
  <li>
    ex. <br>
    Parent parent = em.find(Parent.class, id); <br>
    parent.getChildList().remove(0); <br>
    위처럼 remove하면 delete 쿼리가 발생한다
  </li>
  <li>참조하는 곳이 하나일 때만 사용해야 한다</li>
  <li>특정 엔티티가 개인 소유할 때만 사용해야 한다</li>
  <li>개념적으로 부모를 제거하면 자식들은 고아가 되기 때문에 이때는 CascadeType.remove와 동일하게 작동한다</li>
  <li>
    영속성 전이와 고아 객체 속성 둘다 사용하면 부모 엔티티가 자식 엔티티의 생명 주기를 완전히 관리할 수 있다<br>
  -> 도메인 주도 설계(DDD) Aggregate Root 개념을 구현할 때 유용(부모 엔티티를 관리하는 레포지토리만 만들어서 자식 엔티티까지 관리할 수 있게 하는 것)
  </li>
</ul>

<h3 style="font-weight:bold;">어노테이션 정리</h3>
<ul>
  <li>@ManyToOne</li>
  <span>다대일 [N:1]</span>
  <li>@OneToMany</li>
  <span>일대다 [1:N]</span>
  <li>@OneToOne</li>
  <span>일대일 [1:1]</span>
  <br>
  <li>@ManyToMany (다대다)는 사용을 지양해야 한다.</li>
  <span>@JoinTable을 통해 중간 테이블을 사용해야하는데, 이러면 중간 테이블은<br>
  단순히 매핑해주는 역할만 할 수 있고 다른 컬럼들을 가지지 못하기 때문에 유연하지 못하게 된다. </span>
  <li>@JoinColumn</li>
  <span>외래키가 있는 테이블과 매핑된 엔티티에서 설정해야 한다. (연관관계의 주인, N 쪽)</span> <br>
  <span>반대쪽은 mappedBy로 명시 해줘야 한다.</span>
  <li>@Enumerated</li>
  <span>enum 타입을 사용할 때 필요한 어노테이션</span><br>
  <span>사용시에 반드시 @Enumerated(EnumType.STRING) 처럼 설정해야한다.</span>
  <span>String으로 설정을 안하면 ordinary 설정이 돼서 enum을 수정하기 어려워진다.</span>
  <li>@Id @GeneratedValue</li>
  <span>Id는 테이블의 주키 역할을 한다는 것을 나타내게</span><br>
  <span>GeneratedValue는 주키 생성 전략 설정</span>
  <li>@Embeddable, @Embedded</li>
  <span>하나의 엔티티 내에서 연관성 있는 필드들을 다른 객체로 분리 후 사용하기 위해</span>
  <span>Embeddable은 타입을 정의하는 곳(클래스)에 표시, Embedded는 사용하는 곳(필드)에 표시</span>
</ul>

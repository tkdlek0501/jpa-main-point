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

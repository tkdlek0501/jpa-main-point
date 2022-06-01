# jpa-main-point
JPA를 이용한 웹 애플리케이션 개발 중 의문 사항, 공부 정리

<details>
  <summary><h3 style="font-weight:bold;">영속성 컨텍스트</h3></summary>
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
</details>

<details>
  <summary><h3 style="font-weight:bold;">Dirty Checking(변경 감지)</h3></summary>
<p>엔티티를 변경할 때 사용하는 방법, transaction할 때 변경된 값들을 감지 후 자동으로 update 한다.</p>
<ol>
  <li>transaction이 있는 서비스 계층에 식별자, 변경할 data 전달</li>
  <li>식별자를 통해 엔티티를 조회해서 영속성 컨텍스트로 등록 후 값을 변경(이때 setter를 지양하고 엔티티의 메서드 이용)</li>
  <li>transaction 커밋 시점에 변경 감지 실행</li>
</ol>
</details>  

<details>
  <summary><h3 style="font-weight:bold;">프록시 기초</h3></summary>
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
</details>

<details>
  <summary><h3 style="font-weight:bold;">즉시 로딩과 지연 로딩</h3></summary>
<ul><b>지연 로딩</b><br>
    객체 Member 와 Team이 연관관계일 때, Member만 조회해도 되는데 Team까지 조회되는 경우는 효율적이지 않다.<br>
  -> 지연 로딩(LAZY)으로 해결 가능, 처음 Member를 조회할 때가 아니라 실제로 Team의 필드를 가져올 때 team 조회 쿼리를 발생시켜 조회한다
</ul>
<ul><b>즉시 로딩</b><br>
    만약 Member와 Team을 항상 같이 조회한다면 즉시 로딩(EAGER)을 사용해서 한번에 조회할 수 있다.<br>
    *but, 실무에서는 가급적 지연로딩을 사용해야 한다<br>
    - 즉시 로딩 적용시 예상치 못한 SQL이 발생할 수 있기 때문에<br>
    -> find() 이용시 모든 연관 관계 테이블을 join 해서 가져옴<br>
    - 즉시 로딩은 JPQL에서 N+1 문제를 일으킨다<br>
    -> Member를 createQuery()를 이용해 JPQL로 조회시 Team이 EAGER로 돼있다면 JPQL에 의해
    Member만 조회 쿼리(1)를 보낸 직후 결과 row 수만큼 Team 조회 쿼리(N)을 발생시킨다<br>
    - @ManyToOne, @OneToOne(~ToOne)은 기본 설정이 즉시 로딩이다<br>
    -> 실무에서는 신경써서 LAZY로 설정해야 한다
</ul>
<ol><b>* N+1 문제 해결 방법</b>
  <li>우선 모든 연관관계에서 지연 로딩으로 설정(@~ToOne 에서는 직접 설정 필요)</li>
  <li>JPQL에서 fetch join을 사용해서 필요한 테이블만 join해서 한 번에 가져오기</li>
  <li>@EntityGraph 를 이용하는 방법도 있다</li>
  <li>결론: 모든 연관관계에서 지연 로딩 사용하고, N+1 문제는 JPQL fetch join으로 해결하기</li>
</ol>
</details>

<details>
  <summary><h3 style="font-weight:bold;">영속성 전이(CASCADE)</h3></summary>
<ul>
  <li>특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때 사용</li>
  <li>영속성 전이는 연관관계 매핑하는 것과 아무 관련 없다</li>
  <li>주의점: 자식 엔티티를 관리하는 부모 엔티티가 한 개이고, 생성/ 삭제 등의 생명 주기가 완전히 일치할 때 사용해야 한다. 부모와 관련없이 자식 엔티티만 따로 생성하거나 삭제해야한다면 사용 X</li>
  <li>결론: 설계 처음에는 cascade를 사용하지 않고 설계한 후 라이프 사이클이 완전 동일한 객체라면 cascade를 설정해서 리팩토링하는 설계가 좋다</li>
</ul>
</details>

<details>
  <summary><h3 style="font-weight:bold;">고아 객체</h3></summary>
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
</details>

<details>
  <summary><h3 style="font-weight:bold;">데이터 타입 분류</h3></summary>
<p>1. 엔티티 타입</p>
  <ul>
    <li>@Entity로 정의하는 객체</li>
    <li>데이터가 변해도 식별자로 지속해서 추적 가능</li>
  </ul>
<p>2. 값 타입</p>
  <ul>
    <li>int, Integer, String 등 단순히 값으로 사용하는 자바 기본 타입이나 객체</li>
    <li>식별자가 없고 값만 있으므로 변경시 추적 불가</li>
    <li> 기본값 타입
      <ul>
        <li>기본 타입 int, double</li>
        <li>래퍼 클래스 integer, Long</li>
        <li>String</li>
        <li>생명주기를 엔티티에 의존</li>
        <li>값 타입은 공유하면 안된다. (ex. 회원 이름 변경시 다른 회원의 이름도 함께 변경되면 안됨)</li>
        <li>
          참고) Integer같은 래퍼 클래스나 String 같은 특수 클래스는 주소값을 복사하므로 공유가 된다, 기본값 타입은 이렇게 공유하면 안된다.<br>
          ex)<br>
          Integer a = 10;<br>
          Integer b = a;<br>
          a = 20;<br>
          a와 b는 같은 주소를 바라보므로 20의 값을 가지게 된다.
        </li>
      </ul>
    <li>
    <li> 임베디드 타입
      <ul>
        <li>
          int, String 과 같은 값 타입<br>
          ex) 회원 엔티티: id, name, startDate, endDate, city, street, zipcode <br>
          -> id, name, workPeriod, address (연관된 필드끼리 묶어 클래스로)
        </li>
        <li>
          사용법: @Embeddable(값 타입 정의하는 곳), @Embedded(사용하는 곳), 기본 생성자 필수
        </li>
        <li>
          장점: 재사용 가능, 높은 응집도, 모든 값 타입을 소유하고 있는 엔티티가 생명주기를 관리,
          DB의 변경은 필요 없음, 객체 지향적, 객체와 테이블을 아주 세밀하게 매핑하는 것이 가능
        </li>
        <li>
          한 엔티티에서 중복되는 필드가 있는 값타입 여러개 사용하려면 @AttributeOverride를 사용해 컬럼명 재정의 할 수 있다.
        </li>
        <li>
          @MappedSuperclass VS @Embeddable <br>
          상속 vs 위임의 개념이다.<br>
          @MappedSuperclass 는 비교적 다수의 클래스에서 공통으로 사용할 수 있는 필드를
          묶어놓은 클래스를 지정해서 사용할 때 사용해야 한다 (ex. 생성일, 수정일 등)<br>
          하나의 객체는 하나만 상속할 수 있다.
        </li>
      </ul>
    </li>
    <li> 값 타입 공유 참조
      <ul>임베디드 값타입을 여러 엔티티에서 공유해버리면 위험하다.
        <li>같은 값을 가지는 게 아니라 같은 주소를 바라보기 때문에</li>
        <li>각각 다르게 new 해서 사용해야 한다.</li>
        <li>
          객체 타입은 참조 값을 직접 대입하는 것을 막을 방법이 없다.<br>
          -> 따라서 불변 객체로 만들어 버려서 생성 이후에 값의 수정을 못하게 해야 한다.
        </li>
        <li>결론: 임베디드 값타입은 항상 불변 객체로 만들어 사용해야 한다.</li>
      </ul>
    </li>
    <li> 값 타입 비교
      <ul>
        <li>동일성 비교: 인스턴스의 참조값을 비교, == 사용</li>
        <li>동등성 비교: 인스턴스의 실제값을 비교, equals() 사용</li>
      </ul>
    </li>
    <li> 컬렉션 값 타입
      <ul>
        <li>값 타입을 하나 이상 저장할 때 사용</li>
        <li>@ElementCollection, @CollectionTable 사용</li>
        <li>
          DB는 컬렉션을 같은 테이블에 저장할 수 없다.<br>
          -> 컬렉션을 저장하기 위한 별도의 테이블이 필요하다.
        </li>
        <li>
          수정시에 update가 아니라 delete-insert 된다.
          (side-effect를 방지하기 위해 setter를 허용하지 않는 불변 객체로 만들기 때문)
        </li>
        <li>
          컬렉션 값 타입을 굳이 사용해야 할까? Entity로 전환해서 cascade와 하는 것이 비슷<br>
          - 값타입 컬렉션의 제약사항 <br>
          <ol>
            <li>값타입은 엔티티와 다르게 식별자 개념이 없다.</li>
            <li>값 변경시 추적이 어렵다.</li>
            <li>변경시 주인 엔티티와 연관된 모든 데이터 삭제 후 다시 저장한다.</li>
            <li>값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본키를 구성해야 한다.
              (null 허용 x, 중복 x)</li>
            <li>생명주기를 엔티티에 의존한다.</li>
            <li>공유하지 않는 것이 안전하다.(불변 객체로 만들어야 한다.)</li>
          </ol>
          - 결론: 차라리 Entity로 전환하는 게 나을수도 있다.(고려해야 한다.)<br>
          값 타입을 사용할 때는 추적이 필요없고 정말 간단한 값들에만 사용해야 한다.<br>
          Entity로 전환시에는<br>
          1. 1:다 연관관계로 설정<br>
          2. cascade (영속성 전이) + orphanremoval (고아 객체 제거) 로 설정
        </li>
      </ul>
    </li>
  </ul>
</details>  

<details>
  <summary><h3 style="font-weight:bold;">객체 지향 쿼리 언어 정리</h3></summary>
  <ol>
    <li>JPA 는 다양한 쿼리 방법을 지원한다
      <ul>
        <li>JPQL (표준 문법)</li>
        <li>QueryDSL</li>
        <li>네이티브 SQL도 가능 (특정 DB에 종속적인 SQL 쿼리)</li>
      </ul>
    </li>
    <li>JPQL이란?
      <ul>
        <li>객체 지향 쿼리 언어이고 테이블이 아닌 엔티티를 대상으로 쿼리를 작성하는 문법</li>
        <li>가장 단순한 조회 방법</li>
        <li>추상화된 형태라 SQL에 의존하지는 않지만 SQL로 변환돼서 실행된다.</li>
        <li>문자열 형태이기 때문에 동적 쿼리를 해결하기가 어렵다.</li>
      </ul>
    </li>
    <li>프로젝션
      <ul>
        <li>select 절에 조회할 대상을 지정하는 것, 엔티티의 일부 데이터만을 가져오게 하는 기능</li>
        <li>대상: 엔티티, 임베디드 타입, 스칼라 타입</li>
        <li>임베디드 타입은 조회의 시작점이 될 수 없다는 제약이 있다.<br>
        -> Address가 임베이드 타입이면, String query = "SELECT o.address FROM Order o" / List<Address> addresses = em.createQuery( query, Address.class ).getResultList(); <br>
          이렇게 엔티티로 부터 가져와야 한다.</li>
        <li>여러 타입을 가져오려면 DTO를 따로 만들어서 가져오는 방법이 좋다.<br>
          ex. List<MemberDTO> result = em.createQuery("select new jpql.MemberDTO(m.username, m.age) from MemberDTO m ", MemberDTO.class) <br>
          주의점: 엔티티가 아니기 때문에 new 키워드를 사용해 생성자를 사용하듯이 작성해야 한다.<br>
          패키지 경로를 다 적어줘야 한다. (queryDSL에서는 import 가능)<br>
          DTO에 생성자를 만들어 줘야한다. 
        </li>
      </ul>
    </li>
    <li>페이징 API
      <p>setFirstResult(int startPosition) : 시작 위치 (index)</p>
      <p>setMaxResult(int maxResult) : 조회할 데이터 수</p>
    </li>
    <li>join
      <p>내부 조인 inner join</p>
      <p>외부 조인 left (outer) join</p>
      <p>세타 조인</p>
    </li>
    <li>서브쿼리
      <p>JPA 에서는 where, having 절 + 하이버네이트에서 지원: select 절에서 사용 가능</p>
      <p>but, from 절에서는 사용할 수 없다. -> join으로 풀어서 or 쿼리 두 번으로 나눠서 해결</p>
    </li>
    <li>경로 표현식
      <ul>
        <li>상태필드 : 단순히 값을 저장하기 위한 필드, 경로 탐색의 끝이라 더이상 탐색 X<br>
          ex. m.username
        </li>
        <li>연관필드
          <ul>
            <li>단일 값 연관 필드 : @ManyToOne, @OneToOne / 대상이 엔티티일때<br>
              묵시적 내부 조인(inner join) 발생, 탐색 O <br>
              ex. m.team
            </li>
            <li>*컬렉션 값 연관 필드 : @OneToMany, @ManyToMany / 대상이 컬렉션일때<br>
              묵시적 내부 조인 발생, 탐색 X <br>
              ex. m.orders <br>
              -> 컬렉션이니까 특정 데이터를 가져올 수 없어서 필드 탐색이 불가능하다. <br>
              -> from 절에서 명시적 조인을 통해 별칭 얻어서 별칭으로부터 탐색 가능하다. 
              -> ex. select m.username from Team t join t.members m (O) <br>
              select t.members.username from Team t (X) <br>
              => 결론: 묵시적 조인은 사용하면 안좋다. 명시적 조인을 사용해서 탐색해야 한다.<br>
              - join은 SQL 튜닝에 중요 포인트이고, 묵시적 join은 어떻게 생성될지 파악하기 어렵기 때문에
            </li>
          </ul>
        </li>
      </ul>
    </li>
    <li>https://github.com/tkdlek0501/jpa-basic/blob/main/src/main/java/jpa/basic/example/MemberRepository.java</li>
  </ol>
</details> 

<details>
  <summary><h3 style="font-weight:bold;">Fetch Join 내용 정리</h3></summary>
  <p>JPA를 사용하는 실무에서 가장 중요한 포인트</p>
  <ul>
    <li>SQL join의 종류가 아니다.</li>
    <li>JPQL에서 성능 최적화를 위해 제공하는 기능</li>
    <li>연관된 엔티티나 컬렉션을 SQL 한 번에 함께 조회 (테이블이 아닌 엔티티를 기준으로 조회)</li>
    <li>join fetch 명령어를 사용</li>
    <li>기본은 inner join</li>
  </ul>
  <p>ex. 회원 조회시 연관된 팀도 함께 조회(한 번의 SQL)</p>
  <ul>
    <li>JPQL) select m from Member m join fetch m.team</li>
    <li>SQL) select m.*, t.* from member m inner join team t on m.team_id = t.team_id</li>
  </ul>
  <p>주의점: 1:다 연관관계(컬렉션) 에서 데이터 중복이 일어날 수 있다<br>
    -> distinct로 해결<br>
    SQL의 distinct 기능 + 같은 식별자를 가진 엔티티 중복 제거
  </p>
  <ul>fetch join 과 일반 join의 차이
    <li>일반 join 은 실행시 연관된 엔티티를 함께 조회하지 않음</li>
    <li>JPQL 은 결과 반환시 연관 관계 고려 x, 단지 select 절에 지정한 엔티티만 조회한다</li>
    <li>팀 엔티티만 조회하고, 회원 엔티티는 조회 x</li>
    <li>fetch join 사용할 때만 연관된 엔티티도 함께 조회되는 것(즉시 로딩)</li>
    <li>요약: JPQL에서는 fetch join을 사용할 때만 연관된 엔티티를 함께 조회한다 (그냥 join은 연관 관계 고려 x)</li>
    <li>또한 fetch join은 즉시 로딩이다</li>
  </ul>
  <ol>fetch join의 특징과 한계
    <li>fetch join 대상에는 별칭을 줄 수 없다. <br>
      ex. fetch join t.members m 이렇게 별칭(as)를 주고 where 절에 m.age > 10 이런 조건을 주는 것은 안된다. <br>
      : team 으로부터 member 탐색시 member 전체가 조회되지 않고 일부만 조회하게 되므로 탐색에 누락이 생긴다. 객체 그래프를 탐색한다는 것은 전체를 조회한다는 개념 <br>
      따라서 일부만 조회하고 싶다면 fetch join이 아니라 아예 따로 쿼리를 생성해야 한다. 
    </li>
    <li>둘 이상의 컬렉션은 fetch join 할 수 없다.<br>
      1:다 하나도 데이터 중복이 생기는데 둘 이상이면 예상치 못한 join이 발생할 수 있다.
    </li>
    <li>컬렉션(1:다) fetch join하면 페이징 API 쓸 수 없다.(1:1, 다:1은 페이징 가능)<br>
      1:다 에서 페이징 API를 쓰려면 <br>
      1. 다:1 로 조회하는 쿼리로 변경
      2. 1:다를 쓰려면 @Batchsize 로 in() 쿼리를 추가해서 페이징 해줘야한다. (연관된 엔티티 몇 row 조회할지 설정)
    </li>
    <li>결론(fetch join 사용 유의점)<br>
      fetch join 은 객체 그래프를 유지할 때는 효과적(fetch join의 대상을 별칭으로 지정하고 조건식을 만들면 안된다.)<br>
      모든 것을 fetch join으로 해결할 수는 없다.<br>
      통계 등 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야 한다면, fetch join 보다는 일반 join을 사용해서 필요한 data들만 조회해서 DTO로 반환하는 것이 효과적이다.
    </li>
  </ol>
</details>

<details>
  <summary><h3 style="font-weight:bold;">Fetch Join 사용 주의 사항 및 문제 해결 방법</h3></summary>
<ul> 주의점
  <li>1:N, 즉 컬렉션을 조회할 때는 다음과 같은 사항들을 주의해야 한다.</li>
  <li>1:N을 fetch join 해서 가져오면 데이터 수가 뻥튀기 된다.(N 쪽을 기준으로 row가 만들어지기 때문)</li>
  <li>위 같은 문제를 jpql의 distinct 로 해결할 수는 있지만, (같은 식별자의 data 중복을 제거 + SQL의 distinct)</li>
  <li>1. 1:N 관계를 fetch join하면 paging 불가능 (SQL결과는 뻥튀기 돼있어, JPA가 실제 쿼리가 아닌 메모리 단계에서 페이징 처리를 하기 때문에)</li>
  <li>2. 1:N 관계 fetch join은 1개만 사용해야 한다. (1:N:N:... 이렇게 사용하면 데이터 조회시 부정합 발생 가능)</li>
</ul>
  <ol> 해결방안
    <li>먼저 ToOne 관계는 모두 fetch join을 걸어준다. (N:1, 1:1 관계는 data 중복을 발생시키지 않기 때문에 한번에 join해서 가져오도록 한다.)</li>
    <li>컬렉션은 지연 로딩으로 조회하도록 한다. (컬렉션은 jpql에서 fetch join 걸지 않고, fetchType:LAZY 로 조회되도록 내버려둔다.)</li>
    <li>지연 로딩 성능을 최적화(N+1 문제 방지) 하기 위해 fetch size를 설정해준다. 
      (1+N 방식의 조회가 아니라 IN 쿼리로 1+1 조회가 되도록 해준다. 즉 테이블 당 한개의 쿼리만 날려서 조회된다.)
      <ol>방법
        <li>글로벌(일반적으로 사용) : propertise에서 spring.jpa.properties.hibernate.default_batch_fetch_size: 개수</li>
        <li>개별 : @OneToMany 설정해둔 필드 값에 @BatchSize(size = 개수)</li>
      </ol>
    </li>
    <li>IN query 개수 max는 1000개라고 생각하면 된다. (100~500 사이가 적당)</li>
  </ol>
</details>

<details>
  <summary><h3 style="font-weight:bold;">어노테이션 정리</h3></summary>
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
</details>

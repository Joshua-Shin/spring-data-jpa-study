## 단원별 요점 정리


### 프로젝트 환경설정
#### H2 데이터베이스 설치
- 프로젝트 생성할떄, h2 라이브러리 추가하면, 해당 프로젝트의 spring boot 버전과 호환이 잘 되는 h2 버전을 알아서 설치하잖아.
- 프로젝트에서 라이브러리에서 h2 버전으로 어떤게 설치되었는지 확인하고. 내가 로컬에서 설치한 h2 버전이랑 맞는지 확인해. 이 둘의 버전이 안맞으면 호환이 잘 안될 수 있어.
- 처음 h2 접속할때는 로컬에 db파일이 생성이 안되어있기때문에, 처음에는 jdbc:h2:~/datajpa 로 접근하는거야. 이러면 해당 이름으로 로컬에 파일을 생성하면서 파일로 접속을 하게 됨.
- 그 다음부터는 원격으로 jdbc:h2:tcp://localhost/~/datajpa 으로 접속하는거임. 파일로 접근하게 되면 파일에 락이 걸려서 여러곳에서 동시에 접속이 안됨.

#### 스프링 데이터 JPA와 DB 설정, 동작확인
- application.yml 에서 몇가지 설정 옵션
  - show_sql : true - 콘솔창에 sql 나오게 함 (비추)
  - format_sql : true - sql문이 일자로 나오는게 아니라 좀 예쁘게 포맷팅 되어서 출력됨
  - org.hibernate.SQL : debug - sql문이 로그로 출력되게함. 이걸 권장.
  - org.hibernate.type : true - sql문에 파라미터에 뭐가 들어갔는지 보여주는 옵션. 근데 좀 보여주는게 답답함
  - 그래서 외부라이브러리인 p6spy를 implementation에 넣어서 사용. 이런 외부라이브러리 땡겨오는건 성능을 살짝 깎아먹을 수도.
- 같은 트랜잭션 안에서는 영속성 컨텍스트의 동일성을 보장한다 = 같은 인스턴스임을 보장한다 = savedMember랑 그 안에서 findMember랑 == 비교 하면 true

### 예제 도메인 모델
#### 예제 도메인 모델과 동작확인
- 엔티티 작성한것도 테스트코드를 작성하네.. 테스트코드, tdd 이거에 대한 강의도 언제가는 들어봐야할듯.
- @ToString(of = {"id", "username", "age"}) : 특정 필드 몇개만 선별해서 ToString 메소드 만들어주는 롬복 애노테이션. 양방향 연관관계로 물려있는 엔티티 필드를 여기에다 넣으면 무한 루프 위험.


### 공통 인터페이스 기능
#### 공통 인터페이스 설정
- public class MemberRepository extends JpaRepository<Member, Long> {}
- @Repository 애노테이션 안붙여도 됨.
- Spring data JPA가 알아서 구현체 만들어서 주입해줌


### 쿼리 메소드 기능
#### 메소드 이름으로 쿼리 생성
- findByUsernameAndAgeGreaterThan(String username, int age); 이런것도 그냥 인터페이스에 선언만 해주면 메소드 이름을 바탕으로 쿼리를 생성해버림..
- 당연히 메소드 이름 짓는 규약이 있음.
- https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation
- 미쳤다.

#### JPA NamedQuery
- @NameQuery : JPA 문법중 하나인데, 컴파일 시점에서 문자열로 주어진 쿼리를 파싱하여 잘못된 쿼리는 오류를 발생시켜주는 장점이 있으나, 
- 엔티티에다가 선언한 다음 그걸 가져오고 뭐하고 이러는게 너무 지저분하고 번거로움. 떄문에 실무에서 안씀.

#### @Query, 리포지토리 메소드에 쿼리 정의하기
- 메소드 이름으로 쿼리 생성하는건 파라미터가 1개 혹은 2개 정도 있는 단순하고 간단간단한거용으로 쓰고.
- 파라미터가 3개, 혹은 쿼리 자체가 복잡한거. 이런거느 그냥 쿼리를 직접 정의해서 쓰고 싶잖아. 그떄 사용하는게 이거
- 컴파일 시점에 쿼리 문자열을 파싱해서 오타 냈으면 오류 발생시켜줌. 실무에서 많이 씀
- @Query("select m from Member m where m.username= :username and m.age = :age")
- List \<Member> findUser(@Param("username") String username, @Param("age") int age);

#### @Query, 값, DTO 조회하기
- 단순히 값 하나를 조회
  - @Query("select m.username from Member m")
  - List \<String> findUsernameList();
- DTO로 직접 조회 
  - @Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) from Member m join m.team t")
  - List\<MemberDto> findMemberDto();
  - 잘 안써... QueryDsl 사용하면 쉽게 하는 방법 있대. 이거 뭐 패키지명까지 다 적어줘야하고 너무 지저분하잖아.

#### 파라미터 바인딩
- 컬렉션 파라미터 바인딩
  - @Query("select m from Member m where m.username in :names")
  - List\<Member> findByNames(@Param("names") List\<String> names)

#### 반환 타입
- Member, List\<Member>, Optional\<Member> 처럼 다양한 반환 타입 지원

#### 순수 JPA 페이징과 정렬
- 페이징 하는법 em.createQuery(.....).setFirstResult(offset).setMaxResults(limit).getResultList();
- offset은 전체 결과에서 시작할 idx 지정. 0부터 시작
- limit는 최대 개수.
- offset : 1, limit : 3 이면 1,2,3 이 가져와 지고, offset:2, limit:3이면 2,3,4 가져와짐. 물론 limit보다 남은게 적으면 적은대로 가져와지고
- 이거 jpa 강의에서 했던건데, 내가 이번 프로젝트에서 페이징 하는 일이 없어서 까먹네.

#### 스프링 데이터 JPA 페이징과 정렬
- Page\<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용
- Slice\<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용 안 함. count 쿼리가 매우 무거운 편이기에, limit+1을 조회해서 다음 페이지 여부만 확인해두는거.
- List\<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용 안 함
- List\<Member> findByUsername(String name, Sort sort);
- 사용 예
  - PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username")); // sort 안써도 됨
  - Page\<Member> page = memberRepository.findByAge(10, pageRequest); // Pageable의 구현체가 PageRequest,
  - List\<Member> content = page.getContent(); // 조회된 데이터 
  - page.getTotalElements() // 전체 데이터 수 
  - page.getNumber() // 페이지 번호 
  - page.getTotalPages() // 전체 페이지 번호 
  - page.isFirst()) // 첫번째 항목인가? 
  - page.hasNext() // 다음 페이지가 있는가?
- count 쿼리 분리하기. 
  - count만 뽑아올때는 쿼리에 조인이 필요 없으니까, 분리해서 더 최적화 하는거. 실무에서 매우 중요!
  - @Query(value = “select m from Member m”, countQuery = “select count(m.username) from Member m”)
  - Page\<Member> findMemberAllCountBy(Pageable pageable);
- 페이지를 유지하면서 엔티티를 DTO로 변환하기
  - Page\<Member> page = memberRepository.findByAge(10, pageRequest);
  - Page\<MemberDto> dtoPage = page.map(m -> new MemberDto());

#### 벌크성 수정 쿼리
- JPA를 사용한 벌크성 수정 쿼리
```
int resultCount = em.createQuery(
            "update Member m set m.age = m.age + 1" +
                    "where m.age >= :age")
            .setParameter("age", age)
            .executeUpdate();
```
  - 변경된 개수를 반환. getResultList() 이런거 아니고 executeUpdate()를 해줘야돼.
- 스프링 데이터 JPA를 사용한 벌크성 수정 쿼리
```
@Modifying
@Query("update Member m set m.age = m.age + 1 where m.age >= :age")
int bulkAgePlus(@Param("age") int age);
```
- 벌크 수정 쿼리는 그냥 바로 db에 sql문 날려버림.
- 벌크 수정 쿼리 날린 후에 em.clear()를 안하고 em.find()로 엔티티를 조회하면, db에 반영된 수정된 값이 조회되지 않고, 영속성컨텍스트 1차 쿼리에 있는 애의 값을 조회하게됨.
- 때문에 벌크 수정 쿼리 날린 후에는 안전하게 바로 em.clear() 해주는게 좋아.
- 이를 @Modifiying(clearAutomatically = true) 옵션 줄 수 있음
- Q. 벌크 수정 쿼리 날린 후에, em.flush(); 해버리면, 벌커 수정 쿼리로 변경된 값이 영속성 콘텍스트 1차 캐시에 남아있는 데이터로 다시 변경되는거 아닌가??
- A. 변경감지 프로세스를 생각해보면, 1차 캐시에 남아있는 데이터와, 스냅샷한 데이터의 변경이 없음. 때문에 변경감지로 인한 update sql문이 생성되지 않아! 때문에 em.flush()를 해도 아무런 일이 발생하지 않는거지.
- 스냅샷은 엔티티를 persist하든 find하든 뭘하든 값을 처음 읽어온 최초의 시점에 데이터를 저장해주는거임.

#### @EntityGraph
- JPQL 페치 조인
  - @Query("select m from Member m left join fetch m.team")
  - List\<Member> findMemberFetchJoin();
- EntityGraph
```
//공통 메서드 오버라이드
@Override
@EntityGraph(attributePaths = {"team"}) List<Member> findAll();
//JPQL + 엔티티 그래프 @EntityGraph(attributePaths = {"team"}) @Query("select m from Member m") List<Member> findMemberEntityGraph();
//메서드 이름으로 쿼리에서 특히 편리하다. @EntityGraph(attributePaths = {"team"}) List<Member> findByUsername(String username)
```
- 페치조인 할때 간단간단한건 EntityGraph 쓰고, 조금 복잡한건 JPQL @Query 쓰자

#### JPA Hint & Lock
- 쿼리 힌트
  - @QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
  - Member findReadOnlyByUsername(String username);
  - 읽기전용으로만 엔티티를 조회할때 읽기모드임을 명시해줌. 읽기모드가 아니면 영속성컨텍스트 1차캐시에 스냅샷을 뜨는데 이것부터가 메모리를 잡아먹으니까 진짜 읽기만 할거다 싶으면 이렇게 최적화를 하는거지
  - 그렇다고 뭐 읽기만 하는 메소드에 이걸 다~ 걸어서 최적화할 필요는 없고, 성능상 크게 트래픽을 잡아먹는애들 몇몇만 성능테스트를 해가면서 해놓는게 좋긴해
  - 어차피, 진짜 성능을 더 최적으로 땡겨줘야 하는 상황이면 redis 같은 캐시를 앞에 깔기 때문에, 성능상 이점을 챙기기가 글쎄..
- 락은 db에 락을 걸어두는건데, 너무 깊은 내용이라 거의 스킵


### 확장 기능
#### 사용자 정의 리포지토리 구현
- 나만의 메소드를 따로 정의해서 사용하고 싶을때, 사용자 정의 리포지토리를 따로 구현.
- MemberRepositoryCustom 인터페이스 만들고 여기에 메소드 선언하고, 이를 구현하는 구현체 MemberRepositoryImpl에 해당 메소드를 구현
- 처음에 JpaRepository 상속받은 MemberRepository 인터페이스가 MemberRepositoryCustom 얘를 또 상속 받게 해놓으면 끝
- 보통 QueryDsl를 사용하여 코드를 작성하고 싶을떄 사용.
- 인터페이스 이름은 규약이 없는데 이를 구현하는 구현체 이름 MemberRepositoryImpl에 이건 XxxxImpl 은 정해진거임. 
- 정해진대로 해야 Spring Data Jpa가 이걸 잘 고려해서 구현체를 만들어서 심어줌.
- 근데 꼭 이렇게 MemberRespository에 다 심어서 사용할 필요는 없고 그냥 MemberQueryRepository 이런식으로 클래스를 하나 정의해서 순수 JPA 사용하듯 사용하는것도 고려할 수 있지.
- 핵심 로직이냐 view에 보내는 로직이냐 등등 고려해가면서 클래스를 분리하는거야.

#### Auditing
- 등록일, 수정일, 등록자, 수정자 같은 필드는 웬만한 테이블에 다 적용. 이때 상속이 아니라 공통 속성만 가져오는 기능이 @MappedSuperClass
- 순수 JPA 사용
```
@MappedSuperclass
@Getter
public class JpaBaseEntity {
    @Column(updatable = false)
    private LocalDateTime createdDate;
    private LocalDateTime updatedDate;
    @PrePersist
    public void prePersist() {
        LocalDateTime now = LocalDateTime.now();
        createdDate = now;
        updatedDate = now;
    }
    @PreUpdate
    public void preUpdate() {
        updatedDate = LocalDateTime.now();
    }
}
```
- 지금 내 프로젝트에는 Member 객체 생성할때 따로 내가 LocalDatTime을 넣어줬는데,, 안그래도 되는거였네
- 스프링 데이터 JPA 사용
```
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
public class BaseEntity {
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;
    @LastModifiedDate
    private LocalDateTime lastModifiedDate;
    
    @CreatedBy
    @Column(updatable = false)
    private String createdBy;
    @LastModifiedBy
    private String lastModifiedBy;
}
```
```
@EnableJpaAuditing
@SpringBootApplication
public class DataJpaApplication {
     public static void main(String[] args) {
          SpringApplication.run(DataJpaApplication.class, args);
      }
     @Bean
     public AuditorAware<String> auditorProvider() {
          return () -> Optional.of(UUID.randomUUID().toString()); // 실무에서는 세션 정보나, 스프링 시큐리티 로그인 정보에서 ID를 받음
     }
}
```
- 생성일, 수정일은 거의 모든 테이블에 다 적용되는데, 생성자 수정자는 굳이 쓸 필요 없을때는 생성일, 수정일을 담은 클래스는 BaseTimeEntity라 하고, 이를 상속받는 BaseEntity에는 생성자, 수정자 필드를 넣어서
- 필요에 따라 선별적으로 해당 클래스를 엔티티가 상속받으면 유연하게 대처할 수 있음.

#### Web 확장 - 도메인 클래스 컨버터
- 도메인 클래스 컨버터 적용전
```
@GetMapping("/members/{id}")
public String findMember(@PathVariable("id") Long id) {
    Member member = memberRepository.findById(id).get();
    return member.getUsername();
}
```
- 도메인 클래스 컨버터 적용 후
```
@GetMapping("/members/{id}")
public String findMember(@PathVariable("id") Member member) {
    return member.getUsername();
}
```
- 트랜잭션이 적용되지 않는 상황이기에 딱 조회용으로만 좋고 여기서 member에다가 뭐를 수정하고 그럴때는 사용하면 고려해야될 예외상황들이 너무많음.
- 웬만하면 그냥 쓰지마

#### Web 확장 - 페이징과 정렬
- 예제
```
@GetMapping("/members")
public Page<Member> list(Pageable pageable) {
    Page<Member> page = memberRepository.findAll(pageable);
    return page;
}
```
- /members?page=0&size=3&sort=id,desc&sort=username,desc 로 들어오는 요청을 pageable 파라미터로 받아서 적용됨
- 서버 사이드쪽에서 페이지를 랜더링 하고 그 안에서 지지고볶고 하는 경우에는 엔티티를 그대로 반환해도 크게 상관없는데.
- 특히 api 쪽에서는 엔티티를 그대로 노출해서는 절대 안된다.
- 엔티티 필드 명이 바뀌면 스펙이 바뀌느것이기에 ssr 에서는 그냥 자기가 바꾼 필드명 view에서도 필드명 수정해주면 끝이지만, api로 작업하면 프론트쪽과의 협업에 있어서 큰 문제.
- ssr 에서는 어쨌든 랜더링 까지 다 해서 랜더링한 결과만 보내는거니까 보안상 이슈도 별 문제 없지만, api는 보안상으로도 엔티티를 그냥 다 노출해버리니 문제가 될 수도.
- 내 프로젝트에서 api 보낸것도 아니고 그냥 view에 보내는거니까 엔티티 그대로 보낸것들 냅두자.

### 스프링 데이터 JPA 분석
#### 스프링 데이터 JPA 구현체 분석
- @Transactional 적용
  - 그냥 JPA에서는 적용이 안되어있기때문에 @Transactional 안붙여주면 에러 터지는데, 얘는 기본적으로 리포지토리 계층에 다 선언되어있음.
  - 때문에 서비스 계층에 선언이 되어있으면 그걸 이어서 가고, 선언이 안되어있으면 리포지토리 계층에서 트랜잭션 시작해줌

#### 새로운 엔티티를 구별하는 방법

### 나머지 기능들
#### Specifications (명세)
#### Query By Example
#### Projections
#### 네이티브 쿼리

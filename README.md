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
#### 반환 타입
#### 순수 JPA 페이징과 정렬
#### 스프링 데이터 JPA 페이징과 정렬
#### 벌크성 수정 쿼리
#### @EntityGraph
#### JPA Hint & Lock

### 확장 기능
#### 사용자 정의 리포지토리 구현
#### Auditing
#### Web 확장 - 도메인 클래스 컨버터
#### Web 확장 - 페이징과 정렬

### 스프링 데이터 JPA 분석
#### 스프링 데이터 JPA 구현체 분석
#### 새로운 엔티티를 구별하는 방법

### 나머지 기능들
#### Specifications (명세)
#### Query By Example
#### Projections
#### 네이티브 쿼리

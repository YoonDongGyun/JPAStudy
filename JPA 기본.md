# JPA

- Java Application에서 관계형 데이터베이스를 사용하는 방식을 정의한 인터페이스
- ORM(Object-Relational Mapping) 스팩
  - Java 객체와 관계형 DB간의 매핑 처리를 위한 API
- Java Persistence API (2.2버전)
- Jakarta Persistence API (3.0버전)
  - 2.2버전부터 JCP -> 이클립스 재단으로 이관 진행
  - Jakarta EE 9 버전: JPA 3.0
- 보통 JPA만 단독으로 사용하기 보다는 스프링과 함께 사용
- 스프링 6 버전부터 Jakarta EE 9+ 지원

# JPA 특징
- Annotation을 이용한 Mapping 설정
  -XML 파일을 이용한 매핑 설정도 가능
- String, int, Local Date 등 기본적인 타입에 대한 매핑을 지원
- 커스텀 타입 변환기 지원
  -내가 만든 타입을 DB 칼럼에 매핑 가능
- Value 타입 매핑 지원
  - 한 개 이상 칼럼을 한 개 타입으로 매핑 가능
- 클래스 간 연관 지원: 1대1, 1대 N, N대 M 등
- 상속에 대한 매핑 지원

# JPA를 사용하는 이유
- 반복적인 CRUD SQL을 처리해줄 수 있다.
- SQL문이 아닌 객체를 중심으로 개발할 수 있다 -> 생산성을 높이고, 유지보수도 더 수월하게 할 수 있다
- 쿼리 문을 직접 작성할 필요가 없기 때문에 코드량이 줄어들고, 가독성을 높일 수 있다
- Oracle, MySQL 등 DB 벤더에 따라 조금씩 달라지는 SQL 문법 때문에 애플리케이션이 DB에 종속될 수 밖에 없었는데, JPA는 직접 쿼리를 작성하지 않아 DB 벤더에 독립적으로 개발이 가능하다

# 코드 구조
```java
//persistence.xml 파일에 정의한 영속단위 기준으로 초기화, 필요한 자원 생성
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabegin");
EntityManager entityManager = emf.createEntityManager(); //EntityManager 생성
EntityTransaction transaction = entityManager.getTransaction(); //EntityTransaction 구함

try{
  transaction.begin();//트랜잭션 시작
  
  //entityManager로 DB 작업 진행
  User user = new User("user@user.com", "user", LocalDateTime.now());
  entityManager.persist(user);
  
  transaction.commit();//트랜잭션 커밋 -> 쿼리는 커밋이 진행되는 시점에 실행된다
  //EntityManager 단위로 영속 컨텍스트를 관리하다가 커밋시점에 변경 내역을 DB에 반영하는 것이 특징
} catch(Exception e){
  e.printStackTrace();
  transaction.rollback(); //트랜잭션 롤백
} finally{
  entityManager.close(); //EntityManager 닫음
}

//팩토리 닫음
//사용한 자원 반환
emf.close();
```

# 시작 전 보조 클래스 만들기

- EntityManager를 쉽게 구하기 위한 클래스
```java
public class EMF {
  private static EntityManagerFactory emf;
  
  public static void init(){
    emf = Persistence.createEntityManagerFactory("jpabegin");
  }
  
  public static EntityManager createEntityManager() {
    return emf.createEntityManager();
  }
  
  public static void close() {
    emf.close();
  }
}
```

# Entity 단위 CRUD 처리
- EntityManager가 제공하는 method 이용
  - persist(Object entity): 객체를 입력받아 저장하는 method
  - find(Class <T> entityClass, Object primaryKey): 객체에서 해당하는 식별자를 찾는 method
  - remove(Object entity): 해당 객체를 삭제
  - merge()

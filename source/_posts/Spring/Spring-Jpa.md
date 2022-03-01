---
layout: post
title: Spring Boot - Spring Data JPA (with MySQL)
date: 2020-12-18 17:24:00 +0900
cover: /covers/spring.png
disqusId: afc725463a8aaaa14858b63b71b96aa55af4b662
toc: true
category: Spring
tags:
- spring
- spring-boot
---

Spring Boot에서 Spring Data JPA를 사용하여 MySQL을 다루는 법을 알아보겠습니다.

<!-- more -->

# 1. Dependency 추가

```gradle build.gradle
dependencies {
    runtimeOnly 'mysql:mysql-connector-java'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
}
```

# 2. 설정

```ini application.properties
spring.jpa.hibernate.ddl-auto=create
spring.datasource.url=jdbc:mysql://localhost:3306/mydb?serverTimezone=Asia/Seoul
spring.datasource.username=username
spring.datasource.password=password
```

`spring.jpa.hibernate.ddl-auto`의 값의 의미는 다음과 같다.

- none: 아무 것도 하지 않는다. (MySQL에서의 기본값)
- validate: DB와 엔티티의 스키마가 맞지 않으면 다른 부분을 출력하고 종료한다.
- update: 변경된 엔티티의 스키마를 적용한다.
- create: SessionFactory가 시작될 때, 기존의 테이블을 drop하고 새 테이블을 생성한다.
- create-drop: create의 동작 + SessionFactory가 종료될 때, 테이블을 drop 한다.

<article class="message message-immersive is-warning">
  <div class="message-body">
    <i class="fas fa-exclamation-triangle mr-2"></i>
    만약 본인 DB의 TimeZone이 Asia/Seoul가 아닐 경우 이에 맞게 바꿔주어야합니다.
  </div>
</article>

# 3. Entity 클래스 정의

```java
@Entity
@Table(name="user")
public class UserEntity {
    @Id
    @GeneratedValue
    private Integer id;

    @Column(nullable=false)
    private String name;

    @Column(columnDefinition="DATETIME(0) default CURRENT_TIMESTAMP")
    private OffsetDateTime dateTime;
}
```

- 컬럼 타입에 Integer, String 뿐만 아니라 Double, Char, Boolean등 다양한 Java Primitive Type을 사용할 수 있습니다.
- String은 컬럽 타입이 `VARCHAR(255)`로 자동으로 지정됩니다.
- `@Table`이나 `@Column`에서 `name`을 명시헤주지 않으면 해당 클래스나 멤버 변수의 이름을 snale_case로 변환한 것을 사용합니다.
- `nullable`을 false로 지정해주지 않으면 true가 기본값입니다.
- `columnDefinition`을 통해 컬럼 타입을 직접 지정해줄 수 있습니다.
LocalDateTime, OffsetDateTime, ZonedDateTime은 별다른 지정이 없어도 DATETIME으로 타입이 지정됩니다.
하지만 기본으로 소수점 6자리를 사용하도록 설정되기 때문에 저는 저렇게 길이를 0으로 명시하는 편입니다.
- `columnDefinition` 뒤에 `default [값]`을 붙여서 기본값을 지정해줄 수 있습니다.
컬럼이 시간, 날짜와 관련된 타입일 경우 `CURRENT_TIMESTAMP`를 붙여서 레코드가 생성될 때의 시간을 기본값으로 할 수 있습니다.

## 3.1. Default가 정의된 컬럼 다루기

```java
@Column()
@ColumnDefault("100")
private Integer score; // DB에 NULL이 저장됨!
```

위와 같이 Default값이 설정되어 있는 컬럼에 아무 것도 넣지 않고 save를 하면 DB에는 기본값인 100이 아니라 NULL값이 저장되어 있습니다.
이는 Hibernate가 INSERT 구문을 생성할 때 아래와 같이 모든 값을 명시하기 때문입니다.

```SQL
INSERT INT user (name, score) VALUE ('John', NULL) -- Hibernate가 생성한 Query

INSERT INT user (name) VALUE ('John') -- 이렇게 되어야 score에 기본값이 들어간다.
```

이 문제를 해결하기 위해 2가지 방법이 있습니다.

### 1) @DynamicInsert 사용하기

단순히 엔티티 클래스에 `@DynamicInsert` 어노테이션을 사용하면 됩니다.
`@DynamicInsert` 어노테이션은 값이 NULL인 컬럼을 INSERT 구문에 넣지 않는 기능을 합니다.
추가로 `@DynamicUpdate` 어노테이션도 존재하는데 이름에서 유추할 수 있듯이 값이 NULL인 컬럼을 UPDATE 구문에 넣지 않는 기능을 합니다.

```java
@Entity
@Table(name="user")
@DynamicInsert
public class UserEntity {
  
}
```

아래와 같이 엔티티를 save한 후에도 DB에는 기본값이 저장되어 있으나, 엔티티 객체에서는 해당 컬럼의 값이 그대로 null로 남아있다는 단점이 있습니다.

```java
repository.save(entity);

// DB에는 기본값인 100이 저장되어 있으나 여기서는 null이 저장되어 있음
Integer score = entity.getScore(); // null
```

### 2) 엔티티 클래스 멤버 변수에 기본값 주기

`@ColumnDefault` 어노테이션을 사용하지 않고 단순히 Java 코드 단계에서 기본값을 정의합니다.
DB 스키마에서 기본값을 확인할 수 없는 단점이 있습니다.

```java
@Entity
@Table(name="user")
public class UserEntity {
    @Column()
    private Integer score = 100;
}
```

## 3.2. NOT NULL과 Default를 동시에 정의하기

어떤 컬럼을 `NOT NULL`로 선언하고 해당 컬럼에 기본값을 주고 싶은 경우가 종종있습니다.
하지만 아래와 같이 컬럼을 정의하고 해당 컬럼의 값을 넣지 않고 save를 하면 `org.hibernate.PropertyValueException`이 발생합니다.

```java
@Column(nullable=false)
@ColumnDefault("100")
private Integer score;
```

이 컬럼의 nullable 여부는 INSERT 구문을 생성하기 전에 확인하는데 default값은 INSERT 구문이 실행해야 집어넣어지기 때문입니다.
이 문제를 해결하기 위해서는 아래와 같이 2가지 방법이 있습니다.

```java
// 방법 1
@Column(columnDefinition="INT default 100 NOT NULL")
private Integer score;

// 방법 2
@Column(nullable=false)
private Integer score = 100;
```

1. `columnDefinition`에서 컬럼의 타입과 기본값과 NOT NULL을 동시에 정의하는 방법
2. Java 코드에 직접 기본값을 정의

# 4. Repository 인터페이스 정의

```java
public interface UserRepository extends JpaRepository<UserEntity, Integer> {

}
```

실수로 내용을 빼먹은 것이 아닙니다.
단순히 JpaRepository만 상속하면 준비는 끝났습니다.
JpaRepository를 상속할 때 두번째 제너릭은 Entity의 @Id 컬럼의 타입을 넣어주면 됩니다.

# 5. 간단한 CRUD

## 5.1. SELECT

```java
@Bean
private UserRepository userRepository;

public void select() {
    List<UserEntity> list = repository.findAll();
    Optional<UserEntity> user = repository.findById(1);
    long num = repository.count();
}
```

## 5.2. INSERT

```java
@Bean
private UserRepository userRepository;

public void insert() {
    UserEntity user = new UserEntity("John");
    repository.save(user);

    int id = user.getId(); // 1
}
```

repository에 save한 후 생성된 기본키를 가져올 수 있습니다.

## 5.3. UPDATE

```java
public void update() {
    UserEntity user = repository.findById(1).orElseThrow(() -> new IllegalArgumentException("Not Found"));
    user.setName("James");
    repository.save(user);
}
```

## 5.4. DELETE

```java
public void delete() {
    UserEntity user = userRepository.findById(1).orElseThrow(() -> new IllegalArgumentException("Not Found"));
    userRepository.delete(user);

    // 또는
    repository.deleteById(1);
}
```

# 6. 조건절이 있는 CRUD

## 6.1. 메서드명으로부터 쿼리 생성

```java
public interface UserRepository extends JpaRepository<UserEntity, Integer> {
    List<UserEntity> findByName(String name);
    List<UserEntity> findByNameAndScore(String name, int score);
    long countByName(String name);
}
```

이렇게 단순히 `UserRepository` 인터페이스에 메서드를 추가해주는 것으로 검색 쿼리를 생성할 수 있습니다.
And, Or, Between, LessThan, Like 등 다양한 조건을 추가해줄 수 있습니다.
자세한 정보는
[문서](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)
를 참고해 주세요

<article class="message message-immersive is-primary">
  <div class="message-body">
    <i class="fas fa-info-circle mr-2"></i>
    메서드의 파라미터 이름이 메서드 이름에 적힌 프로퍼티 이름과 같아야할 필요는 없습니다.
    단순히 메서드의 파라미터의 순서대로 바인드되기 때문입니다.
  </div>
</article>

## 6.2. @Query 어노테이션

하지만 위와 같은 방법으로는 SELECT 쿼리만 사용할 수 있을뿐만 아니라 소괄호를 통한 연산 우선순위를 적용하는 등 복잡한 쿼리를 할 수 없습니다.
이제부터 직접 쿼리문을 작성하는 방법을 소개하겠습니다.

```java
public interface UserRepository extends JpaRepository<UserEntity, Integer> {
    @Query("SELECT u FROM UserEntity u WHERE u.name = :name")
    List<UserEntity> findByName(String name);

    @Transactional
    @Modifying
    @Query("UPDATE UserEntity u SET u.name = :to WHERE u.name = :from")
    Integer updateName(String from, String to);

    @Transactional
    @Modifying
    @Query("DELETE FROM #{#entityName} u WHERE u.name = :name")
    Integer deleteByName(String name);
}
```

여기서 `@Query` 어노테이션 안에 있는 쿼리문은 일반적인 SQL이 아니라 JPQL이라는 Java 독자적인 쿼리 언어입니다.
여기서 테이블명이나 컬럼명 대신 엔티티 클래스 이름과 앤티티 클래스의 멤버 번수 이름을 사용합니다.
UPDATE나 DELETE같이 DB를 변경해야하는 경우에는 반드시 `@Transactional`, `@Modifying` 어노테이션을 사용해야합니다.
자세한 사용방법은 [여기](https://www.baeldung.com/spring-data-jpa-query)를 참고해 주세요.

<article class="message message-immersive is-primary">
  <div class="message-body">
    <i class="fas fa-info-circle mr-2"></i>
    JPQL에 엔티티 클래스 이름을 직접 집어넣지 않고 {% raw %}<code>#{#entityName}</code>{% endraw %}을 넣음으로써
    현재 레포지토리의 엔티티 클래스 이름을 자동으로 바인딩 할 수 있습니다.
  </div>
</article>

## 6.3. Example

하지만 위와같은 방법으로는 검색 조건의 수가 유동적인 경우에는 사용할 수 없습니다.
예시 엔티티를 생성해서 비교하는 벙법을 소개하겠습니다.

```java
UserEntity toCompare = new UserEntity(1, "John", null);
Example<UserEntity> ex = Example.of(toCompare);
List<UserEntity> users = repository.findAll(ex);
users.forEach(System.out::println);
```

예시 엔티티를 생성해서 `Example.of`메서드로 예시를 생성하고 `findAll`이나 `findOne` 메서드에 넘겨주면 됩니다.
예시 엔티티에서 값이 NULL아닌 모든 프로퍼티와 비교합니다.
`ExampleMatcher`를 통해 문자열의 대소문자 구분 없음, 정규표현식, 포함관계의 비교를 할 수 있습니다.
자세한 사용방법은 [여기](https://www.baeldung.com/spring-data-query-by-example)를 참고해 주세요.

## 6.4. JpaSpecificationExecutor

하지만 위와같은 방법으로는 검색 조건의 수가 유동적이며 범위 검색이 있는 경우에는 사용할 수 없습니다.
Repository에 JpaSpecificationExecutor를 상속하는 방법을 소개하겠습니다.

```java
public interface UserRepository extends JpaRepository<UserEntity, Integer>, JpaSpecificationExecutor<UserEntity> {

}
```

먼저 Repository 인터페이스에 추가로 `JpaSpecificationExecutor`를 상속합니다. 

```java
public class UserSpecs {
    public static Specification<UserEntity> withName(final String name) {
        return (root, query, builder) -> builder.equal(root.get("name"), name);
    }

    public static Specification<UserEntity> withScoreRange(final Integer from, final Integer to) {
        return (root, query, builder) -> builder.between(root.get("score"), from, to);
    }
}
```

먼저 각각의 조건 항을 정의하는 메서드를 만듭니다.
여기서 `builder`는 `CriteriaBuilder`라는 클래스인데 equal, between 뿐만 아니라 여러분이 생각할 수 있는 모든 것이 다 있습니다.
자세한 정보는 [문서](https://docs.oracle.com/javaee/7/api/javax/persistence/criteria/CriteriaBuilder.html)를 확인해주세요.

```java
Specification<UserEntity> spec = Specification.where(null);

if (map.containsKey("name")) {
    spec = spec.and(UserSpecs.withName(map.get("name")));
}

if (map.containsKey("fromScore") && map.containsKey("toScore")) {
    spec = spec.and(UserSpecs.withScoreRange(map.get("fromScore"), map.get("toScore")));
}

List<UserEntity> users = repository.findAll(spec);
```

위와같이 파라미터로 넘어온 map에 키가 있는지 없는지 체크하면서 조건을 추가해줄 수 있습니다.
여기서 and 메서드말고 or, not 메서드도 사용할 수 있습니다.

# 7. 정렬과 페이징처리

## 7.1. Repository의 기본 메서드

### 1) 정렬

```java
List<UserEntity> list = repository.findAll(Sort.by(Sort.Direction.ASC, "score"));
```

`Sort` 클래스를 통해 정렬을 할 수 있습니다.

### 2) 페이징

```java
List<UserEntity> list = repository.findAll(PageRequest.of(0, 10));

List<UserEntity> list = repository.findAll(PageRequest.of(0, 10, Sort.by(Sort.Direction.ASC, "score")));
```

`PageRequest` 클래스를 통해 페이징 처리를 할 수 있습니다.
페이지는 0부터 시작합니다.
세번째 파라미터로 Sort 객체를 넘겨줌으로써 페이징 처리와 정렬을 동시에 할 수 있습니다.
자세한 정보는
[PageRequest 공식 문서](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/PageRequest.html),
[Sort 공식 문서](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Sort.html)
를 참고해 주세요.

## 7.2. 메서드명으로부터 쿼리 생성

### 1) 정렬

```java
public interface UserRepository extends JpaRepository<UserEntity, Integer> {
    List<UserEntity> findByNameOrderByScoreDesc(String name);
    List<UserEntity> findByName(String name, Sort sort);
}
```

1. 메서드 명뒤에 `OrderBy[프로퍼티이름][Desc|Asc]`를 붙이면 됩니다.
2. 아니면 `Sort` 객체를 맨 끝에 파라미터로 추가하면 됩니다.

### 2) 페이징

```java
public interface UserRepository extends JpaRepository<UserEntity, Integer> {
    List<UserEntity> findByName(String name, Pageable pageable);
    Slice<UserEntity> findByName(String name, Pageable pageable);
    Page<UserEntity> findByName(String name, Pageable pageable);
}
```

Pageable 객체를 맨 끝에 파라미터로 추가하면 됩니다.
반환 타입도 위의 3개중에 아무거나 하나 고르면 JPA가 다 알아서 합니다.

#### Slice

`hasNext`, `hasPrevious`, `isFirst`, `isLast`의 메서드를 통해 다음, 이전 페이지가 존재하는지, 현재 페이지가 처음이나 마지막인지 알 수 있습니다.
이외에도 다양한 기능을 제공하니 자세한 정보는
[문서](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Slice.html)
를 확인해 주세요.

#### Page

위의 Slice를 상속받은 것이기 때문에 Slice의 모든 기능을 쓸 수 있습니다.
내부적으로 COUNT 함수를 실행시켜서 검색 결과의 총 갯수를 구합니다.
COUNT 함수의 오버헤드가 큰 경우 별로 추천하지 않습니다.
`getTotalElements`, `getTotalPages` 메서드를 통해 전체 결과 갯수, 전체 페이지 수를 구할 수 있습니다.
자세한 정보는
[문서](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Page.html)
를 확인해 주세요.

## 7.3. @Query 어노테이션

```java
public interface UserRepository extends JpaRepository<UserEntity, Integer> {
    @Query("SELECT u FROM UserEntity u WHERE u.name = :name")
    Page<UserEntity> findByName(String name, Pageable pageable);
}
```

위에서 한 것 처럼 `Sort`나 `Pageable`객체를 파라미터에 추가하고 반환 타입을 `List`, `Slice`, `Page`로 바꾸면 JPA가 다 알아서 합니다.

## 7.4. Example

```java
List<UserEntity> users = repository.findAll(ex, PageRequest.of(0, 10));
```

메서드 파라미터 뒤에 `Sort`나 `Pageable`객체를 넘겨주면 됩니다.

## 7.5. JpaSpecificationExecutor

```java
List<UserEntity> users = repository.findAll(spec, PageRequest.of(0, 10));
```

마찬가지로 메서드 파라미터 뒤에 `Sort`나 `Pageable`객체를 넘겨주면 됩니다.

# 8. 기본 실행 SQL 파일

resources 폴더 루트에 `import.sql` 이라는 파일이 있으면 Hibernate는 Entity를 통한 Table 자동 생성이 끝난 후 이 파일을 실행합니다.
보통 실행때마다 데이터를 Drop하는 설정에서 요긴하게 쓰입니다.

# Appendix

### dependency에 JPA를 포함했지만 JPA 설정을 건너뛰고 싶은 경우

```
spring.autoconfigure.exclude= \
  org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration, \
  org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration, \
  org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration
```

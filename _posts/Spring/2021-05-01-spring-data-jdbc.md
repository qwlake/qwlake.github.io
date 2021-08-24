---
layout: post
title: "[Spring] Spring Data JDBC"
author: qwlake
categories: Spring
tags: jdbc
---

# Spring Data JDBC

[https://www.baeldung.com/spring-data-jdbc-intro](https://www.baeldung.com/spring-data-jdbc-intro)

> SpringData JDBC는 SpringData JPA만큼 복잡하지 않은 지속성 프레임 워크입니다. 캐시, 지연로드, write-behind 또는 기타 JPA의 많은 기능을 제공하지 않습니다. 그럼에도 불구하고 자체 ORM이 있으며 **매핑 된 엔티티, 리포지토리, 쿼리 주석 및 *JdbcTemplate* 과 같이 *SpringData*** JPA **와** 함께 사용되는 대부분의 기능을 제공합니다 .
명심해야 할 중요한 점은 **SpringData JDBC가 스키마 생성을 제공하지 않는다는 것** 입니다. 결과적으로 우리는 명시 적으로 스키마를 생성해야합니다.

## Adding Entities

```java
public class Person {
    @Id
    private long id;
    private String firstName;
    private String lastName;
    // constructors, getters, setters
}
```

@Table or @Column을 추가할 필요가 없다. Spring Data JDBC의 기본 네이밍 전략은 엔티티와 테이블간에 암묵적으로 매핑되는 것이다.

## Declaring JDBC Repositories

```java
@Repository 
public interface PersonRepository extends CrudRepository<Person, Long> {
}
```

Spring Data JDBC repository는 Repository, CrudRepository, or PagingAndSortingRepository 인터페이스를 확장하여 생성할 수 있다. 
예를 들어 CrudRepository를 구현한 경우 save, delete, findById 등을 사용할 수 있다.

## Customizing JDBC Repositories

```java
@Repository
public interface PersonRepository extends CrudRepository<Person, Long> {

    List<Person> findByFirstName(String firstName);

    @Modifying
    @Query("UPDATE person SET first_name = :name WHERE id = :id")
    boolean updateByFirstName(@Param("id") Long id, @Param("name") String name);
}
```

non-modifying query는 findByFirstName같이 이름을 지정해 주면 쿼리가 알아서 생성됨

다만 modifying query는 @Modifying와 @Query를 붙여야함 (필수인지? Spring Data Jpa처럼 변경 감지 안 되나??)

## Populating the Database

```java
@Component
public class DatabaseSeeder {

    @Autowired
    private JdbcTemplate jdbcTemplate;
    public void insertData() {
        jdbcTemplate.execute("INSERT INTO Person(first_name,last_name) VALUES('Victor', 'Hugo')");
        jdbcTemplate.execute("INSERT INTO Person(first_name,last_name) VALUES('Dante', 'Alighieri')");
        jdbcTemplate.execute("INSERT INTO Person(first_name,last_name) VALUES('Stefan', 'Zweig')");
        jdbcTemplate.execute("INSERT INTO Person(first_name,last_name) VALUES('Oscar', 'Wilde')");
    }
}
```

## Conclusion

Spring Data JDBC는 Spring Data JPA에 비해 속도가 빠름. Spring Data JDBC는 DB랑 다이렉트로 통신하기 때문. 단, Spring Data JDBC의 단점은 데이터베이스의 vendor에 대한 의존성이다. 예를 들어, DB를 MySQL에서 Oracle로 변경 시 문제가 발생할 수 있다.
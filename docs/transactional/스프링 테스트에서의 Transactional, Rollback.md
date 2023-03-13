---
layout: default
title: 스프링 테스트에서의 Transactional, Rollback
nav_order: 1
has_children: false
parent: Transactional
permalink: /docs/transactional/transactional-in-spring-data-test
---


# 스프링 테스트에서의 Transactional, Rollback
{: .no_toc }
<br>
<br>

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

### 테스트에서 중요한 원칙
{: .fs-6 .fw-700 }
- 테스트는 다른 테스트와 격리되어 있어야 한다.
- 테스트는 반복해서 실행할 수 있어야 한다.

### DB 연결 세팅
{: .fs-6 .fw-700 }

#### application.properties 
{: .fs-5 .fw-700 }
application.properties 파일은 애플리케이션 영역과 테스트 영역을 구분해서 지정가능하다.<br>

예를 들면 아래의 디렉터리에 따로 두고 사용할 수 있다.<br>

- src/main/resources/application.properties
- src/test/resources/application.properties

테스트코드에서 코드를 실행할 때에는 src/test/resources/application.properties 파일이 우선순위를 가지게 된다.<br>
<br>
<br>

#### h2 세팅
{: .fs-5 .fw-700 }

h2 데이터베이스는 개발 PC에 설치해서 실행할 수도 있고, 인메모리로 실행할 수도 있다.
h2 접속 주소는 개발PC에 설치해서 사용할지, 인메모리로 사용할지에 따라서 접속주소가 달라지는데 각 접속 주소는 아래와 같아지게 된다.
- 개발 PC 에 접속해서 사용시 : `h2:tcp//localhost/~/test`
- 인메모리로 접속해서 사용시 : `jdbc:h2:mem:db;DB_CLOSE_DELAY=1`
  - 참고) 스프링부트는 데이터베이스에 대한 설정이 없을 경우 임베디드 데이터베이스를 사용한다.
<br>
<br>

##### 애플리케이션 영역 properties 세팅
{: .fs-4 .fw-700 }
개발 PC에 설치해서 사용<br>

src/main/resources/application.properties
```
spring.profiles.active=local
spring.datasource.url=jdbc:h2:tcp//localhost/~/test
spring.datasource.username=sa
```
<br>
<br>

##### 테스트 영역 properties 세팅
{: .fs-4 .fw-700 }

##### 개발PC 의 h2 에 접속하도록 해서 실행
{: .fs-4 .fw-700 }
```
spring.profiles.active=test
spring.datasource.url=jdbc:h2:tcp//localhost/~/test
spring.datasource.username=sa
```
<br>

##### 인메모리 h2 에 접속하도록 해서 실행
{: .fs-4 .fw-700 }
src/test/resources/application.properties
```
spring.profiles.active=test
spring.datasoruce.url=jdbc:h2:mem:db;DB_CLOSE_DELAY=1
spring.datasource.username=sa
```
<br>
<br>

#### h2 용도별 분리
{: .fs-5 .fw-700 }
h2를 애플리케이션에서 사용할 때와 테스트에서 사용할 때의 데이터베이스를 다르게 줄수 있게끔 따로 데이터베이스를 생성하고 지정할 수 있다.<br>

예를 들면 아래와 같이 분리하는 것을 예로 들 수 있다.<br>

- 로컬에서 접속하는 h2 DB
  - jdbc:h2:tcp//localhost/~/test
- test 에서 접속하는 h2 DB
  - jdbc:h2:tcp://localhost/~/testCase

<br>

데이터베이스 파일을 생성하는 방법은 아래와 같다.

- 데이터베이스 서버를 종료하고 다시 실행
- 사용자명은 `sa` 입력
- JDBC URL 에 `jdbc:h2:~/testCase` 입력 (최초한번)
- `~/testcase.mv.db` 파일 생성 확인
- 이후 부터는 `jdbc:h2:tcp//localhost/~/testcase` 이렇게 접속

<br>
<br>

### h2 대안
{: .fs-6 .fw-700 }

실무에서 JPA, Querydsl 만 쓰는 경우는 드물다. Mybatis, JdbcTemplate 으로 raw 쿼리를 작성하는 경우도 꽤 있다. UPSERT 등으로 일정 BATCH 사이즈 만큼의 insert/update 쿼리를 사용하기 위해 주로 사용하는데, 이런 경우는 docker-compose.yml, testcontainer 를 활용해서 가상 DB를 로딩해서 활용하는 방법이 있다.<br>
<br>
<br>

### 테스트 후 데이터 롤백
{: .fs-6 .fw-700 }

예를 들어 이전 테스트케이스에서 어떤 테이블에 A라는 데이터를 저장했고 커밋해둔 상태였다고 해보자. 그리고 이 데이터는 롤백하지 않은 채로 그대로 두었다.<br>

그리고 다른 테스트 케이스에서는 A라는 데이터를 insert 후에 이것이 유효성 체크를 통과 후에 조회를 제대로 하는지 체크하는 테스트를 구현해야 한다고 해보자. 이 경우 A라는 데이터가 이미 저장되어 있기에 중복된 키가 insert 되기에 테스트케이스가 실패하게 된다.<br>

이런 이유로 테스트 케이스에서는 아래와 같은 방식으로 트랜잭션을 롤백하는 코드를 테스트 케이스마다 실행하도록 작성한다.<br>
```plain
// 테스트케이스 A
트랜잭션 시작
테스트 A 실행
트랜잭션 롤백

// 테스트케이스 B
트랜잭션 시작
테스트 B 실행
트랜잭션 롤백
```
<br>
<br>

이렇게 테스트케이스마다 트랜잭션을 롤백시키는 것은 아래의 두가지 방식으로 구현 가능하다.
- 직접 구현 (@BeforeEach, @AfterEach 를 이용)
  - 트랜잭션 매니저 인스턴스를 이용해서 직접 commit, rollback 하는 로직을 작성하는 방식이다.
  - PlatformTransactionManager 를 의존성 주입으로 주입 받아서 이 트랜잭션 매니저로 @BeforeEach, @AfterEach 에서 commit, rollback 을 구현하면 된다.
- @Transactional 사용
  - @Transactional 이 적용된 메서드의 수행이 완료되면 수행했던 테스트에 대한 DB 수정사항을 Rollback 하는 방식

<br>

테스트 후 롤백하는 로직을 직접 구현할 경우의 예제를 정리해보고, @Transactional 을  사용하도록 전환하면 어떻게 코드가 단순해지는지를 정리해보기로 했다.<br>
<br>
<br>

#### 직접 구현할 경우 (@BeforeEach, @AfterEach)
{: .fs-5 .fw-700 }


트랜잭션 매니저 인스턴스를 이용해서 직접 `commit()` , `rollback()` 하는 로직을 작성하는 방식이다. `PlatformTransactionManager` 를 의존성 주입으로 주입받아서 이 트랜잭션 매니저 인스턴스를 이용해 `@BeforeEach` , `@AfterEach` 에서 `commit()` , `rollback()` 메서드를 호출하게끔 구현하는 방식이다.<br>
<br>

아래 코드를 보면,<br>
- @BeforeEach 가 적용된 메서드에서는  `transactionManager.getTransactionManager(new DefaultTransactionDefinition())` 으로 트랜잭션을 시작하고 있다.
- @AfterEach 가 적용된 메서드에서는 `transactionManager.rollback(status)` 를 통해 트랜잭션을 롤백하고 있음을 확인하고 있다.
<br>
<br>

```java
@SpringBootTest
class SomethingTest{
	@Autowired
	PlatformTransactionManager transactionManager;
	// ...

	TransactionStatus status;

	@BeforeEach
	void beforeEach(){
		status = transactionManager.getTransaction(new DefaultTransactionDefinition());
	}

	@AfterEach
	void afterEach(){
		// ...
		transactionManager.rollback(status);
	}

	// ...
	@Test
	public void INSERTION_TEST(){
		// ... 
	}
}
```
<br>
<br>

#### @Transactional 사용방식
{: .fs-5 .fw-700 }
스프링에서는 위와 같이 직접 작성하던 트랜잭션 커밋/롤백 로직을 `@Transactional` 어노테이션 하나로 모두 대체가능하다.<br>

위에서 사용한 코드는 아래와 같이 어노테이션 하나로 대체가 된다.<br>

메서드 레벨에 대해 `@Transactional` 을 적용한 테스트 코드다.<br>

```java
@SpringBootTest
class SomethingTest{

	@Transactional
	@Test
	public void SOME_TEST(){
		// ...
	}
}
```
<br>
또는 아래와 같이 클래스 레벨에 `@Transactional` 을 적용하는 것도 가능하다.<br>
`@Transactional` 을 class 레벨에 선언하면, class 내의 public 클래스들에 모두 `@Transactional` 이 적용되게 된다.<br>
<br>

```java
@SpringBootTest
@Transactional
class SomethingTest{

	@Test
	public void SOME_TEST(){
		// ...
	}
}
```

### 테스트 코드에서 @Transactional 을 사용하는 것의 장점
{: .fs-6 .fw-700 }

- 테스트가 끝난 후 개발자가 직접 데이터를 삭제하지 않아도 되기에 유지보수가 편리해진다.
- 테스트 실행 중에 데이터를 등록하고 중간에 테스트가 종료되도 테스트 중에는 트랜잭션을 커밋하지 않기에, 롤백이 수행되지 않더라도 데이터는 자동으로 롤백된다. (데이터베이스 커넥션이 끊어지면, 커밋되지 않은 데이터는 자동으로 롤백된다.)
- 트랜잭션 범위에서 테스트를 진행하므로 다른 테스트를 진행하더라도 서로 영향을 주지않는다는 장점이 있다.
- @Transactional 을 사용하면 아래의 두가지 원칙을 지킬 수 있게 된다.
  - 테스트는 다른 테스트와 격리해야 한다.
  - 테스트는 반복해서 실행할 수 있어야 한다.
<br>
<br>

### 테스트 코드에서의 @Transactional
{: .fs-6 .fw-700 }

> 별다른 내용은 없지만, 뭔가 아쉬워서 좀 더 정리해봤다.

<br>

`@Transactional` 어노테이션을 테스트에서 사용하면 애플리케이션 영역에서 @Transactional 을 사용할 때와는 다르게 동작한다.<br>

테스트클래스/메서드에서 `@Transactional` 을 사용할 때 스프링은 그 테스트를 트랜잭션 범위에서 실행하고 테스트가 끝나면 **자동으로 트랜잭션을 rollback 한다.**<br>

(애플리케이션 계층에서 @Transactional 은 @Transactional 이 적용된 메서드가 실행이 성공적으로 수행되면 커밋하게끔 동작한다.)<br>

아래 그림은 @Transactional 이 적용된 테스트 케이스가 실행될때의 동작을 그림으로 나타낸 그림이다. 자세히 보면, 트랜잭션의 마지막에 꼭 rollback 을 수행하는 것을 볼 수 있다.<br>
<br>

![1](./img/SPRING-TEST-TRANSACTIONAL/TRANSACTIONAL-ROLLBACK-1.png)

<br>
트랜잭션내에서 SELECT SQL을 수행하고 있다. 이때 같은 트랜잭션 내에서 조회하는 것이기에 INSERT SQL 로 실행한 결과가 조회된다. 하지만, 다른 트랜잭션에서는 INSERT 한 결과는 조회되지 않는다.<br>

테스트가 끝난 후에는 트랜잭션을 강제로 롤백한다. @Transactional 이 적용된 테스트 코드는 테스트가 끝날 때 **트랜잭션을 강제 롤백**한다.<br>

서비스/리포지터리 에 적용한 `@Transactonal` 도 테스트에서 시작한 트랜잭션으로 참여한다. 서비스/리포지터리 코드를 포함해서 테스트 코드가 실행하는 모든 코드는 테스트가 시작된 트랜잭션에 참여한다. (트랜잭션 전파 개념 .. 추후 정리 예정)<br>
<br>
<br>

#### @Commit
{: .fs-5 .fw-700 }
@Transactional 을 테스트 클래스/메서드에 붙였더라도 `@Commit` 을 클래스/메서드에 붙이면 테스트 종료 후 롤백 대신 커밋이 호출된다.
> 가끔 데이터베이스 테이블에 데이터가 잘 저장/수정 됐는지 육안으로 확인하려 할 때 드물게 @Commit, @Rollback 을 사용하기도 한다. 이런 코드를 작성한 후에는 해당 코드를 가급적이면 전체 테스트 범위에 포함되지 않도록 `@Disabled` 처리를 해주자.
<br>

```java
import org.springframework.test.annotation.Commit;

@Commit
@Transactional
@SpringBootTest
class SomethingTest{
    // ...
}
```
<br>
<br>

#### @Rollback(value = false)
{: .fs-5 .fw-700 }

> 가끔 데이터베이스 테이블에 데이터가 잘 저장/수정 됐는지 육안으로 확인하려 할 때 드물게 @Commit, @Rollback 을 사용하기도 한다. 이런 코드를 작성한 후에는 해당 코드를 가급적이면 전체 테스트 범위에 포함되지 않도록 `@Disabled` 처리를 해주자.
<br>

@Transactional 을 테스트 클래스/메서드에 붙였더라도 `@Rollback(value = false)` 를 클래스/메서드에 붙이면 테스트 종료 후 롤백 대신 커밋이 호출된다. (rollback=false 로 줘서 **'롤백하지마!'** 라고 프로그램에게 이야기해주는 것)
<br>

```java
import org.springframework.test.annotation.Commit;

@Rollback(value = false)
@Transactional
@SpringBootTest
class SomethingTest{
    // ...
}
```
<br>
<br>

### etc
진짜 정리하기 귀찮아서 미뤄두고 있었는데, 오늘 갑자기 하루 날 잡고 정리를 하게되었다...

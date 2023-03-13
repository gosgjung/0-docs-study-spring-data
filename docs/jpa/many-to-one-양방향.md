---
layout: default
title: 1 vs N 양방향
nav_order: 4
has_children: false
parent: JPA
permalink: /docs/jpa/many-to-one-mirrored
---


# 1 vs N 단방향
{: .no_toc }
<br>
<br>

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

### 1 vs N 양방향
{: .fs-6 .fw-700 }

양방향을 처음 배울때는 스트레스를 꽤 받는다. 사실은 그냥 코드 보고 테스트 코드 출력결과와 함께 조합해보면 이해가 간다. 그냥 배우면 되는데, 주변에서 자꾸 C언어의 포인터 같다면서 자꾸 긴장시킨다. 그냥 코드 보면 이해간다.
<br>

까먹을 수 있다는 점도 꽤 문제다. 일하다보면 JPA나 Querydsl 만 쓸수 있는 건 아니다. Mybatis 를 쓰거나 JDBCTemplate 을 헤비하게 써야 할때도 있다. 만약 JPA에 대해 까먹은 상태에서 다시 스터디를 시작한다면... 1 vs N 양방향에서는 `mappedBy` 에 어떤 값을 지정하는지만 기억하고 있으면, 언제든지 MyBatis, JdbcTemplate 을 쓰다가도 안 까먹고 다시 기억하고 쓸 수 있다. 이걸 이해했다는 것은 테이블 두개를 같은 키로 매핑할 때와 객체 두개를 같은 키로 매핑할때 왜 다른지를 감으로 이해한 결과가 된다.<br>
<br>

@OneToMany 에 사용하는 `mappedBy` 는 N vs 1 에서 `1` 에 해당하는 측의 매핑을 당하는 필드에 주로 지정한다.<br>
<br>

예를 들어 Employee, Department 객체간의 관계를 보자. Department 입장에서는 여러 명의 Employee 가 존재한다. 따라서 Department 내에는 `List<Employee> employees` 라는 필드가 존재하게 된다. 그리고 이 employees 라는 리스트를 실제 조인이 수행되게끔 하려면 프로그래밍 적으로 별도의 장치가 필요한데, JPA 는 `@OneToMany` 라고 하는 별도의 어노테이션을 사용해서 조인관계를 지정해줄 수 있다.
<br>

이렇게 지정된 Department 클래스 내의 `@ManyToOne` 어노테이션에는 `mappedBy` 라는 속성이 있다. 그리고 상대편 테이블인 Employee 클래스에는 `Department dept;` 와 같이 정의해둔 Department 클래스에 대한 참조 점이 있다. Department 클래스 내의 `employees` 에는 상대편 객체인 `Employee` 객체 내의 `dept` 변수에 대해 조인을 수행하게 끔 `mappedBy = "dept"` 라고 지정해준다. 쉽게 이야기하면, 상대편 테이블의 변수명을 지정해주는 방식이다. 내부적으로는 리플렉션을 통해 주입하기 때문이다.
<br>
<br>

여기 까지만 이해하면 1 vs N 의 50% 는 이해했다. 이 외에 N+1 문제도 있고 연관관계 편의 메서드 등 여러가지 이야기들이 있다.
<br>
<br>

### fetch, optional, cascade
{: .fs-6 .fw-700 }

@ManyToOne 에 대해 부가적으로 지정하는 파라미터들.

**fetch**
- @ManyToOne 에 대한 fetch 기본 디폴트 속성은 FetchType.EAGER 이다.
- @OneToMany 에 대한 fetch 기본 디폴트 속성은 FetchType.LAZY 다.
직관적으로 따져보면 Many 에 해당하는 객체가 많기에 Lazy 로딩하도록 지정
<br>

**optional**
- 기본 join 전략을 outer join 으로 할지, inner join 으로 할지에 대한 옵션
- @ManyToOne 에서는 optional = true 가 기본설정이다. 
- optional = true 로 설정하면 left outer join 을 수행하게 된다.
- 참고) optional = false 일때는 inner join 이 실행된다.
<br>

**cascade**
- 영속성 전이에 대한 옵션
- 연관 엔티티를 같이 저장하거나 삭제할 때 사용
<br>
<br>

### e.g. Employee, Department
{: .fs-6 .fw-700 }

시퀀스 채번 전략은 Postgresql 의 규칙으로 했다.
연관관계의 상대편인 DEPT 테이블에 대한 객체인 Department 객체를 FetchType.LAZY로 로딩하도록 설정해두었다.
<br>
양방향 매핑을 하게 되면 코드가 아래와 같아진다.
<br>
<br>

#### Employee.java
{: .fs-5 .fw-700 }

```java
@Getter
@Entity
@SequenceGenerator(
    name = "employee_sequence",
    schema = "public", sequenceName = "EMP_SEQ",
    initialValue = 1, allocationSize = 1
)
@Table(name = "EMP", schema = "public")
public class Employee {

    @Id @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "employee_sequence")
    @Column(name = "EMPLOYEE_ID")
    private Long id;

    @Column(name = "EMPLOYEE_NAME")
    private String name;

    @ManyToOne(optional = false, fetch = FetchType.LAZY)
    @JoinColumn(name = "DEPT_ID")
    private Department dept;

    public Employee(String name, Department dept){
        this.name = name;
        this.dept = dept;
    }
}
```

<br>
<br>

#### Department.java
{: .fs-5 .fw-700 }

```java
@Entity
@SequenceGenerator(
    schema = "public", sequenceName = "DEPT_SEQ", name = "department_seq",
    initialValue = 1, allocationSize = 1
)
@Table(name = "DEPT", schema = "public")
public class Department {

    @Id @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "department_seq")
    @Column(name = "DEPT_ID")
    private Long id;

    @Column(name = "DEPT_NAME")
    private String deptName;

    @OneToMany(mappedBy = "dept")
    List<Employee> employees = new ArrayList<>();

    public Department(String deptName){
        this.deptName = deptName;
    }
}
```
<br>
<br>

자세히 보면 Department 클래스의 `employees` 필드에 @OneToMany(mappedBy = "dept") 어노테이션이 적용되어 있다. 이 **"dept"** 는 Employee 클래스 내에서 `private Department dept;` 와 같이 선언되어 있다. 이렇듯 **@OneToMany** 의 mappedBy 에는 지정하는 연관관계의 상대편 클래스에 `List`, `Set` 등의 타입으로 선언된 변수의 변수 명을 정의해준다.
<br>

이렇게 하는 이유는 DB 입장에서는 List 와 같은 선형자료구조로 조인되는 것은 아니지만, 객체 입장에서는 선형 자료구조로 매핑되기 때문에 이렇게 `List`, `Set` 등의 컬렉션에 매핑을 해주게 된다.
<br>
<br>




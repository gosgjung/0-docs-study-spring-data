---
layout: default
title: 1 vs N 단방향
nav_order: 1
has_children: false
parent: JPA
permalink: /docs/jpa/many-to-one-single
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

### ManyToOne 단방향
{: .fs-6 .fw-700 }

단방향에서는 따로 정리할 만한 내용이 없다. 단순한 내용들이다. 그래서 optional 에 지정한 값에 따라 쿼리가 어떻게 달라지는지만 간략히 정리해봤다.
<br>
<br>

### fetch, optional, cascade
{: .fs-6 .fw-700 }

@ManyToOne 에 대해 부가적으로 지정하는 파라미터들.

**fetch**
- 지연로딩, 즉시로딩 여부를 결정짓는 옵션
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

//    @OneToMany(mappedBy = "dept")
//    List<Employee> employees = new ArrayList<>();

    public Department(String deptName){
        this.deptName = deptName;
    }
}
```
<br>
<br>


### optional
{: .fs-6 .fw-700 }

위에서 한번 정리했지만 한번 더... 정리해봤다. (쓸 내용이 없어서^^;;)
- optional = true
  - `@ManyToOne` 의 기본 설정은 optional = true 이다.
  - optional = true 로 세팅하면, left outer join 을 수행하게 된다.
  - `@ManyToOne` 의 상대편 테이블은 One 에 해당하는 연관관계이므로, 조인할 대상이 상대적으로 적기 때문에 jpa의 `@ManyToOne` 의 One 에 대한 Join은 left outer join 으로 지정하고 있다.
- optional = false
  - inner join 을 수행하게 된다.
<br>

left outer join 쿼리는 조인을 하는 상대편 테이블 내의 null 이 아닌 로우도 같이 들고 와야 하기에 성능 측면에서는 조금 불리한 측면이 있다.
<br>
<br>

#### optional = true 단건 조회 테스트
{: .fs-5 .fw-700 }
엔티티 매핑은 위에서 이미 정의를 해두었다. 쿼리를 직접 날려서 조인이 어떻게 나오는지 확인해보자.
아래 코드는 단순히 Employee 리포지터리에서 데이터를 가져오는 쿼리다.

```java
@Test
@DisplayName("optional_false_테스트_단건조회")
public void optional_false_테스트_단건조회(){
  em.flush();
  em.clear();
  Employee e1 = employeeRepository.findById(3L).orElseGet(()->{
    return new Employee();
  });

  System.out.println("e1 >>> " + e1.toString());
}
```
<br>

`em.flush()` , `em.clear()` 
- JPA는 영속성 컨테이너(=메모리 내의 테이블 객체 매핑 저장소) 내에 엔티티가 없으면 SQL을 수행하게 된다. SQL이 실제로 어떻게 나오는지 확인하기 위해 강제로 `em.flush()` , `em.clear()` 를 사용했다.
<br>
<br>

위 구문의 sql 로 변환된 실행결과는 아래와 같다.
자세히 보면 left outer join 이 실행된 것을 볼 수 있다.

```sql
select
    employee0_.employee_id as employee1_1_0_,
    employee0_.dept_id as dept_id3_1_0_,
    employee0_.employee_name as employee2_1_0_,
    department1_.dept_id as dept_id1_0_1_,
    department1_.dept_name as dept_nam2_0_1_ 
from
    public.emp employee0_ 
left outer join
    public.dept department1_ 
        on employee0_.dept_id=department1_.dept_id 
where
    employee0_.employee_id=?
```
<br>
<br>


##### 전체 테스트 코드 
{: .fs-4 .fw-700 }

테스트 용도 데이터를 초기하는 부분부터 테스트의 모든 부분들을 코드로 남겼다.

```java
@Transactional
@SpringBootTest
class ManyToOneFetchOptionalTest {

    @Autowired
    EntityManager em;

    @Autowired
    private EmployeeRepository employeeRepository;

    @Autowired
    private DepartmentRepository departmentRepository;

    @BeforeEach
    public void init(){
        Department deptTrader = new Department("트레이더");
        Department deptSinger = new Department("가수");

        em.persist(deptTrader);
        em.persist(deptSinger);

        Employee e1 = new Employee("Beatles", deptSinger);
        Employee e2 = new Employee("Warren Buffet", deptTrader);
        Employee e3 = new Employee("Elvis", deptSinger);
        Employee e4 = new Employee("Fisher", deptTrader);
        Employee e5 = new Employee("Peter Lynch", deptTrader);
        em.persist(e1);
        em.persist(e2);
        em.persist(e3);
        em.persist(e4);
        em.persist(e5);
    }

    @Test
    @DisplayName("optional_false_테스트_단건조회")
    public void optional_false_테스트_단건조회(){
        em.flush();
        em.clear();
        Employee e1 = employeeRepository.findById(3L).orElseGet(()->{
            return new Employee();
        });

        System.out.println("e1 >>> " + e1.toString());
    }
}
```

#### optional = false 단건 조회 테스트
{: .fs-5 .fw-700 }

Employee.java
- Employee.java 의 Department 에 조인하는 @ManyToOne 의 optional 속성을 명시적으로 false 로 지정해줬다.
```java
@Data
@Entity
@Table(name = "EMP", schema = "public")
public class Employee {
  	// ...
    @ManyToOne(optional = false)
    @JoinColumn(name = "DEPT_ID")
    private Department dept;
		// ...
}
```
<br>

`em.flush()` , `em.clear()` 
- JPA는 영속성 컨테이너(=메모리 내의 테이블 객체 매핑 저장소) 내에 엔티티가 없으면 SQL을 수행하게 된다. SQL이 실제로 어떻게 나오는지 확인하기 위해 강제로 `em.flush()` , `em.clear()` 를 사용했다.
<br>
<br>

테스트 코드
- 테스트 코드다. 별 내용 없다. 단건 조회로 데이터를 조회할 뿐이다.
```java
@Test
@DisplayName("optional_false_테스트_단건조회")
public void optional_false_테스트_단건조회(){
	em.flush();
	em.clear();
	Employee e1 = employeeRepository.findById(3L).orElseGet(()->{
		return new Employee();
	});

	System.out.println("e1 >>> " + e1.toString());
}
```
<br>
<br>

SQL 생성결과
```sql
select
    employee0_.employee_id as employee1_1_0_,
    employee0_.dept_id as dept_id3_1_0_,
    employee0_.employee_name as employee2_1_0_,
    department1_.dept_id as dept_id1_0_1_,
    department1_.dept_name as dept_nam2_0_1_ 
from
    public.emp employee0_ 
inner join
    public.dept department1_ 
        on employee0_.dept_id=department1_.dept_id 
where
    employee0_.employee_id=?
```


##### 전체 테스트 코드
{: .fs-4 .fw-700 }

```java
@Transactional
@SpringBootTest
class ManyToOneFetchOptionalTest {

    @Autowired
    EntityManager em;

    @Autowired
    private EmployeeRepository employeeRepository;

    @Autowired
    private DepartmentRepository departmentRepository;

    @BeforeEach
    public void init(){
        Department deptTrader = new Department("트레이더");
        Department deptSinger = new Department("가수");

        em.persist(deptTrader);
        em.persist(deptSinger);

        Employee e1 = new Employee("Beatles", deptSinger);
        Employee e2 = new Employee("Warren Buffet", deptTrader);
        Employee e3 = new Employee("Elvis", deptSinger);
        Employee e4 = new Employee("Fisher", deptTrader);
        Employee e5 = new Employee("Peter Lynch", deptTrader);
        em.persist(e1);
        em.persist(e2);
        em.persist(e3);
        em.persist(e4);
        em.persist(e5);
    }

    @Test
    @DisplayName("optional_false_테스트_단건조회")
    public void optional_false_테스트_단건조회(){
        em.flush();
        em.clear();
        Employee e1 = employeeRepository.findById(3L).orElseGet(()->{
            return new Employee();
        });
    }
}
```

#### optional = true 리스트 조회 테스트 (정리 예정)
{: .fs-5 .fw-700 }


### fetch 테스트 (정리 예정)
{: .fs-6 .fw-700 }


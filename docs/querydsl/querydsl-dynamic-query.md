---
layout: default
title: 동적 쿼리 
nav_order: 3
has_children: false
parent: Querydsl
permalink: /docs/querydsl/querydsl-dynamic-query
---


# Querydsl 의 동적쿼리
{: .no_toc }
Querydsl 에서는 `BooleanBuilder`, `BooleanExpression` 을 활용해서 여러가지 조건식을 메서드에 조합해 하나로 묶을 수 있다. 여러가지 조건식을 조합하는 또 다른 Builder를 만들어두고 사용할수 있기에 재사용성이 높아진다.

Querydsl 에서 동적 쿼리를 사용할 때 아래와 같은 클래스 들을 사용한다.
- BooleanBuilder
- BooleanExpression
<br>
<br>


## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

<br>

### 참고자료
{: .fs-6 .fw-700 }
[인프런 - 실전! Querydsl](https://www.inflearn.com/course/querydsl-%EC%8B%A4%EC%A0%84)
<br>
<br>


### BooleanBuilder
{: .fs-6 .fw-700 }
- BooleanBuilder 에 조건식 BooleanExpression 을 넣어준다.
- 파라미터가 null 이 아닐 경우에 한해 `eq()` , `gt()` , `goe()` , `lt()` , `loe()` 와 같은 비교표현식이 가능하다.

e.g. 보험회사에서 원하는 가입최대연령/몸무게
```java
private BooleanBuilder signUpLimit(Double weight, Integer age){
  BooleanBuilder builder = new BooleanBuilder();

  if(weight != null){
    builder.and(member.weight.le(weight));
  }

  if(ageCond != null){
    builder.and(member.age.le(age));
  }

  return builder;
}
```
<br>
만약 보험회사가 70세 이상, 몸무게 60키로 이하까지를 가입제한을 걸었다가 약관을 바꾸어서 90세 이상, 몸무게 100카로 이하로 가입제한을 늘리려 할 경우 위의 함수의 인자값을 바꾸면 적용된다.
이렇게 되면 애플리케이션 곳곳에 퍼져있는 주요 조건식의 위치에 대해 모두 알고 대비해야 하는 강박은 조금 줄어들게 된다.<br>
<br>

메서드를 자세히 보면, 이름과 나이를 기반으로 회원을 검색하고 있다. 그리고 `BooleanBuilder` 객체 builder 내에는 `BooleanExpression` 들을 `and()` 메서드 내에 넣어서 하나씩 조합하고 있다.<br>
- member.weight.(weight)
- member.age.eq(age)
<br>
<br>

위의 두 메서드 모두 Expression 을 반환한다. BooleanBuilder 는 이렇게 Expression 들을 하나씩 and(), eq() 등을 통해 조합할 수 있다.<br>
<br>

이렇게 만들어진 builder 는 변수화 하는 것이 가능하다.
또는 builder 객체를 반환하는 하나의 공통 메서드를 만들어서 재사용하는 것 역시 가능해진다. <br>
<br>

### BooleanExpression
{: .fs-6 .fw-700 }

이미 위의 예제에서 많은 내용을 봤기에 설명할 내용이 그리 많지 않다. BooleanExpression 은 하나의 조건식이다. 이 조건식 하나 하나가 객체다.<br>
<br>

#### e.g. 1
{: .fs-5 .fw-700 }

```java
// null 처리가 간편해진다.
// stl 등에 대해 알고있지 않아도 되는점은 장점이다.
private BooleanExpression ageLe(Integer age){
  return age == null ? null : member.age.le(age);
}

// null 처리가 간편해진다.
private BooleanExpression weightLe(Double weight){
  return weight == null ? null : member.weight.le(weight);
}
```
<br>
위의 식들은 아래 구문처럼 where 절에 추가해서 사용할 수 있게 된다.

<br>
<br>

```java
private List<Member> searchData1(String username, Integer age){
  QMember member = QMember.member;
  return queryFactory
    .selectFrom(member)
    .where(ageLe(username), weightLe(age))
    .fetch();
}
```

#### e.g. 2
{: .fs-5 .fw-700 }

또는 아래처럼 개별 BooleanExpression 들을 합쳐서 하나의 함수로 공통화하는 경우 역시 있다.

```java
// null 처리가 간편해진다.
private BooleanExpression ageLe(Integer age){
  return age == null ? null : member.age.le(age);
}

// null 처리가 간편해진다.
private BooleanExpression weightLe(Double weight){
  return weight == null ? null : member.weight.le(weight);
}

// ageLe, weightLe 를 함께 조합한다.
private BooleanBuilder signUpLimit(Double weight, Integer age){
  return ageLe(age).and(weightLe(weight));
}

```
<br>
<br>

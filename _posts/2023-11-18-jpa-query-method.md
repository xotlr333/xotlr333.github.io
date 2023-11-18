---
layout: post
title: JPA 쿼리 메소드
date: 2023-11-18
categories: [Spring, JPA]
tags: [JPA]
---

## 쿼리 메소드란

리포지토리는 JpaRepository에서 제공하는 기본 메소드 이외 필요한 메소드를 만들때 사용하는 것이 쿼리 메소드이다. 간단한 쿼리문 대신 사용할 수 있다.

## 쿼리 메소드 생성

쿼리 메소드는 크게 동작을 결정하는 주제와 서술어로 구분한다.

`find…by`, `exists…by` 와 같은 키워드로 쿼리의 주제를 정하고 `by` 는 서술어의 시작을 나타내는 구분자 역할을 한다.

서술어 부분은 검색 및 정렬 조건을 지정하는 영역이다. 기본적으로 엔티티의 속성으로 정의할 수 있고, AND나 OR를 사용해 조건을 확장하는 것도 가능하다.

```java
// (리턴타입) + {주제 + 서술서(속성)} 구조의 메소드
List<Person> findByLastnameAndEmail(String lastName, String email);
```

## 쿼리 메소드의 주제 키워드

### 조회 기능

조회 기능을 하는 키워드는 다양하다

- find…by
- read…by
- get…by
- query…by
- search…by
- stream…by

```java
Optional<Product> findByNumber(Long number);
List<Product> findAllByName(String name);
Product queryByNumber(Long number);
```

### 데이터 존재 여부

특정 데이터가 존재하는지 확인하는 키워드이다.

리턴 타입으로는 boolean 타입을 사용한다.

- exists…by

```java
boolean existsByNumber(Long number);
```

### 레코드 개수

조회 쿼리를 수행한 후 쿼리 결과로 나온 레코드의 개수를 리턴한다.

- count…by

```java
long countByName(String name);
```

### 삭제 기능

삭제 쿼리를 수행한다.

리턴 타입이 없거나 삭제한 횟수를 리턴한다.

- delete…by
- remove…by

```java
void deleteByNumber(Long number)
long removeByName(String name);
```

### 조회 개수 제한

쿼리를 통해 조회도니 결괏값의 개수를 제한하는 키워드이다.

일반적으로 한번의 동작으로 여러 건을 조회할 때 사용된다.

- …First<number>…
- …Top<number>…

```java
List<Product> findFirst5ByName(String name);
List<Product> findTop10ByName(String name);
```

## 쿼리 메소드의 조건자 키워드

### Is

값의 일치를 조건으로 사용하는 조건자 키워드이다.

Equals와 동일한 기능을 수행한다.

```java
Product findByNumberIs(Long number);
Product findByNumberEquals(Long number);
```

### (Is)Not

값의 불일치를 조건으로 사용하는 조건자 키워드이다.

Is는 생략하고 Not만 사용할 수 있다.

```java
Product findByNumberIsNot(Long number);
Product findByNumberNot(Long number);
```

### (Is)Null, (Is)NotNull

값이 Null인지 검사하는 조건자 키워드이다.

```java
List<Product> findByUpdatedAtNull();
List<Product> findByUpdatedAtIsNull();
List<Product> findByUpdatedAtNotNull();
List<Product> findByUpdatedAtIsNotNull();
```

### (Is)True, (Is)False

boolean 타입으로 지정된 컬럼값을 확인하는 키워드이다.

```java
Product findByisActiveTrue();
Product findByisActiveIsTrue();
Product findByisActiveFalse();
Product findByisActiveIsFalse();
```

### And, Or

여러 조건을 묶을 때 사용한다.

```java
Product findByNumberAndName(Long number, String name);
Product findByNumberOrName(Long number, String name);
```

### (Is)GreaterThan, (Is)LessThan, (Is)Between

숫자나 datetime 컬럼을 대상으로 한 비교연산에 사용할 수 있는 키워드이다.

GreaterThan, LessThan 키워드는 비교 대상에 대한 초과/미만의 개념으로 비교 연산을 수행한다.

경계값을 포함하려면 Equals 키워드를 추가하면 된다.

```java
List<Product> findByPriceIsGreaterThan(Long price);
List<Product> findByPriceGreaterThan(Long price);
List<Product> findByPriceGreaterThanEqual(Long price);

List<Product> findByPriceIsLessThan(Long price);
List<Product> findByPriceLessThan(Long price);
List<Product> findByPriceLessThanEqual(Long price);

List<Product> findByPriceIsBetween(Long lowPrice, Long hightPrice);
List<Product> findByPriceBetween(Long lowPrice, Long hightPrice);
```

### (Is)StartingWith(==StartsWith), (Is)EndingWith(==EndsWith), (Is)Containing(==Contains), (Is)Like

컬럼값에서 일부 일치 여부를 확인하는 조건자 키워드이다.

SQL 쿼리문에서 값의 일부를 포함하는 값을 추출할 때 사용하는 ‘%’ 키워드와 동일한 역할을 하는 키워드이다.

Containing 키워드는 문자열의 양 끝, StartingWith 키워드는 문자열읠 앞, EndingWith 키워드는 문자열 끝에 ‘%’가 배치된다.

Like 키워드는 전달하는 값에 %를 명시적으로 입력해야한다.

```java
List<Product> findByNameLike(String name);
List<Product> findByNameIsLike(String name);

List<Product> findByNameContains(String name);
List<Product> findByNameContaining(String name);
List<Product> findByNameIsContaining(String name);

List<Product> findByNameStartsWith(String name);
List<Product> findByNameStartingWith(String name);
List<Product> findByNameIsStartingWith(String name);

List<Product> findByNameEndsWith(String name);
List<Product> findByNameEndingWith(String name);
List<Product> findByNameIsEndingWith(String name);
```

---
layout: post
title: 코틀린 프로퍼티
description:  
summary: 
tags:
minute: 1
---

코틀린 프로퍼티는 자바 필드와 비슷하게 보이지만 다른 개념이다.

```kotlin
//kotlin property
var name: String? = null

//Java field
String name = null
```

둘 다 데이터를 저장한다는 점은 같지만 프로퍼티에는 더 많은 기능이 있다. 기본적으로 프로퍼티는 사용자 정의 세터와 게터를 가질 수 있다.

```kotlin
var name: String? = null
	get() = field?.toUpperCase()
	set(value) {
		if(!value.isNullOrBlank())
			field = value
	}
```

이 코드에서 `field` 라느 식별자를 확인할 수 있다. 이는 프로퍼티의 데이터를 저장해두는 backing field에 대한 레퍼런스다. 이런 백킹 필드는 세터, 게터의 디폴트 구현에 사용되므로 따로 만들지 않아도 디폴트로 생성된다.

참고로 val을 사용해서 읽기 전용 프로퍼티를 만들 때는 `field`가 만들어지지 않는다.

```kotlin
val fullName: String
	get() = "$name $surname"
```

var을 사용해서 만든 읽고 쓸 수 있는 프로퍼티는 게터와 세터를 정의할 수 있다. 이런 프로퍼티를 파생 프로퍼티(derived property)라고 부르며 자주 사용한다.

이처럼 코틀리느이 모든 프로퍼티는 디폴트로 캡슐화되어 있다. 예를 들어 자바 표준 라이브러리 `Date` 를 활용해 객체에 날짜를 저장해서 많이 활용한 상황을 가정해본다. 프로젝트를 진행하는 중에 직렬화 문제 등으로 객체를 더 이상 이런 타입으로 저장할 수 없게 되었는데, 이미 프로젝트 전체에서 이 프로퍼티를 많이 참조하고 있다면 어떻게 해야 할까? 코틀린은 데이터를 millis라는 별도의 프로퍼티로 옮기고, 이를 활용해서 date 프로퍼티에 데이터를 저장하지 않고 wrap/unwrap하도록 코드를 변경하기만 하면 된다.

```kotlin
var data: Date
	get() = Date(millis)
	set(value) { millis = value.time }
```

프로퍼티는 필드가 필요 없다. 오히려 프로퍼티는 개념적으로 접근자(val의 경우 게터, var의 경우 게터와 세터)를 나타낸다. 따라서 코틀린은 인터페이스에도 프로퍼티를 정의할 수 있는 것이다.

```kotlin
interface Pesrson {
	val name: String
}
```

이렇게 코드를 작성하면 이는 게터를 가질 거라는 것을 나타낸다. 따라서 다음과 같이 오버라이드 할 수 있다.

```kotlin
open class Supercomputer {
	open val thenAnswer: Long = 42
}

open class AppleComputer: Supercomputer() {
	override val thenAnswer: Long = 1_800_275_2273
}
```

프로퍼티는 본질적으로 함수다. 따라서 확장 프로퍼티를 만들 수도 있다.

```kotlin
val Context.preferences: SharedPreferences
	get() = PreferencesManager.getDefaultPreferences(this)
```

프로퍼티는 필드가 아니라 접근자를 나타낸다. 이처럼 프로퍼티를 함수 대신 사용할 수 있지만 그렇다고 완전히 대체해서 사용하는 것은 좋지 않다. 다음처럼 알고리즘의 동작을 나타내는 것은 좋지 않다.

```kotlin
var Tree<Int>.sum: Int
	get() = when(this) {
						is Leaf = value
						is Node = left.sum + right.sum
				}
```

여기에서 sum 프로퍼티는 모든 요소를 반복처리하므로, 알고리즘 동작을 나타낸다고 할 수 있다. 이런 프로퍼티는 여러 가지 오해를 불러일으킬 수 있다. 큰 컬렉션의 경우 답을 찾을 때 많은 계산량이 필요하다. 하지만 관습적으로 이런 게터에 그런 계산량이 필요하다고 예상하지 않는다. 따라서 이런 처리는 프로퍼티가 아니라 함수로 구현해야 한다.

원칙적으로 프로퍼티는 상태를 나타내거나 설정하기 위한 목적으로만 사용하는 게 좋다. 다른 로직을 포함하지 말아야 한다. 어떤 것을 프로퍼티로 해야 하는지 판단할 수 있는 간단한 질문이 있다. ‘이 프로퍼티를 함수로 정의할 경우 접두사로 get 또는 set을 붙일 것인가?’ 만약 아니라면 함수로 만들어야 한다.

프로퍼티 대신 함수를 사용하는 것이 좋은 경우를 구체적으로 보면 이렇다.

- 연산 비용이 높거나 복잡도가 큰 경우
- 비즈니스 로직을 포함하는 경우
- 결정적이지 않은 경우 → 같은 동작에서 다른 값이 나올 수 있는 경우
- 변환의 경우. `Int.toDouble()`과 같은 변환 함수로 이루어진다. 따라서 이런 변환을 프로퍼티로 만들면 오해할 수 있다.
- 게터에서 프로퍼티 상태 변경이 일어나야 하는 경우

반대로 상태를 추출/설정할 때는 프로퍼티를 사용해야 한다. 특별한 이유가 없다면 함수를 사용하면 안 된다.

많은 경우 경험적으로 프로퍼티는 상태 집합을, 함수는 행동을 나타낸다고 생각한다.
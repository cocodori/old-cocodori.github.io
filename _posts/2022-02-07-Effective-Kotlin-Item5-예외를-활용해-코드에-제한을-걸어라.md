---
layout: post
title: Effective Kotlin Item5 예외를 활용해 코드에 제한을 걸어라
description: 확실하게 어떤 형태로 동작해야 하는 코드가 있다면 예외를 활용해 제한을 거는 게 좋다
summary: 확실하게 어떤 형태로 동작해야 하는 코드가 있다면 예외를 활용해 제한을 거는 게 좋다
tags: 이펙티브-코틀린
minute: 5
---

### 요약

확실하게 어떤 형태로 동작해야 하는 코드가 있다면, 예외를 활용해 제한을 거는 게 좋다.
코틀린은 다음과 같은 방법을 사용할 수 있음

- require
- check
- assert
- return or throw와 함께 사용하는 Elivs operator

**예시**

```kotlin
//Stack<T>
fun pop(num: Int = 1): List<T> {
	require(num <= size) { “Cannot remove more elements than current size” }
	check(isOpen) {“cannot pop from closed stack”}
	val ret = collection.take(num)
	collection = collection.drop(num)
	assert(ret.size == num)
	return ret
}
```

이렇게 제한을 걸면 다양한 이점이 있음

- 문서를 읽지 않아도 문제를 확인할 수 있다.
- 문제가 있을 경우 함수가 예상하지 못한 동작을 하지 않고 throw한다. 이런 제한으로 인해서 문제를 놓치지 않을 수 있고 코드가 더 안정적으로 작동하게 된다.
- 코드가 어느 정도 자체적으로 검사된다. 이와 관련된 테스트를 줄일 수 있다.
- 스마트 캐스트 기능을 활용할 수 있다.

## Argument

함수를 정의할 때 타입 시스템을 활용해서 아규먼트에 제한을 거는 코드를 많이 사용한다.
**몇 가지 예**

- 숫자를 아규먼트로 받아서 팩토리얼을 계산한다면 숫자는 양의 정수여야 한다.
- 좌표를 아규먼트로 받아서 클러스터를 찾을 때는 비어 있지 않은 좌표 목록이 필요하다.
- 사용자로부터 이메일 주소를 입력받을 때는 값이 입력되어 있는지, 그리고 이메일 형식이 올바른지 확인해야 한다.
일반적으로 이런 제한을 걸 때 `require` 를 사용함. `require` 는 조건을 만족하지 않을 경우 `IllegalArgumentException` 발생 시킴.

```kotlin
fun factorial(n: Int): Long {
	require(n>=0)
	return if(n<=1) 1 else factorial(n-1) * n
}

fun findClusters(points: List<Point>): List<Cluster> {
	require(points.isNotEmpty())
}

fun sendEmail(user: User, message: String) {
	requireNotNull(user.email)
	require(isValidEmail(user.email))
}
```

이와 같은 형태의 입력 유효성 검사 코드는 함수의 가장 앞부분에 배치되므로 읽는 사람도 쉽게 확인할 수 있다.
`require` 함수는 조건을 만족하지 못하면 무조건 throw를 발생시키기 때문에 제한을 무시할 수 없다. 그리고 다음처럼 코드 블록을 사용해서 지연 메시지를 정의할 수 있다.

```kotlin
fun factorial(n: Int): Long {
	require(n>=0) {“cannot calculate factorial of ….“ }
	return if(n<=1) 1 else factorial(n-1) * n
}
```

## 상태

어떤 구체적인 조건을 만족할 때만 함수를 사용할 수 있게 해야 할 때가 있다.

- 어떤 객체가 미리 초기화되어 있어야만 처리를 하고 싶은 함수
- 사용자가 로그인 했을 때만 처리하게 하고 싶은 함수
- 객체를 사용할 수 있는 시점에 사용하고 싶은 함수

상태와 관련된 제한을 걸 때는 일반적으로 `check` 함수를 사용한다.

```kotlin
fun speak(text: String) {
	check(isInitialize)
	//…
}

fun getUserInfo(): UserInfo {
	checkNotNull(token)
	//…
}

fun next(): T {
	check(isOpen)
}
```

`check` 함수는 지정된 예측을 만족하지 못할 때 `IllegalStatementException` 을 throw 한다. 상태가 올바른지 확인할 때 사용한다. 지연 메시지는 `require` 과 마찬가지로 코드 블록을 사용해서 변경할 수 있다. 함수 전체에 대한 어떤 예측이 있을 때는 일반적으로 `require`블록을 배치하고 그 뒤에 `check` 를 배치한다.
이런 확인은 사용자가 규약을 어기고 사용하면 안 되는 곳에서 함수를 호출하고 있다고 의심될 때 사용한다.

스스로 구현한 내용을 확인할 때는 일반적으로 `assert` 라는 또 다른 함수를 사용한다.

## Assert

함수가 올바르게 구현되었다면 확실하게 참을 낼 수 있는 코드들이 있다. 예를 들어 어떤 함수가 10개의 요소를 리턴한다면, ‘함수가 10개의 요소를 리턴하는가?’라는 코드는 참이다. 그런데 함수가 올바르게 구현되어 있지 않을 수도 있다. 처음부터 구현을 잘못했을 수도 있고, 해당 코드를 이후에 다른 누가 변경해서 제대로 동작하지 않는 것일 수도 있다. 이런 구현 문제로 발생하는 추가적인 문제를 예방하려면 단위 테스트를 사용하는 게 좋다.

```kotlin
Class StackTest {
	@Test
	fun ‘Stack pops correct number of elements’() {
		val stack = Stack(20) { it }
		val ret = tack.pop(10)
		assertEquals(10, ret.size)
	}
}
```

이 코드는 스택이 10개의 요소를 pop하면 10개의 요소가 나온다는 보편적인 사실을 테스트한다.
하지만 하나의 경우만 테스트해서는 모든 상황에서 괜찮을지 알 수 없다. 따라서 모든 pop 호출 위치에서 제대로 동작하는지 확인해도 좋을 것이다.

```kotlin
fun pop(num: Int = 1): List<T> {
	//…
	assert(ret.size == num)
	return ret
}
```

이런 조건은 코틀린/JVM에서만 활성화되며, -ea JVM 옵션을 활성화해야 확인할 수 있다. 이런 코드도 코드가 예상대로 동작하는지 확인하므로 테스트라고 할 수 있다. 다만 프로덕션 환경에서는 오류가 발생하지 않는다. 테스트할 때만 활성화되므로 오류가 발생해도 사용자가 알아차릴 수 없다.
만약 이 코드가 심각한 결과를 초래할 가능성이 있다면 `assert` 대신 `check` 를 사용하는 게 좋다.

`assert` 장점

- Assert 계열 함수는 코드를 자체 점검하며 더 효율적으로 테스트할 수 있다.
- 특정 상황이 아닌 모든 상황에 대한 테스트를 할 수 있다.
- 실행 시점에 정확하게 어떻게 되는지 확인할 수 있다.
- 실제 코드가 더 빠른 시점에 실패하게 만든다. 따라서 예상하지 못한 동작이 언제 어디서 실행되었는지 쉽게 찾을 수 있다.

## 정리

- `require` : 아규먼트와 관련된 예측을 정의할 때 사용하는 범용적인 방법
- `check` : 상태와 관련된 예측을 정의할 때 사용하는 범용적인 방법
- `assert` : 테스트 모드에서 테스트를 할 때 사용하는 범용적인 방법
- `return`과 `throw` 와 함께 Elvis 연산자 사용하기
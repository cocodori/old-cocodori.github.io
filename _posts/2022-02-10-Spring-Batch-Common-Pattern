---
layout: post
title: Common Batch Pattern
description: 일부 배치 job은 스프링 배치가 제공하는 컴포넌트 만으로 구성할 수 있다.
summary: 
tags: Spring-Batch
minute: 3
---

# Common Batch Pattern

일부 배치 job은 스프링 배치가 제공하는 컴포넌트 만으로 구성할 수 있다. 예를 들어 `ItemReader` , `ItemWriter` 구현체만으로 다양한 시나리오를 구현할 수 있다. 하지만 커스텀 코드가 필요할 때가 훨씬 많다. 주로 `Tasklet` , `ItemReader` , `ItemWriter` 와 다양한 리스너 인터페이스를 구현하는 것부터 시작한다. 제일 간단한 배치는 스프링 배치의 `ItemReader` 를 사용하면 되지만 아이템을 가공하고 write할 때는 `ItemWriter` , `ItemProcessor` 를 커스텀 해야 할 때가 많다.

커스텀 비즈니스 로직에서 흔히 쓰이는 몇 가지 패턴이 있다. 예제는 주로 리스너 인터페이스를 사용한다. `ItemReader` , `ItemWriter` 는 필요하다면 리스너 인터페이스도 구현할 수 있다.

## Logging ITem Processing and Failures

step에서 에러가 발생하면 아이템별로 특정 채널에 로그를 남기거나 데이터베이스에 레코드를 넣는 식의 특별한 처리를 많이 한다. 청크 지향 `Step` 을 사용하면 `read` 메소드에서 생긴 에러는 `ItemReadListener` 로, `write` 메소드에서 생긴 에러는 `ItemWriteListener` 로 쉽게 구현할 수 있다. 아래 예제에 있는 리스너는 read와 write 실패를 모두 로깅한다.

```java
public class ItemFailureLoggerListener extends ItemListenerSupport {
	private static Log logger = LogFactory.getLog("item.error")

	public void onReadError(Exception ex) {
		logger.error("Encountered error on read", e);
	}

	public void onWriteError(Exception ex, List<? extends Object> items) {
		logger.error("Encountered error on write", ex);
	}
}
```

리스너를 구현했다면 아래 예제처럼 step에 등록해야 한다.

```java
@Bean
public Step simpleStep() {
	return this.stepBuilderFactory.get("simpleStep")
									...
									.listener(new ItemFailureLoggerListener())
									.build();
}
```

> 리스너가 `onError()` 메소드 안에서 처리하는 일은 트랜잭션 안에 있어야 롤백할 수 있다. `onError()` 안에서 데이터베이스 같은 트랜잭션 리소스를 사용한다면 메소드에 선언적인 트랜잭션을 추가해서 전파(propagation) 속성을 `REQUIRES_NEW` 로 설정해야 한다.
> 

## Stopping a Job Manually for Business Reasons

스프링 배치는 `JobLauncher` 인터페이스로 `stop()` 메소드를 제공하지만 애플리케이션 개발자가 사용하는 용도는 아니다. 가끔은 비즈니스 로직 안에서 job을 중단하는 게 더 편하거나 합리적이다.

가장 간단한 방법은 `RuntimeException` 을 던지는 것이다(무한으로 재시도하거나 스킵하지 않는 exception).

예를 들어 아래처럼 커스텀 exception을 사용할 수 있다:

```java
public class PoisonPillItemProcessor<T> implements ItemProcessor<T, T> {
	@Override
	public T process(T item) throws Exception {
		if (isPoisonPill(item))
			throw new PoisonPillException("Poison pill detected: " + item);

		return item;
	}
}
```

간단하게  step을 종료하는 또 다른 방법은 아래 예제처럼 `ItemReader` 에서 null을 리턴하는 것이다.

```java
public class EarlyCompletionItemReader implements ItemReader<T> {
	private ItemReader<T> delegate;

	public void setDelegate(ItemReader<T> delegate) { ... }

	public T read() throws Exception {
		T item = delegate.read();
		if (isEndItem(item))
			return null
		
		return item;
	}
}
```

이 예제는 사실 아이템이 null이면 배치가 완료된 것으로 판단하는 `CompletionPolicy` 디폴트 구현체가 있어야 동작한다. 더 복잡한 completion policy를 구현했다면 아래 예제처럼 `SimpleStepFactoryBean` 으로 `Step` 에 주입하면 된다.

```java
@Bean
public Step simpleStep() {
	return this.stepBuilderFatory.get("simpleStep")
					.<String, String>chunk(new SpecialCompletionPolicy())
					.reader(reader())
					.writer(writer())
					.build();
}
```

다른 방법은 아이템을 처리하는 동안 `Step` 구현체가 체크하는 `StepExecution` 플래그를 설정하는 것이다. 이 방법을 사용하려면 현재 `StepExecution` 에 접근하는 `StepListener` 를 구현하고 `Step` 에 등록해야 한다. 다음은 플래그를 설정하는 리스너 예시:

```java
public class CustomItemWriter extends ItemListenerSupport implements StepListener {
	private StepExecution stepExecution;

	public void beforeStep(StepExecution stepExecution) {
		this.stepExecution = stepExecution;
	}

	public void afterRead(Object item) {
		if (isPoisonPill(item))
			stepExecution.setTerminateOnly();
	}
}
```

이 플래그를 사용하면 디폴트 설정대로 step이 `JobInterruptedException` 을 던진다. 이 동작은 `StepInterruptionPolicy` 가 제어한다. 하지만 exception을 던지거나 던지지 않는 동작만 가능해서 항상 비정상적으로 job을 종료한다.

## Adding a Footer Record

Flat 파일에 write할 때는 모든 처리가 끝난 후에 파일 끝에 footer 레코드를 적는 경우가 종종 있다. 스프링 배치는 이를 위한 FlatFileFooterCallback 인터페이스를 지원한다. `FlatFileFooterCallback` 은 (`FlatFileHeaderCallback` 도 있음) `FlatFileItemWriter` 의 선택적인 프로퍼티이며, 아래 예제처럼 item writer에 추가할 수 있다:

```java
@Bean
public FlatFileItemWriter<String> itemWriter(Resource outputResource) {
	return new FlatFileItemWriterBuilder<String>()
						.name("itemWriter")
						.resource(outputResource)
						.lineAggregator(lineAggregator())
						.headerCallback(headerCallback())
						.footerCallback(footerCallback())
						.build()
}
```

footer callback 인터페이스는 꼬리말을 적을 때 호출하는 메소드 하나만 가지고 있다. 인터페이스 정으ㅣ느ㄴ 다음과 같다:

```java
public interface FlatFileFooterCallback {
	void writeFooter(Writer writer) throw IOException;
}
```


참고: https://godekdls.github.io/Spring%20Batch/commonbatchpatterns/
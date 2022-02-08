---
layout: post
title: SpringBatch JsonItemReader
description: 스프링 배치는 JSON 처리에 사용할 수 있는 ItemReader를 제공한다.
summary: 스프링 배치는 JSON 처리에 사용할 수 있는 ItemReader를 제공한다.
tags: Spring-Batch
minute: 3
---

스프링 배치는 JSON 처리에 사용할 수 있는 ItemReader를 제공한다.

`JsonItemReader`는 JSON 청크를 읽어서 객체로 파싱한다는 면에서 `StackEventItemReader` 동작 개념과 거의 동일하다. JSON 문서는 객체로 구성된 배열이 최상단에 하나만 존재하는 완전한 형태의 문서여야 한다. `JsonItemReader` 가 동작할 때 실제 파싱 작업은 `JsonObjectReader` 인터페이스의 구현체에게 위임된다.

인터페이스는 `StackEventItemReader` 에서 언마샬러가 XML을 객체로 파싱하는 것과 유사한 방법으로 실제로 JSON을 객체로 파싱하는 역할을 한다. 스프링 배치는 애플리케이션 개발에 즉시 사용할 수 있도록 `JsonObjectReader` 인터페이스 구현체 두 개를 제공한다. 하나는 Jackson을 파싱 엔진으로 사용하며 나머지 하나는 Gson을 사용한다. 예제에서는 Jackson 구현체를 사용한다.

**예시 데이터**

```json
[
  {
    "firstName":"Laura",
    "middleInitial":"0",
    "lastName":"Minella",
    "address":"2039 Wall Street",
    "city":"Omaha",
    "state":"IL",
    "zipCode":"35446",
    "transactions": [
      {
       "accountNumber": 829433,
       "transactionDate": "2010-10-14 05:48:59",
       "amount": 26.08
      }
    ]
  }
]
```

`JsonItemReader` 를 구성하려면 스프링 배치가 제공하는 builder를 사용한다. 빌더에는 세 가지 의존성이 필요하다.

- batch name
- `JsonObjectReader`
- 읽을 리소스

`JsonItemReader` 의 다른 구성 항목에는 입력이 반드시 존재해야 하는지 나타내는 플래그(strict.기본값 true), 상태를 저장해야 하는지를 나타내는 플래그(saveState) 현재 ItemCount가 있다.

**파일을 읽어들이는 `JsonItemReader`**

```kotlin
@Configuration
class TestReader {
    @Bean
    @StepScope
    fun customerFileReader(
        @Value("#{jobParameters['customerFile']}") inputFile: Resource
    ): JsonItemReader<Customer> {
        val objectMapper = ObjectMapper()
        objectMapper.dateFormat = SimpleDateFormat("yyyy-MM-dd HH:mm:ss")

        val jsonObjectReader: JacksonJsonObjectReader<Customer> =
            JacksonJsonObjectReader(Customer::class.java)
        jsonObjectReader.setMapper(objectMapper)

        return JsonItemReaderBuilder<Customer>()
            .name("customerFileReader")
            .jsonObjectReader(jsonObjectReader)
            .resource(inputFile)
            .build()
    }
}
```

먼저 `ObjectMapper` 인스턴스를 생성한다. `ObjectMapper` 클래스는 Jackson이 JSON을 읽고 쓰는 데 사용하는 주요 클래스다. 애플리케이션 개발할 때 대부분 `ObjectMapper` 생성하는 코드를 직접 작성할 필요가 없다. 그러나 예제에서는 입력 파일의 날짜 형식을 지정해야 한다. 즉, 사용하려는 ObjectMapper를 커스터마이징 해야 한다. 

1.  `ObjectMapper` 인스턴스를 생성한다.
    1. 입력 파일의 `transactionDate` 를 처리할 날짜 형식을 지정한다.
    2. 이어서 `JacksonJsonObjectReader` 를 생성한다.
        1. 이때 두 가지 의존성이 필요하다. 
            1. 반환할 클래스(`Customer`)
            2. 커스터마이징된 `ObjectMapper` 
                1. 마지막으로 `JsonItemReader` 인스턴스를 구성한다.
                2. 새로운 `JsonItemReader` 를 생성한 뒤 이름, `JsonObjectReader` 객체, 읽어들일 리소스를 구성했다. 그리고 build 메서드를 호출해 `JsonItemReader` 인스턴스를 생성한다.
                

참고: 스프링 배치 완벽가이드
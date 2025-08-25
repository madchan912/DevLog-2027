# [Spring Data JPA] 새로운 Entity인지 판단하는 방법

## ❓ Question

Spring Data JPA에서 새로운 Entity인지 판단하는 방법은 무엇일까요?

---

### 💡 Version 1: 핵심 요약 답변 (1분)

> `save()` 메소드가 내부적으로 `persist()`를 호출할지 `merge()`를 호출할지 결정하는 로직입니다. 판단 기준은 크게 두 가지입니다.
>
> 1.  **기본 전략 (@Id 필드 확인):** ID가 객체 타입이면 `null` 여부, primitive 타입이면 `0` 여부를 검사합니다. 이는 `@GeneratedValue` 전략과 잘 맞습니다.
>
> 2.  **ID 직접 할당 전략 (Persistable 구현):** 상품 코드나 UUID처럼 ID를 직접 할당할 때 발생하는 불필요한 `SELECT` 쿼리 문제를 해결하기 위해 `Persistable` 인터페이스를 구현합니다. `isNew()` 메소드를 오버라이드하여, 보통 `@CreatedDate` 필드의 `null` 여부로 새로운 엔티티인지 직접 판단하도록 구현합니다.

---

### 📚 Version 2: 심화 상세 답변 (Deep Dive)

> `save()` 메소드는 엔티티의 새로운 객체 여부에 따라 `persist()`(INSERT)와 `merge()`(UPDATE)를 선택하는데, 이 판단은 다음의 우선순위에 따라 이루어집니다.
>
> **1️⃣ 순위: `@Version` 필드 확인**
>
> -   가장 먼저 `@Version` 필드의 `null` 여부를 확인하여 새로운 엔티티인지 판단하고 로직을 즉시 종료합니다.
>
> **2️⃣ 순위: `Persistable` 인터페이스의 `isNew()` 구현**
>
> -   엔티티가 `Persistable`을 구현했다면, JPA 기본 로직 대신 개발자가 직접 오버라이드한 `isNew()` 메소드의 결과를 따릅니다.
> -   ID를 직접 할당할 때 발생하는 성능 문제를 해결하는 가장 권장되는 방법이며, 보통 `@CreatedDate` 필드가 `persist` 시점에 자동으로 채워지는 원리를 활용합니다.
>
> **3️⃣ 순위: `@Id` 필드 확인 (기본 Fallback 전략)**
>
> -   위 두 조건에 해당하지 않을 때, `@Id` 필드가 참조 타입이면 `null` 여부를, primitive 타입이면 `0` 여부를 기준으로 판단하는 기본 전략이 동작합니다.
>
> 이처럼 Spring Data JPA는 여러 계층의 판단 전략을 통해 엔티티의 상태를 명확히 구분하여 효율적인 영속성 관리를 수행합니다.
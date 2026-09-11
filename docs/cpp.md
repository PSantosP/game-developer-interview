# C++

[학습 안내](README.md) · [차시별 질문·실습](questions.md)

작성 기준과 출처 표시는 [학습 안내](README.md)를 따릅니다. 본문 예제는 개념 설명용 코드 조각입니다.

<a id="cpp-01"></a>
## 객체의 수명과 저장 공간

문헌 보충: 클래스 객체의 수명 종료는 엄밀히 소멸자 호출의 시작과 연결됩니다. 생성·소멸 중 멤버 사용에는 별도 규칙이 있으므로 “수명 전후에는 어떤 접근도 불가능”으로 과장하지 않습니다. [C++ 작업 초안 — lifetime](https://eel.is/c++draft/basic.life). 이 링크는 갱신되는 초안이며 최신 규칙 전체를 C++17에 그대로 적용하지 않습니다.

<!-- RECALL_CARD_START -->
**기억할 기준: 공간은 자리, 수명은 사용 가능한 기간**

**쉬운 예:** 지역 vector가 사라질 때 원소 저장 공간도 관리 객체의 규칙에 따라 정리됩니다.

**가리고 떠올리기:** 지역 객체 주소 반환과 vector 값 반환은 왜 다를까요?

<details>
<summary>면접에서 짧게 말하기</summary>

공간 확보와 객체의 초기화·파괴를 구분합니다. 반환값은 별도 결과 객체가 될 수 있지만 지역 객체 주소는 수명 종료 후 사용할 수 없습니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](cpp-examples.md#cpp-01)

> “객체의 수명과 메모리를 확보하는 것은 어떻게 다른가요?”

- 저장 공간은 객체가 놓일 자리이고, 수명은 그 자리에서 객체를 유효하게 사용할 수 있는 기간입니다.
- 일반적인 클래스 객체는 초기화가 완료되면 수명이 시작됩니다. 생성자가 필요한 타입이라면 공간만 확보했다고 사용할 수 있는 객체가 되지는 않습니다.
- 자동 저장 기간의 지역 객체는 보통 스택을 이용하지만, C++ 개념을 단순히 ‘지역 변수는 무조건 스택’으로 외우지는 않습니다.
- 지역 `vector` 객체의 수명과 그 객체가 관리하는 동적 원소 저장 공간을 구분해야 합니다.

추가 질문: 지역 변수의 주소를 반환하면 왜 위험한가요? `vector`를 함수에서 값으로 반환하는 것도 같은 문제인가요?

참고: [Microsoft — 소멸자와 객체 수명 종료](https://learn.microsoft.com/en-us/cpp/cpp/destructors-cpp?view=msvc-170), [vector](https://learn.microsoft.com/en-us/cpp/standard-library/vector-class?view=msvc-170).

<a id="cpp-02"></a>
## 포인터와 참조

<!-- RECALL_CARD_START -->
**기억할 기준: 참조는 별명, 소유권은 별도 계약**

**쉬운 예:** 타깃 포인터가 있다고 그 타깃을 내가 삭제해야 하는 것은 아닙니다.

**가리고 떠올리기:** 포인터가 null이 아니면 대상도 살아 있을까요?

<details>
<summary>면접에서 짧게 말하기</summary>

포인터·참조의 형태만으로 수명과 소유권을 보장하지 않습니다. 필수 여부·수정 여부·비용과 API 계약으로 인자를 선택합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](cpp-examples.md#cpp-02)

> “포인터와 참조는 어떻게 다르고, 함수 인자는 어떻게 고르나요?”

- 포인터는 주소를 값으로 가지며, `nullptr`로 대상이 없음을 표현하거나 다른 대상을 가리키도록 바꿀 수 있습니다.
- 참조는 초기화할 때 대상에 연결되며 이후 다른 대상에 다시 연결할 수 없습니다. 기존 객체를 참조하는 것만으로 그 수명이 늘어나지는 않습니다. 다만 지역 const 참조에 임시 객체를 직접 바인딩하는 등의 문맥에는 수명 연장 규칙이 있으므로 구분합니다.
- 작은 값은 값 전달, 복사 비용이 있는 읽기 전용 입력은 `const T&`, 변경할 필수 대상은 `T&`, 대상 없음이 의미 있는 경우는 `T*` 등을 검토합니다.
- 원시 포인터나 참조라는 표기만으로 소유권을 판단할 수는 없습니다. API의 수명 계약도 확인합니다.

추가 질문: `const T*`와 `T* const`는 어떻게 다른가요? `const T&`를 멤버에 저장해도 대상이 계속 살아 있나요?

참고: [C++ Core Guidelines — 함수 인자 전달](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rf-in).

<a id="cpp-03"></a>
## RAII

<!-- RECALL_CARD_START -->
**기억할 기준: 정리를 객체의 소멸에 연결**

**쉬운 예:** 잠금 관리 객체가 스코프를 벗어나면 잠금이 풀립니다.

**가리고 떠올리기:** 함수 중간에 return해도 자원이 정리되게 하려면 어떻게 할까요?

<details>
<summary>면접에서 짧게 말하기</summary>

RAII는 자원의 획득·해제를 객체 수명에 묶습니다. 정상적인 스코프 종료와 스택 해제 경로에서 정리 책임을 일관되게 만듭니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](cpp-examples.md#cpp-03)

> “RAII가 무엇이며 게임 개발에서 어디에 쓰이나요?”

- 자원의 획득과 해제를 관리 객체의 수명에 연결하는 방식입니다. 메모리뿐 아니라 파일, 잠금, 그래픽스 자원에도 적용합니다.
- 정상적으로 스코프를 벗어나거나 예외로 스택이 풀릴 때 소멸자가 정리하므로, 여러 반환 경로마다 해제 코드를 반복할 필요가 줄어듭니다.
- 프로그램의 강제 종료까지 소멸자 실행을 보장한다는 뜻은 아닙니다.

예: 파일을 관리하는 객체를 지역 변수로 두면 함수 중간에서 반환해도 그 객체가 파일을 닫습니다.

추가 질문: 잠금을 직접 `lock`/`unlock`하는 것보다 `lock_guard`를 쓰는 이유는 무엇인가요?

참고: [C++ Core Guidelines — RAII](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rr-raii).

<a id="cpp-04"></a>
## 스마트 포인터와 소유권

<!-- RECALL_CARD_START -->
**기억할 기준: 누가 마지막까지 살려 둘 것인가**

**쉬운 예:** 부모가 자식을 소유하고 자식은 부모를 관찰만 하는 관계를 생각합니다.

**가리고 떠올리기:** 서로 shared_ptr를 가지면 왜 마지막 소유자가 사라지지 않을까요?

<details>
<summary>면접에서 짧게 말하기</summary>

단독 소유는 unique_ptr, 공유 소유는 shared_ptr, 비소유 관찰은 weak_ptr를 검토합니다. 순환 참조와 대상 자체의 동시성은 별도로 해결합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](cpp-examples.md#cpp-04)

> “unique_ptr, shared_ptr, weak_ptr의 차이는 무엇인가요?”

- `unique_ptr`: 한 소유자가 객체를 관리합니다. 복사할 수 없고 이동으로 소유권을 넘깁니다.
- `shared_ptr`: 여러 소유자가 수명을 공유합니다. 마지막 강한 소유자가 사라지면 관리 대상이 파괴됩니다.
- `weak_ptr`: 공유 대상을 소유하지 않고 관찰합니다. `lock()`으로 사용 가능한 `shared_ptr`를 얻었는지 확인합니다.
- 서로를 `shared_ptr`로 소유하면 순환 때문에 해제되지 않을 수 있습니다. 관계 중 소유하지 않는 쪽을 약한 참조로 표현할 수 있습니다.
- `shared_ptr`는 대상 객체의 멤버 접근까지 스레드 안전하게 만들지 않습니다.

추가 질문: 부모와 자식이 서로를 알아야 한다면 양쪽 모두 소유권이 필요한가요?

참고: [C++ Core Guidelines — 스마트 포인터](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rr-summary-smartptrs).

<a id="cpp-05"></a>
## 복사와 이동

<!-- RECALL_CARD_START -->
**기억할 기준: move는 이동을 허용하는 표현**

**쉬운 예:** 가방을 복제하는 복사와 내부 자원을 넘겨받는 이동을 구분합니다.

**가리고 떠올리기:** std::move를 썼는데 복사 생성자가 불릴 수 있을까요?

<details>
<summary>면접에서 짧게 말하기</summary>

std::move 자체는 자원을 옮기지 않고 이동 연산이 선택될 기회를 줍니다. 타입의 오버로드와 const 여부에 따라 복사가 선택될 수도 있습니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](cpp-examples.md#cpp-05)

> “복사와 이동의 차이, std::move의 역할을 설명해 보세요.”

- 복사는 대상의 상태를 복제하고, 이동은 타입이 제공하는 이동 연산을 통해 자원을 넘겨받을 기회를 줍니다.
- `std::move` 자체는 데이터를 옮기는 함수가 아닙니다. 이동 연산이 선택될 수 있도록 표현식의 값 범주를 변환합니다.
- 실제로 이동할지는 타입과 오버로드에 달려 있습니다. 이동 연산이 없거나 `const` 조건 등이 맞지 않으면 복사가 일어날 수 있습니다.
- 이동 후 상태를 무조건 ‘비어 있음’으로 가정하면 안 됩니다. 해당 타입의 계약을 확인합니다.

<details>
<summary>예제 — 어느 줄에서 소유권이 이동하나요?</summary>

```cpp
#include <memory>
#include <utility>

auto first = std::make_unique<int>(42);
auto second = std::move(first);
// second가 소유합니다. unique_ptr의 이동 후 first는 비어 있습니다.
```

`std::move(first)`로 얻은 표현식을 사용해 `second`의 이동 생성자가 호출될 때 소유권이 넘어갑니다.

</details>

추가 질문: 이름이 있는 `T&&` 변수는 표현식으로 사용할 때도 항상 rvalue인가요?

참고: [Microsoft — rvalue 참조와 이동](https://learn.microsoft.com/en-us/cpp/cpp/rvalue-reference-declarator-amp-amp?view=msvc-170).

<a id="cpp-06"></a>
## Rule of Zero와 Rule of Five

<!-- RECALL_CARD_START -->
**기억할 기준: 직접 자원을 관리할수록 특별 멤버가 연결됨**

**쉬운 예:** vector와 unique_ptr로 멤버를 구성하면 직접 해제 코드를 줄일 수 있습니다.

**가리고 떠올리기:** 소멸자만 추가했는데 복사·이동 정책도 봐야 하는 이유는 무엇일까요?

<details>
<summary>면접에서 짧게 말하기</summary>

우선 Rule of Zero를 지향합니다. 직접 자원을 관리한다면 복사·이동·소멸의 관계를 함께 정의하거나 금지합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](cpp-examples.md#cpp-06)

> “자원을 소유하는 클래스에서 복사와 소멸을 함께 생각해야 하는 이유는 무엇인가요?”

- 원시 포인터로 자원을 직접 소유하면서 기본 복사를 사용하면 주소만 복사될 수 있습니다. 두 객체가 같은 자원을 해제하면 문제가 됩니다.
- 직접 관리해야 한다면 소멸자, 복사 생성자, 복사 대입, 이동 생성자, 이동 대입을 함께 검토합니다. 반드시 다섯 함수를 모두 직접 구현하라는 뜻은 아닙니다. 필요한 연산을 삭제할 수도 있습니다.
- 가능하면 `vector`, `string`, `unique_ptr`처럼 수명을 관리하는 멤버를 사용해 특별 멤버 함수의 직접 구현을 피합니다. 이것이 Rule of Zero의 방향입니다.

추가 질문: `unique_ptr` 멤버가 있는 클래스의 기본 복사는 가능한가요?

참고: [C++ Core Guidelines — 특별 멤버 함수](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rc-zero).

<a id="cpp-07"></a>
## 가상 소멸자

<!-- RECALL_CARD_START -->
**기억할 기준: 부모 포인터로 삭제할 때의 계약**

**쉬운 예:** 기반 포인터가 실제 파생 객체를 소유하는 상황을 생각합니다.

**가리고 떠올리기:** 기반 소멸자가 비가상일 때 delete가 안전할까요?

<details>
<summary>면접에서 짧게 말하기</summary>

기반 포인터를 통한 다형적 삭제를 허용하면 가상 소멸자가 필요합니다. 허용하지 않는 설계라면 그런 삭제 경로 자체를 막습니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](cpp-examples.md#cpp-07)

> “기반 클래스 포인터로 파생 객체를 삭제할 때 무엇을 주의해야 하나요?”

- 일반적인 다형적 삭제에서 기반 소멸자가 비가상이면 정의되지 않은 동작이 됩니다. 단순히 ‘파생 소멸자만 생략된다’고 설명하면 부족합니다.
- 기반 포인터로 삭제하도록 설계했다면 공개 가상 소멸자를 제공합니다.
- 기반 타입을 통해 삭제하지 못하게 설계할 때는 보호된 비가상 소멸자라는 선택도 있습니다.

<details>
<summary>예제 — 다형적으로 소유하는 인터페이스</summary>

```cpp
struct IAttackRule
{
    virtual ~IAttackRule() = default;
    virtual int Evaluate() const = 0;
};
```

일반 C++ 인터페이스의 개념 예제입니다. Unreal의 `UINTERFACE` 작성 예제는 아닙니다.

</details>

추가 질문: 가상 함수가 전혀 없는 클래스에도 항상 가상 소멸자가 필요한가요?

참고: [Microsoft — 가상 소멸자](https://learn.microsoft.com/en-us/cpp/cpp/destructors-cpp?view=msvc-170).

<a id="cpp-08"></a>
## vector의 크기와 용량

<!-- RECALL_CARD_START -->
**기억할 기준: size는 원소 수, capacity는 확보한 여유**

**쉬운 예:** reserve(100)는 좌석을 확보하는 것이며 원소 100개를 생성하지 않습니다.

**가리고 떠올리기:** reserve 후 아직 size가 0인데 v[0]에 써도 될까요?

<details>
<summary>면접에서 짧게 말하기</summary>

reserve는 용량, resize는 원소 수를 바꿉니다. 접근 가능한 원소 범위는 size로 판단합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](cpp-examples.md#cpp-08)

> “reserve와 resize는 어떻게 다른가요?”

- `size()`는 존재하는 원소 수, `capacity()`는 재할당 없이 담을 수 있는 원소 수입니다.
- `reserve(n)`은 필요한 용량을 확보하며 원소 수를 늘리지 않습니다.
- `resize(n)`은 원소 수를 바꿉니다. 커지면 원소를 추가하고 작아지면 뒤쪽 원소를 파괴합니다.
- `reserve(100)` 후 `size()`가 0이라면 `v[0]`에 쓰면 안 됩니다.

추가 질문: 매번 `push_back` 직전에 `reserve(size() + 1)`을 호출하면 어떤 비용이 생길 수 있나요?

참고: [Microsoft — vector](https://learn.microsoft.com/en-us/cpp/standard-library/vector-class?view=msvc-170).

<a id="cpp-09"></a>
## vector와 참조 무효화

구체 조건: 재할당 없는 push_back도 기존 end 반복자는 무효화합니다. erase는 삭제 위치 **및 그 이후**의 반복자·참조에 영향을 줍니다. [C++ 작업 초안 — vector modifiers](https://eel.is/c++draft/vector.modifiers).

<!-- RECALL_CARD_START -->
**기억할 기준: 재할당하면 예전 원소 주소가 남지 않음**

**쉬운 예:** vector 원소의 주소를 저장한 뒤 push_back으로 공간이 옮겨질 수 있습니다.

**가리고 떠올리기:** reserve를 했으면 모든 erase와 삽입에도 참조가 안전할까요?

<details>
<summary>면접에서 짧게 말하기</summary>

재할당은 기존 원소의 참조·포인터·반복자를 무효화합니다. 재할당 없는 수정도 연산별 무효화 규칙을 확인합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](cpp-examples.md#cpp-09)

> “원소를 추가한 뒤 기존 포인터가 위험해지는 이유는 무엇인가요?”

- 재할당이 발생하면 저장 공간이 바뀌어 기존 원소를 가리키던 포인터·참조·반복자가 무효화됩니다.
- 재할당이 없더라도 삽입·삭제 위치에 따라 일부 반복자와 참조가 무효화됩니다.
- 인덱스를 저장하면 주소 변경은 피할 수 있지만, 삭제나 정렬 이후에도 같은 개체를 뜻한다는 보장은 없습니다. 안정적인 개체 식별이 필요하면 ID와 조회 구조를 별도로 설계합니다.

추가 질문: `reserve`를 했다는 이유만으로 저장한 포인터가 게임 종료까지 안전하다고 할 수 있나요?

참고: [Microsoft — vector의 재할당과 무효화](https://learn.microsoft.com/en-us/cpp/standard-library/vector-class?view=msvc-170).

<a id="cpp-10"></a>
## 람다와 수명

<!-- RECALL_CARD_START -->
**기억할 기준: 콜백 실행 시점까지 무엇이 살아 있는가**

**쉬운 예:** 함수 종료 후 실행할 람다가 지역 변수를 참조 캡처하면 문제가 됩니다.

**가리고 떠올리기:** this 캡처가 현재 객체의 수명을 늘려 주나요?

<details>
<summary>면접에서 짧게 말하기</summary>

캡처 방식과 콜백 실행 시점에 필요한 대상 수명을 확인합니다. 값 캡처라도 내부 포인터 대상까지 소유하는 것은 아닙니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](cpp-examples.md#cpp-10)

> “참조 캡처를 비동기 작업에 사용할 때 무엇을 주의해야 하나요?”

- 작업이 실행될 때 참조 대상이 이미 파괴되었다면 댕글링 참조가 됩니다.
- 값 캡처는 값을 저장하지만, 원시 포인터를 값으로 복사해도 가리키는 객체의 수명은 늘어나지 않습니다. `[this]`도 객체 전체를 복사하지 않습니다.
- 필요한 데이터를 독립된 값으로 넘기거나, 소유권 공유·약한 참조·취소 및 작업 완료 대기 중 요구사항에 맞는 방법을 선택합니다.
- 수명이 보장되어도 여러 스레드의 동시 접근 문제는 별도로 해결해야 합니다.

추가 질문: 비동기 콜백이 실행되기 전에 화면이나 Actor가 사라질 수 있다면 어떻게 설계하겠습니까?

참고: [C++ Core Guidelines — 외부로 전달되는 람다의 참조 캡처](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rf-value-capture).

<a id="cpp-11"></a>
## const와 캐스팅

<!-- RECALL_CARD_START -->
**기억할 기준: 읽기 제한과 실제 타입 검증을 구분**

**쉬운 예:** const_cast로 실제 const 객체를 수정하려는 것은 안전한 우회가 아닙니다.

**가리고 떠올리기:** 캐스팅이 컴파일된다는 이유만으로 변환 결과 사용도 안전할까요?

<details>
<summary>면접에서 짧게 말하기</summary>

const 계약을 유지하고 변환의 전제 조건을 확인합니다. static_cast·dynamic_cast 등의 목적을 구분하며 const 제거로 잘못된 수정 경로를 만들지 않습니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](cpp-examples.md#cpp-11)

> “const가 붙으면 객체가 어떤 경로로도 바뀌지 않나요?”

- `const T*`는 그 포인터를 통한 대상 변경을 제한합니다. 원래 객체가 비상수라면 다른 비상수 접근 경로에서 변경될 수 있습니다.
- `T* const`는 포인터 자체의 재지정을 막습니다. 대상의 변경을 막는 표기는 아닙니다.
- `static_cast`는 다운캐스팅 시 실제 동적 타입을 검사하지 않습니다. `dynamic_cast`는 다형적 타입의 런타임 검사가 필요한 경우 사용합니다. UObject의 `Cast`는 엔진의 별도 체계입니다.
- `const_cast`로 표기를 벗겨도 원래 상수인 객체를 수정하면 정의되지 않은 동작입니다. `reinterpret_cast`도 아무 타입으로 역참조해도 된다는 허가가 아닙니다.

예측: `int n = 1; const int* p = &n; n = 2;` 이후 `*p`는 2입니다. `p`를 통한 변경만 제한했기 때문입니다.

추가 질문: 다운캐스팅 대신 공통 인터페이스를 사용할 수 있는 상황은 무엇인가요?

참고: [Microsoft — const](https://learn.microsoft.com/en-us/cpp/cpp/const-cpp?view=msvc-170), [Casting](https://learn.microsoft.com/en-us/cpp/cpp/casting?view=msvc-170).

<a id="cpp-12"></a>
## 가상 호출과 객체 배치

<!-- RECALL_CARD_START -->
**기억할 기준: 값으로 잘라 복사하면 파생 부분을 잃을 수 있음**

**쉬운 예:** 파생 객체를 기반 타입 값 컨테이너에 넣으면 슬라이싱이 생길 수 있습니다.

**가리고 떠올리기:** virtual을 붙이면 값 복사로 사라진 파생 상태도 돌아올까요?

<details>
<summary>면접에서 짧게 말하기</summary>

가상 호출은 객체의 실제 타입과 유효한 수명을 전제로 합니다. 다형적 객체를 기반 값으로 복사하는 슬라이싱과 배치 비용을 구분합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](cpp-examples.md#cpp-12)

> “virtual 호출은 일반 함수 호출과 무엇이 다른가요?”

- 가상 호출은 객체의 동적 타입에 맞는 재정의 함수를 선택합니다. 흔한 구현은 가상 함수 테이블과 이를 가리키는 포인터를 사용하지만 정확한 배치가 C++ 표준으로 고정되어 있지는 않습니다.
- 다중 상속, 정렬, 패딩, ABI에 따라 크기와 주소 관계가 달라집니다. ‘가상 함수가 있으면 항상 몇 바이트 증가한다’고 일반화하지 않습니다.
- 기반 객체에 값으로 복사하면 파생 부분이 잘리는 슬라이싱이 발생할 수 있습니다. 다형적으로 다루려면 참조·포인터와 수명 정책을 함께 설계합니다.
- 성능은 호출 수, 인라이닝 가능성, 메모리 접근 패턴 등을 측정합니다. 컴파일러가 구체 타입을 알아 가상 호출을 최적화할 수도 있습니다.

추가 질문: 포인터 배열의 순회에서 가상 호출과 캐시 미스 비용을 어떻게 나누겠습니까?

참고: [Microsoft — Virtual Functions](https://learn.microsoft.com/en-us/cpp/cpp/virtual-functions?view=msvc-170).

<a id="cpp-13"></a>
## 템플릿과 컴파일·링크

<!-- RECALL_CARD_START -->
**기억할 기준: 컴파일은 번역, 링크는 정의 연결**

**쉬운 예:** 선언만 보고 컴파일돼도 필요한 함수 정의가 없으면 링크에 실패할 수 있습니다.

**가리고 떠올리기:** 템플릿 정의를 cpp로 숨기면 어떤 인스턴스가 생성되지 않을까요?

<details>
<summary>면접에서 짧게 말하기</summary>

필요한 템플릿 정의를 인스턴스화 지점에 제공하거나 명시적 인스턴스화를 설계합니다. 문법 오류와 정의 누락을 구분합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](cpp-examples.md#cpp-13)

> “템플릿 구현은 왜 헤더에 두는 경우가 많나요?”

- 템플릿은 타입·값을 매개변수로 코드를 표현합니다. 특수화를 인스턴스화하려면 보통 정의가 보여야 하므로 헤더에 구현을 둡니다. 명시적 인스턴스화로 분리하는 방식도 있습니다.
- 컴파일러는 번역 단위를 처리하고 링커는 생성된 코드와 라이브러리 사이의 심벌을 연결합니다. 선언만 있고 정의가 없거나 라이브러리가 빠지면 링크 오류가 날 수 있습니다.
- 헤더 가드는 한 번역 단위에서 중복 포함을 막습니다. 여러 번역 단위에 비인라인 함수 정의를 중복해서 두는 문제까지 해결하지 않습니다.
- Unreal에서는 C++ 컴파일·링크와 UHT의 리플렉션 코드 생성 단계도 구분합니다.

추가 질문: ‘헤더를 포함했으니 링크도 된다’는 설명은 왜 부족한가요?

참고: [Microsoft — Templates](https://learn.microsoft.com/en-us/cpp/cpp/templates-cpp?view=msvc-170), [Translation units and linkage](https://learn.microsoft.com/en-us/cpp/cpp/program-and-linkage-cpp?view=msvc-170).

<a id="cpp-14"></a>
## 컨테이너 선택과 캐시

<!-- RECALL_CARD_START -->
**기억할 기준: 복잡도와 실제 접근 패턴을 같이 보기**

**쉬운 예:** 적을 매 프레임 순회한다면 연속 저장의 캐시 이점이 중요할 수 있습니다.

**가리고 떠올리기:** 평균 O(1) 탐색 컨테이너가 작은 배열의 순회보다 항상 빠를까요?

<details>
<summary>면접에서 짧게 말하기</summary>

데이터 크기·순회·삽입·삭제·주소 안정성과 메모리 배치를 함께 고려합니다. 후보를 같은 작업으로 측정해 선택합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](cpp-examples.md#cpp-14)

> “해시 테이블은 항상 배열보다 빠른가요?”

- `vector`는 연속 저장이라 순회와 인덱스 접근에 적합합니다. 중간 삽입·삭제에는 뒤 원소 이동 비용이 생길 수 있습니다.
- `unordered_map`은 평균적으로 빠른 키 조회를 제공하지만 해시 계산, 충돌, 할당 비용이 있습니다. 최악 복잡도와 평균 복잡도를 구분합니다.
- `map`은 정렬된 키 순서와 로그 시간 연산을 제공합니다. `list`의 상수 시간 삽입은 삽입 위치를 이미 알고 있는 등의 조건을 따집니다.
- 빅오는 입력 크기 증가 경향을 설명합니다. 작은 데이터와 순회 위주 작업에서는 캐시 지역성 등으로 실제 결과가 달라집니다.
- 비교 시 같은 데이터와 연산, 최적화 빌드, 반복 횟수, 결과 소비, 워밍업을 기록합니다. 시간 측정 루프 안에 로그를 넣지 않습니다.

추가 질문: 적 20마리와 아이템 ID 10만 개 조회에 같은 구조를 고르겠습니까?

참고: [Microsoft — Containers](https://learn.microsoft.com/en-us/cpp/standard-library/stl-containers?view=msvc-170), [unordered_map](https://learn.microsoft.com/en-us/cpp/standard-library/unordered-map-class?view=msvc-170).

<a id="cpp-15"></a>
## 동시성과 데이터 경쟁

<!-- RECALL_CARD_START -->
**기억할 기준: 같은 데이터에 겹치는 접근의 규칙**

**쉬운 예:** 두 스레드가 같은 점수를 동기화 없이 증가시키면 데이터 경쟁이 생깁니다.

**가리고 떠올리기:** 카운터를 atomic으로 만들면 주변 여러 필드의 불변식도 보호될까요?

<details>
<summary>면접에서 짧게 말하기</summary>

충돌하는 접근의 동기화와 소유권을 설계합니다. atomic은 여러 변수의 복합 작업 전체를 자동으로 보호하지 않으며 잠금 범위와 실행 순서도 봅니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](cpp-examples.md#cpp-15)

> “atomic으로 바꾸면 멀티스레드 로직이 안전해지나요?”

- 동기화되지 않은 충돌 접근 중 쓰기가 있고 비원자적 데이터가 관여하는 데이터 경쟁은 C++에서 정의되지 않은 동작입니다.
- `atomic`은 해당 원자 연산을 보장합니다. 여러 연산으로 구성된 ‘재고가 있을 때만 감소’ 같은 복합 규칙을 저절로 보장하지는 않습니다.
- `mutex`와 RAII 잠금으로 임계 구역을 보호할 수 있습니다. 일관된 잠금 순서와 짧은 잠금 범위를 검토합니다.
- 수명 보장과 동시 접근 보장은 별개입니다. 공유 포인터가 살아 있어도 대상의 동시 변경이 안전한 것은 아닙니다.
- 작업 스레드에서 계산한다면 입력 복사, 결과 전달, 취소·종료 순서를 명확히 합니다.

추가 질문: 작업 스레드가 게임 스레드를 기다리고 게임 스레드도 그 작업을 기다리면 어떻게 되나요?

참고: [C++ Core Guidelines — Concurrency](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#s-concurrency).

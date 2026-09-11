# C++

[학습 안내](README.md) · [차시별 질문·실습](questions.md)

작성 기준과 출처 표시는 [학습 안내](README.md)를 따릅니다. 본문 예제는 개념 설명용 코드 조각입니다.

<a id="cpp-01"></a>
## 객체의 수명과 저장 공간

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

[예제와 해설로 이해하기](cpp-examples.md#cpp-06)

> “자원을 소유하는 클래스에서 복사와 소멸을 함께 생각해야 하는 이유는 무엇인가요?”

- 원시 포인터로 자원을 직접 소유하면서 기본 복사를 사용하면 주소만 복사될 수 있습니다. 두 객체가 같은 자원을 해제하면 문제가 됩니다.
- 직접 관리해야 한다면 소멸자, 복사 생성자, 복사 대입, 이동 생성자, 이동 대입을 함께 검토합니다. 반드시 다섯 함수를 모두 직접 구현하라는 뜻은 아닙니다. 필요한 연산을 삭제할 수도 있습니다.
- 가능하면 `vector`, `string`, `unique_ptr`처럼 수명을 관리하는 멤버를 사용해 특별 멤버 함수의 직접 구현을 피합니다. 이것이 Rule of Zero의 방향입니다.

추가 질문: `unique_ptr` 멤버가 있는 클래스의 기본 복사는 가능한가요?

참고: [C++ Core Guidelines — 특별 멤버 함수](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rc-zero).

<a id="cpp-07"></a>
## 가상 소멸자

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

[예제와 해설로 이해하기](cpp-examples.md#cpp-09)

> “원소를 추가한 뒤 기존 포인터가 위험해지는 이유는 무엇인가요?”

- 재할당이 발생하면 저장 공간이 바뀌어 기존 원소를 가리키던 포인터·참조·반복자가 무효화됩니다.
- 재할당이 없더라도 삽입·삭제 위치에 따라 일부 반복자와 참조가 무효화됩니다.
- 인덱스를 저장하면 주소 변경은 피할 수 있지만, 삭제나 정렬 이후에도 같은 개체를 뜻한다는 보장은 없습니다. 안정적인 개체 식별이 필요하면 ID와 조회 구조를 별도로 설계합니다.

추가 질문: `reserve`를 했다는 이유만으로 저장한 포인터가 게임 종료까지 안전하다고 할 수 있나요?

참고: [Microsoft — vector의 재할당과 무효화](https://learn.microsoft.com/en-us/cpp/standard-library/vector-class?view=msvc-170).

<a id="cpp-10"></a>
## 람다와 수명

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

[예제와 해설로 이해하기](cpp-examples.md#cpp-15)

> “atomic으로 바꾸면 멀티스레드 로직이 안전해지나요?”

- 동기화되지 않은 충돌 접근 중 쓰기가 있고 비원자적 데이터가 관여하는 데이터 경쟁은 C++에서 정의되지 않은 동작입니다.
- `atomic`은 해당 원자 연산을 보장합니다. 여러 연산으로 구성된 ‘재고가 있을 때만 감소’ 같은 복합 규칙을 저절로 보장하지는 않습니다.
- `mutex`와 RAII 잠금으로 임계 구역을 보호할 수 있습니다. 일관된 잠금 순서와 짧은 잠금 범위를 검토합니다.
- 수명 보장과 동시 접근 보장은 별개입니다. 공유 포인터가 살아 있어도 대상의 동시 변경이 안전한 것은 아닙니다.
- 작업 스레드에서 계산한다면 입력 복사, 결과 전달, 취소·종료 순서를 명확히 합니다.

추가 질문: 작업 스레드가 게임 스레드를 기다리고 게임 스레드도 그 작업을 기다리면 어떻게 되나요?

참고: [C++ Core Guidelines — Concurrency](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#s-concurrency).

# C++ — 예측하고 실행하며 이해하기

[학습 목차](README.md) · [개념 요약](cpp.md) · [차시별 과제](questions.md)

각 예제는 헤더와 main을 포함한 독립 프로그램입니다. 한 번에 하나만 main.cpp에 넣고 C++17 이상으로 빌드하세요. 먼저 출력 순서를 예상하고 해설을 펼칩니다. 검증 환경과 결과는 [검증 기록](verification.md)에 기록합니다. 휴대폰에서는 상황 → 코드 → 출력 → 이유 → 변형 순서로 읽습니다.

<a id="cpp-01"></a>
## 1. 저장 공간과 객체의 수명

<!-- EXPLANATION_PASS -->

**객체를 쓸 수 있는 기간부터 따지는 이유**

장비 목록을 지역 vector에 담으면 두 층이 생깁니다. vector 객체는 원소 저장 공간을 관리하고, 그 공간에는 Item 객체들이 들어갑니다. 공간 확보, Item 생성, Item 소멸, 공간 반환을 같은 사건으로 보면 reserve와 clear의 동작을 설명하기 어렵습니다.

예제에서 reserve(1) 다음에는 자리는 있지만 Item은 없습니다. emplace_back(7)이 Item을 초기화하고 size를 1로 만듭니다. clear를 넣으면 Item은 소멸하지만 vector는 살아 있고 capacity도 유지됩니다. 이후 스코프가 끝나면 vector가 남은 저장 공간을 반환합니다.

지역 객체의 주소를 반환하는 함수는 끝난 객체를 가리키게 되지만, vector를 값으로 반환하는 함수는 호출자에게 결과 객체를 제공합니다. 주소를 복사했는지, 결과 객체를 구성했는지가 판단 기준입니다.

장비 목록을 함수 안에서 만들었다고 생각해 봅시다. vector라는 관리 객체와 실제 장비가 들어 있는 공간은 구분됩니다. 목록을 값으로 반환하는 것과 지역 객체의 주소를 반환하는 것도 다릅니다.

~~~cpp
#include <iostream>
#include <vector>
struct Item {
    int id;
    explicit Item(int n) : id(n) { std::cout << "create " << id << '\n'; }
    ~Item() { std::cout << "destroy " << id << '\n'; }
};
int main() {
    std::cout << "before\n";
    {
        std::vector<Item> items;
        items.reserve(1);
        items.emplace_back(7);
        std::cout << "inside " << items.size() << '\n';
    }
    std::cout << "after\n";
}
~~~

<details>
<summary>예상 출력과 해설</summary>

~~~text
before
create 7
inside 1
destroy 7
after
~~~

reserve는 자리를 확보할 뿐 Item을 만들지 않습니다. emplace_back에서 Item이 만들어집니다. 안쪽 스코프가 끝나면 vector가 원소를 파괴하고 저장 공간을 해제합니다. 하나만 넣어 재할당 중 복사·소멸 로그가 섞이지 않도록 했습니다.

</details>

직접 변형: inside 출력 직후 items.clear()를 넣으세요. Item의 소멸 시점과 vector 자체의 수명 종료를 구분할 수 있나요?

추가 질문 해설: vector를 값으로 반환하면 반환된 객체가 원소를 소유할 수 있습니다. 지역 변수의 주소만 반환하면 대상의 수명이 끝난 뒤 주소만 남습니다. 반환 시 복사 생략·이동 여부는 표현식과 타입에 따라 판단합니다.

<a id="cpp-02"></a>
## 2. 참조와 임시 객체 수명 연장

<!-- EXPLANATION_PASS -->

**인자 표기는 호출자와의 약속이다**

체력 숫자 하나를 읽는 함수라면 값을 복사해도 부담이 작습니다. 큰 인벤토리를 잠깐 읽는 함수는 const 참조로 불필요한 복사를 피할 수 있습니다. 반드시 존재하는 인벤토리를 수정하면 참조, 장착 무기가 없을 수도 있으면 포인터로 그 조건을 드러낼 수 있습니다.

참조에 대입하는 것은 연결 대상을 바꾸는 일이 아닙니다. int a=1, b=2; int& r=a; 이후 r=b는 a에 2를 넣으며 r은 계속 a를 가리킵니다. 반면 포인터는 다른 객체의 주소로 재지정할 수 있습니다.

함수가 참조를 보관해 나중에 쓴다면 인자 선택만으로 문제가 끝나지 않습니다. 호출자가 먼저 사라질 수 있는지 따져야 합니다. 임시 객체의 지역 참조 수명 연장은 특정 초기화 문맥의 규칙이며, 그 참조를 다른 곳에 전달한다고 계속 연장되지는 않습니다.

참조는 별명입니다. 기존 객체에 별명을 붙였다고 대상이 영구히 살아 있지는 않습니다. 다만 임시 객체를 참조로 직접 초기화하는 특정 문맥에는 별도의 수명 연장 규칙이 있습니다.

~~~cpp
#include <iostream>
struct Token {
    Token() { std::cout << "create\n"; }
    ~Token() { std::cout << "destroy\n"; }
};
int main() {
    {
        const Token& token = Token{};
        (void)token;
        std::cout << "still alive\n";
    }
    std::cout << "finished\n";
}
~~~

<details>
<summary>예상 출력과 해설</summary>

~~~text
create
still alive
destroy
finished
~~~

이 선언의 임시 Token은 세미콜론에서 사라지지 않고 지역 참조의 수명까지 살아 있습니다. 이미 수명이 끝난 지역 객체의 참조를 반환하면 이 규칙으로 구제되지 않습니다. 참조를 다른 참조에 연결해도 수명이 계속 연장되지는 않습니다.

</details>

직접 변형: 참조 대신 일반 지역 Token을 만들고 출력이 같아도 생성 방식은 어떻게 다른지 설명하세요.

추가 질문 해설: 함수 인자의 const 참조를 멤버에 저장해도 호출자가 준 객체의 수명이 그 멤버에 맞춰 연장되지는 않습니다. 임시 객체 직접 바인딩의 예외와 기존 객체 참조를 구분합니다. [공식 설명](https://learn.microsoft.com/en-us/cpp/cpp/temporary-objects?view=msvc-170).

<a id="cpp-03"></a>
## 3. 중간 반환에서도 정리되는 RAII

<!-- EXPLANATION_PASS -->

**여러 종료 경로를 하나의 소멸자로 모으기**

파일을 열고 검증 A, 검증 B를 거쳐 저장한다고 합시다. 각 실패 지점에서 직접 파일을 닫으면 새로운 return을 추가할 때 닫기 한 줄을 빠뜨리기 쉽습니다. RAII에서는 파일을 관리하는 객체를 만든 뒤, 그 객체의 소멸자가 닫기를 맡습니다.

실행은 자원 획득 → 관리 객체 구성 → 작업 → 스코프 종료 → 소멸자의 해제 순서입니다. 중간 return도 정상적인 스코프 종료이므로 같은 해제 경로를 이용합니다. 관리 객체 생성 자체가 실패했다면 완성된 객체의 소멸자를 기대할 수 없으므로 생성 과정의 부분 자원도 안전하게 관리해야 합니다.

lock_guard도 같은 원리입니다. 잠금을 얻은 범위 안에서 공유 데이터를 다루고 범위를 나가면 잠금을 풉니다. 잠금이 필요한 작업까지만 범위에 포함해야 다른 스레드를 불필요하게 오래 기다리게 하지 않습니다.

데이터 검증 실패로 함수 중간에서 반환할 때도 자원 정리는 필요합니다. 반환 경로마다 해제를 반복하는 대신 정리 책임을 지역 관리 객체에 맡깁니다.

~~~cpp
#include <iostream>
struct Scope {
    Scope() { std::cout << "acquire\n"; }
    ~Scope() { std::cout << "release\n"; }
};
void Load(bool valid) {
    Scope resource;
    if (!valid) { std::cout << "reject\n"; return; }
    std::cout << "use\n";
}
int main() { Load(false); Load(true); }
~~~

<details>
<summary>예상 출력과 해설</summary>

~~~text
acquire
reject
release
acquire
use
release
~~~

첫 호출도 return하면서 소멸자를 실행합니다. Scope는 수명 관찰용 모형이며 실제 파일·잠금은 fstream, lock_guard 등 자원에 맞는 타입을 사용합니다. 프로세스 강제 종료까지 소멸자 실행을 보장하지는 않습니다.

</details>

직접 변형: 실패 조건을 둘로 늘려도 release 호출을 직접 추가하지 않고 정리를 유지하세요.

추가 질문 해설: lock_guard는 정상적인 스코프 종료 때 잠금을 해제하므로 unlock 누락을 줄입니다. 다만 잘못된 잠금 순서의 교착 상태까지 해결하는 것은 아닙니다.

<a id="cpp-04"></a>
## 4. 소유와 관찰을 구분하기

<!-- EXPLANATION_PASS -->

**소유한다는 말의 실제 의미**

소유권은 대상을 알고 있다는 뜻이 아니라 언제 정리할지 책임진다는 뜻입니다. 인벤토리만 아이템을 소유하고 UI가 보여 주기만 한다면, UI까지 공동 소유자가 될 필요는 없습니다. 다만 관찰 방식은 실제 소유 구조에 맞아야 하며 weak_ptr는 shared_ptr가 관리하는 대상에 사용합니다.

예제의 shared_ptr를 복사하면 강한 소유자가 늘고, weak_ptr를 만들면 강한 소유자는 늘지 않습니다. lock이 성공하면 작업하는 동안 사용할 임시 강한 소유자를 얻습니다. 그 임시 소유자까지 사라져야 마지막 소유자 조건이 충족됩니다.

부모와 자식이 서로 강하게 소유하면 바깥 참조를 지워도 서로를 유지합니다. 자식에서 부모로 가는 관찰을 약하게 바꾸면 소유의 고리를 끊을 수 있습니다. 공유할 필요가 없다면 처음부터 unique_ptr로 정리 책임을 한 곳에 두는 편이 설명하기 쉽습니다.

부모와 자식이 서로를 알아야 해도 양쪽 모두 서로를 소유해야 하는 것은 아닙니다. 탐색 관계와 소유 관계를 구분해야 순환 소유를 피할 수 있습니다.

~~~cpp
#include <iostream>
#include <memory>
struct Node {
    ~Node() { std::cout << "destroy\n"; }
};
int main() {
    std::weak_ptr<Node> observer;
    {
        auto owner = std::make_shared<Node>();
        observer = owner;
        {
            auto second = observer.lock();
            std::cout << "owners " << owner.use_count() << '\n';
        }
        std::cout << "owners " << owner.use_count() << '\n';
    }
    std::cout << "expired " << observer.expired() << '\n';
}
~~~

<details>
<summary>예상 출력과 해설</summary>

~~~text
owners 2
owners 1
destroy
expired 1
~~~

observer는 강한 소유자 수를 늘리지 않습니다. lock이 성공해 second가 만들어진 동안은 소유자가 둘입니다. 대상 소멸과 제어 블록 해제는 다릅니다. weak_ptr가 남아 있으면 대상이 소멸해도 관련 제어 정보는 더 오래 유지될 수 있습니다.

</details>

직접 변형: 부모는 자식을 shared_ptr, 자식은 부모를 weak_ptr로 가리키게 하세요. 외부 소유자를 제거했을 때 두 소멸 로그를 확인하세요.

추가 질문 해설: 양방향 shared_ptr이면 외부 소유자가 없어져도 서로의 강한 참조가 남을 수 있습니다. use_count는 이 단일 스레드 예제의 관찰 도구이며 동시 접근 안전성 판정 수단이 아닙니다.

<a id="cpp-05"></a>
## 5. std::move와 실제 이동은 다른 단계

<!-- EXPLANATION_PASS -->

**복사 비용과 이동 생성자 선택을 연결하기**

큰 vector를 복사하면 별도 원소 저장 공간과 원소 복사가 필요합니다. 일반적인 allocator를 쓰는 vector의 이동 생성은 그 저장 공간을 새 vector가 관리하도록 넘길 수 있어 원소 전체를 복사하는 일을 줄입니다. 사용자 정의 타입의 이동은 작성된 구현에 달려 있으므로 모든 이동이 항상 싸다는 뜻은 아닙니다.

아래 Box 예제는 실제 버퍼를 옮기지 않고 어떤 생성자가 선택되는지만 출력합니다. b(a)는 원본을 유지하는 복사를, c(std::move(a))는 이동 생성자를 선택합니다. fixed는 const이므로 std::move를 써도 수정 가능한 Box&&에 연결되지 않아 복사가 선택됩니다.

자원을 가진 타입에서는 선택된 이동 생성자가 실제 소유 정보를 넘기는 일을 해야 합니다. std::move는 그 함수를 선택할 수 있는 표현식을 만들 뿐입니다. 이동 뒤 원본에 어떤 연산을 해도 되는지는 해당 타입의 계약을 따릅니다. 아래 추가 실험은 호출 로그와 별개로 vector의 자원 소유를 관찰합니다.

큰 버퍼를 넘기기 전에 어느 생성자가 선택되는지부터 관찰합니다. 아래 Box는 실제 자원이 없는 호출 관찰용 타입입니다.

~~~cpp
#include <iostream>
#include <utility>
struct Box {
    Box() = default;
    Box(const Box&) { std::cout << "copy\n"; }
    Box(Box&&) noexcept { std::cout << "move\n"; }
};
int main() {
    Box a;
    Box b(a);
    Box c(std::move(a));
    const Box fixed;
    Box d(std::move(fixed));
}
~~~

<details>
<summary>예상 출력과 해설</summary>

~~~text
copy
move
copy
~~~

a는 이름이 있는 lvalue라 복사 생성자가 선택됩니다. std::move(a)에는 이동 생성자가 맞습니다. 하지만 std::move는 fixed의 const를 제거하지 않습니다. 일반적인 Box&& 이동 생성자에 연결할 수 없어 const Box& 복사 생성자가 선택됩니다.

</details>

직접 변형: 이동 생성자를 제거한 경우와 삭제 선언한 경우를 각각 빌드하세요. 후보 자체가 없는 경우와 선택된 후보가 삭제된 경우는 다릅니다.

추가 질문 해설: 이름이 있는 T&& 변수도 그 이름을 표현식으로 쓰면 lvalue입니다. 전달 참조 문맥의 forward와 의도적인 move는 목적이 다릅니다. 이동 후 상태는 타입의 계약을 따라야 합니다.


**추가 실험 — 실제 원소 저장 공간의 소유권**

Box는 호출 선택만 보여 주므로 이번에는 원소가 있는 vector를 사용합니다. 다음은 별도의 독립 프로그램입니다.

```cpp
// experiment: move-buffer.cpp
#include <iostream>
#include <utility>
#include <vector>
int main() {
    std::vector<int> original(100000, 7);
    const int* oldStorage = original.data();
    std::vector<int> copied = original;
    std::vector<int> moved = std::move(original);
    copied[0] = 9;
    std::cout << std::boolalpha;
    std::cout << (copied.data() != oldStorage) << '\n';
    std::cout << (moved.data() == oldStorage) << '\n';
    std::cout << copied[0] << ' ' << moved[0] << '\n';
}
```

<details>
<summary>저장 공간과 결과 해설</summary>

```text
true
true
9 7
```

copied는 원소를 따로 소유하므로 값을 바꾸어도 moved의 원소는 바뀌지 않습니다. 여기서는 allocator 인자가 없는 vector 이동 생성자를 사용합니다. 이동 전 원소 주소는 이동 후 moved의 원소를 가리킵니다. original을 역참조하거나 original의 이동 후 size를 고정값으로 검사하지 않습니다.

원소 전체의 복사와 저장 공간 소유 이전을 구분하는 실험이며 속도 측정 결과는 아닙니다. 다른 allocator를 명시하는 이동 생성이나 이동 대입까지 같은 비용이라고 일반화하지 않습니다.

</details>

참고: [Microsoft — vector 생성자](https://learn.microsoft.com/en-us/cpp/standard-library/vector-class?view=msvc-170).

<a id="cpp-06"></a>
## 6. Rule of Zero가 실제로 줄이는 코드

<!-- EXPLANATION_PASS -->

**특별 멤버 함수를 함께 검토해야 하는 이유**

원시 포인터로 버퍼를 가진 클래스를 기본 복사하면 보통 주소만 복사됩니다. 두 객체가 같은 주소를 각각 delete하면 중복 해제 문제가 생깁니다. 소멸자를 작성했다면 복사 때 새 버퍼를 만들지, 복사를 금지할지, 이동 때 소유권을 넘길지도 함께 결정해야 합니다.

vector 멤버는 자신의 복사·이동·해제를 이미 구현합니다. 바깥 클래스가 특별 멤버 함수를 직접 쓰지 않아도 멤버의 규칙을 조합할 수 있습니다. unique_ptr 멤버라면 복사는 기본적으로 불가능하고 이동 가능한 설계가 됩니다. 이것이 Rule of Zero 예제가 확인하는 차이입니다.

직접 소멸자를 선언하면 암시적 이동 연산 생성에 영향을 주므로 로그만 넣었다가 이동이 없어지는 경우도 살펴야 합니다. Rule of Five는 다섯 함수를 무조건 작성하는 양식이 아니라 소유 정책과 다섯 연산이 모순되지 않는지 확인하는 기준입니다.

이미 자원 관리를 제공하는 컨테이너를 멤버로 두면 클래스가 복사·소멸을 직접 구현하지 않아도 됩니다.

~~~cpp
#include <iostream>
#include <memory>
#include <type_traits>
#include <vector>
struct Inventory { std::vector<int> ids{1, 2}; };
struct Exclusive { std::unique_ptr<int> id; };
int main() {
    Inventory a;
    Inventory b = a;
    b.ids[0] = 9;
    std::cout << a.ids[0] << ' ' << b.ids[0] << '\n';
    std::cout << std::is_copy_constructible_v<Exclusive> << ' '
              << std::is_move_constructible_v<Exclusive> << '\n';
}
~~~

<details>
<summary>예상 출력과 해설</summary>

~~~text
1 9
0 1
~~~

vector 멤버 복사는 원소를 복사해 a와 b의 값이 독립적입니다. Exclusive는 unique_ptr 멤버 때문에 기본 복사가 불가능합니다. 특별 멤버 함수를 직접 작성하지 않아도 멤버의 계약이 반영됩니다.

</details>

직접 변형: Exclusive에 사용자 선언 소멸자를 추가하고 이동 가능 여부를 다시 확인하세요. 자동 이동 생성 조건을 조사한 뒤 필요한 연산을 명시적으로 default하는 방식을 검토하세요.

추가 질문 해설: Rule of Five는 다섯 함수를 기계적으로 구현하라는 뜻이 아닙니다. 자원을 직접 관리한다면 각 연산을 구현·삭제·기본화할지 함께 결정해야 합니다.

<a id="cpp-07"></a>
## 7. 기반 타입으로 소유하는 객체의 소멸

<!-- EXPLANATION_PASS -->

**생성한 타입과 삭제하는 타입이 다를 때**

여러 공격 규칙을 기반 클래스 포인터 하나로 보관하면 호출자는 구체적인 규칙 타입을 몰라도 함수를 부를 수 있습니다. 그런데 마지막 정리도 기반 포인터로 한다면 실제 파생 객체의 정리 경로가 필요합니다.

예제는 가상 소멸자를 통해 파생 소멸자, 기반 소멸자 순으로 정리되는 모습을 보여 줍니다. 여기서 virtual을 지우고 실행 결과만 관찰해 안전성을 판단하면 안 됩니다. 일반적인 기반 포인터 delete에서 비가상 소멸자는 정의되지 않은 동작이므로 특정 출력은 보장이 아닙니다.

기반 타입을 외부에서 삭제하지 못하게 할 설계라면 protected 비가상 소멸자로 잘못된 사용을 컴파일 단계에서 막는 선택도 있습니다. 핵심은 모든 클래스에 virtual을 붙이는 것이 아니라 기반 타입을 통한 삭제가 API에 허용되어 있는지입니다.

~~~cpp
#include <iostream>
#include <memory>
struct Rule {
    virtual ~Rule() { std::cout << "base\n"; }
};
struct HeavyRule : Rule {
    ~HeavyRule() override { std::cout << "derived\n"; }
};
int main() { std::unique_ptr<Rule> rule = std::make_unique<HeavyRule>(); }
~~~

<details>
<summary>예상 출력과 해설</summary>

~~~text
derived
base
~~~

unique_ptr의 대상 타입은 Rule이지만 가상 소멸자를 통해 파생 소멸 절차가 수행됩니다. virtual을 제거한 뒤 base만 출력된다는 관찰로 동작을 규정하면 안 됩니다. 일반적인 기반 포인터 삭제에서 실제 타입이 다르고 소멸자가 비가상이면 정의되지 않은 동작입니다.

</details>

직접 변형: 파생 객체에 vector 멤버를 추가하고 멤버 정리가 어느 소멸 단계에 연결되는지 설명하세요.

추가 질문 해설: 모든 클래스에 가상 소멸자가 필요한 것은 아닙니다. 다형적 삭제를 허용하는지 결정하고, 허용하지 않는 기반 타입은 접근 제어로 삭제 경로를 막는 설계도 가능합니다.

<a id="cpp-08"></a>
## 8. 예약한 자리와 존재하는 원소

<!-- EXPLANATION_PASS -->

**빈 자리와 실제 원소를 구분하기**

아이템 100개를 받을 예정이라고 reserve(100)을 해도 아직 아이템은 0개입니다. reserve는 추가할 때 저장 공간을 자주 바꾸지 않도록 준비하는 작업이고, resize는 실제 목록의 길이를 바꾸는 작업입니다.

빈 vector<int>에 reserve(100)만 한 뒤 v[0]을 쓰면 존재하지 않는 원소에 접근합니다. push_back으로 넣거나 resize로 원소를 먼저 만들어야 합니다. vector<int>의 resize 증가분은 기본 allocator에서 0으로 초기화되며 사용자 타입에서는 그 타입의 생성 규칙을 따릅니다.

최종 개수를 대략 알 때 한 번 예약하는 것은 도움이 될 수 있습니다. 매번 size()+1만큼 예약하면 구현의 여유 용량 증가 전략을 방해해 재할당을 자주 유발할 수 있습니다. 예제는 capacity의 정확한 증가 배수를 외우는 대신 size와 원소 존재 여부를 관찰합니다.

~~~cpp
#include <iostream>
#include <vector>
int main() {
    std::vector<int> values;
    values.reserve(4);
    std::cout << values.size() << ' ' << (values.capacity() >= 4) << '\n';
    values.resize(2);
    std::cout << values.size() << ' ' << values[0] << '\n';
    values.push_back(9);
    std::cout << values.size() << ' ' << values.back() << '\n';
}
~~~

<details>
<summary>예상 출력과 해설</summary>

~~~text
0 1
2 0
3 9
~~~

reserve 직후 원소 수는 0입니다. 기본 allocator의 int 원소는 resize로 추가할 때 0으로 초기화됩니다. capacity는 최소 요구치 이상이라는 조건만 사용했으며 정확히 4라고 가정하지 않았습니다.

</details>

직접 변형: clear 이후 size와 capacity를 비교하세요. shrink_to_fit은 비구속 요청이라는 점도 문서에서 확인하세요.

추가 질문 해설: 매번 reserve(size+1)하면 컨테이너의 기하급수적 성장 전략을 방해해 재할당·이동을 반복하게 만들 수 있습니다. 예상량이 있으면 묶어서 예약하고 실제 성장 패턴을 관찰합니다.

<a id="cpp-09"></a>
## 9. 인덱스가 유효해도 같은 아이템은 아니다

<!-- EXPLANATION_PASS -->

**주소 안정성과 대상의 정체성은 다르다**

인벤토리의 두 번째 슬롯을 선택했다고 합시다. 재할당 후에는 이전 주소가 무효가 될 수 있고, 첫 번째 아이템 삭제 후에는 두 번째 인덱스가 다른 아이템을 뜻할 수 있습니다. 하나는 저장 위치 문제이고 다른 하나는 대상 식별 문제입니다.

예제의 삭제 전후 ID를 비교하면 인덱스가 범위 안이어도 선택 대상이 바뀔 수 있음을 볼 수 있습니다. ID로 다시 찾는 구조는 저장 위치와 대상 식별을 분리합니다. ID 역시 재사용 정책이 필요하므로 오래된 ID가 새 개체를 가리키지 않도록 세대 번호 등을 선택할 수 있습니다.

재할당이 없는 push_back은 기존 원소 참조를 유지하지만 기존 end 반복자는 바뀝니다. 중간 erase는 삭제 지점 및 그 이후 참조·반복자를 무효화합니다. 안전성을 판단할 때는 reserve 호출 유무보다 실제로 수행한 연산과 보관한 핸들의 종류를 봅니다.

~~~cpp
#include <algorithm>
#include <iostream>
#include <vector>
struct Item { int id; };
int main() {
    std::vector<Item> items{{10}, {20}};
    const int wanted = items[0].id;
    items.erase(items.begin());
    std::cout << "index0 " << items[0].id << '\n';
    auto found = std::find_if(items.begin(), items.end(),
        [wanted](const Item& item) { return item.id == wanted; });
    std::cout << "found " << (found != items.end()) << '\n';
}
~~~

<details>
<summary>예상 출력과 해설</summary>

~~~text
index0 20
found 0
~~~

원소를 삭제하면 뒤 원소가 이동합니다. 인덱스 0은 유효하지만 다른 아이템입니다. ID는 찾으려는 개체를 명시하며 조회 실패를 처리할 수 있게 합니다. ID 재사용 시스템에는 세대 번호 같은 정책이 추가로 필요할 수 있습니다.

</details>

직접 변형: 삭제 대신 역순 정렬을 하고 인덱스 조회와 ID 조회를 비교하세요.

추가 질문 해설: reserve는 일부 재할당을 피할 뿐, 삭제·삽입에 따른 무효화와 논리적 식별 문제를 모두 해결하지 않습니다. 이미 무효화된 포인터를 역참조하는 실습은 하지 않습니다.

<a id="cpp-10"></a>
## 10. 등록 당시 값과 실행 당시 값

<!-- EXPLANATION_PASS -->

**값을 복사해도 포인터의 대상까지 복사되지는 않는다**

콜백을 등록한 순간과 실제 호출 순간 사이에 화면이 닫힐 수 있습니다. 지역 숫자를 값으로 캡처하면 그 숫자의 복사본은 콜백 안에 남지만, 포인터를 값으로 캡처하면 주소만 남습니다. this 캡처 역시 객체를 살려 두는 소유권이 아닙니다.

아래 예제는 같은 스코프에서 값을 바꾼 후 호출해 값 캡처는 등록 당시 값, 참조 캡처는 현재 값을 읽는 모습을 비교합니다. 살아 있지 않은 대상을 읽는 코드를 실행하지 않고도 두 저장 방식의 차이를 볼 수 있습니다.

나중에 실행할 작업에는 필요한 숫자만 복사할지, 공유 소유권을 유지할지, 약한 참조가 만료되면 취소할지 정합니다. 작업 결과가 불필요해졌다면 대상이 살아 있어도 적용하지 않아야 합니다. 수명 검사와 요청 취소는 서로 다른 질문입니다.

~~~cpp
#include <iostream>
int main() {
    int score = 10;
    auto snapshot = [score] { return score; };
    auto live = [&score] { return score; };
    score = 20;
    std::cout << snapshot() << ' ' << live() << '\n';
}
~~~

<details>
<summary>예상 출력과 해설</summary>

~~~text
10 20
~~~

snapshot은 등록 당시 int 값을 저장했습니다. live는 살아 있는 score에 접근합니다. 두 호출 모두 score의 수명 안에 있어 안전합니다. 외부로 콜백을 반환하는 순간 대상 수명을 다시 검토해야 합니다.

</details>

직접 변형: shared_ptr를 값 캡처하고 외부 소유자를 reset한 뒤 실행하세요. 원시 포인터 값 캡처와 차이를 설명하세요.

추가 질문 해설: 비동기는 완료를 지금 기다리지 않는 구조, 병렬은 작업이 실제로 동시에 진행되는 성질입니다. UObject가 사라질 수 있는 콜백은 엔진의 약한 참조와 취소·결과 폐기 정책을 검토합니다.

<a id="cpp-11"></a>
## 11. const가 제한하는 대상

<!-- EXPLANATION_PASS -->

**const가 막는 접근 경로를 읽기**

체력을 읽기만 하는 함수에 const 포인터를 주면 그 함수를 통한 변경을 제한할 수 있습니다. 원래 객체를 다른 곳에서 변경하는 것까지 막는 전역 잠금은 아닙니다. 그래서 예제에서 n을 다른 경로로 바꾸면 p로 읽는 값도 바뀝니다.

const T*는 대상에 대한 접근을, T* const는 포인터 변수의 재지정을 제한합니다. 둘 다 붙이면 두 제약이 함께 적용됩니다. const_cast로 실제 const 객체를 수정하는 것은 허용되지 않으므로 강제로 컴파일된다는 사실을 안전성으로 판단하면 안 됩니다.

캐스팅은 타입 관계를 표현하거나 검사하는 별도 도구입니다. 기반 포인터가 정말 파생 타입인지 모르는 상황에서 static_cast로 단정하지 않습니다. 자주 다운캐스팅한다면 필요한 동작을 기반 인터페이스로 표현할 수 있는지 먼저 검토합니다.

~~~cpp
#include <iostream>
int main() {
    int a = 10;
    int b = 20;
    const int* readOnly = &a;
    int* const fixedAddress = &a;
    *fixedAddress = 30;
    std::cout << *readOnly << '\n';
    readOnly = &b;
    std::cout << *readOnly << '\n';
}
~~~

<details>
<summary>예상 출력과 해설</summary>

~~~text
30
20
~~~

readOnly를 통한 쓰기는 막았지만 원래 a는 상수 객체가 아닙니다. fixedAddress는 재지정 불가능하지만 대상은 변경할 수 있습니다.

</details>

직접 변형: readOnly를 통해 값을 쓰는 문장과 fixedAddress를 다른 주소로 바꾸는 문장을 각각 추가해 컴파일 오류를 확인하세요. 캐스팅으로 오류를 숨기지 않습니다.

추가 질문 해설: 다운캐스팅이 반복되면 기반 인터페이스가 실제 요구를 표현하는지 검토합니다. 런타임 타입 분기가 정당한 경우도 있으므로 캐스팅 횟수만으로 설계가 나쁘다고 결론내리지 않습니다.

<a id="cpp-12"></a>
## 12. 슬라이싱은 virtual로 되돌릴 수 없다

<!-- EXPLANATION_PASS -->

**참조를 통한 다형성과 값 복사의 차이**

Boss를 Enemy&로 받으면 함수는 원래 Boss 객체를 계속 봅니다. Enemy를 값으로 받으면 새 Enemy 객체가 만들어지고 Boss에만 있는 부분은 그 객체에 들어가지 않습니다. virtual은 현재 존재하는 객체의 동적 타입에 따라 호출할 뿐 잃어버린 파생 부분을 복구하지 않습니다.

예제의 값 전달 결과와 참조 전달 결과가 다른 이유가 여기 있습니다. vector<Enemy>에 Boss를 넣는 설계도 값 저장 과정에서 같은 문제를 만납니다. 다형적 소유가 필요하면 가상 소멸자를 갖춘 기반 타입과 포인터 소유 정책을 함께 선택합니다.

가상 함수 테이블은 흔한 구현 기법입니다. 성능을 비교하려면 호출 방식 외에 객체가 흩어진 정도도 통제해야 합니다. 서로 다른 객체 배치로 측정한 결과를 virtual 한 가지의 비용이라고 결론 내리면 원인을 잘못 짚습니다.

~~~cpp
#include <iostream>
struct Enemy {
    virtual ~Enemy() = default;
    virtual int Damage() const { return 1; }
};
struct Boss : Enemy {
    int Damage() const override { return 10; }
};
int ByValue(Enemy enemy) { return enemy.Damage(); }
int ByReference(const Enemy& enemy) { return enemy.Damage(); }
int main() {
    Boss boss;
    std::cout << ByValue(boss) << ' ' << ByReference(boss) << '\n';
}
~~~

<details>
<summary>예상 출력과 해설</summary>

~~~text
1 10
~~~

ByValue는 새 Enemy 객체를 만들며 Boss의 기반 부분만 복사합니다. ByReference는 원래 Boss를 참조해 재정의된 함수가 호출됩니다.

</details>

직접 변형: vector<Enemy>와 vector<unique_ptr<Enemy>>에 Boss를 저장하는 경우를 비교하세요. 다형성과 할당·간접 접근 비용을 함께 설명하세요.

추가 질문 해설: 가상 호출 비용을 측정할 때 객체 배치를 같게 두고 호출 방식을 바꾸는 등 한 조건을 분리해야 합니다. 메모리 배치까지 바꾸면 캐시 효과가 섞입니다.

<a id="cpp-13"></a>
## 13. 템플릿과 번역 단위

<!-- EXPLANATION_PASS -->

**선언을 본다는 것과 실행할 코드가 있다는 것**

컴파일러는 보통 각 cpp와 그 파일이 포함한 헤더를 하나의 번역 단위로 처리합니다. 함수 선언은 호출할 때 필요한 타입 정보를 주고, 정의는 실제 일을 제공합니다. 일반 함수는 다른 cpp에서 생성된 정의를 링커가 연결할 수 있습니다.

템플릿은 사용할 타입으로 구체화할 정의가 필요합니다. 선언만 main.cpp에 보이고 정의는 다른 cpp에 있어도, 그 cpp에서 필요한 타입을 명시적으로 인스턴스화하지 않았다면 연결할 코드가 생기지 않을 수 있습니다. 그래서 템플릿 정의를 헤더에 두는 방식을 흔히 사용합니다.

아래 단일 파일 예제는 타입별 호출을 먼저 확인합니다. 이어지는 세 파일 실험은 선언만 보이는 상황에서 링크가 실패하는 이유, 정의를 헤더에 옮기는 해결, 지원 타입을 명시하는 해결을 따로 비교합니다. 헤더 가드는 한 번역 단위의 중복 포함을 막는 것이므로 이 문제의 해결책이 아닙니다.

~~~cpp
#include <iostream>
template<class T>
T Add(T a, T b) { return a + b; }
int main() {
    std::cout << Add(2, 3) << '\n';
    std::cout << Add(1.5, 2.5) << '\n';
}
~~~

<details>
<summary>예상 출력과 해설</summary>

~~~text
5
4
~~~

호출 타입에 맞는 템플릿 특수화가 사용됩니다. 아무 타입이나 가능한 것이 아니라 + 연산과 반환 변환 등 요구사항을 만족해야 합니다.

</details>

직접 변형:

1. 일반 함수 AddInt의 선언만 add.h에 넣고 main에서 호출해 링크 오류를 확인합니다.
2. 정의를 add.cpp에 넣고 두 cpp를 함께 빌드합니다.
3. 템플릿으로 바꾸어 선언만 헤더에 둔 경우와 정의까지 헤더에 둔 경우를 비교합니다.
4. 명시적 인스턴스화로 지원 타입을 고정하는 선택과 헤더 구현을 비교합니다.

추가 질문 해설: 헤더 포함은 선언을 보여 주는 일이고 링크는 정의를 연결하는 일입니다. 헤더 가드가 여러 번역 단위의 비인라인 정의 중복까지 해결하지는 않습니다.


**추가 실험 — 세 파일로 링크 오류의 원인 확인하기**

아래 세 파일을 같은 빈 폴더에 저장합니다. 첫 구성은 의도적으로 링크에 실패합니다. main이 필요한 Add<int> 정의를 어떤 번역 단위에서도 생성하지 않는 상황입니다.

```cpp
// experiment: add.h
#pragma once
template<class T> T Add(T a, T b);
```

```cpp
// experiment: add.cpp
#include "add.h"
template<class T> T Add(T a, T b) { return a + b; }
```

```cpp
// experiment: template-main.cpp
#include "add.h"
#include <iostream>
int main() { std::cout << Add(2, 3) << '\n'; }
```

Visual Studio의 x64 Native Tools 터미널에서 이 폴더로 이동해 실행합니다.

```text
cl /nologo /std:c++17 /EHsc /W4 /WX /utf-8 template-main.cpp add.cpp /Fe:template-test.exe
```

<details>
<summary>왜 실패하며 어떻게 고치나요?</summary>

main을 컴파일할 때 Add<int>의 선언으로 호출 형식을 알 수 있지만 본체는 보이지 않습니다. add.cpp에는 템플릿 정의가 있어도 int를 대상으로 인스턴스화하는 사용이나 지시가 없습니다. MSVC에서는 이 구성의 Add<int> 연결이 LNK2019로 실패하는 것을 확인했습니다. 오류 번호와 표현은 다른 도구에서 달라질 수 있습니다.

해결 A: add.h의 선언을 `template<class T> T Add(T a, T b) { return a + b; }`로 바꾸고 add.cpp에서는 중복된 정의를 제거해 include만 남깁니다. 같은 명령으로 빌드하면 5를 출력합니다. 호출하는 번역 단위가 필요한 타입의 정의를 만들 수 있습니다.

해결 B: 처음 구성으로 되돌리고 add.cpp의 정의 아래에 `template int Add<int>(int, int);`를 넣습니다. 같은 명령으로 빌드하면 역시 5를 출력합니다. 이번에는 add.cpp가 int용 코드를 명시적으로 제공합니다.

B에서 main의 호출을 Add(1.5, 2.5)로 바꾸면 double용 정의가 제공되지 않아 링크에 실패합니다. A에서는 해당 타입의 연산이 유효하므로 4를 출력합니다. 명시적 인스턴스화의 지원 타입 제한과 헤더 정의의 유연성이 드러나는 비교입니다.

</details>

참고: [Microsoft — 템플릿 소스 구성](https://learn.microsoft.com/en-us/cpp/cpp/source-code-organization-cpp-templates?view=msvc-170).

<a id="cpp-14"></a>
## 14. 비교 횟수와 실행 시간은 다르다

<!-- EXPLANATION_PASS -->

**자료 구조는 어떤 작업이 많은지로 고르기**

매 프레임 적 전체를 순회하는 작업과 아이템 ID 하나를 찾는 작업은 다릅니다. vector는 연속 원소 순회에 적합하고 해시 구조는 키 조회에 유리할 수 있지만, 해시 계산과 버킷 접근에도 비용이 있습니다.

예제는 네 원소를 선형 검색한 비교 횟수를 보여 줍니다. 해시 조회의 CPU 명령 수나 실행 시간을 같은 숫자로 계산한 예제가 아닙니다. 실제 시간을 비교하려면 데이터 구성 시간을 조회 시간과 분리하고, 성공·실패 조회 비율과 데이터 크기를 맞춰야 합니다.

키 순서가 필요하면 map, 빠른 전체 순회가 필요하면 vector 등 요구를 먼저 적습니다. list의 삽입이 상수 시간이라는 말에도 위치를 이미 알고 있다는 조건이 있습니다. 삽입 위치 검색까지 포함하면 전체 작업 비용이 달라집니다.

~~~cpp
#include <algorithm>
#include <iostream>
#include <unordered_map>
#include <vector>
int main() {
    std::vector<int> ids{10, 20, 30, 40};
    int comparisons = 0;
    auto found = std::find_if(ids.begin(), ids.end(), [&](int id) {
        ++comparisons;
        return id == 40;
    });
    std::unordered_map<int, int> index{{10, 0}, {20, 1}, {30, 2}, {40, 3}};
    std::cout << (found != ids.end()) << ' ' << comparisons << '\n';
    std::cout << index.at(40) << '\n';
}
~~~

<details>
<summary>예상 출력과 해설</summary>

~~~text
1 4
3
~~~

선형 탐색은 네 번 비교했습니다. 해시 조회가 한 CPU 명령으로 끝났다는 뜻은 아닙니다. 해시 계산·충돌 확인·메모리 접근과 구성 비용이 있습니다. 이 예제는 타이밍 벤치마크가 아닙니다.

</details>

직접 변형: 입력 크기를 16·1,024·65,536으로 늘려 성공·실패 조회를 섞으세요. 구성 시간과 조회 시간을 분리하고 Release 빌드에서 측정합니다. 결과 합계를 소비하고 타이밍 루프에서 로그를 제거합니다.

추가 질문 해설: 작은 데이터에서 연속 메모리의 단순 탐색이 유리할 수 있습니다. 큰 데이터의 키 조회가 많으면 해시 구조가 유리할 수 있으나 순서·갱신·최악 시간 요구를 함께 봅니다.

<a id="cpp-15"></a>
## 15. 원자 연산과 복합 규칙

<!-- EXPLANATION_PASS -->

**한 번의 원자 연산과 여러 단계의 규칙**

남은 탄약이 1일 때 두 작업이 각각 1을 읽고 발사한 뒤 감소시키면 총 두 번 발사할 수 있습니다. 읽기와 감소를 각각 원자 연산으로 바꾸어도 두 단계를 하나의 조건부 작업으로 묶지는 못합니다.

아래 카운터는 ++라는 원자적 증가만 필요하므로 atomic으로 갱신 손실을 막습니다. join 후 출력해 두 작업의 완료도 확인합니다. 반면 탄약 확인·소비·발사 허용을 함께 결정하려면 하나의 잠금 범위에서 조건을 검사하고 변경하거나 적절한 비교 교환 알고리즘을 설계해야 합니다.

mutex를 쓸 때는 그 데이터에 접근하는 모든 경로가 같은 규칙을 따라야 합니다. 한 경로만 잠가서는 보호되지 않습니다. 서로 기다리는 작업을 join할 때 잠금을 쥐고 있으면 교착될 수 있으므로 데이터 보호 범위와 작업 종료 순서를 함께 봅니다.

~~~cpp
#include <atomic>
#include <iostream>
#include <thread>
int main() {
    std::atomic<int> count{0};
    auto work = [&count] {
        for (int i = 0; i < 1000; ++i) ++count;
    };
    std::thread a(work);
    std::thread b(work);
    a.join();
    b.join();
    std::cout << count.load() << '\n';
}
~~~

<details>
<summary>예상 출력과 해설</summary>

~~~text
2000
~~~

원자 증가가 갱신을 잃지 않도록 하고 join이 작업 완료 뒤 읽게 합니다. count의 수명도 작업보다 깁니다. 비원자 int에서 우연히 2000이 나와도 안전성 증거는 아닙니다.

</details>

직접 생각할 반례: stock을 읽어 양수이면 감소하는 두 단계에서, 두 작업이 모두 1을 읽고 차례로 감소하면 -1이 될 수 있습니다. 각 단계가 원자적이어도 합친 규칙은 하나의 원자 연산이 아닙니다.

추가 질문 해설: 전체 판정을 mutex로 보호하거나 올바른 비교 후 교환 루프를 설계합니다. 잠금 순서가 순환하고 서로 기다리면 교착이 가능하므로 일관된 순서·수명·대기 관계를 함께 봅니다.

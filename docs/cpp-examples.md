# C++ — 예측하고 실행하며 이해하기

[학습 목차](README.md) · [개념 요약](cpp.md) · [차시별 과제](questions.md)

각 예제는 헤더와 main을 포함한 독립 프로그램입니다. 한 번에 하나만 main.cpp에 넣고 C++17 이상으로 빌드하세요. 먼저 출력 순서를 예상하고 해설을 펼칩니다. 검증 환경과 결과는 [검증 기록](verification.md)에 기록합니다. 휴대폰에서는 상황 → 코드 → 출력 → 이유 → 변형 순서로 읽습니다.

<a id="cpp-01"></a>
## 1. 저장 공간과 객체의 수명

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

<a id="cpp-06"></a>
## 6. Rule of Zero가 실제로 줄이는 코드

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

<a id="cpp-14"></a>
## 14. 비교 횟수와 실행 시간은 다르다

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

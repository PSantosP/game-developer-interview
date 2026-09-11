# Unreal Engine

[학습 안내](README.md) · [차시별 질문·실습](questions.md)

작성 기준과 출처 표시는 [학습 안내](README.md)를 따릅니다. 구체적인 에디터 위치와 API는 실습할 엔진 버전에서 확인합니다.

<a id="ue-01"></a>
## 일반 C++ 객체와 UObject

<!-- RECALL_CARD_START -->
**기억할 기준: 엔진 객체 시스템이 필요한가**

**쉬운 예:** 체력 계산식만 담는 작은 값과 월드에 등장하는 Actor는 다른 역할입니다.

**가리고 떠올리기:** 모든 클래스를 UObject로 만들 필요가 있을까요?

<details>
<summary>면접에서 짧게 말하기</summary>

엔진의 리플렉션·GC 등이 필요한 객체는 알맞은 엔진 생성 경로를 사용합니다. 일반 C++ 값과 자원 관리 객체는 별도로 설계합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](unreal-walkthroughs.md#ue-01)

> “일반 C++ 객체와 UObject는 무엇이 다른가요?”

- `UObject`는 Unreal의 객체 시스템에 참여하는 기본 클래스입니다. 리플렉션, 직렬화, GC 등 엔진 기능과 연결됩니다.
- 일반적인 UObject 인스턴스는 `NewObject`, Actor는 `SpawnActor`, 생성자에서 기본 서브오브젝트는 `CreateDefaultSubobject`처럼 목적에 맞는 엔진 경로로 생성합니다.
- `UObject`에 대한 원시 포인터를 지역 변수로 보관하는 것만으로 GC가 그 참조를 추적한다고 가정하면 안 됩니다.

추가 질문: 계산만 수행하는 작은 자료형까지 모두 UObject로 만들 필요가 있나요?

참고: [Epic — Object Pointers](https://dev.epicgames.com/documentation/en-us/unreal-engine/object-pointers-in-unreal-engine), [Actor Lifecycle](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-actor-lifecycle).

<a id="ue-02"></a>
## UObject 포인터 선택

<!-- RECALL_CARD_START -->
**기억할 기준: 유지·관찰·나중에 로드를 구분**

**쉬운 예:** 조준 대상은 사라질 수 있고 무기 에셋은 아직 로드되지 않았을 수 있습니다.

**가리고 떠올리기:** TObjectPtr라는 이름만으로 모든 위치에서 GC 추적될까요?

<details>
<summary>면접에서 짧게 말하기</summary>

GC가 추적하는 강한 참조, 약한 관찰, 소프트 에셋 참조를 목적에 맞게 고릅니다. 선언 위치와 추적 경로, 사용 시점 유효성을 확인합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](unreal-walkthroughs.md#ue-02)

> “TObjectPtr, TWeakObjectPtr, TSoftObjectPtr는 언제 사용하나요?”

| 형태 | 대표 용도 | 주의점 |
|---|---|---|
| `UPROPERTY()`의 `TObjectPtr<T>` | 살아 있는 UObject가 유지하는 강한 참조 | 멤버가 GC 추적 경로에 있어야 함 |
| `TWeakObjectPtr<T>` | 사라질 수 있는 대상 관찰 | 사용 직전 유효성 확인 |
| `TSoftObjectPtr<T>` | 경로로 에셋을 가리키고 필요할 때 로드 | 선언만으로 로드되지 않음 |

`TObjectPtr`라는 타입명만으로 어디서든 GC 안전성이 확보되지는 않습니다. 또한 강한 참조가 있어도 Actor의 명시적인 `Destroy()`를 막지는 않습니다.

추가 질문: 적을 조준하는 참조와 나중에 로드할 무기 에셋 참조에 같은 포인터를 고르겠습니까?

참고: [Epic — Object Pointers](https://dev.epicgames.com/documentation/en-us/unreal-engine/object-pointers-in-unreal-engine).

<a id="ue-03"></a>
## 생성자와 BeginPlay

<!-- RECALL_CARD_START -->
**기억할 기준: 기본 구성과 플레이 중 준비를 분리**

**쉬운 예:** 기본 컴포넌트는 생성자에서 만들고 플레이 중 연결은 적절한 초기화 시점에 처리합니다.

**가리고 떠올리기:** BeginPlay라면 다른 모든 Actor의 준비도 끝났을까요?

<details>
<summary>면접에서 짧게 말하기</summary>

생성자는 기본값·기본 구성을 맡고 BeginPlay는 플레이 시작 작업을 맡습니다. 다른 객체와 복제 데이터 준비는 별도 조건으로 확인합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](unreal-walkthroughs.md#ue-03)

> “생성자와 BeginPlay는 어떤 작업을 나눠 맡아야 하나요?”

- 생성자에서는 기본값과 기본 컴포넌트 구성을 설정합니다. 클래스 기본 객체(CDO) 생성 등에도 사용되므로 플레이 중인 월드가 준비되었다고 가정하지 않습니다.
- `BeginPlay`는 플레이 시작 단계의 초기화에 사용합니다. 다른 Actor의 `BeginPlay`가 모두 끝났거나 클라이언트에 필요한 참조가 모두 복제되었다고 보장하지는 않습니다.
- 로드된 Actor, 스폰된 Actor, 에디터에서 복제된 Actor는 그 이전 경로가 다를 수 있습니다. 모든 경로를 하나의 단순한 호출 순서로 외우지 않습니다.

추가 질문: 다른 Actor를 찾는 코드가 생성자에서는 실패하고 플레이 중에는 성공하는 이유는 무엇일까요?

참고: [Epic — Actor Lifecycle](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-actor-lifecycle).

<a id="ue-04"></a>
## Destroy와 EndPlay

<!-- RECALL_CARD_START -->
**기억할 기준: 게임에서 끝남과 메모리 회수는 다른 시점**

**쉬운 예:** 적이 Destroy된 뒤 늦게 도착한 로드 콜백이 그 적을 사용할 수 있습니다.

**가리고 떠올리기:** 포인터가 남아 있다는 이유로 종료된 Actor를 써도 될까요?

<details>
<summary>면접에서 짧게 말하기</summary>

EndPlay에서 게임플레이 작업과 구독을 정리하고 이후 콜백의 유효성을 확인합니다. Destroy와 실제 GC 회수를 같은 순간으로 보지 않습니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](unreal-walkthroughs.md#ue-04)

> “Destroy를 호출하면 Actor의 메모리가 즉시 해제되나요?”

- 게임플레이에서의 종료와 메모리 회수를 구분해야 합니다. `Destroy`로 종료 절차를 밟고, 실제 메모리 회수는 이후 GC 단계에서 이루어집니다.
- `EndPlay`는 명시적 파괴 외에도 레벨 전환이나 플레이 종료 등으로 호출될 수 있습니다.
- 등록한 이벤트, 타이머, 진행 중인 작업은 대상 수명에 맞게 정리합니다. 특히 파괴 이후 콜백에서 예전 대상을 사용하는 경로를 확인합니다.

추가 질문: 포인터가 `nullptr`이 아니면 파괴 중인 Actor도 사용해도 되나요?

참고: [Epic — Actor Lifecycle](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-actor-lifecycle).

<a id="ue-05"></a>
## 게임 전체 상태의 위치

<!-- RECALL_CARD_START -->
**기억할 기준: 상태가 누구에게 얼마나 오래 필요한가**

**쉬운 예:** 매치 규칙과 모든 플레이어가 볼 점수, 맵을 넘어 유지할 로컬 설정은 수명이 다릅니다.

**가리고 떠올리기:** GameInstance에 저장하면 다른 클라이언트에도 자동 복제될까요?

<details>
<summary>면접에서 짧게 말하기</summary>

서버 규칙은 GameMode, 공유 매치 상태는 GameState, 프로세스 단위 지속 상태는 GameInstance 등을 검토합니다. 지속과 네트워크 공유는 별개입니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](unreal-walkthroughs.md#ue-05)

> “GameMode, GameState, GameInstance의 역할은 무엇인가요?”

- `GameMode`: 규칙과 참가·스폰 등의 서버 측 판단을 담당합니다. 네트워크 클라이언트에는 해당 서버의 GameMode 인스턴스가 없습니다.
- `GameState`: 남은 시간이나 경기 상태처럼 클라이언트들이 알아야 하는 게임 상태를 담고 복제합니다.
- `GameInstance`: 게임 인스턴스 수명 동안 유지되어 맵 전환을 넘는 데이터에 사용할 수 있습니다. 서버와 클라이언트 사이에 자동으로 공유·복제되는 저장소는 아닙니다.

추가 질문: 라운드 승패와 로컬 옵션 설정은 각각 어디에 두겠습니까?

참고: [Epic — Gameplay Framework](https://dev.epicgames.com/documentation/en-us/unreal-engine/gameplay-framework-in-unreal-engine).

<a id="ue-06"></a>
## 플레이어와 조종 대상

<!-- RECALL_CARD_START -->
**기억할 기준: 플레이어의 정체성과 몸을 나누기**

**쉬운 예:** 죽어서 Pawn을 교체해도 플레이어의 점수는 유지되어야 할 수 있습니다.

**가리고 떠올리기:** PlayerController는 모든 클라이언트가 모든 사람의 것을 갖고 있을까요?

<details>
<summary>면접에서 짧게 말하기</summary>

Controller의 조종, Pawn의 몸, PlayerState의 플레이어 상태를 구분합니다. 재스폰 수명과 각 네트워크 역할에서의 존재 범위를 확인합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](unreal-walkthroughs.md#ue-06)

> “PlayerController, Pawn, PlayerState를 구분하는 이유는 무엇인가요?”

- `PlayerController`: 플레이어의 제어 주체입니다. 서버와 해당 소유 클라이언트에 존재하며 다른 모든 클라이언트에 똑같이 존재하지는 않습니다.
- `Pawn`: 실제로 조종되는 월드 내 대상입니다. `Character`는 캐릭터 이동 등을 제공하는 Pawn의 하위 클래스입니다.
- `PlayerState`: 점수 등 플레이어 상태를 표현하고 다른 클라이언트에도 복제할 수 있습니다.
- 캐릭터 사망과 재스폰 시 Pawn을 교체할 수 있으므로, 함께 사라질 데이터와 남길 데이터를 수명에 따라 구분합니다. 맵 전환에서의 유지·복사는 별도 정책을 확인합니다.

추가 질문: 차량에 탑승해 Pawn이 바뀌어도 유지해야 하는 점수는 어디에 두겠습니까?

참고: [Epic — Gameplay Framework](https://dev.epicgames.com/documentation/en-us/unreal-engine/gameplay-framework-in-unreal-engine).

<a id="ue-07"></a>
## 변수 복제와 RPC

문헌 보충: Unreal RPC는 반환값을 직접 받는 일반 함수 호출과 다릅니다. 서버 응답이 필요하면 복제 상태나 별도 응답 경로를 설계합니다. 클라이언트에서 NetMulticast를 호출한다고 서버와 모든 클라이언트로 전파되지는 않습니다. [Epic — RPC 실행 규칙](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-procedure-calls-in-unreal-engine).

<!-- RECALL_CARD_START -->
**기억할 기준: 상태 전달과 사건 요청을 구분**

**쉬운 예:** 체력은 나중에 접속한 사람도 알아야 하지만 한 번의 입력 요청은 실행 사건입니다.

**가리고 떠올리기:** RPC만으로 현재 체력을 보내면 늦은 접속자는 어떻게 알까요?

<details>
<summary>면접에서 짧게 말하기</summary>

지속 상태는 복제 속성, 필요한 호출은 RPC로 설계합니다. 소유권·전달 조건·관련성과 상태 복구를 확인합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](unreal-walkthroughs.md#ue-07)

> “변수 복제와 RPC는 어떻게 다른가요?”

- 변수 복제는 서버의 상태를 클라이언트에 전달하는 데 사용합니다. 모든 중간 값 변화가 각각 이벤트처럼 도착하는 것은 아닙니다.
- RPC는 지정된 원격 실행 규칙에 따라 함수를 호출하는 방법입니다. 호출 방향, Actor 소유권, 연결, 복제 설정을 함께 확인해야 합니다.
- 계속 유지되어야 하는 상태와 일회성 요청을 구분합니다. 체력은 상태이며 공격 버튼 입력은 서버에 전달할 요청이 될 수 있습니다.
- `Reliable`은 게임 규칙 검증이나 과거 호출의 재생을 대신하지 않습니다. 늦게 접속한 사람에게 과거 Multicast가 자동 재생되지는 않습니다.

추가 질문: 문을 열었다는 Multicast만 보냈다면 나중에 접속한 사람은 열린 문을 볼 수 있나요?

참고: [Epic — Networking Overview](https://dev.epicgames.com/documentation/en-us/unreal-engine/networking-overview-for-unreal-engine).

<a id="ue-08"></a>
## 서버 권위 공격 판정

<!-- RECALL_CARD_START -->
**기억할 기준: 요청은 클라이언트, 확정은 서버**

**쉬운 예:** 클라이언트가 공격 버튼을 눌렀다고 피해 수치를 그대로 확정하지 않습니다.

**가리고 떠올리기:** 클라이언트가 보낸 명중 대상과 피해를 서버가 바로 믿어도 될까요?

<details>
<summary>면접에서 짧게 말하기</summary>

서버는 공격 가능 상태·거리·대상 등을 검증해 결과를 확정합니다. 클라이언트 표현 및 예측과 권위 있는 판정을 구분합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](unreal-walkthroughs.md#ue-08)

> “클라이언트에서 공격했을 때 서버는 무엇을 검증해야 하나요?”

- 서버가 최종 게임 상태를 결정하는 모델에서는 클라이언트가 보낸 피해량과 피격 결과를 그대로 확정하지 않습니다.
- 요청자의 소유권과 현재 행동 가능 상태, 자원·쿨다운, 대상·거리 등 게임 규칙을 서버에서 판단하도록 설계합니다.
- 클라이언트는 입력 반응을 빠르게 보여 줄 수 있지만, 서버 결과와 다를 때 보정할 흐름이 필요합니다.

설계 예: ‘적 체력을 30 줄여라’보다 ‘이 공격을 시도했다’를 전달하고, 서버가 유효한 공격인지 판단해 체력을 변경합니다. 구체적인 판정 시점과 지연 보상은 게임 요구사항에 따라 정합니다.

추가 질문: 서버가 현재 위치만으로 판정하면 지연이 큰 플레이어에게 어떤 문제가 생기나요?

참고: [Epic — Networking Overview](https://dev.epicgames.com/documentation/en-us/unreal-engine/networking-overview-for-unreal-engine). 검증 항목과 예시는 서버 권위 모델을 적용한 설계 예입니다.

<a id="ue-09"></a>
## GAS 구성 요소

<!-- RECALL_CARD_START -->
**기억할 기준: 능력·수치·효과의 역할 분리**

**쉬운 예:** 대시 실행, 스태미나 값, 스태미나 감소와 쿨다운은 서로 다른 책임입니다.

**가리고 떠올리기:** GameplayAbility 하나에 모든 영구 수치를 직접 보관하면 어떤 수명 문제가 생길까요?

<details>
<summary>면접에서 짧게 말하기</summary>

ASC가 능력·효과를 관리하고 Ability는 실행 흐름, AttributeSet은 수치, GameplayEffect는 수치 변화 등을 담당합니다. 태그로 조건과 상태를 표현합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](unreal-walkthroughs.md#ue-09)

> “Ability, Effect, AttributeSet, ASC는 어떤 역할인가요?”

- `GameplayAbility`: 공격·회피·회복 같은 행동의 실행 흐름을 표현합니다.
- `GameplayEffect`: 피해·버프 등 속성 및 태그에 영향을 주는 효과를 표현합니다.
- `AttributeSet`: 체력·공격력 같은 속성의 정의와 관련 처리를 담습니다.
- `AbilitySystemComponent`: 어빌리티와 효과 등을 관리하는 중심 컴포넌트입니다.
- `GameplayTag`: 행동과 상태를 이름의 계층으로 표현합니다. 태그 이름만 만들었다고 행동 차단이 저절로 구현되지는 않습니다.

추가 질문: 공격 모션은 실행됐는데 피해가 적용되지 않는다면 Ability 실행, Effect 적용, Attribute 변경 중 어디까지 확인하겠습니까?

참고: [Epic — Gameplay Ability System](https://dev.epicgames.com/documentation/en-us/unreal-engine/gameplay-ability-system-for-unreal-engine).

<a id="ue-10"></a>
## C++ 자원 관리와 Unreal GC의 경계

<!-- RECALL_CARD_START -->
**기억할 기준: RAII와 GC가 책임지는 대상이 다름**

**쉬운 예:** 파일 잠금은 C++ 관리 객체로, UObject 참조는 엔진의 추적 규칙으로 다룹니다.

**가리고 떠올리기:** UObject를 일반 shared_ptr로 감싸면 엔진 GC도 그 소유권을 알까요?

<details>
<summary>면접에서 짧게 말하기</summary>

일반 자원은 RAII로 관리하고 UObject는 엔진의 생성·참조·GC 규칙을 따릅니다. 두 소유권 체계를 임의로 섞지 않습니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](unreal-walkthroughs.md#ue-10)

> “일반 C++ 스마트 포인터를 UObject에 그대로 사용해도 되나요?”

- 일반 `std::unique_ptr`와 `std::shared_ptr`의 기본 삭제 방식으로 UObject의 수명을 관리하지 않습니다. 엔진 객체 시스템의 생성·종료·GC 경로를 사용합니다.
- 일반 C++ 자료의 소유권과 UObject에 대한 참조 유지는 별개의 문제입니다.
- UObject 멤버의 추적되는 참조, 비소유 관찰, 에셋 경로 참조를 구분해 객체 포인터를 고릅니다. 일반 C++ 객체에서 GC 강한 참조가 필요한 경우에는 `TStrongObjectPtr` 같은 엔진의 지원 수단을 검토합니다.

추가 질문: ‘스마트 포인터니까 자동으로 안전하다’는 설명에서 빠진 조건은 무엇인가요?

참고: [Epic — Object Pointers](https://dev.epicgames.com/documentation/en-us/unreal-engine/object-pointers-in-unreal-engine).

<a id="ue-11"></a>
## 리플렉션·CDO·Blueprint

<!-- RECALL_CARD_START -->
**기억할 기준: 클래스 기본값과 실행 인스턴스를 구분**

**쉬운 예:** Blueprint 기본 체력은 클래스 기본값이며 각 적의 현재 체력과 다릅니다.

**가리고 떠올리기:** CDO에 플레이 중 한 캐릭터의 상태를 저장하면 무엇이 잘못될까요?

<details>
<summary>면접에서 짧게 말하기</summary>

리플렉션은 엔진이 타입 정보를 다루는 통로이고 CDO는 클래스 기본값의 기준입니다. 런타임 상태는 적절한 인스턴스에 둡니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](unreal-walkthroughs.md#ue-11)

> “UPROPERTY를 붙이는 것과 C++ 멤버를 선언하는 것은 어떻게 다른가요?”

- 리플렉션은 엔진이 타입·멤버 정보를 다룰 수 있게 합니다. UHT가 관련 선언을 처리하고 생성 코드를 C++ 빌드에 연결합니다.
- `UPROPERTY`의 지정자에 따라 에디터 편집, Blueprint 접근, 직렬화 등 참여 방식이 달라집니다. 아무 지정자 없이 모든 기능이 활성화되는 것은 아닙니다.
- CDO는 클래스의 기본값을 가진 객체입니다. 생성자에서 실행 중인 플레이어를 찾거나 세계 상태를 바꾸면 기본값 구성과 런타임 작업이 섞입니다.
- Blueprint에 저장된 재정의 값이 있다면 C++ 기본값을 바꾸어도 그 값이 그대로 남을 수 있습니다.
- C++에는 재사용할 규칙과 불변 조건을, Blueprint에는 조합과 콘텐츠 설정을 두는 설계를 검토합니다. 이는 절대적인 성능 규칙이 아니라 변경 빈도와 협업을 고려한 선택입니다.

추가 질문: 에디터에서 편집 가능하다는 것과 Blueprint에서 쓰기 가능하다는 것은 같은가요?

참고: [Epic — Unreal Object Handling](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-object-handling-in-unreal-engine).

<a id="ue-12"></a>
## Component·Subsystem과 책임

<!-- RECALL_CARD_START -->
**기억할 기준: 재사용할 기능과 서비스의 수명 선택**

**쉬운 예:** 여러 Actor의 체력 기능은 Component, 월드 범위 서비스는 해당 Subsystem을 검토할 수 있습니다.

**가리고 떠올리기:** 맵 전환 때 끝나야 할 서비스를 GameInstance 범위에 두면 어떤 정리가 필요할까요?

<details>
<summary>면접에서 짧게 말하기</summary>

책임과 소유자·수명에 따라 Component와 Subsystem을 선택합니다. 큰 관리자 하나에 모든 기능을 모으기보다 범위와 의존성을 드러냅니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](unreal-walkthroughs.md#ue-12)

> “ActorComponent와 Subsystem은 어떻게 고르나요?”

- Component는 Actor에 속한 기능을 나눕니다. 인벤토리나 상호작용처럼 여러 Actor에 조합할 기능에 사용할 수 있습니다.
- Subsystem은 GameInstance·World·LocalPlayer 등 선택한 대상의 수명에 연결되는 서비스에 사용할 수 있습니다. 모든 Subsystem이 프로세스에 하나인 전역 객체는 아닙니다.
- 시작·종료 책임과 참조 대상의 수명을 먼저 결정합니다. 맵 전환을 넘는 서비스가 이전 World의 Actor를 계속 잡고 있지 않은지 확인합니다.
- 클래스를 분리했다고 결합도가 자동으로 줄지는 않습니다. 호출 방향, 데이터 소유자, 초기화 의존성을 함께 봅니다.

추가 질문: 로컬 사용자별 UI 서비스와 월드별 적 목록에 같은 Subsystem 수명이 맞을까요?

참고: [Epic — Subsystems](https://dev.epicgames.com/documentation/en-us/unreal-engine/programming-subsystems-in-unreal-engine).

<a id="ue-13"></a>
## Delegate·Timer와 종료

<!-- RECALL_CARD_START -->
**기억할 기준: 등록한 작업에도 종료 경로가 필요**

**쉬운 예:** 피격 알림을 구독한 UI나 공격 타이머는 화면·Actor 종료 뒤에도 연결이 남을 수 있습니다.

**가리고 떠올리기:** 약한 바인딩이 있으면 타이머와 논리 상태 정리를 전부 생략해도 될까요?

<details>
<summary>면접에서 짧게 말하기</summary>

Delegate·Timer의 대상 수명과 등록 해제를 설계합니다. 메모리 접근 안전성과 게임 규칙상 작업 취소를 따로 확인합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](unreal-walkthroughs.md#ue-13)

> “이벤트로 바꾸면 Tick보다 항상 좋은가요?”

- 상태가 바뀔 때만 필요한 갱신은 이벤트가 적합할 수 있습니다. 매 프레임 필요한 연속 계산은 Tick이 자연스러울 수 있습니다. Timer도 작업량을 없애 주지는 않습니다.
- 단일·멀티캐스트, Dynamic 여부와 바인딩 대상에 따라 용도와 비용이 다릅니다. Blueprint 연결이 필요한지도 판단합니다.
- UObject를 인식하는 바인딩과 원시 포인터·일반 람다 캡처의 수명 처리는 같지 않습니다. `this`를 캡처한 람다가 객체를 소유한다고 가정하지 않습니다.
- 구독과 해제를 짝지어 중복 등록을 막고, 타이머 핸들을 보관해 필요할 때 취소합니다. 객체에 연결되지 않은 람다 타이머는 객체별 일괄 정리만으로 충분한지 확인해야 합니다.

추가 질문: UI를 열고 닫을 때마다 동일한 이벤트를 구독하면 세 번째 열기에서 어떤 현상이 생길까요?

참고: [Epic — Delegates](https://dev.epicgames.com/documentation/en-us/unreal-engine/delegates-and-lambda-functions-in-unreal-engine), [Timers](https://dev.epicgames.com/documentation/en-us/unreal-engine/gameplay-timers-in-unreal-engine).

<a id="ue-14"></a>
## 입력에서 이동까지

<!-- RECALL_CARD_START -->
**기억할 기준: 입력 숫자를 월드의 움직임으로 바꾸기**

**쉬운 예:** 앞으로 입력한 값은 카메라 기준 방향과 결합해 이동 요청으로 이어질 수 있습니다.

**가리고 떠올리기:** 카메라가 90도 돌아가도 월드 X축을 앞으로 쓰면 어떻게 움직일까요?

<details>
<summary>면접에서 짧게 말하기</summary>

입력 값의 의미와 좌표계를 확인하고 기준 회전에서 이동 방향을 구합니다. 입력 처리와 실제 이동 컴포넌트의 적용을 연결합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](unreal-walkthroughs.md#ue-14)

> “Input Action을 만들었는데 왜 입력이 들어오지 않을까요?”

- Action은 행동과 값의 타입을 표현하고 Mapping Context는 키와 행동의 대응을 담습니다. 로컬 플레이어에 Context가 적용되고 입력 함수가 바인딩되는 경로를 확인합니다.
- Modifier는 입력값을 변환하고 Trigger는 행동의 발동 조건을 판단합니다. `Triggered`가 언제 호출되는지는 Trigger 설정에 달려 있습니다.
- 로컬 입력 값, 카메라 기준 방향, 월드 이동 방향을 구분합니다. 카메라가 돌아가도 전진 방향이 월드 X로 고정된다면 어느 좌표계를 사용했는지 확인합니다.
- 입력을 받았다는 사실과 CharacterMovement가 실제 이동을 수행한 결과는 별도로 확인합니다.

추가 질문: 입력 컨텍스트 전환 후 같은 키로 UI와 공격이 동시에 실행된다면 무엇을 보겠습니까?

참고: [Epic — Enhanced Input](https://dev.epicgames.com/documentation/en-us/unreal-engine/enhanced-input-in-unreal-engine).

<a id="ue-15"></a>
## Trace·Sweep과 공격 판정

<!-- RECALL_CARD_START -->
**기억할 기준: 선 검사와 부피를 가진 검사를 구분**

**쉬운 예:** 빠르게 움직인 검의 현재 위치만 검사하면 프레임 사이의 적을 놓칠 수 있습니다.

**가리고 떠올리기:** Sweep 한 번이 회전하는 긴 검의 모든 궤적을 보장할까요?

<details>
<summary>면접에서 짧게 말하기</summary>

Trace·Sweep의 형상·시작·끝·채널을 정의합니다. 시간 간격과 회전 궤적, 중복 명중 정책을 별도로 설계하고 시각화합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](unreal-walkthroughs.md#ue-15)

> “Overlap 이벤트와 Trace 질의는 어떤 차이가 있나요?”

- Overlap 이벤트는 겹침 관계의 변화를 알리는 경로이고 Trace는 특정 시점에 공간을 질의하는 경로입니다. Line은 선, Sweep은 부피가 있는 형상을 이동시켜 검사합니다.
- 채널·오브젝트 타입·응답·무시 대상과 단일/다중 결과 조건을 확인합니다. 화면에 선을 그렸다고 충돌 설정이 올바른 것은 아닙니다.
- 빠른 무기는 현재 위치만 검사하면 프레임 사이 공간을 건너뛸 수 있습니다. 이전·현재 위치를 사용하는 검사를 검토하되 회전 궤적까지 완전히 덮는지는 별도 문제입니다.
- 한 공격의 여러 샘플이 같은 대상을 맞힐 수 있으므로 공격 단위 중복 처리 정책을 정합니다. 이전 공격의 기록을 언제 초기화하는지도 중요합니다.

추가 질문: 샘플 수를 두 배로 늘렸을 때 정확도와 질의 비용을 어떻게 비교하겠습니까?

참고: [Epic — Traces Overview](https://dev.epicgames.com/documentation/en-us/unreal-engine/traces-in-unreal-engine---overview). 무기 샘플링은 질의를 적용한 설계 예입니다.

<a id="ue-16"></a>
## Montage·Notify·Root Motion

<!-- RECALL_CARD_START -->
**기억할 기준: 애니메이션 종료와 공격 상태 정리를 연결**

**쉬운 예:** 몽타주가 피격으로 중단되면 정상 종료 때만 하던 공격 잠금 해제가 누락될 수 있습니다.

**가리고 떠올리기:** Notify 하나를 꼭 받는다는 가정으로 종료를 처리해도 될까요?

<details>
<summary>면접에서 짧게 말하기</summary>

Montage의 재생·중단·블렌드 종료와 Notify의 역할을 나눕니다. 정상·취소·사망 경로에서 상태를 정리하고 Root Motion의 이동 권한도 확인합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](unreal-walkthroughs.md#ue-16)

> “공격 애니메이션이 끝나지 않고 중단되면 무엇을 정리해야 하나요?”

- Montage는 애니메이션 재생을 섹션 등으로 제어합니다. Slot을 포함한 최종 포즈 경로도 맞아야 재생 결과가 보입니다.
- Notify는 애니메이션 시점에 이벤트를 연결합니다. 마지막 Notify 하나만을 필수 정리 경로로 삼으면 중단 상황을 놓칠 수 있습니다.
- 정상 종료·블렌드아웃·중단·사망을 구분해 공격 판정, 이동 제한, 이벤트 구독을 정리합니다. 중복 콜백에서도 결과가 일관되도록 설계합니다.
- Root Motion은 루트의 이동을 캐릭터 이동에 활용합니다. 메시가 움직이는 것과 충돌 캡슐이 함께 움직이는 것을 구분해 관찰합니다. 추출·적용 설정과 네트워크 정책도 확인합니다.

추가 질문: 애니메이션은 전진하는데 캡슐은 제자리에 있다면 어느 설정부터 확인하겠습니까?

참고: [Epic — Montage](https://dev.epicgames.com/documentation/en-us/unreal-engine/animation-montage-in-unreal-engine), [Root Motion](https://dev.epicgames.com/documentation/en-us/unreal-engine/root-motion-in-unreal-engine).

<a id="ue-17"></a>
## 에셋 비동기 로딩

<!-- RECALL_CARD_START -->
**기억할 기준: 로드 완료 때도 요청이 유효한가**

**쉬운 예:** 아이콘을 요청한 UI가 닫힌 뒤 완료 콜백이 올 수 있습니다.

**가리고 떠올리기:** 에셋 로드가 성공했어도 콜백에서 확인할 것은 무엇일까요?

<details>
<summary>면접에서 짧게 말하기</summary>

요청 핸들·대상 수명·실패·취소를 관리합니다. 완료 시 원래 요청이 아직 유효한지 확인하고 필요 없는 자원을 정리합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](unreal-walkthroughs.md#ue-17)

> “Soft 참조로 바꾸면 끊김이 없어지나요?”

- Soft 참조는 경로를 가지고 필요할 때 로드하도록 설계할 수 있게 합니다. 동기 로드를 호출하면 여전히 기다림이 생길 수 있습니다.
- 로딩 요청, 완료, 소비, 해제를 구분합니다. 완료 전에 요청자가 사라졌거나 다른 에셋으로 선택이 바뀌었는지 확인합니다.
- 로드 이후에도 필요한 동안 에셋을 유지할 참조 또는 핸들의 수명을 설계합니다. 경로만 가지고 있다고 로드된 객체가 계속 유지되지는 않습니다.
- PIE에서는 이미 에셋이 메모리에 있어 문제가 가려질 수 있습니다. 시작 상태를 기록하고 패키지에서도 에셋 포함 및 로딩을 확인합니다.

추가 질문: 무기 A를 요청하고 바로 B로 바꿨는데 A가 늦게 도착하면 어떻게 처리하겠습니까?

참고: [Epic — Asynchronous Asset Loading](https://dev.epicgames.com/documentation/en-us/unreal-engine/asynchronous-asset-loading-in-unreal-engine).

<a id="ue-18"></a>
## GAS의 활성화·비용·종료

문헌 보충: EndAbility는 실행 종료이며 이미 적용한 모든 효과와 비용의 자동 환불이 아닙니다. Task와 외부 구독의 정리, 환불 정책을 구분합니다. [Epic — GAS 실행 흐름](https://dev.epicgames.com/documentation/en-us/unreal-engine/understanding-the-unreal-engine-gameplay-ability-system).

<!-- RECALL_CARD_START -->
**기억할 기준: 시작할 수 있음과 비용 확정·종료를 나누기**

**쉬운 예:** 대시 시 스태미나와 쿨다운을 확인하고 성공·취소 시 남은 태스크와 상태를 정리합니다.

**가리고 떠올리기:** Commit 실패 뒤에도 이동 태스크를 계속 실행하면 어떤 문제가 생길까요?

<details>
<summary>면접에서 짧게 말하기</summary>

활성화 조건, 비용·쿨다운 확정, 실행, 종료·취소를 연결합니다. 예측과 서버 확정이 어긋나는 경로까지 고려합니다.

</details>
<!-- RECALL_CARD_END -->

[예제와 해설로 이해하기](unreal-walkthroughs.md#ue-18)

> “Ability가 한 번 실행된 뒤 다시 실행되지 않는다면 무엇을 확인하나요?”

- 부여·활성화 가능 조건·실행·종료를 구분합니다. ASC 초기화, 소유자와 실제 행동 대상, 태그 조건을 따라갑니다.
- 비용·쿨다운을 적용하는 Commit의 성공 여부와 실패 후 종료 경로를 확인합니다. 효과가 적용되었다고 애니메이션까지 성공했다는 뜻은 아닙니다.
- 비동기 Task 완료뿐 아니라 취소·중단에도 EndAbility로 이어지는지 확인합니다. 실행 중 태그나 등록한 콜백이 남으면 재실행을 막을 수 있습니다.
- 예측은 지원되는 작업을 로컬에서 먼저 수행하고 서버 결과와 조정하는 체계입니다. 임의의 게임 상태 변경이 전부 자동으로 되돌려지는 것은 아닙니다.
- ASC 위치는 재스폰 시 남길 상태와 복제 요구를 보고 정합니다. 모든 게임에서 PlayerState가 유일한 답은 아닙니다.

추가 질문: 비용 차감 직후 Montage 재생이 실패하면 환불 여부와 종료 처리를 어떻게 정하겠습니까?

참고: [Epic — Gameplay Ability System](https://dev.epicgames.com/documentation/en-us/unreal-engine/gameplay-ability-system-for-unreal-engine). 실패 후 환불 여부는 게임 규칙에 따른 설계 선택입니다.

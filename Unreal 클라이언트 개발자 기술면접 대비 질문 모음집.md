# Unreal 클라이언트 개발자 기술면접 대비 질문 모음집

> **이해·암기 복습:** [전체 92개 주제의 쉬운 설명과 접힌 답변](./README.md#recall-guide)을 먼저 활용하세요. 질문집에서는 답을 보기 전에 뜻·예시·주의할 조건을 자기 말로 설명하고, 실습 질문은 예상 결과와 실제 확인 결과를 구분합니다.

[통합 README](./README.md#목차) · [원문 Unity 질문 모음집](./질문%20모음집.md) · [C++·Unreal 12차시 기초 실습](./README.md#study-questions)

원본의 **차시 → 주제 → 핵심·심화 질문 → 실습·조사·경험 발표 → 읽기 자료** 구성을 참고해 이 포크에서 새로 작성했습니다. **10차시·20개 주제·160개 본문 문항**이며 꼬리 질문은 문항 수에 포함하지 않습니다. 원저자 안단태·이주연의 원문과 이 확장 질문집의 작성·검토는 구분합니다. [원본 저장소](https://github.com/salt26/game-developer-interview)의 저작자 표시와 [CC BY-NC 4.0](./LICENSE)을 유지합니다.

대상은 Unreal C++ 게임플레이·클라이언트 개발 면접을 준비하는 사람입니다. 신입·주니어는 💯 질문과 설명부터, 경험자는 실패 시나리오·실습·설계 선택까지 준비합니다. GAS·고급 네트워크·렌더링 심화는 지원 직무에 맞춰 선택합니다.

- 💯: 이 질문집의 기초 우선순위입니다. 실제 출제 확률을 뜻하지 않습니다.
- 표시 없는 문항: 답변을 준비할 권장 질문입니다.
- 😎: 심화 질문입니다. 엔진 버전과 설계 전제를 함께 답합니다.
- 🔨: 직접 구현·관찰해 발표하는 실습입니다.
- 📊: 공식 문서·소스·측정으로 검토하는 조사입니다.
- 🗽: 자신의 경험 또는 가정한 설계 판단을 말하는 질문입니다.

질문은 공개 자료를 검토해 새로 만든 학습용 문항이며 특정 기업의 기출이라고 주장하지 않습니다. 공개 후기의 역할은 주제 선정 참고이고, 기술 근거는 공식 문서입니다. [원본 비교·조사 출처·선정 기준](#research)을 확인할 수 있습니다. 조사일은 **2026-09-11**입니다.

## 공부하는 방법

먼저 각 절의 1·2번에 짧게 답하고, 3·4번에서 조건을 바꾸어 설명합니다. 막히면 그 절의 **답변 점검과 해설**을 펼치고 README 또는 공식 읽기 자료로 돌아갑니다. 이후 실습으로 확인하고 자신의 결과를 말합니다. 한 차시의 두 주제를 하루에 모두 끝낼 필요는 없습니다.

API 이름이 기억나지 않으면 역할·입력·대상·수명부터 설명하고 공식 서명을 확인해 이어갑니다. 실습 기록에는 엔진 버전·빌드·실행 모드·재현 순서·예상·실제 결과를 남깁니다. 이 문서의 실습은 수행 요구사항이며 엔진에서 이미 검증된 결과가 아닙니다.

**읽기 순서:** 처음 배우면 1→2→3차시를 먼저 진행하고 이후 관심 주제를 고릅니다. 네트워크는 프레임워크 이후, GAS는 이벤트·네트워크 이후에 읽습니다. C++ 문법·자료 구조·운영체제·알고리즘은 Unreal API 공부와 별도로 계속 준비합니다.

<a id="toc"></a>
## 목차

* [1차시 — C++와 엔진 객체 수명](#day-01)
  * [C++ 수명·소유권·다형성](#cpp-ownership)
  * [UObject·GC·Actor 종료](#uobject-gc)
* [2차시 — 리플렉션과 게임 프레임워크](#day-02)
  * [리플렉션·CDO·Blueprint·빌드](#reflection-build)
  * [Gameplay Framework·재스폰·맵 전환](#framework)
* [3차시 — 자료 구조와 비동기 흐름](#day-03)
  * [TArray·컨테이너·문자열](#containers)
  * [Delegate·Timer·비동기·스레드](#async-events)
* [4차시 — 메모리와 렌더링 성능](#day-04)
  * [메모리·할당·프로파일링](#memory-profile)
  * [렌더링·CPU/GPU 병목](#render-profile)
* [5차시 — 입력·수학과 전투 판정](#day-05)
  * [Enhanced Input·이동·게임 수학](#input-math)
  * [충돌·Trace·공격 판정](#collision-combat)
* [6차시 — 애니메이션과 UI](#day-06)
  * [Animation Blueprint·Montage·종료](#animation)
  * [UMG·UI 수명·갱신 비용](#ui)
* [7차시 — 복제와 네트워크 반응성](#day-07)
  * [네트워크 상태·RPC·소유권](#replication)
  * [이동 예측·보정·서버 판정](#prediction)
* [8차시 — 에셋과 저장·배포](#day-08)
  * [에셋 참조·비동기 로딩·Asset Manager](#assets)
  * [저장·로드·쿠킹·패키징](#save-package)
* [9차시 — AI와 구조·테스트](#day-09)
  * [AIController·Behavior Tree·Perception·Navigation](#ai)
  * [Component·Subsystem·설계·검증](#architecture-test)
* [10차시 — GAS와 종합 면접](#day-10)
  * [GAS·GameplayTag·비용·취소](#gas)
  * [종합 디버깅·코드 리뷰·경험 발표](#interview-case)
* [원본 비교·조사 출처·선정 기준](#research)
* [면접 전 최종 점검](#final-check)


<a id="day-01"></a>
## 1차시 — C++와 엔진 객체 수명


<a id="cpp-ownership"></a>
### C++ 수명·소유권·다형성

포인터·참조·스코프를 먼저 설명합니다. 일반 C++의 수명 규칙을 다음 절의 Unreal GC와 비교합니다.

읽기: [수명과 저장 공간](./README.md#cpp-01) · [RAII](./README.md#cpp-03) · [복사·이동](./README.md#cpp-05) · [가상 소멸자](./README.md#cpp-07)

<a id="ueq-01-01"></a>
* **UEQ-01-01** 💯 객체의 수명과 저장 공간은 어떻게 다른가요? 지역 vector 객체와 그 원소를 나누어 설명하세요.

<a id="ueq-01-02"></a>
* **UEQ-01-02** 💯 RAII는 조기 반환이나 예외 경로에서 어떤 실수를 줄이나요? 파일과 잠금 중 하나로 설명하세요.

<a id="ueq-01-03"></a>
* **UEQ-01-03** unique_ptr·shared_ptr·weak_ptr를 장비 소유자와 UI 관찰자에 배치한다면 어떻게 고르겠습니까?

<a id="ueq-01-04"></a>
* **UEQ-01-04** 작업 큐에 this를 캡처한 람다를 넣고 화면을 닫았습니다. 언제 위험하며 무엇을 보관하거나 취소해야 하나요?

<a id="ueq-01-05"></a>
* **UEQ-01-05** 😎 기반 포인터 삭제, 슬라이싱, std::move를 각각 설명하고, 이동 생성자를 없앤 경우와 삭제 선언한 경우를 비교하세요.

<a id="ueq-01-06"></a>
* **UEQ-01-06** 🔨 생성·복사·이동·소멸 로그를 출력하는 일반 C++ 타입으로 스코프 종료와 소유권 이전을 보여주세요.

<a id="ueq-01-07"></a>
* **UEQ-01-07** 📊 “지역 변수는 항상 스택에 있고, 이동 후 객체는 항상 비어 있다”는 설명을 표준 보장과 구현 선택으로 나누어 검토하세요.

<a id="ueq-01-08"></a>
* **UEQ-01-08** 🗽 직접 작성한 코드에서 소유권을 명확히 하려고 바꾼 부분을 설명하세요. 경험이 없다면 가상의 개선안이라고 밝히세요.

**꼬리 질문:** 3번에서 양방향 shared_ptr 관계가 생기면 외부 참조를 지워도 정리될까요? 4번에서 주소를 값 캡처하면 문제가 해결되나요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* 수명은 사용 가능한 기간이고 저장 공간 확보만으로 모든 객체가 초기화되지는 않습니다.

* RAII는 완성된 관리 객체의 소멸 경로에 해제를 모읍니다. 강제 프로세스 종료까지 보장하지는 않습니다.

* 관찰과 소유를 구분합니다. 값으로 복사한 포인터도 대상 수명을 연장하지 않습니다.

* 다형적 삭제 계약, 복사로 잘린 파생 부분, 이동 후보 선택은 각각 다른 문제입니다. 실행 한 번의 우연한 결과를 언어 보장으로 삼지 않습니다.

**실습 발표에서 확인할 것:** 객체 ID별 생성·소멸을 연결하고, 반환된 값 객체와 반환된 지역 주소의 차이를 설명합니다. 수명이 끝난 주소를 역참조하는 실습은 하지 않습니다.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Microsoft — C++ 수명과 RAII](https://learn.microsoft.com/en-us/cpp/cpp/object-lifetime-and-resource-management-modern-cpp?view=msvc-170)

[↑ 목차](#toc)

---


<a id="uobject-gc"></a>
### UObject·GC·Actor 종료

앞 절의 일반 C++ 소유권을 먼저 봅니다. Unity fake null을 Unreal의 규칙으로 그대로 옮기지 않습니다.

읽기: [일반 C++과 UObject](./README.md#ue-01) · [포인터 선택](./README.md#ue-02) · [Destroy와 EndPlay](./README.md#ue-04)

<a id="ueq-02-01"></a>
* **UEQ-02-01** 💯 일반 C++ 객체, UObject, Actor를 어떤 경로로 생성하고 종료하나요?

<a id="ueq-02-02"></a>
* **UEQ-02-02** 💯 UPROPERTY의 TObjectPtr, TWeakObjectPtr, TSoftObjectPtr는 각각 무엇을 유지하거나 관찰하나요?

<a id="ueq-02-03"></a>
* **UEQ-02-03** NewObject에 Outer를 넘기는 것과 GC가 추적하는 참조를 보관하는 것은 같은 의미인가요?

<a id="ueq-02-04"></a>
* **UEQ-02-04** Destroy 호출 뒤 포인터가 null이 아닙니다. 곧바로 공격 대상으로 사용해도 될까요?

<a id="ueq-02-05"></a>
* **UEQ-02-05** 😎 일반 C++ 관리자에서 UObject를 유지해야 할 때 선택할 수 있는 엔진 참조 수단과 정리 책임을 조사하세요.

<a id="ueq-02-06"></a>
* **UEQ-02-06** 🔨 연습 Actor에서 명시적 파괴·맵 전환·PIE 종료의 EndPlay 이유를 각각 기록해 보여주세요.

<a id="ueq-02-07"></a>
* **UEQ-02-07** 📊 추적되는 강한 참조와 약한 참조의 차이를 문서와 통제된 GC 실험으로 확인하세요.

<a id="ueq-02-08"></a>
* **UEQ-02-08** 🗽 대상이 먼저 사라지는 UI·타이머·비동기 작업을 어떻게 처리했는지 설명하세요.

**꼬리 질문:** 2번에서 Soft 경로는 유효한데 Get이 null일 수 있나요? 4번에서 강한 참조를 추가하면 명시적 Actor 파괴가 막힐까요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* 생성 목적에 따라 NewObject·SpawnActor·CreateDefaultSubobject를 구분합니다.

* TObjectPtr 타입명만으로 모든 저장 위치가 추적되지는 않습니다. 추적 경로와 유지하는 객체의 도달 가능성을 봅니다.

* Weak는 관찰, Soft는 에셋 경로이며 로드와 로드 후 유지는 별도입니다.

* 플레이 종료와 메모리 회수는 다릅니다. EndPlay에서 외부 등록을 정리하고 늦은 결과는 적용 전에 유효성을 검사합니다.

**실습 발표에서 확인할 것:** 종료 이유·객체 식별자·콜백 발생 시점을 기록합니다. 파괴된 대상에 접근하지 않았다는 로그와 설계 근거를 함께 제시합니다.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Epic — Object Pointers](https://dev.epicgames.com/documentation/en-us/unreal-engine/object-pointers-in-unreal-engine) · [Epic — Actor Lifecycle](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-actor-lifecycle)

[↑ 목차](#toc)

---


<a id="day-02"></a>
## 2차시 — 리플렉션과 게임 프레임워크


<a id="reflection-build"></a>
### 리플렉션·CDO·Blueprint·빌드

일반 C++ 선언과 정의를 구분한 뒤 UHT·UBT·컴파일·링크의 역할을 나눕니다.

읽기: [템플릿과 빌드](./README.md#cpp-13) · [리플렉션·CDO](./README.md#ue-11)

<a id="ueq-03-01"></a>
* **UEQ-03-01** 💯 UCLASS·UPROPERTY·UFUNCTION은 엔진이 C++ 선언을 다루는 방식에 어떤 역할을 하나요?

<a id="ueq-03-02"></a>
* **UEQ-03-02** 💯 CDO, Blueprint 기본값, 배치 인스턴스의 재정의 값을 구분해 설명하세요.

<a id="ueq-03-03"></a>
* **UEQ-03-03** EditDefaultsOnly와 BlueprintReadOnly는 같은 접근을 제한하나요? C++와 Blueprint 책임은 어떻게 나누겠습니까?

<a id="ueq-03-04"></a>
* **UEQ-03-04** 헤더를 포함했는데 다른 모듈의 함수를 연결하지 못합니다. include와 Build.cs 의존성을 어떻게 나눠 확인하나요?

<a id="ueq-03-05"></a>
* **UEQ-03-05** 😎 모듈의 Public/Private 의존성, 공개 헤더, API 내보내기 지정자가 외부 사용에 주는 영향을 설명하세요.

<a id="ueq-03-06"></a>
* **UEQ-03-06** 🔨 연습 Blueprint에 값을 재정의하고 C++ 기본값 변경 전후를 새 Blueprint와 비교해 보여주세요.

<a id="ueq-03-07"></a>
* **UEQ-03-07** 📊 리플렉션 코드 생성 실패·일반 컴파일 실패·링크 실패의 진단을 각각 찾아 원인 단계로 분류하세요.

<a id="ueq-03-08"></a>
* **UEQ-03-08** 🗽 디자이너가 바꿀 값과 프로그래머가 보장할 규칙을 구분한 경험 또는 설계안을 설명하세요.

**꼬리 질문:** 4번의 해결로 모든 모듈을 Public 의존성에 넣으면 어떤 결합이 생기나요? CDO 기본값 변경을 기존 에셋의 재정의 초기화와 혼동하지 않았나요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* 엔진 메타데이터 참여와 일반 C++ 타입 선언을 구분합니다. 지정자에 따라 편집·Blueprint·직렬화 참여 방식이 달라집니다.

* 저장된 재정의는 C++ 기본값과 다른 층입니다. 수정한 클래스와 보고 있는 인스턴스부터 확인합니다.

* 헤더는 선언을 보이고 모듈 의존성은 빌드 관계를 표현합니다. 정의 누락·라이브러리 연결·내보내기를 오류 단계에 따라 좁힙니다.

* 공개 인터페이스에서 필요한 의존성과 구현 안에서만 필요한 의존성을 구분해 불필요한 전파를 줄입니다.

**실습 발표에서 확인할 것:** 원본 에셋 대신 연습용을 사용합니다. 코드값·Blueprint값·인스턴스값을 변경 전후 표로 남겨 재정의가 원인인지 설명합니다.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Epic — Object Handling](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-object-handling-in-unreal-engine) · [Epic — Modules](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-modules)

[↑ 목차](#toc)

---


<a id="framework"></a>
### Gameplay Framework·재스폰·맵 전환

Actor 종료를 알고 진행합니다. 서버·소유 클라이언트·다른 클라이언트를 구분해 그립니다.

읽기: [게임 전체 상태](./README.md#ue-05) · [플레이어와 Pawn](./README.md#ue-06) · [Component와 Subsystem](./README.md#ue-12)

<a id="ueq-04-01"></a>
* **UEQ-04-01** 💯 GameMode·GameState·GameInstance에 각각 어떤 데이터를 두겠습니까?

<a id="ueq-04-02"></a>
* **UEQ-04-02** 💯 PlayerController·Pawn·Character·PlayerState를 나누는 이유는 무엇인가요?

<a id="ueq-04-03"></a>
* **UEQ-04-03** 점수·현재 체력·로컬 음량·라운드 종료 판정을 어디에 둘지 수명과 복제 요구로 설명하세요.

<a id="ueq-04-04"></a>
* **UEQ-04-04** 재스폰하면 UI가 이전 Pawn을 계속 바라봅니다. 참조 교체와 이벤트 재구독을 어디에서 연결하겠습니까?

<a id="ueq-04-05"></a>
* **UEQ-04-05** 😎 일반 맵 이동과 seamless travel에서 객체 유지·상태 복사를 확인할 항목은 무엇인가요?

<a id="ueq-04-06"></a>
* **UEQ-04-06** 🔨 Pawn 교체 전후 식별자와 점수·체력을 기록하고 남길 값과 초기화할 값을 보여주세요.

<a id="ueq-04-07"></a>
* **UEQ-04-07** 📊 서버·소유 클라이언트·관찰 클라이언트에서 각 프레임워크 객체의 존재 여부를 표로 조사하세요.

<a id="ueq-04-08"></a>
* **UEQ-04-08** 🗽 특정 데이터를 GameInstance에 넣었다가 수명이 맞지 않았던 사례나 대안을 설명하세요.

**꼬리 질문:** 클라이언트 UI가 GameMode를 조회해도 될까요? GameInstance에 저장하면 재실행 후에도 남거나 서버와 자동 공유되나요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* 규칙 판단, 공유할 경기 상태, 실행 인스턴스 수명을 나눕니다. GameInstance는 자동 복제·디스크 저장이 아닙니다.

* 조종 주체와 몸을 분리하면 Pawn 교체를 설명할 수 있습니다. 체력의 위치도 게임 규칙에 따른 선택입니다.

* 다른 플레이어의 Controller가 모든 클라이언트에 있다는 가정을 버리고 공개 상태 경로를 사용합니다.

* 맵 전환 유지 여부는 이동 방식과 실제 초기화·복사 경로로 확인합니다. 재스폰 검증을 모든 여행의 검증으로 확장하지 않습니다.

**실습 발표에서 확인할 것:** 새 Pawn으로 UI가 바뀌고 이전 소스의 콜백이 남지 않는지 확인합니다. 점수 유지와 체력 초기화를 각각 검증합니다.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Epic — Gameplay Framework](https://dev.epicgames.com/documentation/en-us/unreal-engine/gameplay-framework-in-unreal-engine)

[↑ 목차](#toc)

---


<a id="day-03"></a>
## 3차시 — 자료 구조와 비동기 흐름


<a id="containers"></a>
### TArray·컨테이너·문자열

C++ 복사·이동과 참조 무효화를 먼저 봅니다. 엔진 타입을 STL과 이름만 대응시키지 않습니다.

읽기: [vector와 무효화](./README.md#cpp-09) · [컨테이너와 캐시](./README.md#cpp-14)

<a id="ueq-05-01"></a>
* **UEQ-05-01** 💯 TArray에서 Num과 확보된 용량, Reserve와 원소 추가를 구분해 설명하세요.

<a id="ueq-05-02"></a>
* **UEQ-05-02** 💯 FString·FName·FText를 편집 가능한 문자열·식별 이름·사용자 표시 문구에 어떻게 고르나요?

<a id="ueq-05-03"></a>
* **UEQ-05-03** 전체 적 순회와 아이템 ID 조회에는 각각 어떤 컨테이너가 적합한가요? 순서와 갱신 비용도 고려하세요.

<a id="ueq-05-04"></a>
* **UEQ-05-04** 배열 원소 주소를 저장한 뒤 추가·삭제하면 무엇이 깨질 수 있나요? 인덱스로 바꾸면 충분한가요?

<a id="ueq-05-05"></a>
* **UEQ-05-05** 😎 순서 보존 삭제와 순서를 바꿀 수 있는 삭제를 비교하고, 타입·allocator에 따른 이동 비용을 설명하세요.

<a id="ueq-05-06"></a>
* **UEQ-05-06** 🔨 삭제 전후 아이템 ID와 선택 인덱스를 기록해 같은 인덱스가 다른 아이템을 뜻하는 상황을 보여주세요.

<a id="ueq-05-07"></a>
* **UEQ-05-07** 📊 같은 입력으로 구성 비용·성공 조회·실패 조회·전체 순회 비용을 나눠 비교하세요.

<a id="ueq-05-08"></a>
* **UEQ-05-08** 🗽 문자열·컨테이너 선택을 가독성·메모리·현지화 요구 때문에 바꾼 사례를 설명하세요.

**꼬리 질문:** FText를 문자열로 바꿨다가 복원하면 현지화 정보를 그대로 보존한다고 가정할 수 있나요? 작은 배열에서도 해시 조회가 항상 빠를까요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* 자리를 예약하는 것과 원소 생성은 다릅니다. 변경 연산별 주소·반복자 유효성을 확인합니다.

* FString은 문자열 조작, FName은 이름 식별, FText는 현지화 가능한 표시 텍스트라는 용도를 출발점으로 봅니다.

* 주소 안정성과 논리적 ID는 다른 문제입니다. 삭제·정렬·ID 재사용 조건을 설명합니다.

* 빅오만으로 실제 시간 우위를 단정하지 않습니다. 동일 데이터와 빌드에서 비용을 분리해 측정합니다.

**실습 발표에서 확인할 것:** 결과 ID·연산 횟수·시간의 의미를 구분합니다. 루프 안 로그와 데이터 구성 비용이 측정을 덮지 않게 합니다.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Epic — TArray](https://dev.epicgames.com/documentation/en-us/unreal-engine/array-containers-in-unreal-engine) · [Epic — String Handling](https://dev.epicgames.com/documentation/en-us/unreal-engine/string-handling-in-unreal-engine)

[↑ 목차](#toc)

---


<a id="async-events"></a>
### Delegate·Timer·비동기·스레드

객체 수명을 먼저 봅니다. 콜백이 실행되는 시점과 실행 스레드를 별도 질문으로 다룹니다.

읽기: [람다와 수명](./README.md#cpp-10) · [동시성](./README.md#cpp-15) · [Delegate와 종료](./README.md#ue-13)

<a id="ueq-06-01"></a>
* **UEQ-06-01** 💯 Delegate에 함수를 등록하는 것과 Broadcast로 호출하는 것은 어떻게 다른가요?

<a id="ueq-06-02"></a>
* **UEQ-06-02** 💯 Tick·Timer·이벤트·비동기 작업은 각각 어떤 요구에 맞나요?

<a id="ueq-06-03"></a>
* **UEQ-06-03** AddUObject와 this를 캡처하는 일반 람다의 수명 처리는 어떻게 다른가요?

<a id="ueq-06-04"></a>
* **UEQ-06-04** UI를 세 번 열면 체력 갱신이 세 번 호출됩니다. 등록 소스·핸들·해제를 어떻게 추적하나요?

<a id="ueq-06-05"></a>
* **UEQ-06-05** 😎 계산을 작업 스레드로 옮길 때 데이터 경쟁·게임 스레드 적용·대기 의존성을 어떻게 설계하나요?

<a id="ueq-06-06"></a>
* **UEQ-06-06** 🔨 구독·해제·Broadcast 로그를 넣어 열기·닫기·소스 파괴 이후의 콜백 수를 보여주세요.

<a id="ueq-06-07"></a>
* **UEQ-06-07** 📊 “비동기이면 항상 별도 스레드다”와 “atomic이면 여러 단계의 규칙도 안전하다”를 반례로 검토하세요.

<a id="ueq-06-08"></a>
* **UEQ-06-08** 🗽 취소된 요청의 결과가 늦게 도착한 문제를 해결한 경험 또는 설계를 설명하세요.

**꼬리 질문:** 멀티캐스트 구독 순서에 로직이 의존하면 괜찮을까요? 수명이 안전한 결과라도 이전 요청의 결과이면 적용해도 될까요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* 등록은 호출 목록을 구성하고 Broadcast가 실행합니다. 실행 순서에 의존하지 않습니다.

* 객체 기반 약한 바인딩도 숨겨진 UI의 중복 등록을 자동 제거하는 규칙은 아닙니다.

* 작업 입력·결과 전달·취소를 나누고 대상 유효성과 최신 요청 여부를 함께 확인합니다.

* 데이터 수명, 스레드 안전, 완료 대기는 서로 다른 조건입니다. 작업 간 의존성의 순환도 확인합니다.

**실습 발표에서 확인할 것:** 한 번의 상태 변경에 필요한 갱신이 한 번 발생하고, 종료 뒤 옛 소스에서 갱신되지 않는지 확인합니다.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Epic — Multicast Delegates](https://dev.epicgames.com/documentation/en-us/unreal-engine/multicast-delegates-in-unreal-engine) · [Epic — Tasks System](https://dev.epicgames.com/documentation/unreal-engine/tasks-systems-in-unreal-engine?lang=en-US)

[↑ 목차](#toc)

---


<a id="day-04"></a>
## 4차시 — 메모리와 렌더링 성능


<a id="memory-profile"></a>
### 메모리·할당·프로파일링

수명과 컨테이너를 먼저 봅니다. 수치가 늘어난 이유를 누수·캐시·풀로 나눠 가설을 세웁니다.

읽기: [RAII](./README.md#cpp-03) · [성능 측정](./README.md#profiling)

<a id="ueq-07-01"></a>
* **UEQ-07-01** 💯 누수, 의도적으로 유지한 캐시, allocator가 보유한 공간을 어떻게 구분하나요?

<a id="ueq-07-02"></a>
* **UEQ-07-02** 💯 매 프레임 동적 할당이 문제가 되는 조건과 조사할 지표를 설명하세요.

<a id="ueq-07-03"></a>
* **UEQ-07-03** 오브젝트 풀은 생성 비용을 줄이는 대신 어떤 메모리·상태 초기화 책임을 늘리나요?

<a id="ueq-07-04"></a>
* **UEQ-07-04** 인벤토리를 열고 닫을 때마다 메모리가 증가합니다. 어느 시점과 할당 경로를 비교하겠습니까?

<a id="ueq-07-05"></a>
* **UEQ-07-05** 😎 메모리 단편화·캐시 지역성·할당 수의 감소를 서로 다른 성능 요인으로 설명하세요.

<a id="ueq-07-06"></a>
* **UEQ-07-06** 🔨 Memory Insights에서 반복 작업 전후의 살아 있는 할당과 호출 경로를 비교해 보여주세요.

<a id="ueq-07-07"></a>
* **UEQ-07-07** 📊 한 가지 할당 감소 변경의 전후를 동일 장면·빌드·반복 횟수로 측정하세요.

<a id="ueq-07-08"></a>
* **UEQ-07-08** 🗽 메모리 증가를 감수해 지연을 줄인 선택이 있다면 목표와 측정 결과를 설명하세요.

**꼬리 질문:** 프로세스 메모리가 즉시 줄지 않았다는 사실만으로 누수라고 할 수 있나요? 풀에 돌려준 객체의 이벤트와 타이머는 누가 초기화하나요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* 총량 그래프와 살아 있는 할당의 수명·호출 경로를 함께 봅니다.

* 안정화되는 캐시인지 반복마다 누적되는 보관인지 구분하려면 여러 구간을 관찰해야 합니다.

* 풀은 보관 비용과 재사용 초기화가 필요합니다. 이전 사용자의 상태·구독·타이머가 남는지도 검사합니다.

* 기록 설정과 심벌·호출 스택 수집 조건을 명시해야 다른 사람이 같은 측정을 재현할 수 있습니다.

**실습 발표에서 확인할 것:** 측정 시작·반복·정리 후 시점을 고정해 기록합니다. 원인 가설과 실제 확인한 할당을 따로 적습니다.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Epic — Memory Insights](https://dev.epicgames.com/documentation/en-us/unreal-engine/memory-insights-in-unreal-engine)

[↑ 목차](#toc)

---


<a id="render-profile"></a>
### 렌더링·CPU/GPU 병목

벡터·행렬 기초와 프레임 시간 개념을 확인합니다. 고급 렌더링 기능은 지원 직무에 맞춰 확장합니다.

읽기: [렌더링과 병목](./README.md#rendering) · [측정과 패키징](./README.md#profiling)

<a id="ueq-08-01"></a>
* **UEQ-08-01** 💯 월드의 객체가 화면에 그려지기까지 CPU와 GPU가 담당하는 일을 설명하세요.

<a id="ueq-08-02"></a>
* **UEQ-08-02** 💯 드로우 콜을 줄였는데 프레임 시간이 그대로인 이유를 어떻게 조사하나요?

<a id="ueq-08-03"></a>
* **UEQ-08-03** 평균 FPS, 프레임 시간, 순간적인 hitch는 각각 무엇을 보여주나요?

<a id="ueq-08-04"></a>
* **UEQ-08-04** 해상도를 낮춰도 느립니다. CPU 병목 외에 어떤 조건을 확인해야 하나요?

<a id="ueq-08-05"></a>
* **UEQ-08-05** 😎 LOD·인스턴싱·컬링이 각각 줄이는 일과 비용·화질의 교환 관계를 설명하세요.

<a id="ueq-08-06"></a>
* **UEQ-08-06** 🔨 같은 장면에서 CPU/GPU 시간을 기록하고 한 가지 렌더링 조건만 바꿔 비교하세요.

<a id="ueq-08-07"></a>
* **UEQ-08-07** 📊 지원 프로젝트의 Nanite·Lumen 등 사용 기능 하나를 골라 해당 버전·플랫폼의 제약과 측정 지표를 조사하세요.

<a id="ueq-08-08"></a>
* **UEQ-08-08** 🗽 화질과 성능의 절충을 제안했다면 목표 기기·예산·결과와 남은 문제를 설명하세요.

**꼬리 질문:** Game·Render·GPU 시간을 단순 합산해 FPS를 구해도 될까요? 병목을 옮긴 변경과 전체 시간을 줄인 변경을 구분했나요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* 파이프라인의 병렬성과 대기 때문에 비용 숫자를 단순 합산하지 않습니다.

* 픽셀·지오메트리·드로우 제출·게임 로직·프레임 제한 등 가능한 원인을 분리합니다.

* 한 조건씩 바꾸어 어떤 가설을 지지하는지 설명합니다. 품질 변화도 결과에 포함합니다.

* 기능을 켰다는 사실은 최적화 증거가 아닙니다. 대상 빌드에서 측정한 변화가 필요합니다.

**실습 발표에서 확인할 것:** 장면·카메라·해상도·빌드·장치를 기록하고 전후 트레이스를 보관합니다. 측정하지 않은 수치는 예상으로 표시합니다.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Epic — Performance Profiling](https://dev.epicgames.com/documentation/en-us/unreal-engine/introduction-to-performance-profiling-and-configuration-in-unreal-engine) · [Epic — Unreal Insights](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-insights-in-unreal-engine)

[↑ 목차](#toc)

---


<a id="day-05"></a>
## 5차시 — 입력·수학과 전투 판정


<a id="input-math"></a>
### Enhanced Input·이동·게임 수학

입력값·로컬 방향·월드 방향을 구별합니다. 수평 이동과 자유 비행은 다른 요구입니다.

읽기: [입력에서 이동](./README.md#ue-14) · [위치·방향·회전](./README.md#math)

<a id="ueq-09-01"></a>
* **UEQ-09-01** 💯 Input Action·Mapping Context·Modifier·Trigger의 역할을 설명하세요.

<a id="ueq-09-02"></a>
* **UEQ-09-02** 💯 카메라가 돌아가도 W가 카메라 앞을 향하게 하려면 어떤 값과 좌표계를 사용하나요?

<a id="ueq-09-03"></a>
* **UEQ-09-03** 위치와 방향 변환은 어떻게 다른가요? 내적·외적을 실제 게임 판정에 연결해 설명하세요.

<a id="ueq-09-04"></a>
* **UEQ-09-04** 입력 함수는 호출되는데 Pawn이 안 움직입니다. 입력 소비와 실제 이동을 어떻게 나눠 보나요?

<a id="ueq-09-05"></a>
* **UEQ-09-05** 😎 회전 보간·회전 합성 순서·쿼터니언의 용도를 설명하고 정규화할 벡터가 0인 조건을 처리하세요.

<a id="ueq-09-06"></a>
* **UEQ-09-06** 🔨 카메라 방향을 바꾸며 입력값과 월드 이동 방향을 표시하는 실습을 보여주세요.

<a id="ueq-09-07"></a>
* **UEQ-09-07** 📊 UI와 게임플레이 Context가 같은 키를 사용할 때 우선순위·트리거·소비 설정을 조사하세요.

<a id="ueq-09-08"></a>
* **UEQ-09-08** 🗽 조작감 요구 때문에 이동·카메라 처리를 바꾼 경험 또는 제안을 설명하세요.

**꼬리 질문:** Triggered는 매 프레임 무조건 호출되나요? 전체 시야각과 반각, 방향 판정과 거리 제한을 구분했나요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* 키 매핑, 값 가공, 발동 조건, 콜백, 이동 의도 소비를 순서대로 분리합니다.

* 방향을 같은 공간에서 계산하고 지상 이동에 사용할 회전 축을 정합니다.

* 내적 각도 조건과 거리는 별도입니다. 0 벡터의 방향은 예외 정책이 필요합니다.

* 입력이 없으면 Context·바인딩을, 값이 있으면 방향·이동 구현·충돌을 확인합니다.

**실습 발표에서 확인할 것:** 카메라 회전 전후의 입력과 이동 벡터를 비교합니다. 단순 이동 성공뿐 아니라 축·좌표계가 맞는 이유를 설명합니다.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Epic — Enhanced Input](https://dev.epicgames.com/documentation/en-us/unreal-engine/enhanced-input-in-unreal-engine)

[↑ 목차](#toc)

---


<a id="collision-combat"></a>
### 충돌·Trace·공격 판정

앞 절의 공간 계산을 알고 진행합니다. 한 번의 공격을 어떤 기간으로 정의할지 먼저 정합니다.

읽기: [Trace·Sweep](./README.md#ue-15) · [서버 공격 판정](./README.md#ue-08)

<a id="ueq-10-01"></a>
* **UEQ-10-01** 💯 Overlap 이벤트·Hit 이벤트·Line Trace·Sweep을 어떤 상황에 고르나요?

<a id="ueq-10-02"></a>
* **UEQ-10-02** 💯 채널·오브젝트 타입·응답 설정·무시 대상은 질의 결과에 어떻게 관여하나요?

<a id="ueq-10-03"></a>
* **UEQ-10-03** 화면에 디버그 선은 보이지만 피격이 검출되지 않습니다. 무엇부터 확인하나요?

<a id="ueq-10-04"></a>
* **UEQ-10-04** 빠른 검이 프레임 사이에서 적을 지나칩니다. 샘플링 위치·주기·형상을 어떻게 바꾸겠습니까?

<a id="ueq-10-05"></a>
* **UEQ-10-05** 😎 이전·현재 위치의 직선 Sweep이 회전하는 검의 전체 궤적을 완전히 덮는지 검토하세요.

<a id="ueq-10-06"></a>
* **UEQ-10-06** 🔨 한 공격의 여러 샘플이 같은 적을 맞혀도 피해가 한 번만 적용되도록 구현하고 보여주세요.

<a id="ueq-10-07"></a>
* **UEQ-10-07** 📊 채널 기반 다중 질의의 blocking/overlap 결과와 종료 조건을 사용 API 문서에서 확인하세요.

<a id="ueq-10-08"></a>
* **UEQ-10-08** 🗽 정확도와 질의 비용 사이에서 선택한 기준을 실제 경험 또는 설계 가정으로 설명하세요.

**꼬리 질문:** 공격이 취소되면 이미 맞힌 대상 목록과 판정 활성 상태를 언제 정리하나요? 다음 공격에서 같은 적을 다시 맞힐 수 있나요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* 이벤트가 필요한지 특정 시점의 질의가 필요한지부터 선택합니다.

* 모양·채널·응답·무시 대상을 함께 확인합니다. 디버그 그림은 설정의 정답을 보장하지 않습니다.

* 공격별 대상 기록으로 중복을 제어하고 시작·종료·취소 수명을 맞춥니다.

* 샘플 증가는 오차와 비용을 함께 바꿉니다. 다중 질의가 모든 관통 결과를 무조건 반환한다고 가정하지 않습니다.

**실습 발표에서 확인할 것:** 1개 대상·여러 대상·공격 중단·다음 공격 재타격을 각각 확인합니다. 대상별 공격 ID와 피해 적용 횟수를 남깁니다.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Epic — Collision Overview](https://dev.epicgames.com/documentation/unreal-engine/collision-in-unreal-engine---overview?lang=en-US)

[↑ 목차](#toc)

---


<a id="day-06"></a>
## 6차시 — 애니메이션과 UI


<a id="animation"></a>
### Animation Blueprint·Montage·종료

공격 판정 기간과 이벤트를 알고 진행합니다. 이동 상태와 애니메이션 표현을 분리합니다.

읽기: [Montage·Notify·Root Motion](./README.md#ue-16)

<a id="ueq-11-01"></a>
* **UEQ-11-01** 💯 상태 머신과 Montage는 각각 어떤 애니메이션 요구에 적합한가요?

<a id="ueq-11-02"></a>
* **UEQ-11-02** 💯 Montage 재생 요청이 성공했는데 화면에 보이지 않을 때 Slot과 최종 포즈 경로를 어떻게 확인하나요?

<a id="ueq-11-03"></a>
* **UEQ-11-03** Notify를 이용한 판정 시점과 게임 규칙의 공격 성공 여부는 어떻게 다른가요?

<a id="ueq-11-04"></a>
* **UEQ-11-04** 피격으로 공격이 중단된 뒤 이동 제한이 풀리지 않습니다. 어떤 종료 경로를 조사하나요?

<a id="ueq-11-05"></a>
* **UEQ-11-05** 😎 Root Motion에서 메시 이동·캡슐 이동·네트워크 보정은 어떻게 나누어 검증하나요?

<a id="ueq-11-06"></a>
* **UEQ-11-06** 🔨 자연 종료와 강제 중단 양쪽에서 공격 판정·이동 제한·이벤트 구독이 정리되는 모습을 보여주세요.

<a id="ueq-11-07"></a>
* **UEQ-11-07** 📊 사용하는 Montage 종료·블렌드아웃 콜백의 호출 시점과 중단 인자를 해당 버전 API에서 확인하세요.

<a id="ueq-11-08"></a>
* **UEQ-11-08** 🗽 애니메이터와 타이밍·전환·루트 이동 문제를 협의한 경험 또는 협업 절차를 설명하세요.

**꼬리 질문:** 마지막 Notify가 오지 않으면 어떻게 되나요? 블렌드아웃과 종료 알림이 모두 오면 정리 함수를 두 번 호출해도 괜찮나요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* 재생 선택과 실제 포즈 출력 경로를 구분합니다.

* Notify는 시점 연결에 쓰되 필수 정리가 마지막 프레임 하나에만 의존하지 않게 합니다.

* 정상·취소·중단이 공통 정리 결과에 도달하고 중복 호출도 일관된 결과를 내도록 설계합니다.

* 사용 API와 재생 대상·콜백 연결을 명시합니다. 애니메이션 한 번 재생됨을 네트워크 검증으로 확대하지 않습니다.

**실습 발표에서 확인할 것:** 중단 전후 판정 활성과 이동 제한을 로그로 확인합니다. 다시 공격할 수 있는지까지 보여주세요.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Epic — Animation Montage](https://dev.epicgames.com/documentation/en-us/unreal-engine/animation-montage-in-unreal-engine)

[↑ 목차](#toc)

---


<a id="ui"></a>
### UMG·UI 수명·갱신 비용

Delegate와 로컬 플레이어 개념을 먼저 봅니다. 화면이 숨겨진 것과 객체가 소멸한 것은 다릅니다.

읽기: [UI 구독과 해제](./README.md#ue-example-13) · [플레이어 상태](./README.md#ue-06)

<a id="ueq-12-01"></a>
* **UEQ-12-01** 💯 UI가 게임 상태를 직접 소유하는 것과 상태를 읽어 표시하는 것은 어떤 차이가 있나요?

<a id="ueq-12-02"></a>
* **UEQ-12-02** 💯 체력 표시를 매 프레임 조회하는 방식과 상태 변경 이벤트로 갱신하는 방식을 비교하세요.

<a id="ueq-12-03"></a>
* **UEQ-12-03** 위젯을 열 때 초기 값 표시와 이후 변경 구독을 왜 함께 고려하나요?

<a id="ueq-12-04"></a>
* **UEQ-12-04** 재스폰 후 UI가 옛 Pawn의 체력을 표시합니다. 무엇을 교체하고 해제해야 하나요?

<a id="ueq-12-05"></a>
* **UEQ-12-05** 😎 Invalidation·Retainer·가상화된 목록이 해결하는 비용은 어떻게 다른지 조사하세요.

<a id="ueq-12-06"></a>
* **UEQ-12-06** 🔨 화면을 반복해 열고 닫고 재스폰한 뒤 구독 수와 표시값이 올바른지 보여주세요.

<a id="ueq-12-07"></a>
* **UEQ-12-07** 📊 많은 아이템을 표시할 때 위젯 구성·레이아웃·갱신 비용을 나누어 측정하세요.

<a id="ueq-12-08"></a>
* **UEQ-12-08** 🗽 UI의 빠른 재열기와 메모리 절약 사이에서 무엇을 선택했는지 설명하세요.

**꼬리 질문:** 숨겨진 위젯이면 모든 메모리·갱신 비용이 사라지나요? 이벤트로 바꾸었는데 최초 값이 비어 있는 이유는 무엇일까요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* 표시 상태의 소스와 UI 수명을 나누고 소스 교체 시 이전 구독을 해제합니다.

* 초기 갱신과 변경 이벤트를 연결합니다. 모든 UI가 항상 이벤트 하나로 대체되는 것은 아닙니다.

* Invalidation은 변경 없는 정보 재사용, Retainer는 렌더 결과 활용과 갱신 정책을 검토하는 출발점입니다.

* 목록의 전체 데이터 수와 실제 생성된 표시 항목 수를 구분합니다. 적용한 도구와 대상 버전에서 비용을 측정합니다.

**실습 발표에서 확인할 것:** 열기 직후·체력 변경·닫기·재열기·Pawn 교체의 표시와 콜백 수를 기록합니다. 성능 결과는 별도 트레이스로 증명합니다.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Epic — UMG Optimization](https://dev.epicgames.com/documentation/unreal-engine/optimization-guidelines-for-umg-in-unreal-engine?lang=en-US) · [Epic — Multicast Delegates](https://dev.epicgames.com/documentation/en-us/unreal-engine/multicast-delegates-in-unreal-engine)

[↑ 목차](#toc)

---


<a id="day-07"></a>
## 7차시 — 복제와 네트워크 반응성


<a id="replication"></a>
### 네트워크 상태·RPC·소유권

프레임워크의 객체 배치와 이벤트를 먼저 봅니다. 서버가 권위를 갖는 모델을 전제로 답합니다.

읽기: [변수 복제와 RPC](./README.md#ue-07) · [서버 권위](./README.md#ue-08)

<a id="ueq-13-01"></a>
* **UEQ-13-01** 💯 Listen Server·Dedicated Server·Client의 역할과 선택 기준을 설명하세요.

<a id="ueq-13-02"></a>
* **UEQ-13-02** 💯 변수 복제와 RPC는 어떤 요구를 각각 해결하나요?

<a id="ueq-13-03"></a>
* **UEQ-13-03** Server·Client·NetMulticast RPC는 호출 위치와 owning connection에 따라 어떻게 실행되나요?

<a id="ueq-13-04"></a>
* **UEQ-13-04** 문 열기를 Multicast로만 구현했더니 늦게 접속한 사용자는 닫힌 문을 봅니다. 어떻게 바꾸겠습니까?

<a id="ueq-13-05"></a>
* **UEQ-13-05** 😎 Reliable을 많이 쓰면 어떤 문제가 생길 수 있나요? 관련성·휴면·복제 빈도는 어떤 비용을 조절하나요?

<a id="ueq-13-06"></a>
* **UEQ-13-06** 🔨 서버·소유 클라이언트·관찰 클라이언트에서 문 상태와 요청 경로를 기록하고 늦은 접속도 보여주세요.

<a id="ueq-13-07"></a>
* **UEQ-13-07** 📊 RPC 실행 표를 문서와 대조하고 소유하지 않은 Actor에서 요청한 경우를 통제된 실습으로 확인하세요.

<a id="ueq-13-08"></a>
* **UEQ-13-08** 🗽 로컬에서는 정상인데 멀티플레이에서 실패한 문제를 어떻게 좁혔는지 설명하세요.

**꼬리 질문:** Reliable이면 늦은 접속에 과거 RPC가 재생되나요? C++ 서버에서 값을 대입할 때 OnRep도 자동 호출된다고 가정해도 되나요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* 현재 상태와 일회성 호출을 나눕니다. 호출 신뢰성은 과거 상태 저장 기능이 아닙니다.

* 소유권은 RPC 경로에 중요하며 호출이 도착해도 게임 규칙 검증은 별도입니다.

* C++ 대입 경로와 복제 수신 표시 경로를 분리하고 필요한 공통 갱신을 연결합니다.

* 변수 선언만으로 전체 복제가 완성되지 않습니다. Actor·속성 설정과 전달 대상 조건을 확인합니다.

**실습 발표에서 확인할 것:** 정상 요청·잘못된 소유 경로·늦은 접속을 나눠 기록합니다. 모든 클라이언트에 같은 객체들이 있다는 가정을 피합니다.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Epic — Networking Overview](https://dev.epicgames.com/documentation/en-us/unreal-engine/networking-overview-for-unreal-engine) · [Epic — Remote Procedure Calls](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-procedure-calls-in-unreal-engine)

[↑ 목차](#toc)

---


<a id="prediction"></a>
### 이동 예측·보정·서버 판정

앞 절의 상태 복제와 RPC를 먼저 이해합니다. 예측은 반응성과 권위 사이의 설계입니다.

읽기: [서버 공격 판정](./README.md#ue-08) · [입력에서 이동](./README.md#ue-14)

<a id="ueq-14-01"></a>
* **UEQ-14-01** 💯 클라이언트 입력 후 서버 응답을 기다리지 않고 움직이는 이유는 무엇인가요?

<a id="ueq-14-02"></a>
* **UEQ-14-02** 💯 예측·서버 확인·보정·스무딩을 서로 다른 과정으로 설명하세요.

<a id="ueq-14-03"></a>
* **UEQ-14-03** CharacterMovement를 사용하면서 위치를 별도 RPC로 계속 덮으면 어떤 충돌을 의심해야 하나요?

<a id="ueq-14-04"></a>
* **UEQ-14-04** 클라이언트는 적을 맞혔다고 보는데 서버는 빗나갔다고 판단합니다. 어떤 시점과 데이터를 비교하나요?

<a id="ueq-14-05"></a>
* **UEQ-14-05** 😎 지연 보상에서 과거 상태 보관·되감기 범위·악용 제한을 어떻게 정하겠습니까?

<a id="ueq-14-06"></a>
* **UEQ-14-06** 🔨 통제된 지연·손실 조건에서 이동과 서버 판정을 관찰하고 정상 연결과 비교해 보여주세요.

<a id="ueq-14-07"></a>
* **UEQ-14-07** 📊 커스텀 이동 상태를 추가할 때 엔진의 이동 저장·전송·재현 경로 중 무엇을 확장해야 하는지 조사하세요.

<a id="ueq-14-08"></a>
* **UEQ-14-08** 🗽 반응성·공정성·구현 비용 사이의 선택을 설명하세요. 직접 구현하지 않은 엔진 기능은 구분하세요.

**꼬리 질문:** 예측 결과가 서버에서 거부되면 연출·자원·이동을 무엇까지 복구하나요? 지연 수치 하나만으로 모든 차이를 설명할 수 있나요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* 로컬 반응과 서버 권위가 공존하며 차이를 수정하는 경로가 필요합니다.

* CharacterMovement의 기존 네트워크 흐름과 별도 위치 변경을 중복 구현하지 않았는지 봅니다.

* 입력 시각·관측 위치·서버 처리 시각을 구분하고 보정 발생 조건을 기록합니다.

* 지연 보상은 게임별 선택입니다. 엔진이 모든 공격의 과거 판정을 자동 구현한다고 가정하지 않습니다.

**실습 발표에서 확인할 것:** 같은 입력 경로에서 정상·지연·손실 조건을 비교하고 서버/클라이언트 역할을 로그에 남깁니다. 에디터 창 하나의 화면만으로 판정하지 않습니다.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Epic — Networked Character Movement](https://dev.epicgames.com/documentation/en-us/unreal-engine/understanding-networked-movement-in-the-character-movement-component-for-unreal-engine) · [Epic — Remote Procedure Calls](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-procedure-calls-in-unreal-engine)

[↑ 목차](#toc)

---


<a id="day-08"></a>
## 8차시 — 에셋과 저장·배포


<a id="assets"></a>
### 에셋 참조·비동기 로딩·Asset Manager

Soft 참조와 콜백 수명을 먼저 봅니다. 원문의 Addressable 주제를 Unreal의 실제 관리 방식으로 바꾼 절입니다.

읽기: [Soft 참조](./README.md#ue-02) · [비동기 로딩](./README.md#ue-17)

<a id="ueq-15-01"></a>
* **UEQ-15-01** 💯 Hard 참조와 Soft 참조가 에셋 의존성과 로딩 시점에 주는 차이는 무엇인가요?

<a id="ueq-15-02"></a>
* **UEQ-15-02** 💯 비동기 로딩의 요청·완료·적용·유지·해제를 각각 누가 책임져야 하나요?

<a id="ueq-15-03"></a>
* **UEQ-15-03** Primary Asset과 Secondary Asset을 구분하는 목적은 무엇인가요?

<a id="ueq-15-04"></a>
* **UEQ-15-04** A 무기를 요청한 직후 B로 바꾸었는데 A가 늦게 완료됩니다. 최신 표시를 어떻게 보장하나요?

<a id="ueq-15-05"></a>
* **UEQ-15-05** 😎 Asset Bundle·쿠킹 규칙·로드 핸들 수명을 어떤 요구에 연결해 검토하나요?

<a id="ueq-15-06"></a>
* **UEQ-15-06** 🔨 두 에셋을 빠르게 교체 요청하고 이전 요청 완료가 최신 선택을 덮지 않도록 보여주세요.

<a id="ueq-15-07"></a>
* **UEQ-15-07** 📊 에디터에서 이미 로드된 경우와 새 패키지 실행에서의 로드 시간·포함 여부를 비교하세요.

<a id="ueq-15-08"></a>
* **UEQ-15-08** 🗽 시작 시간·메모리·전환 지연을 고려해 미리 로드할 대상을 정한 이유를 설명하세요.

**꼬리 질문:** Soft 포인터의 Get이 null이면 경로가 없는 건가요? 로딩 완료 뒤에도 Soft 경로만 보관하면 에셋이 계속 유지되나요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* 경로 참조·실제 로드·로드 후 유지가 별개라는 점이 핵심입니다.

* 완료 시 요청자의 생존뿐 아니라 최신 요청 번호 또는 현재 선택을 확인합니다.

* Asset Manager는 에셋 식별·관리 정책을 정리하는 수단이며 모든 Soft 경로의 쿠킹을 자동 보장한다고 가정하지 않습니다.

* 핸들·참조 수명과 플랫폼의 에셋 포함 규칙을 실제 빌드에서 확인합니다.

**실습 발표에서 확인할 것:** A→B 요청 순서, 완료 순서, 실제 적용 대상, 요청자 종료 후 행동을 각각 기록합니다.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Epic — Asynchronous Asset Loading](https://dev.epicgames.com/documentation/en-us/unreal-engine/asynchronous-asset-loading-in-unreal-engine) · [Epic — Asset Management](https://dev.epicgames.com/documentation/en-us/unreal-engine/asset-management-in-unreal-engine)

[↑ 목차](#toc)

---


<a id="save-package"></a>
### 저장·로드·쿠킹·패키징

에셋 로딩과 객체 ID를 먼저 봅니다. 메모리 주소와 저장 가능한 식별자는 다릅니다.

읽기: [성능 측정과 패키징](./README.md#profiling)

<a id="ueq-16-01"></a>
* **UEQ-16-01** 💯 SaveGame에 저장할 데이터와 런타임 객체 참조를 어떻게 나누나요?

<a id="ueq-16-02"></a>
* **UEQ-16-02** 💯 Build·Cook·Stage·Package·실행 확인은 각각 어떤 단계인가요?

<a id="ueq-16-03"></a>
* **UEQ-16-03** 저장 형식이 바뀌면 기존 저장 파일을 어떻게 읽거나 거절하겠습니까?

<a id="ueq-16-04"></a>
* **UEQ-16-04** PIE에서는 보이는 에셋이 패키지에서 누락됩니다. 어떤 로그와 설정부터 확인하나요?

<a id="ueq-16-05"></a>
* **UEQ-16-05** 😎 저장 중 종료·손상·동시 저장 요청에 대해 어떤 완료·실패·복구 정책을 두겠습니까?

<a id="ueq-16-06"></a>
* **UEQ-16-06** 🔨 새 실행에서 저장값을 복원하고 파일 없음·이전 버전·로드 실패 조건을 분리해 보여주세요.

<a id="ueq-16-07"></a>
* **UEQ-16-07** 📊 패키지 첫 실행에서 시작 맵·입력·동적 에셋 로드를 확인하고 빌드 성공과 실행 성공을 따로 기록하세요.

<a id="ueq-16-08"></a>
* **UEQ-16-08** 🗽 배포 환경에서만 발생한 문제를 재현한 경험이나 조사 절차를 설명하세요.

**꼬리 질문:** Actor의 포인터 주소를 저장하면 다음 실행에서도 같은 대상을 찾을 수 있나요? 비동기 저장 요청을 보냈다는 것이 디스크 기록 완료인가요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* 안정적인 ID와 복원할 값·관계를 저장하고 새 객체에 연결하는 정책을 정합니다.

* 저장 요청과 완료를 나누며 파일 없음·지원하지 않는 형식·읽기 실패를 별도 처리합니다.

* 버전 변환과 손상 복구는 프로젝트 정책입니다. SaveGame 사용만으로 모든 상황의 원자적 저장을 보장하지 않습니다.

* 빌드 산출물 생성 후 실제 시작과 필요한 콘텐츠 로드를 확인해야 배포 동작을 판단할 수 있습니다.

**실습 발표에서 확인할 것:** 테스트 파일로만 실패 조건을 재현합니다. 정상 저장 데이터와 사용자 파일을 덮어쓰지 않으며 기대·실제·복구 결과를 기록합니다.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Epic — Saving and Loading](https://dev.epicgames.com/documentation/en-us/unreal-engine/saving-and-loading-your-game-in-unreal-engine) · [Epic — Packaging](https://dev.epicgames.com/documentation/en-us/unreal-engine/packaging-your-project)

[↑ 목차](#toc)

---


<a id="day-09"></a>
## 9차시 — AI와 구조·테스트


<a id="ai"></a>
### AIController·Behavior Tree·Perception·Navigation

Controller·Pawn과 상태 전환을 이해한 뒤 진행합니다. 일반 클라이언트 직무에서는 기초, AI 직무에서는 심화까지 준비합니다.

읽기: [플레이어와 조종 대상](./README.md#ue-06) · [책임 분리](./README.md#ue-12)

<a id="ueq-17-01"></a>
* **UEQ-17-01** 💯 AIController·Pawn·Behavior Tree·Blackboard의 역할을 설명하세요.

<a id="ueq-17-02"></a>
* **UEQ-17-02** 💯 Task·Decorator·Service를 행동·조건·상태 갱신 관점에서 구분하세요.

<a id="ueq-17-03"></a>
* **UEQ-17-03** 적을 감지하는 것, 공격 대상으로 선정하는 것, 경로를 구하는 것은 어떻게 다른가요?

<a id="ueq-17-04"></a>
* **UEQ-17-04** AI가 플레이어를 알아보는데 이동하지 않습니다. 인지·Blackboard·트리·내비게이션 중 어디를 확인하나요?

<a id="ueq-17-05"></a>
* **UEQ-17-05** 😎 여러 AI가 하나의 행동 에셋을 쓸 때 개별 실행 상태의 저장 위치와 중단 처리를 조사하세요.

<a id="ueq-17-06"></a>
* **UEQ-17-06** 🔨 순찰→추적→대상 상실 후 복귀를 구현하고 현재 키와 실행 노드를 보여주세요.

<a id="ueq-17-07"></a>
* **UEQ-17-07** 📊 감지 실패·목표 키 누락·경로 없음·Task 종료 누락을 각각 구분할 로그와 도구를 조사하세요.

<a id="ueq-17-08"></a>
* **UEQ-17-08** 🗽 AI 반응성과 갱신 비용 사이에서 업데이트 주기나 이벤트를 선택한 이유를 설명하세요.

**꼬리 질문:** Blackboard에 대상이 남아 있지만 대상이 파괴되면 어떻게 하나요? 감지 상실과 즉시 기억 삭제가 같은 게임 규칙인가요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* Behavior Tree는 행동 분기를 실행하고 Blackboard 키는 판단에 쓸 정보를 보관합니다.

* Perception의 자극 정보와 게임의 대상 선정·기억 정책을 분리합니다.

* 인지가 정상이어도 경로가 없거나 이동 요청·Task 종료가 잘못되면 이동은 실패할 수 있습니다.

* 대상 수명과 행동 중단 시 정리를 확인합니다. 인스턴스 공유 방식은 사용 노드와 버전에서 검토합니다.

**실습 발표에서 확인할 것:** 현재 대상·실행 분기·이동 결과를 함께 표시합니다. 대상 상실과 파괴, 경로 없는 위치를 각각 재현합니다.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Epic — Behavior Trees](https://dev.epicgames.com/documentation/en-us/unreal-engine/behavior-trees-in-unreal-engine) · [Epic — AI Perception](https://dev.epicgames.com/documentation/en-us/unreal-engine/ai-perception-in-unreal-engine) · [Epic — Navigation System](https://dev.epicgames.com/documentation/en-us/unreal-engine/navigation-system-in-unreal-engine)

[↑ 목차](#toc)

---


<a id="architecture-test"></a>
### Component·Subsystem·설계·검증

객체 수명과 프레임워크를 먼저 봅니다. 패턴 이름보다 데이터 소유자와 의존 방향을 설명합니다.

읽기: [Component·Subsystem](./README.md#ue-12) · [객체 소유권](./README.md#cpp-04)

<a id="ueq-18-01"></a>
* **UEQ-18-01** 💯 상속·Component 조합·Subsystem 중 무엇을 선택할지 기준을 설명하세요.

<a id="ueq-18-02"></a>
* **UEQ-18-02** 💯 World·GameInstance·LocalPlayer 범위의 서비스는 수명과 사용자 수가 어떻게 다른가요?

<a id="ueq-18-03"></a>
* **UEQ-18-03** 인벤토리와 장비·UI·저장을 연결할 때 어떤 의존을 직접 참조하고 어떤 것을 이벤트로 풀겠습니까?

<a id="ueq-18-04"></a>
* **UEQ-18-04** 맵을 바꾼 뒤 관리자에 이전 Actor가 남습니다. 소유·등록·해제·약한 참조를 어떻게 검토하나요?

<a id="ueq-18-05"></a>
* **UEQ-18-05** 😎 에디터 전용 의존성이 런타임 모듈로 들어오지 않도록 어떤 경계를 두겠습니까?

<a id="ueq-18-06"></a>
* **UEQ-18-06** 🔨 순수 계산 테스트와 월드에서의 기능 테스트를 하나씩 설계하고 무엇을 증명하는지 설명하세요.

<a id="ueq-18-07"></a>
* **UEQ-18-07** 📊 Automation·Functional Test·수동 PIE·멀티클라이언트 검증의 범위 차이를 조사하세요.

<a id="ueq-18-08"></a>
* **UEQ-18-08** 🗽 팀원이 기능을 추가할 때 변경 파일이 줄도록 설계를 개선한 경험 또는 제안을 설명하세요.

**꼬리 질문:** 클래스를 여러 개로 나누면 결합도가 자동으로 낮아지나요? 한 개의 단위 테스트가 통과하면 에디터 배선까지 검증된 건가요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* 기능이 붙는 대상과 시작·종료 범위를 먼저 정합니다.

* 오래 사는 서비스가 짧게 사는 객체를 보관할 때 맵 종료와 등록 해제 책임이 필요합니다.

* 테스트는 실제로 검증한 경계를 명시합니다. 계산 성공과 에셋 연결·복제·패키지 성공을 구분합니다.

* 테스트 순서나 이전 실행 상태에 의존하지 않게 준비·정리를 설계합니다.

**실습 발표에서 확인할 것:** 테스트 입력·기대 결과·초기 상태·정리를 명시하고 실패할 때 어떤 회귀를 잡는지 보여주세요.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Epic — Subsystems](https://dev.epicgames.com/documentation/en-us/unreal-engine/programming-subsystems-in-unreal-engine) · [Epic — Automation Test Framework](https://dev.epicgames.com/documentation/en-us/unreal-engine/automation-test-framework-in-unreal-engine)

[↑ 목차](#toc)

---


<a id="day-10"></a>
## 10차시 — GAS와 종합 면접


<a id="gas"></a>
### GAS·GameplayTag·비용·취소

프레임워크·이벤트·네트워크 기초 이후 진행합니다. GAS 사용 직무에는 핵심, 미사용 직무에는 선택 심화입니다.

읽기: [GAS 구성](./README.md#ue-09) · [활성화·비용·종료](./README.md#ue-18)

<a id="ueq-19-01"></a>
* **UEQ-19-01** 💯 Ability·ASC·AttributeSet·GameplayEffect·GameplayTag는 각각 무엇을 맡나요?

<a id="ueq-19-02"></a>
* **UEQ-19-02** 💯 Ability의 부여·활성화 가능 검사·실행·Commit·종료를 구분하세요.

<a id="ueq-19-03"></a>
* **UEQ-19-03** 직접 체력 값을 바꾸는 방식과 Effect를 적용하는 방식은 어떤 요구에서 달라지나요?

<a id="ueq-19-04"></a>
* **UEQ-19-04** 비용 차감 뒤 Montage가 실패했습니다. 종료와 환불을 어떻게 처리하겠습니까?

<a id="ueq-19-05"></a>
* **UEQ-19-05** 😎 Owner와 Avatar, ASC 위치, 재스폰 초기화, 예측과 서버 거부를 함께 설명하세요.

<a id="ueq-19-06"></a>
* **UEQ-19-06** 🔨 단순 회복 Ability에서 정상 실행·비용 부족·중단·재실행 결과를 보여주세요.

<a id="ueq-19-07"></a>
* **UEQ-19-07** 📊 사용 버전의 효과 지속·중첩·태그 조건을 조사하고 프로젝트 설정과 대조하세요.

<a id="ueq-19-08"></a>
* **UEQ-19-08** 🗽 GAS를 채택하거나 자체 시스템을 유지한 이유를 규칙 재사용·복잡도·협업 비용으로 설명하세요.

**꼬리 질문:** 태그 이름을 만들기만 하면 공격이 차단되나요? EndAbility가 이미 차감한 모든 비용을 자동 환불하나요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* 행동 흐름과 속성 변경 규칙을 분리하고 ASC가 관리하는 관계를 설명합니다.

* 부여는 활성 상태와 다릅니다. Commit 성공과 후속 작업 성공도 별개입니다.

* 실패·취소에서도 정리 경로가 필요하고 환불은 게임 규칙에 따른 선택입니다.

* Owner·Avatar가 다른 설계에서는 새 Pawn 연결과 복제 도착 시점까지 봅니다. 예측이 모든 임의 변경을 자동 복구하지는 않습니다.

**실습 발표에서 확인할 것:** 실행 전후 속성·태그·활성 상태를 비교하고 취소 이후 다시 실행되는지 확인합니다. 한 번의 모션 재생을 GAS 전체 검증으로 표시하지 않습니다.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Epic — Understanding GAS](https://dev.epicgames.com/documentation/en-us/unreal-engine/understanding-the-unreal-engine-gameplay-ability-system) · [Epic — Gameplay Effects](https://dev.epicgames.com/documentation/en-us/unreal-engine/gameplay-effects-for-the-gameplay-ability-system-in-unreal-engine)

[↑ 목차](#toc)

---


<a id="interview-case"></a>
### 종합 디버깅·코드 리뷰·경험 발표

앞의 개념 중 실제 구현한 것을 선택합니다. 상용 경험·개인 실습·기존 엔진 기능을 구분합니다.

읽기: [질문과 실습](./README.md#study-questions) · [성능 측정](./README.md#profiling)

<a id="ueq-20-01"></a>
* **UEQ-20-01** 💯 자신이 만든 기능 하나를 입력→상태 변경→표시→종료 순서로 설명하세요.

<a id="ueq-20-02"></a>
* **UEQ-20-02** 💯 버그의 증상·재현 조건·원인 증거·수정·회귀 확인을 순서대로 설명하세요.

<a id="ueq-20-03"></a>
* **UEQ-20-03** 팀 코드에서 원시 포인터·이벤트 구독·모듈 의존성을 리뷰한다면 어떤 질문부터 하겠습니까?

<a id="ueq-20-04"></a>
* **UEQ-20-04** 사망 직후 로딩이 완료되고 UI가 열리면서 크래시가 납니다. 가장 작은 재현부터 어떻게 만들겠습니까?

<a id="ueq-20-05"></a>
* **UEQ-20-05** 😎 개선 후 평균 FPS는 같지만 끊김이 줄었다면 어떤 기록으로 효과와 한계를 설명하겠습니까?

<a id="ueq-20-06"></a>
* **UEQ-20-06** 🔨 한 기능의 정상·취소·대상 파괴 조건을 직접 보여주고 해당 코드를 자료 없이 설명하세요.

<a id="ueq-20-07"></a>
* **UEQ-20-07** 📊 지원 공고의 요구 기술과 자신의 구현 근거를 연결하고 모르는 항목은 조사 계획으로 남기세요.

<a id="ueq-20-08"></a>
* **UEQ-20-08** 🗽 자신의 구현, 동료의 구현, 엔진이 제공한 기능, AI 도움을 받은 부분을 나누어 설명하세요.

**꼬리 질문:** 코드를 보지 않고 변경 이유를 설명할 수 있나요? 같은 문제를 다른 조건으로 바꾸면 어떤 가정이 깨지나요?

<details>
<summary>답변 점검과 해설 — 먼저 말해 본 뒤 펼치세요</summary>

* 경험은 수행한 범위와 확인한 결과를 말합니다. 예상 결과를 본인 성과로 바꾸지 않습니다.

* 큰 증상을 입력·수명·동기화·표시 경계로 나누고 최소 재현으로 원인을 좁힙니다.

* 개선 수치에는 환경과 비교 조건이 필요합니다. 평균 외에 프레임 시간과 꼬리 지연도 검토합니다.

* 대안을 선택한 이유와 한계를 말하고, 미구현 항목은 설계 제안으로 구분합니다.

**실습 발표에서 확인할 것:** 짧은 시연과 소유·데이터 흐름 그림, 원인 로그 또는 측정, 수정 범위, 남은 문제를 준비합니다. 공개할 수 있는 연습 자료를 사용합니다.

위 항목은 답변을 점검할 기준입니다. 다른 설계를 택했다면 요구조건·장단점·검증 근거를 함께 설명하세요.

</details>

공식 읽기 자료: [Epic — Programming Career Paths](https://www.epicgames.com/site/earlycareers/career-paths?lang=en) · [Epic — Automation Test Framework](https://dev.epicgames.com/documentation/en-us/unreal-engine/automation-test-framework-in-unreal-engine)

[↑ 목차](#toc)

---


<a id="research"></a>
## 원본 비교·조사 출처·선정 기준

### 원본에서 가져온 방식과 Unreal에서 바꾼 범위

비교 기준은 이 포크가 보유한 원본 질문집 본문입니다. 5차시·10개 주제, 최상위 질문 목록 113개를 확인했습니다. 들여쓴 꼬리 질문과 안내 문구는 별도입니다. 문항 수는 학습 품질이나 출제 적중률을 뜻하지 않습니다.

| 원본의 구성·주제 | Unreal 질문집에서의 대응 |
|---|---|
| 차시별 발표와 읽기 자료 | 10차시, 주제별 README 연결과 공식 자료 |
| GC·Fake Null | C++ 수명, UObject 추적 참조, Destroy와 EndPlay |
| 메모리 최적화·프로파일링 | TArray·문자열 선택, Memory Insights, 재사용 수명 |
| 렌더링·Rendering Profiling | CPU/GPU 병목, 프레임 시간, 기능별 비용 검증 |
| 멀티스레딩·비동기 | Delegate, Timer, 작업 의존성, 결과 적용 수명 |
| 객체지향·디자인 패턴 | Framework, Component, Subsystem, 테스트 경계 |
| Addressable | Soft 참조, 로드 핸들, Asset Manager, 쿠킹 |
| 네트워크 | 상태와 RPC, 소유 경로, 이동 예측·보정, 저장 |
| LINQ·Reflection | 컨테이너 비용, 엔진 리플렉션·UHT·모듈 |
| Unreal 직무에 필요한 추가 범위 | 입력·충돌·애니메이션·UMG·AI·GAS |

Unity 기능을 Unreal 이름으로만 치환하지 않았습니다. 엔진마다 객체 수명·복제·빌드 모델이 달라 질문의 전제도 바꿨습니다. 기존 12차시·84문항은 기초를 따라가는 실습 경로로 유지하고, 이 질문집은 주제별 면접 발표와 자기 점검에 사용합니다.

### 면접 준비 범위를 판단한 공개 근거

1. [Epic 공식 Career Paths](https://www.epicgames.com/site/earlycareers/career-paths?lang=en)는 C++, Unreal, 자료 구조·알고리즘과 수학·물리 배경, 구현이 드러나는 포트폴리오를 안내합니다. 이에 C++·공간 계산·직접 설명할 경험을 엔진 질문과 함께 배치했습니다. 공식 기출 목록이나 모든 회사의 채용 기준은 아닙니다.
2. [국내 지원자의 언리얼 클라이언트 취준 회고](https://econo-my.tistory.com/53?category=1225230)는 면접 후 CS·C++·엔진 기본기의 부족을 돌아보고 학습을 보완한 개인 사례입니다. 기본기를 생략하지 않는 구성에 참고했으며 특정 질문의 출제 증거로 사용하지 않았습니다.
3. [국내 클라이언트 면접 후기](https://shung2.tistory.com/1246?category=1204172)는 지원자가 자신의 면접과 코드 설명 경험을 적은 글입니다. 오래된 개인 기록이며 회사별 현재 절차·질문을 보장하지 않습니다. 구현 이유를 말하는 종합 발표 문항의 참고입니다.
4. [Unreal 면접에 관한 공개 토론](https://www.reddit.com/r/unrealengine/comments/1t7cvsi/unreal_job_interview/)에는 메모리·수학·리플렉션·복제와 본인 프로젝트 꼬리 질문을 준비하라는 경험담이 있습니다. 작성자의 신원·경력은 독립 검증하지 않았고, 주제 범위 참고로만 사용했습니다.

검색에서 찾은 일반적인 “Top 50”·연도별 면접 요약은 회사 기출이나 기술 정답의 증거로 채택하지 않았습니다. 공개 후기 몇 편으로 출제 빈도를 계산할 수 없으므로 💯는 학습 우선순위만 뜻합니다. 한국·해외 모든 회사의 면접을 망라했다는 의미도 아닙니다.

### 기술 출처와 검토 방법

각 절 끝의 공식 읽기 자료는 그 절의 기술 개념을 확인할 근거입니다. 실패 시나리오와 실습 요구는 이 포크에서 새로 구성했습니다. 답변 점검은 유일한 정답 문장이 아니라 설명에 필요한 구분과 확인 범위입니다.

Epic 문서는 버전 선택에 따라 내용이 바뀝니다. 조사 당시 기본 페이지는 대체로 UE 5.8로 표시되었으며, 프로젝트가 UE 5.7 등 다른 버전이면 해당 버전으로 전환해 API·옵션·복제 시스템 차이를 확인하세요. 이 자료는 최신 버전 사용을 요구하지 않습니다.

| 기술 범위 | 확인한 공식 문서 |
|---|---|

| C++ 수명·소유권·다형성 | [Microsoft — C++ 수명과 RAII](https://learn.microsoft.com/en-us/cpp/cpp/object-lifetime-and-resource-management-modern-cpp?view=msvc-170) |
| UObject·GC·Actor 종료 | [Epic — Object Pointers](https://dev.epicgames.com/documentation/en-us/unreal-engine/object-pointers-in-unreal-engine) · [Epic — Actor Lifecycle](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-actor-lifecycle) |
| 리플렉션·CDO·Blueprint·빌드 | [Epic — Object Handling](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-object-handling-in-unreal-engine) · [Epic — Modules](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-modules) |
| Gameplay Framework·재스폰·맵 전환 | [Epic — Gameplay Framework](https://dev.epicgames.com/documentation/en-us/unreal-engine/gameplay-framework-in-unreal-engine) |
| TArray·컨테이너·문자열 | [Epic — TArray](https://dev.epicgames.com/documentation/en-us/unreal-engine/array-containers-in-unreal-engine) · [Epic — String Handling](https://dev.epicgames.com/documentation/en-us/unreal-engine/string-handling-in-unreal-engine) |
| Delegate·Timer·비동기·스레드 | [Epic — Multicast Delegates](https://dev.epicgames.com/documentation/en-us/unreal-engine/multicast-delegates-in-unreal-engine) · [Epic — Tasks System](https://dev.epicgames.com/documentation/unreal-engine/tasks-systems-in-unreal-engine?lang=en-US) |
| 메모리·할당·프로파일링 | [Epic — Memory Insights](https://dev.epicgames.com/documentation/en-us/unreal-engine/memory-insights-in-unreal-engine) |
| 렌더링·CPU/GPU 병목 | [Epic — Performance Profiling](https://dev.epicgames.com/documentation/en-us/unreal-engine/introduction-to-performance-profiling-and-configuration-in-unreal-engine) · [Epic — Unreal Insights](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-insights-in-unreal-engine) |
| Enhanced Input·이동·게임 수학 | [Epic — Enhanced Input](https://dev.epicgames.com/documentation/en-us/unreal-engine/enhanced-input-in-unreal-engine) |
| 충돌·Trace·공격 판정 | [Epic — Collision Overview](https://dev.epicgames.com/documentation/unreal-engine/collision-in-unreal-engine---overview?lang=en-US) |
| Animation Blueprint·Montage·종료 | [Epic — Animation Montage](https://dev.epicgames.com/documentation/en-us/unreal-engine/animation-montage-in-unreal-engine) |
| UMG·UI 수명·갱신 비용 | [Epic — UMG Optimization](https://dev.epicgames.com/documentation/unreal-engine/optimization-guidelines-for-umg-in-unreal-engine?lang=en-US) · [Epic — Multicast Delegates](https://dev.epicgames.com/documentation/en-us/unreal-engine/multicast-delegates-in-unreal-engine) |
| 네트워크 상태·RPC·소유권 | [Epic — Networking Overview](https://dev.epicgames.com/documentation/en-us/unreal-engine/networking-overview-for-unreal-engine) · [Epic — Remote Procedure Calls](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-procedure-calls-in-unreal-engine) |
| 이동 예측·보정·서버 판정 | [Epic — Networked Character Movement](https://dev.epicgames.com/documentation/en-us/unreal-engine/understanding-networked-movement-in-the-character-movement-component-for-unreal-engine) · [Epic — Remote Procedure Calls](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-procedure-calls-in-unreal-engine) |
| 에셋 참조·비동기 로딩·Asset Manager | [Epic — Asynchronous Asset Loading](https://dev.epicgames.com/documentation/en-us/unreal-engine/asynchronous-asset-loading-in-unreal-engine) · [Epic — Asset Management](https://dev.epicgames.com/documentation/en-us/unreal-engine/asset-management-in-unreal-engine) |
| 저장·로드·쿠킹·패키징 | [Epic — Saving and Loading](https://dev.epicgames.com/documentation/en-us/unreal-engine/saving-and-loading-your-game-in-unreal-engine) · [Epic — Packaging](https://dev.epicgames.com/documentation/en-us/unreal-engine/packaging-your-project) |
| AIController·Behavior Tree·Perception·Navigation | [Epic — Behavior Trees](https://dev.epicgames.com/documentation/en-us/unreal-engine/behavior-trees-in-unreal-engine) · [Epic — AI Perception](https://dev.epicgames.com/documentation/en-us/unreal-engine/ai-perception-in-unreal-engine) · [Epic — Navigation System](https://dev.epicgames.com/documentation/en-us/unreal-engine/navigation-system-in-unreal-engine) |
| Component·Subsystem·설계·검증 | [Epic — Subsystems](https://dev.epicgames.com/documentation/en-us/unreal-engine/programming-subsystems-in-unreal-engine) · [Epic — Automation Test Framework](https://dev.epicgames.com/documentation/en-us/unreal-engine/automation-test-framework-in-unreal-engine) |
| GAS·GameplayTag·비용·취소 | [Epic — Understanding GAS](https://dev.epicgames.com/documentation/en-us/unreal-engine/understanding-the-unreal-engine-gameplay-ability-system) · [Epic — Gameplay Effects](https://dev.epicgames.com/documentation/en-us/unreal-engine/gameplay-effects-for-the-gameplay-ability-system-in-unreal-engine) |
| 종합 디버깅·코드 리뷰·경험 발표 | [Epic — Programming Career Paths](https://www.epicgames.com/site/earlycareers/career-paths?lang=en) · [Epic — Automation Test Framework](https://dev.epicgames.com/documentation/en-us/unreal-engine/automation-test-framework-in-unreal-engine) |


### 문서 검증 범위

10차시·20개 주제의 160개 문항 식별자와 중복 여부, 20개 해설 접기 블록, 새 질문집 및 진입 경로의 로컬 링크 117개를 검사해 통과했습니다. 주제별 공식 읽기 자료와 조사 근거로 공식 페이지 32개를 연결했습니다. 원본 Unity 질문 본문과 기존 C++·Unreal 설명·실행 예제는 유지합니다. 이 개정은 질문 문서 작성이며 Unreal 엔진 빌드·PIE·네트워크 실습 완료를 의미하지 않습니다.

<a id="final-check"></a>
## 면접 전 최종 점검

* 핵심 질문에 개념 한 문장과 구체적인 예시로 답할 수 있나요?
* 누가 데이터를 소유하고 언제 만들고 정리하는지 설명할 수 있나요?
* 실패 조건 하나를 추가하면 어느 경로가 달라지는지 말할 수 있나요?
* 측정·실습·실제 경험과 아직 확인하지 않은 가정을 구분했나요?
* 엔진이 제공한 기능과 자신이 구현한 부분을 구분했나요?
* 지원 직무가 요구하는 심화 영역을 선택했나요?

[↑ 목차](#toc) · [통합 README로 돌아가기](./README.md#목차)

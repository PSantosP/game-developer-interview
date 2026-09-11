# Unreal — 객체와 데이터의 흐름을 따라가기

[학습 목차](README.md) · [개념 요약](unreal.md) · [실습 단계](practice.md)

이 문서는 완성 프로젝트가 아닌 읽기용 구현 사례입니다. C++ 조각은 해당 클래스·함수의 일부이며 필요한 선언·헤더·모듈·에셋 연결을 모두 포함하지 않습니다. '의사코드'는 동작 순서를 설명하며 실제 API와 구분합니다. 엔진 빌드·PIE 결과는 [검증 기록](verification.md)을 확인하세요.

휴대폰에서는 각 절의 상황과 흐름을 읽고 결과를 예상합니다. PC 실습은 [준비 및 단계](practice.md)에 따라 작은 성공 조건 하나씩 진행합니다. 처음에는 GAS·네트워크를 동시에 붙이지 않습니다.

<a id="ue-01"></a>
## 1. 무기 설정을 생성할 때 new를 바로 쓰면 안 되는 이유

<!-- EXPLANATION_PASS -->

**엔진이 알아야 하는 객체인가**

단순한 데미지 계산 값은 일반 구조체로 충분할 수 있습니다. 반면 에디터 속성, Blueprint, 직렬화와 연결할 객체는 엔진이 타입과 참조를 알아야 합니다. UObject 생성 경로는 C++ 생성자 호출뿐 아니라 엔진 객체 등록을 함께 수행하므로 일반 new로 대체하지 않습니다.

예제에서 NewObject는 설정 객체를 만들고 Settings 멤버는 그 결과를 유지합니다. 여기서 this는 Outer를 지정하며, UPROPERTY로 추적되는 멤버 참조는 GC의 도달 경로에 참여합니다. 두 역할을 같은 소유 개념으로 섞지 않습니다.

Actor는 월드에 참여하므로 SpawnActor 경로를 사용하고, 기본 컴포넌트는 생성자의 CreateDefaultSubobject로 구성합니다. 무엇을 만들려는지 먼저 고르면 API 선택이 이름 암기에서 역할 판단으로 바뀝니다.

무기 수치만 계산하는 일반 C++ 구조체와 에디터·GC에 참여하는 UObject는 관리 체계가 다릅니다. 모든 데이터를 UObject로 만들 필요도 없고, UObject를 일반 포인터 소유권 규칙으로 해제해서도 안 됩니다.

| 대상 | 생성 의도 | 종료 책임 |
|---|---|---|
| 일반 값 구조체 | 계산·복사 | 일반 C++ 수명 |
| UObject | 엔진 객체 생성 | 엔진 객체 시스템과 GC |
| Actor | World에 참여 | Destroy·EndPlay 후 GC |
| 기본 Component | 클래스 기본 구성 | 소유 Actor와 엔진 수명 |

~~~cpp
// UObject 파생 UWeaponSettings가 이미 선언되어 있다는 전제.
// 살아 있는 UObject 소유자의 멤버:
UPROPERTY()
TObjectPtr<UWeaponSettings> Settings;

// 소유자의 런타임 초기화 함수 일부:
Settings = NewObject<UWeaponSettings>(this);
~~~

흐름: 소유자가 객체를 생성 → GC가 추적하는 멤버로 참조 → 필요한 동안 사용 → 참조와 도달 가능성이 사라지면 엔진이 회수합니다. Outer와 GC 강한 참조는 동일한 개념이 아닙니다. Outer를 넘겼다는 이유만으로 임의의 모든 자식 수명이 보장된다고 외우지 않습니다.

확인 질문: 지역 변수에만 NewObject 결과를 저장하고 함수를 끝내면 누가 유지하나요? 다른 도달 가능한 참조가 없다면 장기 보관 계약이 없습니다. Actor는 World에 등록되는 생성 경로라는 점도 일반 UObject와 구분해야 합니다.

근거: [Object Handling](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-object-handling-in-unreal-engine).

<a id="ue-02"></a>
## 2. 조준 대상과 무기 에셋에는 다른 참조가 필요하다

<!-- EXPLANATION_PASS -->

**유지할 대상과 없어져도 되는 대상을 나누기**

현재 장착 설정은 사용하는 동안 유지해야 하지만 조준한 적은 게임 규칙에 따라 파괴될 수 있습니다. 아직 보지 않은 아이콘은 경로만 알고 필요할 때 로드하는 편이 적절할 수 있습니다. 이 세 요구가 강한 참조, 약한 관찰, Soft 경로 참조의 출발점입니다.

예제의 AimTarget.Get이 null이면 조준 표시를 지우고 대상 사용을 멈춥니다. Icon.Get이 null이면 경로가 없는지, 경로는 있지만 아직 로드하지 않았는지를 나눠 판단합니다. 후자라면 로딩 요청과 대체 표시가 필요합니다.

GC로부터의 유지와 게임플레이에서의 생존은 다릅니다. 강한 참조가 남아도 Actor의 Destroy 요청을 막지는 않습니다. 따라서 공격 가능한지는 포인터뿐 아니라 사망·무적 등의 게임 상태도 검사해야 합니다.

조준한 적은 죽을 수 있습니다. 반면 인벤토리에 표시할 무기 이미지는 아직 로드하지 않았을 수 있습니다. '포인터 하나'라는 공통점보다 필요한 수명과 로딩 의도를 먼저 봅니다.

~~~cpp
// UCLASS의 멤버 예:
TWeakObjectPtr<AActor> AimTarget;

UPROPERTY(EditDefaultsOnly)
TSoftObjectPtr<UTexture2D> Icon;

// 게임 스레드에서 조준 대상 사용:
if (AActor* Target = AimTarget.Get())
{
    const FVector Position = Target->GetActorLocation();
    // Position으로 표시 위치를 계산한다.
}
~~~

위 Target 사용은 유효한 객체를 얻은 같은 동기 흐름 안의 예입니다. 얻은 원시 포인터를 나중의 비동기 콜백에 저장하면 수명 문제가 다시 생깁니다. Icon.Get이 null이라는 사실만으로 에셋 경로가 비었다고 판단하지도 않습니다. 유효 경로지만 미로드 상태일 수 있습니다.

<details>
<summary>대상이 사라지면 무엇이 달라져야 하나요?</summary>

조준 UI는 표시를 해제하고 다음 대상을 찾을 수 있어야 합니다. 무기 아이콘은 로딩 중 대체 표시를 사용하고 완료 후 다시 갱신할 수 있습니다. 강한 참조를 잡아 조준 대상의 명시적인 Actor 파괴까지 막으려는 설계는 맞지 않습니다.

</details>

근거: [Object Pointers](https://dev.epicgames.com/documentation/en-us/unreal-engine/object-pointers-in-unreal-engine).

<a id="ue-03"></a>
## 3. 생성자에 기본 구성, BeginPlay에 플레이 시작 작업

<!-- EXPLANATION_PASS -->

**기본 구성과 플레이 시작 작업의 경계**

문에 Mesh가 있다는 구조는 인스턴스가 플레이하기 전부터 정의되어야 합니다. 반면 현재 플레이어를 찾거나 시작 상태를 월드에 표시하는 일은 런타임 의존성이 있습니다. 생성자가 CDO 구성에도 실행되기 때문에 두 일을 분리합니다.

예제는 생성자에서 기본 Mesh를 만들고 루트로 지정한 뒤 BeginPlay에서 시작 로그를 남깁니다. BeginPlay로 옮겼다는 것만으로 다른 Actor의 준비가 보장되지는 않습니다. A가 B의 준비된 데이터를 필요로 한다면 B가 준비를 알리거나 등록받는 명시적인 연결이 필요합니다.

참조가 null이면 생성 실패인지, 아직 할당하지 않았는지, 클라이언트 복제가 도착하지 않았는지 구분합니다. 임의의 Delay로 성공하는 순간을 기다리기보다 필요한 데이터가 준비되는 사건에 연결해야 타이밍 변화에도 동작합니다.

문 Actor가 항상 Mesh를 갖는다는 구조는 생성자에, 플레이 시작 때 문 상태를 표시하는 일은 런타임에 둡니다. 생성자는 CDO 구성에도 관여하므로 실행 중 플레이어가 있다고 가정하면 안 됩니다.

~~~cpp
// ADoorLab 생성자의 일부. Mesh는 선언된 UStaticMeshComponent 멤버.
Mesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("DoorMesh"));
SetRootComponent(Mesh);
~~~

~~~cpp
// ADoorLab::BeginPlay 본문 일부
Super::BeginPlay();
UE_LOG(LogTemp, Log, TEXT("Door ready: %s"), *GetName());
~~~

여기서 CreateDefaultSubobject는 장면마다 임의의 Actor를 찾는 작업이 아닙니다. 클래스의 기본 구성 요소를 만드는 작업입니다. 스폰된 Actor와 로드된 Actor는 BeginPlay 이전 초기화 경로가 다를 수 있습니다.

디버깅 순서: 생성자에서 실패한 World 조회를 BeginPlay로 옮기기 전에 그 참조가 정말 시작 때 준비되는지 판단 → 다른 Actor와의 의존이 있다면 명시적인 등록·준비 이벤트 설계 → 클라이언트에서는 복제 도착 시점까지 구분합니다.

근거: [Actor Lifecycle](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-actor-lifecycle).

<a id="ue-04"></a>
## 4. 파괴 요청·플레이 종료·메모리 회수

<!-- EXPLANATION_PASS -->

**종료한 객체의 콜백이 남는 이유**

Actor를 끝냈다는 게임 규칙과 메모리 회수는 같은 시점이 아닙니다. 또한 Actor 밖의 타이머나 이벤트 등록이 남아 있으면 종료 뒤에도 작업을 시도할 수 있습니다. 따라서 정리는 메모리 소멸자만 기다리지 않고 플레이 종료 수명에 맞춥니다.

예제는 EndPlay에서 자신이 관리하는 타이머 핸들을 취소합니다. 이벤트를 구독했다면 같은 종료 경로에서 구독도 해제하고, 비동기 결과는 취소하거나 도착 시 대상과 요청의 유효성을 확인합니다. 객체가 먼저 사라질 수 있는 외부 소스는 접근 전에 검사합니다.

EndPlay 이유가 사망에 의한 파괴인지, 레벨 전환인지, PIE 종료인지 기록하면 재현 조건을 나눌 수 있습니다. 체력 0 처리만 테스트해서는 맵 전환 중 뒤늦은 콜백 문제를 확인할 수 없습니다.

체력이 0인 적이 아직 메모리에 있다고 다시 공격 대상으로 사용하면 안 됩니다. 게임에서 끝났다는 상태와 메모리가 반환됐다는 상태는 다릅니다.

~~~text
피해 처리
  → 사망 규칙 결정
  → Destroy 요청
  → EndPlay 등 플레이 종료 처리
  → 이후 GC 과정에서 객체 메모리 정리
~~~

Actor의 Destroy는 즉시 delete와 같지 않습니다. 호출이 허용되지 않아 실패하는 경우도 있고, 즉시 파괴·메모리 반환을 가정하면 안 됩니다. 정리 로그는 EndPlay의 이유와 Actor 식별자를 함께 남깁니다.

~~~cpp
// AEnemyLab::EndPlay 본문 일부
GetWorldTimerManager().ClearTimer(AttackTimer);
UE_LOG(LogTemp, Log, TEXT("EndPlay %s reason=%d"),
       *GetName(), static_cast<int32>(EndPlayReason));
Super::EndPlay(EndPlayReason);
~~~

AttackTimer는 이 Actor가 관리하는 타이머 핸들이라는 전제입니다. 일반 람다 등 객체와 직접 연결되지 않은 작업도 별도 해제 책임이 있는지 봅니다. 레벨 전환·PIE 종료도 검사하므로 EndPlay를 '사망 전용 함수'로 쓰지 않습니다.

반례: 파괴 직후 포인터가 null이 아닌 것만 확인하고 접근하면 종료 중인 대상을 사용할 수 있습니다. 게임플레이 유효성 검사와 콜백 취소를 함께 설계합니다.

근거: [Actor Lifecycle](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-actor-lifecycle).

<a id="ue-05"></a>
## 5. 라운드 규칙과 화면 표시의 소유자

<!-- EXPLANATION_PASS -->

**규칙을 결정하는 곳과 결과를 보여 주는 곳**

클라이언트가 각자 라운드 종료를 확정하면 서로 다른 승패를 표시할 수 있습니다. 서버 GameMode가 규칙을 판단하고 공유할 결과를 GameState에 반영하면 클라이언트 UI는 전달받은 상태를 표시할 수 있습니다.

이 흐름에서 GameState에 새 변수를 선언하는 것만으로 자동 복제되지는 않습니다. 해당 속성의 복제 설정과 화면 갱신 경로까지 연결해야 합니다. 로컬 옵션처럼 네트워크 공유가 필요 없는 값은 GameInstance 계열에 둘 수 있지만 별도 저장을 하지 않으면 프로그램 재실행까지 보존되지는 않습니다.

값이 안 보이면 서버 판단 로그 → 서버 상태 값 → 클라이언트 수신 값 → UI 갱신 순으로 봅니다. UI에서 GameMode를 찾는 코드부터 늘리는 대신 어느 경계에서 정보가 끊겼는지 확인합니다.

라운드가 끝났는지 판단하는 곳과 남은 시간을 보여 주는 곳을 분리합니다. 클라이언트 UI가 서버 전용 GameMode를 찾는 방식은 멀티플레이에서 성립하지 않습니다.

~~~text
서버 GameMode: 승리 조건 판정
        ↓
서버 GameState: 공개할 경기 상태 갱신
        ↓ 복제
클라이언트 GameState → 로컬 UI 표시

각 실행 인스턴스의 GameInstance: 맵을 넘어 유지할 로컬 데이터
~~~

| 데이터 | 가능한 위치 | 먼저 물어볼 조건 |
|---|---|---|
| 라운드 종료 규칙 | GameMode | 서버가 결정하는가 |
| 모두가 보는 점수판 | GameState 또는 PlayerState | 누구에게 복제할 것인가 |
| 로컬 옵션 | GameInstance 계열 서비스 | 맵 전환 뒤에도 필요한가 |

정답 암기보다 요구조건을 설명하세요. GameInstance에 넣었다고 네트워크 동기화가 되지는 않습니다. 데이터 위치와 복제 정책은 각각 필요합니다.

근거: [Gameplay Framework](https://dev.epicgames.com/documentation/en-us/unreal-engine/gameplay-framework-in-unreal-engine).

<a id="ue-06"></a>
## 6. 재스폰에서 바뀌는 몸과 남는 플레이어

<!-- EXPLANATION_PASS -->

**몸이 바뀌어도 남아야 하는 정보**

플레이어가 죽고 새 캐릭터를 받거나 차량에 탑승하면 조종 대상인 Pawn이 바뀔 수 있습니다. Pawn에 넣은 데이터를 모두 그대로 유지하려 하면 이미 종료한 몸과 새 몸의 상태가 섞입니다.

Controller는 조종 연결을 맡고 PlayerState에는 점수처럼 플레이어에 귀속되는 상태를 둘 수 있습니다. 새 Pawn은 자신의 체력과 이동 상태를 초기화합니다. 실제로 무엇을 남길지는 게임 규칙에 따라 정하며 PlayerState라는 이름만으로 모든 여행·접속 상황의 영구 보존이 해결되지는 않습니다.

다른 플레이어의 점수를 보려면 그 사람의 Controller가 내 클라이언트에 있을 것이라고 가정하지 않습니다. 공개할 상태의 복제 경로를 읽습니다. 재스폰 실습은 먼저 Pawn 식별자가 바뀌는지, 다음으로 점수 소유자가 유지되는지를 각각 확인합니다.

죽은 Pawn을 새 Pawn으로 교체할 때 점수까지 사라지면 데이터가 잘못된 수명에 묶였을 수 있습니다.

~~~text
PlayerController ── Possess ──> Pawn A
       │                         체력 0 → 종료
       └──────── Possess ──> Pawn B
                                 새 체력

PlayerState: 같은 플레이어의 점수 등
~~~

실습에서는 체력과 점수를 동시에 바꾸지 말고 먼저 Pawn 교체 전후 이름만 관찰합니다. 그 다음 체력 초기화, 마지막으로 점수 유지 조건을 붙입니다. PlayerState도 맵 이동·연결 종료까지 무조건 유지되는 저장 파일은 아닙니다.

<details>
<summary>관찰 클라이언트의 PlayerController를 찾으면 될까요?</summary>

다른 플레이어의 Controller가 모든 클라이언트에 존재하는 것은 아닙니다. 다른 플레이어를 표시할 때 필요한 공개 상태는 PlayerState 등의 복제 경로로 읽도록 설계합니다. 소유자만 필요한 정보와 모두에게 필요한 정보를 구분합니다.

</details>

근거: [Gameplay Framework](https://dev.epicgames.com/documentation/en-us/unreal-engine/gameplay-framework-in-unreal-engine).

<a id="ue-07"></a>
## 7. 문 열기에서 배우는 상태 복제

<!-- EXPLANATION_PASS -->

**복제 변수는 현재 사실을 전달한다**

문을 여는 Multicast는 그 호출을 받는 시점의 연출에 쓸 수 있지만, 늦게 들어온 사용자에게 과거 호출을 재생해 주는 기록은 아닙니다. bOpen 같은 현재 상태를 복제하면 새로 문을 알게 된 클라이언트도 현재 상태에 맞춰 표시할 수 있습니다.

서버가 bOpen을 변경하고 복제 시스템이 관련 클라이언트에 전달하면 OnRep_Open에서 문 표시 함수를 호출하는 구성을 생각할 수 있습니다. C++ 서버의 직접 대입은 클라이언트 수신 콜백과 다른 경로이므로 서버 화면에도 표시가 필요하면 공통 표시 함수를 직접 연결합니다.

모든 값 변경이 각각 전달되는 이벤트 큐는 아닙니다. 빠르게 열었다 닫은 중간 상태까지 반드시 처리해야 한다면 요구 자체를 별도로 설계해야 합니다. 단순 현재 상태 표시와 반드시 처리해야 하는 개별 사건을 먼저 구별합니다.

문이 열렸다는 순간의 연출과 현재 열림 상태를 분리합니다. 늦게 접속한 사용자는 과거 이벤트를 보지 못해도 현재 상태를 알아야 합니다.

~~~cpp
// 복제되는 Actor의 선언 일부:
UPROPERTY(ReplicatedUsing=OnRep_Open)
bool bOpen = false;

UFUNCTION()
void OnRep_Open();
~~~

~~~cpp
// GetLifetimeReplicatedProps 본문 일부:
// #include "Net/UnrealNetwork.h"
Super::GetLifetimeReplicatedProps(OutLifetimeProps);
DOREPLIFETIME(ADoorLab, bOpen);
~~~

Actor의 복제 활성화 등 전체 설정이 필요합니다. OnRep는 클라이언트에서 복제 상태를 표시하는 진입점으로 사용하고, C++ 서버의 일반 대입이 같은 OnRep를 자동으로 호출한다고 가정하지 않습니다. 공통 표시 함수로 서버 표시와 클라이언트 표시를 연결할 수 있습니다.

| 관찰 시점 | 기대할 내용 |
|---|---|
| 서버가 문 상태 변경 | 서버 원본 값이 변경 |
| 기존 클라이언트 | 해당 Actor가 관련 있고 복제되면 상태 반영 |
| 늦은 접속 | 현재 상태 수신 후 표시 |
| 연속 변경 | 모든 중간 값 이벤트가 각각 도착한다고 가정하지 않음 |

근거: [Networking Overview](https://dev.epicgames.com/documentation/en-us/unreal-engine/networking-overview-for-unreal-engine).

<a id="ue-08"></a>
## 8. 요청 권한과 공격 규칙은 두 번 확인한다

<!-- EXPLANATION_PASS -->

**요청이 도착하는 것과 허용되는 것은 다르다**

클라이언트는 자신이 조종하는 Pawn 또는 Controller를 통해 문 열기나 공격을 요청할 수 있습니다. 서버 RPC의 소유 경로가 맞아야 호출이 실행되고, 실행된 뒤에도 거리·상태·쿨다운 등 규칙 검사가 필요합니다.

문을 소유하지 않은 클라이언트가 문 자체의 Server RPC를 호출하는 문제와, 서버가 너무 먼 문 열기를 거부하는 문제는 원인이 다릅니다. 첫 경우는 호출 경로, 두 번째는 게임 규칙입니다. 요청에는 의도를 담고 최종 피해량이나 성공 여부는 서버가 결정하도록 설계합니다.

지연이 있으면 클라이언트가 본 위치와 서버의 현재 위치가 다를 수 있습니다. 단순 서버 현재 위치 판정부터 이해한 뒤 장르 요구에 따라 예측·지연 보상을 추가합니다. 네트워크 소유권 확인이 조작 방지의 모든 규칙을 대신하지는 않습니다.

클라이언트가 서버 RPC를 보낼 수 있는 소유 경로와 서버가 공격을 인정하는 규칙은 다릅니다. 호출이 도착했다는 사실은 공격 유효성의 증거가 아닙니다.

~~~text
의사코드:
소유 클라이언트: 입력 → 자신의 Controller/Pawn을 통해 서버 요청
서버:
  요청한 행동을 지금 할 수 있는가?
  거리·쿨다운·자원·대상은 유효한가?
  유효하면 서버 상태 변경
클라이언트:
  복제된 확정 상태를 표시
~~~

월드의 문 Actor를 클라이언트가 소유하지 않는데 그 Actor에서 Server RPC를 호출하는 예제는 실행 경로부터 막힐 수 있습니다. 소유한 객체를 통해 요청하고 서버가 대상 문을 검증하는 경로를 검토합니다. Client가 보내는 임의 피해량을 그대로 적용하지 않습니다.

확인 순서: RPC 미도착이면 소유·복제·호출 위치 → 도착 후 거부면 규칙 로그 → 서버 값은 맞는데 화면이 다르면 복제·표시 경로. 이 세 문제를 한 번에 고치려고 하지 않습니다.

근거: [RPC 실행 규칙](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-procedure-calls-in-unreal-engine).

<a id="ue-09"></a>
## 9. GAS에서 '회복 행동'의 경로

<!-- EXPLANATION_PASS -->

**왜 체력 변경을 여러 역할로 나누는가**

회복 행동 하나만 있으면 함수에서 체력을 직접 바꾸는 구현도 가능합니다. 하지만 회복량 수정, 지속 회복, 중첩 버프, 취소와 네트워크 요구가 늘어나면 행동마다 같은 규칙을 반복하기 쉽습니다. GAS는 행동 흐름과 효과 정의, 속성 관리의 공통 처리를 나누는 틀을 제공합니다.

회복 예에서는 Ability가 언제 어떤 대상에게 효과를 적용할지 결정하고, GameplayEffect가 적용할 변경을 표현하며, ASC가 적용을 관리합니다. AttributeSet은 Health 같은 속성과 관련 처리 지점입니다. UI는 확정된 속성 변화를 받아 표시하며, 최대 체력 제한 같은 규칙은 프로젝트에서 적절한 처리 경로에 구현해야 합니다.

이 분리가 자동 완성을 뜻하지는 않습니다. Ability 부여, ActorInfo 초기화, Effect 구성, 속성 등록, UI 구독이 필요합니다. 작은 프로젝트에 GAS가 항상 필수인 것도 아닙니다. 여러 행동이 공통 규칙을 공유할 때의 이점과 설정 비용을 함께 설명합니다.

처음에는 공격·애니메이션·예측을 모두 넣지 말고 회복 수치 하나의 흐름을 봅니다.

~~~text
입력
 → ASC에 부여된 Ability 활성화 요청
 → 활성화 조건 검사
 → GameplayEffect 적용
 → AttributeSet의 Health 변화
 → 상태 변경을 받은 UI 갱신
 → Ability 종료
~~~

Ability는 실행 흐름, Effect는 적용할 효과, AttributeSet은 속성 정의와 처리, ASC는 이를 관리하는 연결점입니다. UI가 Effect를 직접 해석해 체력을 계산하는 대신 확정된 속성 변화에 반응하게 설계합니다.

실패 분리: 부여가 안 됨 / 활성화 조건 거부 / Effect 생성 또는 적용 실패 / 속성은 바뀌었지만 UI 구독 없음 / 종료 누락. 각 단계에 한 로그를 두면 'GAS가 안 된다'를 구체적인 질문으로 바꿀 수 있습니다.

직접 답하기: Health는 감소했는데 연출이 안 나오면 피해 계산부터 다시 작성해야 할까요? 속성 변화 증거를 보존하고 표현 단계로 조사 범위를 좁힙니다.

근거: [GAS 구성](https://dev.epicgames.com/documentation/en-us/unreal-engine/gameplay-ability-system-for-unreal-engine).

<a id="ue-10"></a>
## 10. 일반 데이터와 UObject를 함께 사용할 때

<!-- EXPLANATION_PASS -->

**일반 데이터 계산과 엔진 객체 적용을 분리하기**

경로 계산이 오래 걸린다고 작업 스레드에서 Actor를 바로 수정하면 수명과 스레드 접근 문제가 함께 생깁니다. 계산에 필요한 좌표만 복사하면 작업은 엔진 객체 수명과 덜 얽히고, 완료 결과를 게임 스레드에서 적용하는 경계를 만들 수 있습니다.

결과가 도착하면 먼저 현재 요청인지 확인하고, 다음으로 대상이 유효한지 확인합니다. 대상이 살아 있어도 더 최신 목적지 요청이 생겼다면 옛 결과는 버립니다. 약한 참조는 대상 수명을 늘리지 않고 이 확인을 돕습니다.

순수 C++ 버퍼는 vector나 unique_ptr로 관리하고 UObject 참조는 엔진의 지원 방식으로 관리합니다. std::shared_ptr가 원시 주소를 보관할 수 있다는 이유만으로 UObject의 기본 삭제 책임까지 맡기지 않습니다.

경로 탐색 결과처럼 순수 데이터는 일반 C++ 값으로 전달하고, 결과를 적용할 Actor는 약한 참조로 다시 확인하는 식으로 경계를 나눌 수 있습니다.

~~~text
작업 입력: 복사 가능한 숫자·좌표 목록
       ↓
백그라운드 계산: UObject를 임의로 수정하지 않음
       ↓
게임 스레드로 결과 전달
       ↓
요청 세대 확인 + 대상 유효성 확인
       ↓
살아 있는 대상에만 적용
~~~

이는 설계 예이며 구체적인 작업 API 코드는 아닙니다. std::shared_ptr를 썼다고 UObject의 생성·종료 경로가 일반 delete로 바뀌는 것은 아닙니다. 반대로 순수 자료까지 모두 UObject로 만들면 GC와 엔진 의존성을 불필요하게 늘릴 수 있습니다.

꼬리 질문 해설: 로컬 원시 포인터 사용 자체가 금지되는 것은 아닙니다. 장기 보관·비동기 사용·GC 추적이 필요한 문맥인지가 중요합니다.

근거: [Object Pointers](https://dev.epicgames.com/documentation/en-us/unreal-engine/object-pointers-in-unreal-engine).

<a id="ue-11"></a>
## 11. C++ 기본값을 바꿨는데 에디터 값은 그대로인 이유

<!-- EXPLANATION_PASS -->

**에디터의 값이 어디에서 왔는지 추적하기**

C++ 기본값 10을 Blueprint에서 20으로 재정의했다면 C++ 값을 15로 바꿔도 Blueprint의 의도적인 20을 보존할 수 있습니다. 배치 인스턴스에도 별도 재정의가 있을 수 있어 화면에 보이는 값만으로 C++ 변경 실패라고 단정하기 어렵습니다.

먼저 어떤 클래스의 어떤 인스턴스를 보고 있는지 확인한 뒤 C++ 기본값, Blueprint 기본값, 인스턴스 재정의를 순서대로 비교합니다. 새 연습 Blueprint와 기존 Blueprint를 비교하면 저장된 재정의와 코드 기본값을 구분하기 쉽습니다.

EditDefaultsOnly는 기본값 편집 범위를, BlueprintReadOnly는 Blueprint 그래프에서의 접근을 나타냅니다. 편집 가능한 것과 런타임 그래프에서 쓸 수 있는 것은 다른 권한입니다. 복제 역시 별도 지정과 설정이 필요합니다. 지정자 이름 하나로 여러 기능을 묶어 외우지 않습니다.

먼저 세 층을 구분합니다. C++ 클래스 기본값, Blueprint 클래스가 저장한 기본값, 배치 인스턴스의 재정의 값입니다. 어느 층을 보고 있는지 모르면 같은 수치를 계속 수정하게 됩니다.

~~~cpp
// UCLASS의 멤버 선언 일부:
UPROPERTY(EditDefaultsOnly, BlueprintReadOnly, Category="Combat")
float BaseDamage = 10.0f;
~~~

| 지정자 | 설명할 질문 |
|---|---|
| EditDefaultsOnly | 클래스 기본값을 어디서 바꿀 수 있는가 |
| EditInstanceOnly | 배치 인스턴스마다 조정할 값인가 |
| BlueprintReadOnly | Blueprint 읽기와 쓰기를 어떻게 제한하는가 |
| Replicated | 네트워크 설정을 추가로 어떻게 연결하는가 |

UPROPERTY 하나만 붙이면 편집·Blueprint 쓰기·복제가 모두 된다는 뜻이 아닙니다. 지정자들의 목적이 다릅니다.

실습 순서: 새 기본값과 재정의된 값을 각각 확인 → C++ 기본값 변경 → Blueprint 기본값과 배치 인스턴스를 나누어 확인 → 재정의 초기화 전후 비교. 중요한 재정의 값을 무작정 초기화하지 말고 연습 에셋으로 비교합니다.

근거: [Object Handling](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-object-handling-in-unreal-engine).

<a id="ue-12"></a>
## 12. 전역 관리자부터 만들지 않기

<!-- EXPLANATION_PASS -->

**서비스의 범위가 곧 정리의 범위다**

한 Actor에 붙는 인벤토리 기능은 Component로 조합할 수 있습니다. 한 World의 적 목록은 World 수명, 로컬 플레이어의 UI 서비스는 LocalPlayer 수명, 맵을 넘어 유지할 서비스는 GameInstance 수명을 검토합니다.

World가 끝나면 그 월드의 목록도 함께 정리되는 구조가 자연스럽습니다. 더 오래 사는 서비스에 목록을 두면 이전 월드 Actor를 언제 해제하거나 목록에서 제거할지 별도 책임이 생깁니다. 접근하기 쉽다는 이유만으로 모든 것을 GameInstance에 넣으면 이 경계가 흐려집니다.

PIE의 여러 World에서 각 서비스 이름과 소속을 기록해 보세요. 수명이 맞는데도 초기화가 실패하면 참조하는 다른 서비스의 준비 순서를 확인합니다. 클래스를 나눈 것과 의존성을 정리한 것은 별개의 작업입니다.

적 목록은 World마다 다를 수 있고, UI는 로컬 플레이어마다 다를 수 있습니다. '어디서나 접근 가능'보다 '누구와 함께 시작하고 끝나는가'를 먼저 결정합니다.

~~~text
Actor → ActorComponent: 그 Actor의 기능
World → WorldSubsystem: 해당 World 범위 서비스
GameInstance → GameInstanceSubsystem: 맵 전환을 넘는 서비스
LocalPlayer → LocalPlayerSubsystem: 로컬 사용자별 기능
~~~

맵 전환 뒤 서비스가 살아도 이전 Actor가 계속 살아 있다는 뜻은 아닙니다. 장수명 서비스에서 단수명 대상을 추적한다면 등록·해제·약한 참조·World 종료 처리를 설계합니다. 서버 프로세스와 클라이언트 프로세스 사이의 전역 공유도 아닙니다.

실습은 먼저 인스턴스별 식별자를 로그에 남겨 서로 다른 PIE World를 구분합니다. 그 다음 등록 수가 종료 후 줄어드는지 확인합니다. 테스트를 위해 새 클래스를 만들 때는 선택한 Subsystem 부모의 지원 범위와 초기화 API를 사용하는 엔진 버전에서 확인합니다.

근거: [Subsystems](https://dev.epicgames.com/documentation/en-us/unreal-engine/programming-subsystems-in-unreal-engine).

<a id="ue-13"></a>
## 13. UI가 열릴수록 콜백이 늘어나는 버그

<!-- EXPLANATION_PASS -->

**등록한 함수는 누가 호출하는가**

체력 소스가 체력 값을 바꾼 뒤 OnHealthChanged.Broadcast(새 값)를 호출하는 구조를 생각해 봅시다. UI는 미리 AddUObject로 자신의 갱신 함수를 목록에 등록합니다. 등록 순간에 화면 갱신 함수가 실행되는 것이 아니라 나중의 Broadcast가 그 함수를 호출합니다.

FDelegateHandle은 그 등록 항목을 나중에 제거할 표식입니다. 닫을 때 같은 소스에서 Remove하고 핸들을 초기화해야 다시 열 때 구독이 쌓이지 않습니다. 등록 직후 현재 체력을 표시하려면 별도 초기 갱신도 필요합니다. 다음 변경 이벤트가 올 때까지 빈 화면으로 기다리는 문제가 생길 수 있기 때문입니다.

AddUObject의 약한 대상 참조는 객체를 영구히 살려 두지 않습니다. 하지만 숨겨진 UI는 아직 살아 있을 수 있으므로 자동 수명 처리만으로 닫기 구독 해제가 대체되지는 않습니다. Broadcast는 일반적인 동기 호출이며 멀티캐스트 구독 함수의 실행 순서에 의존하지 않습니다.

~~~text
잘못된 흐름:
열기 → 구독
닫기 → 화면만 숨김
다시 열기 → 다시 구독
체력 변경 → 같은 UI에 여러 알림
~~~

해결은 화면 갱신 함수를 빠르게 만드는 것보다 구독 수명부터 맞추는 것입니다.

~~~cpp
// 선언된 native multicast delegate와 FDelegateHandle을 사용하는 예:
HealthHandle = Source->OnHealthChanged.AddUObject(
    this, &UHealthView::RefreshHealth);

// 종료 시 Source가 유효한지 먼저 확인한 뒤:
Source->OnHealthChanged.Remove(HealthHandle);
HealthHandle.Reset();
~~~

Source와 delegate 시그니처는 프로젝트별 선언이 필요합니다. Dynamic delegate는 바인딩·해제 API가 다르므로 위 코드를 그대로 혼용하지 않습니다. AddUObject의 수명 처리와 일반 AddLambda의 this 캡처도 같지 않습니다.

실험: 열기·닫기 3회 후 한 번 변경 → 1회 콜백. 소스가 먼저 종료되는 경우도 재현합니다. 핸들을 저장했지만 잘못된 소스에서 제거하면 구독이 남을 수 있으므로 소스의 정체성도 보관·검증합니다.

근거: [Delegates](https://dev.epicgames.com/documentation/en-us/unreal-engine/delegates-and-lambda-functions-in-unreal-engine).

<a id="ue-14"></a>
## 14. 입력은 들어오는데 캐릭터가 안 움직인다

<!-- EXPLANATION_PASS -->

**입력값이 이동 결과가 되기까지**

W 키 자체가 월드 이동을 수행하는 것은 아닙니다. Mapping Context가 키를 Action에 연결하고 Modifier와 Trigger 처리를 거친 값이 바인딩한 함수로 전달됩니다. 그 함수는 입력 축을 카메라 기준 전방·우측 벡터와 조합해 이동 의도를 만듭니다.

예를 들어 전진 축이 1이고 우측 축이 0이면 카메라 yaw로 만든 전방 방향을 사용합니다. 지상 이동에서 카메라 pitch까지 그대로 포함하면 위아래 성분이 섞일 수 있으므로 원하는 이동 평면을 정합니다. AddMovementInput은 이동 입력을 누적하고 CharacterMovement 같은 이동 구현이 이를 소비합니다. 모든 Pawn이 자동으로 움직이는 것은 아닙니다.

콜백이 없으면 Context·소유·바인딩을, 값은 있는데 방향이 틀리면 축과 좌표계를, 방향도 맞는데 못 움직이면 이동 모드·충돌·이동 구현을 봅니다. 로그 한 줄에 모두 정상이라고 표시하지 말고 경계별 값을 확인합니다.

적용된 Mapping Context의 키 대응 → Modifier·Trigger 평가 → Action 이벤트와 값 → 바인딩한 함수 → 이동 방향 → 이동 수행을 분리합니다.

~~~cpp
// Axis2D 입력을 읽는 핸들러 일부:
const FVector2D Axis = Value.Get<FVector2D>();
const FRotator YawOnly(0.0f, Controller->GetControlRotation().Yaw, 0.0f);
const FVector Forward = FRotationMatrix(YawOnly).GetUnitAxis(EAxis::X);
const FVector Right = FRotationMatrix(YawOnly).GetUnitAxis(EAxis::Y);
AddMovementInput(Forward, Axis.Y);
AddMovementInput(Right, Axis.X);
~~~

Controller가 유효하고 해당 Pawn이 입력을 받는 전제입니다. 이 예제의 X=좌우, Y=앞뒤 축 계약에 맞게 Input Modifier를 구성해야 합니다. 축 배치가 다른 Context에 그대로 붙이지 않습니다.

확인 순서: Axis 값 출력 → 카메라 Yaw 출력 → Forward·Right 시각화 → 이동 제한·MovementMode 확인. 입력값에 DeltaTime을 무조건 곱하지 않습니다. 사용한 이동 API가 값을 어떤 방식으로 소비하는지 확인합니다.

반례: 카메라를 위로 올렸을 때 전진 속도가 줄면 pitch까지 포함한 전방을 바닥 이동에 썼는지 조사합니다.

근거: [Enhanced Input](https://dev.epicgames.com/documentation/en-us/unreal-engine/enhanced-input-in-unreal-engine).

<a id="ue-15"></a>
## 15. 검 끝의 현재 위치만 보면 놓치는 구간

<!-- EXPLANATION_PASS -->

**빠르게 휘두르는 검이 대상을 놓치는 이유**

프레임 A에서는 검 끝이 적 왼쪽, 프레임 B에서는 오른쪽에 있으면 현재 위치만 검사하는 방법은 중간 통과를 놓칠 수 있습니다. 이전 위치와 현재 위치 사이를 질의하면 그 사이를 보완할 수 있습니다.

Line Trace는 선을 검사하고 Sweep은 구나 캡슐 같은 형상의 이동 구간을 검사합니다. 검 전체를 덮으려면 검 끝 한 점만이 아니라 날의 여러 위치를 샘플링할 수 있습니다. 그래도 회전 곡선과 두 점 사이 직선은 다르므로 빠른 회전에서는 오차가 남습니다.

여러 샘플이 한 적을 맞혀도 한 공격에서 한 번만 피해를 주려면 이미 처리한 대상을 공격별로 기록합니다. 공격 시작 때 새 기록을 만들고 종료 때 판정을 끕니다. 디버그 선은 질의 경로를 보여 줄 뿐 실제 충돌 채널과 응답 설정의 정답을 보증하지 않습니다.

~~~text
이전 프레임 검 끝 A ── 적 ── 현재 프레임 검 끝 B

B에서만 검사: 적을 이미 지나쳤을 수 있음
A→B 검사: 두 위치 사이를 검사
회전하는 검 전체: 끝점 하나만으로는 여전히 빈틈 가능
~~~

~~~cpp
// Actor 메서드 안의 질의 예. PreviousTip, CurrentTip은 같은 월드 공간.
FHitResult Hit;
FCollisionQueryParams Params;
Params.AddIgnoredActor(this);
const bool bHit = GetWorld()->SweepSingleByChannel(
    Hit, PreviousTip, CurrentTip, FQuat::Identity,
    ECC_Visibility, FCollisionShape::MakeSphere(5.0f), Params);
if (bHit && Hit.GetActor())
{
    // 공격 단위 중복 검사 후 서버 판정 경로에 전달.
}
~~~

Visibility는 설명용 채널이며 실제 전투 채널과 충돌 응답을 설계해야 합니다. Single 결과가 필요한지 Multi가 필요한지, blocking hit와 overlap의 반환 규칙도 해당 API에서 확인합니다. 채널 설정과 샘플링은 서로 다른 정확도 조건입니다.

실습: 정지→느린 이동→빠른 이동 순서로 선과 구를 표시합니다. 같은 적을 여러 프레임 맞힐 때 공격당 피해 횟수를 따로 세세요. 샘플을 늘리기 전에 무엇을 놓쳤는지 시각화합니다.

근거: [Traces](https://dev.epicgames.com/documentation/en-us/unreal-engine/traces-in-unreal-engine---overview).

<a id="ue-16"></a>
## 16. 애니메이션이 중단되어도 공격 상태는 정리되어야 한다

<!-- EXPLANATION_PASS -->

**재생 성공과 공격 상태 종료를 따로 보기**

Montage 재생을 요청해도 AnimGraph의 Slot 경로가 최종 포즈로 연결되지 않으면 원하는 모습이 보이지 않을 수 있습니다. 반대로 모션이 보이더라도 공격 판정과 이동 제한을 누가 끄는지는 별도 로직입니다.

Notify로 판정 시작·종료 시점을 연결할 수 있지만 피격이나 사망으로 재생이 중단되면 마지막 시점까지 도달하지 않을 수 있습니다. 정상 종료와 중단 모두가 공통 정리 함수로 이어지고, 여러 종료 알림이 와도 판정 해제와 구독 해제가 한 번의 종료와 같은 결과를 내도록 설계합니다.

Root Motion은 애니메이션 루트 이동을 추출해 캐릭터 이동에 사용하는 과정입니다. 메시만 전진하는지 캡슐도 움직이는지 나누어 관찰하면 애니메이션 표현 문제와 이동 적용 문제를 구별할 수 있습니다. 네트워크에서는 권위와 보정 설정도 별도로 필요합니다.

공격 상태와 애니메이션 상태를 한 bool로 대충 묶으면 취소·연속 공격에서 오래된 콜백이 새 공격을 끝낼 수 있습니다.

~~~text
의사코드:
StartAttack:
  AttackId 증가
  공격 상태 시작
  Montage 콜백은 이 AttackId를 기억

FinishAttack(callbackId):
  callbackId가 현재 AttackId와 다르면 오래된 콜백 → 무시
  이미 종료했다면 → 무시
  판정 비활성화 / 이동 제한 해제 / 구독 정리 / 상태 종료
~~~

정상 완료뿐 아니라 중단·사망도 같은 정리 원칙으로 합류하게 합니다. 결과가 두 번 들어와도 중복 피해나 상태 역전이 없어야 합니다. 종료 이벤트의 구체적 순서와 Montage delegate 시그니처는 실제 엔진 버전에서 확인합니다.

| 실험 | 기대 조건 |
|---|---|
| 정상 완료 | 공격 상태 해제 |
| 다른 행동으로 중단 | 판정·이동 제한 해제 |
| 공격 중 사망 | 사망 후 피해 판정 없음 |
| 즉시 다음 공격 | 이전 콜백이 새 공격 종료시키지 않음 |

메시만 이동하고 캡슐은 남는 경우 Root Motion 추출·적용 경로를 따로 관찰합니다. 애니메이션 성공과 게임플레이 정리 성공은 별도입니다.

근거: [Montage](https://dev.epicgames.com/documentation/en-us/unreal-engine/animation-montage-in-unreal-engine), [Root Motion](https://dev.epicgames.com/documentation/en-us/unreal-engine/root-motion-in-unreal-engine).

<a id="ue-17"></a>
## 17. 늦게 도착한 로딩 결과가 최신 선택을 덮는 버그

<!-- EXPLANATION_PASS -->

**로드 완료가 곧 적용 허가는 아니다**

무기 A 아이콘을 요청한 뒤 B로 바꾸면 B의 요청이 최신입니다. A가 더 늦게 완료되었다는 이유로 화면에 적용하면 최신 선택을 과거 결과가 덮습니다. 로딩 성공 여부와 지금 필요한 결과인지를 나눠야 합니다.

요청 때 번호를 저장하고 완료 시 현재 번호와 비교하면 오래된 결과를 버릴 수 있습니다. 요청자가 이미 종료됐으면 UI 접근도 하지 않습니다. 취소를 요청했더라도 완료와 경합할 수 있으므로 적용 시점 검사를 함께 둡니다.

Soft 참조는 경로를 보관하며 동기 또는 비동기 로드는 별도 요청입니다. 완료 뒤 계속 사용할 에셋은 추적되는 강한 참조나 적절한 로드 핸들로 유지해야 합니다. PIE에서 미리 로드된 에셋이 패키지에서 포함·로드되는지까지 확인해야 배포 동작을 판단할 수 있습니다.

~~~text
시각 1: 무기 A 요청 (요청 번호 1)
시각 2: 무기 B 요청 (요청 번호 2)
시각 3: B 완료 → 현재 번호 2와 일치 → 표시
시각 4: A 완료 → 현재 번호 2와 다름 → 폐기
~~~

로딩 완료만 검사하면 A가 최신 선택을 덮습니다. 대상 수명과 요청의 최신성은 별도의 조건입니다.

~~~text
의사코드:
OnLoaded(requestId, weakView, asset):
  view 유효성 확인
  requestId == view.currentRequest 확인
  asset 로딩 성공 확인
  view가 사용할 기간의 강한 참조/핸들 유지
  표시 갱신
~~~

실패 시 대체 이미지를 유지할지 재시도할지 정합니다. 요청 취소만으로 늦은 콜백이 절대 오지 않는다고 가정하지 말고 완료 경로도 방어합니다. 동기 로딩을 Soft 참조로 감쌌다고 비동기가 되지는 않습니다.

실습: A·B 선택 순서와 완료 순서를 의도적으로 달리하고, 완료 전에 UI를 닫습니다. 에디터에서 이미 로드된 에셋인지 기록하고 패키지에서도 확인해야 합니다.

근거: [Asynchronous Loading](https://dev.epicgames.com/documentation/en-us/unreal-engine/asynchronous-asset-loading-in-unreal-engine).

<a id="ue-18"></a>
## 18. GAS 실패 경로를 끝까지 따라가기

<!-- EXPLANATION_PASS -->

**Ability 실행은 시작보다 종료까지가 한 단위다**

Ability를 부여했다는 것은 실행할 수 있는 항목을 ASC가 알고 있다는 뜻이지 현재 활성화되어 있다는 뜻은 아닙니다. 활성화 조건을 통과한 뒤 실행 흐름에서 Commit을 호출해 비용·쿨다운 적용의 성공을 처리하고 작업을 진행합니다.

Commit 실패면 게임 규칙에 맞게 종료해야 합니다. 성공 후 Montage가 실패했다면 이미 소비한 비용을 돌려줄지 결정하고 정리해야 합니다. EndAbility를 호출했다고 이전 비용이나 적용한 모든 효과가 자동 환불되는 것은 아닙니다.

비동기 Task의 완료·취소·중단에서 어떤 경로가 종료로 이어지는지 그려 보세요. 활성 상태와 차단 태그, 외부 콜백이 남으면 다음 실행에 영향을 줄 수 있습니다. 재스폰으로 Avatar가 바뀌는 설계에서는 ASC가 남더라도 새 Pawn을 가리키도록 초기화되는 경로가 필요합니다.

Ability의 성공 로그 하나보다 활성화 요청부터 종료까지 이어지는 경로가 중요합니다.

~~~text
부여 → 활성화 조건 → Commit 성공?
                    ├─ 아니오 → 실패 종료
                    └─ 예 → 실행/Task
                              ├─ 완료 → 정상 종료
                              ├─ 취소 → 취소 정리와 종료
                              └─ 실행 실패 → 정책 처리와 종료
~~~

Commit 이후 애니메이션 시작이 실패했을 때 비용 환불 여부는 게임 규칙입니다. 환불 정책을 명시하고, 수치를 임의로 직접 더해 복제·효과 정책과 충돌하지 않게 해야 합니다. 일반적인 자동 환불을 가정하지 않습니다.

~~~text
관찰 로그 양식(예상 형식이며 실제 실행 로그가 아님):
AbilityId / Owner / Avatar / Role
Activate accepted 또는 rejected + 이유
Commit success 또는 failed
Task completed 또는 canceled
EndAbility + 종료 이유
~~~

실습은 자원 소모만 성공시킨 다음 부족 조건을 추가합니다. 그 후 Task 취소를 붙이고 마지막에 네트워크를 추가합니다. ASC가 PlayerState에 남고 Pawn이 교체되는 설계라면 새 Avatar로 초기화되는 경로와 클라이언트 복제 도착 시점을 확인합니다.

면접 답변에서 구분할 것: Ability가 부여되어 있음, 실행 가능함, 실행 중임, 종료됨은 서로 다른 상태입니다. ‘한 번 됐다’보다 실패 후 다시 실행되는지까지 증명해야 합니다.

근거: [GAS](https://dev.epicgames.com/documentation/en-us/unreal-engine/gameplay-ability-system-for-unreal-engine).


## GAS 세부 참고

- [ASC와 Attribute 연결](https://dev.epicgames.com/documentation/unreal-engine/gameplay-ability-system-component-and-gameplay-attributes-in-unreal-engine)
- [Ability Task의 비동기 실행과 종료](https://dev.epicgames.com/documentation/unreal-engine/gameplay-ability-tasks-in-unreal-engine)

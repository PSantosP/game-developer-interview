# 개념 문헌 검토 — 2026-09-11

[통합 README와 복습 목차](../README.md#recall-guide)

이번 검토는 **설명과 질문의 전제를 문헌에 대조하는 작업**입니다. 컴파일, 예제 실행, Unity·Unreal 에디터, PIE, 네트워크, 성능 측정은 실행하지 않았습니다. 이전 C++ 실행 기록은 [검증 기록](verification.md)에 별도 보존합니다.

## 범위와 판단 방법

원본의 기술 주제 56개, C++ 15개, Unreal 18개, 공통 기초 3개의 본문·회상 설명을 검토 범위로 삼았습니다. 연결된 C++ 예제 해설, Unreal 구현 흐름, 12차시 과제와 두 질문집도 개념 및 질문 전제 관점에서 살폈습니다. 개인적인 채용 조언은 기술적 사실이나 출제 확률의 보장으로 취급하지 않습니다.

공식 매뉴얼·언어 문서·제작사 소스·저자 공개 교재를 우선했습니다. 아래는 중요한 오류와 조건을 대조한 기록이며 모든 외부 링크의 전체 내용을 검증했다는 뜻은 아닙니다. 문서에 없는 게임별 선택은 설계 예시로 유지하고, 기기 성능·실행 순서의 관찰 결과는 나중에 확인할 실습으로 남겼습니다.

Unity는 주로 **6.0(6000.0)** 문서, Addressables는 **2.3** 문서를 사용했습니다. 기존 버전별 C# 지원 링크는 해당 버전의 범위로 읽습니다. Epic의 버전 미지정 링크는 검토 시 **5.8**로 열리는 페이지가 있어, UE5 공통 개념과 개별 버전의 API·설정을 구분했습니다. C++ 자료는 C++17 기초를 유지하며 갱신되는 작업 초안의 새 기능을 C++17 지원이라고 소개하지 않습니다.

## 주요 정정

| 주제 | 고친 오해 또는 빠진 조건 | 대조 근거 |
|---|---|---|
| 값/참조 타입 | 참조 타입도 기본 인자는 값 전달. ref 등의 전달 규칙은 별개 | [C# 매개변수](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/method-parameters) |
| 얕은 복사 | 단순 참조 대입과 새 객체의 얕은 복제 구분 | [MemberwiseClone](https://learn.microsoft.com/en-us/dotnet/api/system.object.memberwiseclone?view=net-10.0) |
| .NET GC | 누수 방지 보장·보편적 3세대·전체 시간 중단 일반화 제거 | [GC 기초](https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals) |
| Unity GC | 보수적/점진적 분류 구분, 점진적 모드는 중단 분산 | [수집 모드](https://docs.unity3d.com/6000.0/Documentation/Manual/performance-incremental-garbage-collection.html), [관리 메모리](https://docs.unity3d.com/6000.0/Documentation/Manual/performance-managed-memory-introduction.html) |
| Lifecycle | 활성 조건, FixedUpdate의 0회/다회 실행과 시간 보정 조건 | [실행 순서](https://docs.unity3d.com/6000.0/Documentation/Manual/execution-order.html), [FixedUpdate](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/MonoBehaviour.FixedUpdate.html) |
| 직렬화 | 메모리 재배치와 구분, 일반 클래스·필드·관리 참조 규칙 | [Unity 직렬화](https://docs.unity3d.com/6000.0/Documentation/Manual/script-serialization-rules.html) |
| 비동기 | async의 자동 병렬화 부정, 실행 위치와 취소 구분 | [완료와 재개](https://docs.unity3d.com/6000.0/Documentation/Manual/async-awaitable-continuations.html) |
| Awaitable·UniTask | 풀링·단일 소비·스레드 전환·재사용 조건 | [Awaitable](https://docs.unity3d.com/6000.0/Documentation/Manual/async-awaitable-introduction.html), [UniTask](https://github.com/Cysharp/UniTask) |
| Unity null | 관리 래퍼와 엔진 객체, 지연 파괴, delegate 검사와 구분 | [Destroy](https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Object.Destroy.html) |
| delegate·클로저 | 타입 호환과 인스턴스 생성/전달 구분, 캡처가 항상 새 할당이라는 단정 제거 | [C# delegate](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/using-delegates) |
| 컬렉션 | SortedDictionary는 제네릭, List와 ArrayList 구분, 성능 순위 일반화 제거 | [SortedDictionary](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.sorteddictionary-2) |
| Addressables | 핸들 해제와 메모리 반환 구분, 원격 콘텐츠와 코드 업데이트 구분 | [메모리 관리](https://docs.unity3d.com/Packages/com.unity.addressables@2.3/manual/MemoryManagement.html) |
| 프로파일러 | WaitForPresent 하나만으로 GPU 병목 확정하지 않음 | [프로파일러 마커](https://docs.unity3d.com/6000.0/Documentation/Manual/profiler-markers.html) |
| 배칭 | SRP Batcher와 드로우 수 감소 구분, 메모리·FPS 이득 보장 제거 | [SRP Batcher](https://docs.unity3d.com/6000.0/Documentation/Manual/SRPBatcher.html) |
| UGUI Image | Source Image=None이면 무조건 개별 드로우라는 전제 철회 | [Image.mainTexture 소스](https://github.com/Unity-Technologies/uGUI/blob/main/com.unity.ugui/Runtime/UGUI/UI/Core/Image.cs) |
| Mip | 0이 원본이고 번호가 커질수록 축소, 약 1/3 추가 용량의 조건 | [Mip 레벨](https://docs.unity3d.com/6000.0/Documentation/Manual/texture-mipmaps-introduction.html) |
| DI | 생성자 전달과 자동 컨테이너 기능·제어 반전 구분 | [Fowler — DI](https://martinfowler.com/articles/injection.html) |
| 프로세스·스레드 | 전부 복사/스택만 복사라는 정의 제거 | [Processes and Threads](https://learn.microsoft.com/en-us/windows/win32/procthread/about-processes-and-threads) |
| 단편화·페이징 | 디스크 스와핑과 페이징 구분, 풀의 무단편화 보장 제거 | [Free-Space Management](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-freespace.pdf), [Paging](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-paging.pdf) |
| Mutex·Semaphore | 소유권과 허가 카운트 구분 | [Semaphores](https://pages.cs.wisc.edu/~remzi/OSTEP/threads-sema.pdf) |
| 교착·스케줄링 | 비선점 의미·복구 정책·단순 RR 모형의 조건 | [Concurrency Bugs](https://pages.cs.wisc.edu/~remzi/OSTEP/threads-bugs.pdf), [Scheduling](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-sched.pdf) |
| C++ 수명·소유권 | 핵심 설명 유지, 소멸 중 규칙과 암시적 수명 일반화 경계 보충 | [Lifetime](https://eel.is/c++draft/basic.life), [Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines) |
| vector | 삭제 위치 자체의 무효화와 기존 end 반복자 명시 | [vector modifiers](https://eel.is/c++draft/vector.modifiers) |
| UObject·프레임워크 | 생성·도달 가능성·Outer·Actor 종료·공유 상태의 구분 유지 | [Object Pointers](https://dev.epicgames.com/documentation/en-us/unreal-engine/object-pointers-in-unreal-engine), [Actor Lifecycle](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-actor-lifecycle), [Framework](https://dev.epicgames.com/documentation/en-us/unreal-engine/gameplay-framework-in-unreal-engine) |
| Unreal RPC | 반환값 없음, 호출 주체·소유 연결·Multicast 전파 조건 보충 | [RPC](https://dev.epicgames.com/documentation/en-us/unreal-engine/remote-procedure-calls-in-unreal-engine) |
| 입력 | Context 적용 후 Modifier/Trigger 평가라는 흐름으로 정정 | [Enhanced Input](https://dev.epicgames.com/documentation/en-us/unreal-engine/enhanced-input-in-unreal-engine) |
| 로딩·GAS | 경로/유지 분리와 종료/환불 분리 유지·보충 | [비동기 로딩](https://dev.epicgames.com/documentation/en-us/unreal-engine/asynchronous-asset-loading-in-unreal-engine), [GAS](https://dev.epicgames.com/documentation/en-us/unreal-engine/understanding-the-unreal-engine-gameplay-ability-system) |

## 다른 분야의 확인 기준

- **DB·네트워크:** 키·참조 무결성은 [PostgreSQL 제약 문서](https://www.postgresql.org/docs/current/ddl-constraints.html), 계층 구분은 [OSI 설명](https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/)과 대조했습니다. DBMS 제약의 세부 동작과 OSI 모형을 모든 제품의 구현으로 일반화하지 않습니다.
- **UI·AI·저장·서비스:** Unreal 질문집은 [UMG](https://dev.epicgames.com/documentation/en-us/unreal-engine/optimization-guidelines-for-umg-in-unreal-engine), [Behavior Tree](https://dev.epicgames.com/documentation/en-us/unreal-engine/behavior-tree-in-unreal-engine---overview), [SaveGame](https://dev.epicgames.com/documentation/en-us/unreal-engine/saving-and-loading-your-game-in-unreal-engine), [Subsystem](https://dev.epicgames.com/documentation/en-us/unreal-engine/programming-subsystems-in-unreal-engine)의 책임과 한계를 기준으로 읽습니다. 원자적 저장·성능 향상·AI 성공을 API 이름만으로 보장하지 않습니다.
- **애니메이션:** [Root Motion](https://dev.epicgames.com/documentation/en-us/unreal-engine/root-motion-in-unreal-engine) 추출과 이동 적용을 구분합니다. 중단 시 정리와 공격 정책은 구현해야 할 설계이며 엔진이 자동으로 완성해 주는 기능이 아닙니다.

## 공부할 때 남겨 둘 구분

1. **개념:** 소유권·값 전달·실행 위치·상태 복제의 뜻을 설명합니다.
2. **조건:** 런타임·엔진 버전·객체 수명·네트워크 역할을 덧붙입니다.
3. **실습:** 할당량·프레임 시간·실제 콜백·빌드 성공은 나중에 직접 확인합니다.

원본 Span 예제의 공백 인덱스, 생성자 대소문자, 익명 메서드 매개변수 표기에는 읽기상 문제를 표시했습니다. 이 예제들을 컴파일 성공본으로 제시하지 않습니다. 실습 질문은 삭제하지 않았으며 예상 결과를 이미 얻은 성과로 바꾸지 않았습니다.

Unity 질문집의 읽기 링크는 정정된 이 포크의 README를 가리키도록 바꿨습니다. 설명이 고쳐졌는데 질문집에서 다시 수정 전 원문으로 넘어가는 혼선을 줄이기 위한 변경입니다.

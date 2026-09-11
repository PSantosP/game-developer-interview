# 게임 수학·렌더링·성능

[학습 안내](README.md) · [차시별 질문·실습](questions.md)

이 문서는 C++·Unreal 실습에서 함께 사용할 공통 개념을 보충합니다. 작성 기준과 출처 표시는 학습 안내를 따릅니다.

<a id="math"></a>
## 위치·방향·회전

> “적이 내 앞에 있는지 어떻게 판정하나요?”

- 위치 두 개의 차 `Target - Origin`은 원점에서 목표까지의 방향과 거리를 담습니다. 길이를 1로 만들면 거리 정보를 제거한 단위 방향이 됩니다. 거리가 0에 가까운 경우도 처리합니다.
- 단위 전방 벡터와 단위 목표 방향의 내적은 두 방향 사이 각도의 코사인입니다. 시야 반각의 코사인과 비교하면 원뿔 범위를 검사할 수 있습니다. 내적에 거리 제한이 포함되지는 않습니다.
- 외적은 두 벡터에 수직인 방향을 만들며 입력 순서를 바꾸면 부호가 바뀝니다. 좌우 판정은 좌표계와 기준 축을 명시합니다.
- 위치 변환에는 이동이 필요하지만 방향에는 평행이동을 적용하지 않습니다. 로컬·월드·카메라 공간 중 어느 공간에서 계산하는지 먼저 정합니다.
- 회전 합성은 순서에 영향을 받습니다. 쿼터니언은 회전 표현과 보간에 유용하지만 곱셈 순서나 최단 경로 같은 조건을 없애 주지는 않습니다.

<details>
<summary>수치 예제 — 전방 60도 안에 있는가?</summary>

전방이 `(1, 0, 0)`, 같은 위치에서 본 목표 방향이 `(1, 1, 0)`이면 정규화한 목표 방향과의 내적은 약 `0.707`입니다. 전체 시야각이 60도라면 반각 30도의 코사인 약 `0.866`과 비교하므로 범위 밖입니다. 전체 각도와 반각을 혼동하지 않습니다.

</details>

추가 질문: 목표가 바로 위에 있을 때도 수평 시야 검사에서 같은 공식을 그대로 쓰겠습니까?

참고: [Microsoft — Vector dot product](https://learn.microsoft.com/en-us/windows/win32/api/directxmath/nf-directxmath-xmvector3dot), [Quaternion multiplication](https://learn.microsoft.com/en-us/windows/win32/api/directxmath/nf-directxmath-xmquaternionmultiply). 위 예제는 기본 벡터 연산을 적용한 계산입니다.

<a id="rendering"></a>
## 렌더링과 병목

> “드로우 콜을 줄이면 FPS가 반드시 오르나요?”

- CPU에서 그릴 데이터를 준비하고 GPU가 기하 처리·래스터화·셰이딩 등의 작업을 수행합니다. 실제 엔진은 여러 패스와 병렬 작업을 사용하므로 하나의 직렬 함수 호출처럼 보지 않습니다.
- 드로우 호출과 상태 변경 비용을 줄여도 GPU의 픽셀 처리나 다른 CPU 작업이 병목이면 전체 프레임 시간 변화가 작을 수 있습니다.
- 해상도, 오버드로우, 그림자, 재질 복잡도, 지오메트리 수는 서로 다른 비용에 영향을 줍니다. 한 번에 한 조건을 바꾸어 가설을 확인합니다.
- LOD·인스턴싱 등도 메모리·전환 품질·컬링 단위와 함께 판단합니다. 특정 기능을 켰다는 사실은 최적화 결과가 아닙니다.

추가 질문: 해상도를 낮춰도 프레임 시간이 거의 같다면 다음에 어떤 CPU/GPU 구간을 확인하겠습니까?

참고: [Microsoft — Graphics pipeline](https://learn.microsoft.com/en-us/windows/win32/direct3d11/overviews-direct3d-11-graphics-pipeline), [Epic — Unreal Insights](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-insights-in-unreal-engine). 병목 비교는 측정을 위한 가설이며 측정 결과를 단정하지 않습니다.

<a id="profiling"></a>
## 성능 측정과 패키징

> “에디터에서 잘 되면 배포 가능한 상태인가요?”

- PIE, 독립 실행, 패키지 실행은 별도로 확인합니다. 쿠킹에서 에셋이 빠지거나 시작 맵·설정 차이로 실행 결과가 달라질 수 있습니다.
- 빌드·쿠킹·스테이징·패키징 단계와 런타임 실패를 구분합니다. 마지막 실패 문장만 보지 말고 최초 원인 로그를 찾습니다.
- 성능은 장면, 장치, 빌드 설정, 해상도, 개체 수, 실행 시간을 맞춰 비교합니다. 평균 FPS 외에 프레임 시간과 일시적인 지연도 봅니다.
- Insights의 타이밍 등 적절한 트레이스를 이용해 어느 작업이 시간을 쓰는지 확인합니다. 사용한 채널과 기록 구간을 남깁니다.
- 변경 하나를 적용하고 같은 조건에서 재측정합니다. 빨라졌더라도 메모리 증가나 동작 오류가 생겼는지 확인합니다.

추가 질문: 에셋을 미리 로드해 끊김을 줄였을 때 시작 시간과 최대 메모리는 어떻게 달라질까요?

참고: [Epic — Unreal Insights](https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-insights-in-unreal-engine), [Packaging](https://dev.epicgames.com/documentation/en-us/unreal-engine/packaging-your-project).

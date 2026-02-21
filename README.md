# 박태훈 | Unity 클라이언트 프로그래머

> 1998년생, 군필 (의경 만기제대)  
> 📧 hunobas.dev@gmail.com

---

## 핵심 역량 요약

| 역량 | 근거 |
|------|------|
| 사내 프레임워크 적응 및 기여 | My Little Puppy: L-series(LAssetBundle/LCoroutine/LFlexAnimation/LSolidEditor 등) 독자 프레임워크에 빠르게 적응하고 실무 기여 |
| 공통 라이브러리 설계·배포 | ASCII Image UGUI Renderer — UPM 플러그인으로 패키징, 외부 팀도 바로 설치 및 사용 가능 |
| 다양한 코드베이스 분석 | 낯선 코드에서 버그 원인을 추적/수정한 사례 다수 (루트모션 3건, FSM 포함) |
| 기획자 친화적 데이터 기반 설계 | ScriptableObject·UDataTable 기반, 코드 수정 없이 밸런싱 가능한 구조 구축 |

---

## 1. 사내 프레임워크 환경에서의 실무 경험

**My Little Puppy (드림모션, 2025.01~03 / Unity URP 3D)**

- `LAssetBundle`, `LCoroutine`, `LFlexAnimation`, `LUILayerBase`, `LFlexTween`, `Wine(위젯 변수 관리)` 등 사내 전용 시스템 하에서 콘텐츠 구현
- Jenkins 빌드 파이프라인, Slack 에러 알림, OneNote 이슈 트래킹 등 팀 DevOps 루틴에 즉시 적응
- **총 283 커밋** 동안 버그 11건 해결, 주요 콘텐츠 기믹 4건 구현, Steam 데모 11개 국어 출시

---

## 2. 공통 라이브러리 설계 및 배포

![ASCII 변환 예시](https://github.com/user-attachments/assets/1531dd5e-f462-4718-ba35-75a071ef0641)

**ASCII Image UGUI Renderer** — [UPM 플러그인 | GitHub](https://github.com/Hunobas/AsciiImageUGUI-UPM)

인게임 터미널 UI의 아스키 아트 렌더러를 팀 외부에서도 설치해 사용할 수 있도록 **UPM(Unity Package Manager) 플러그인**으로 패키징 및 배포했습니다.

초기 구현은 160×90 그리드 × 4×4 슈퍼샘플(230,400회 픽셀 접근)으로 CPU 27.6ms(프레임 비중 70.4%)를 차지했습니다.

**개선:** 비동기 GPU Readback + 색상 변경 구간에만 RichText 태그 삽입 + 색상 해상도 다운샘플링

| 지표 | Before | After |
|------|--------|-------|
| CPU 시간 | 27.6ms | 2.15ms |
| 프레임 비중 | 70.4% | 3.5% |

> 아트 팀이 프로그래머 호출 없이 인스펙터에서 직접 색상·밝기를 조정할 수 있도록 **에디터 미리보기** 기능 포함

---

## 3. 코드베이스 분석 및 버그 해결

### 루트모션/FSM 버그 3건 (My Little Puppy)

낯선 코드베이스에서 재현 조건을 파악하고 원인을 추적한 사례입니다.

<details>
<summary><b>사례 1: FPS에 따라 루트모션 회전이 달라지는 문제</b></summary>

| 문제 상태 | 정상 상태 |
|-----------|-----------|
| ![문제](https://github.com/user-attachments/assets/c038982c-4e66-4c04-a3c1-a4878d3d48c5) | ![정상](https://github.com/user-attachments/assets/f0d4974a-a54d-4b7d-870c-e7b6ad794d5c) |

**원인:** `ActorPosDir`의 선형 보간과 루트모션 각도 업데이트가 중복 적용  
**해결:** 루트모션 전용 위치/각도 업데이트 메서드를 분리, 외부 보간 로직 제외

</details>

<details>
<summary><b>사례 2: 루트모션 종료 시 캐릭터가 앞으로 튀는 문제</b></summary>

| 문제 상태 | 정상 상태 |
|-----------|-----------|
| ![문제](https://github.com/user-attachments/assets/4e19abda-d7a0-41ea-a84e-c6ffce35ca67) | ![정상](https://github.com/user-attachments/assets/53eb6482-83c2-4f4d-bbe5-2e215c2b7f15) |

**원인:** Walk 상태의 `WalkToIdle` 속도값이 초기화되지 않고 Idle까지 전달됨  
**해결:** 루트모션 상태머신 종료 시점에 Idle 블렌딩 속도를 0으로 초기화

</details>

<details>
<summary><b>사례 3: 컷씬 일시정지 시 NPC 위치가 튀는 문제</b></summary>

| 문제 상태 | 정상 상태 |
|-----------|-----------|
| ![문제](https://github.com/user-attachments/assets/436f5f08-5091-4b73-bb91-3871885ca447) | ![정상](https://github.com/user-attachments/assets/9bb6594b-afca-43f8-b29a-31fe61e320e3) |

**원인:** `PlayableDirector`가 컷씬 모드에선 `LateUpdate`에서 위치를 재조정하지만 일시정지 모드엔 이 처리가 없었음  
**해결:** 일시정지 모드에서도 이전 모드가 컷씬이면 `HandleGameActors()` 호출

</details>

---

## 4. Unity 렌더링 최적화

**목성의 노래 (팀 프로젝트 5인, 2025.06~2026.01)**

400만 버텍스 + 300개 머터리얼로 구성된 씬의 FPS 불안정 문제를 해결했습니다.

<img width="1548" height="591" alt="최적화 전후" src="https://github.com/user-attachments/assets/6b35a453-6a45-4258-9635-3bcff6062e97" />

| 지표 | Before | After |
|------|--------|-------|
| Batches | 2,623 | 910 |
| FPS | 30~60 | 120+ |

**접근:** MeshBaker로 방 단위 텍스처 아틀라스 + 콤바인 메쉬, 오클루전 컬링 설정  
**에디터 확장:** 작업 반복을 줄이기 위해 MeshBaker 자동화 에디터 스크립트 직접 제작

- 📂 [에디터 확장 코드](https://github.com/Hunobas/Song-Of-Jupitor/blob/main/Scripts/Editor/MB3_ApplyCombinedMaterialToSourceObjects.cs)
- 📜 [최적화 개발일지 | Velog](https://velog.io/@po127992/목성의-노래-MeshBaker-최적화-삽질기-텍스처-아틀라스만-vs-콤바인-메쉬까지)

---

## 프로젝트 요약

| 프로젝트 | 엔진 | 기간 | 규모 | 주요 기여 |
|----------|------|------|------|-----------|
| [목성의 노래](https://github.com/Hunobas/Song-Of-Jupitor) | Unity | 2025.06~2026.01 | 5명 | FSM 아키텍처, 렌더링 최적화, ASCII UPM 플러그인 |
| [My Little Puppy](https://ethereal-judo-1f1.notion.site/My-Little-Puppy-1c6486e2cdb980fcbc33f487a01bd7fc) | Unity | 2025.01~03 | 38명 | 사내 프레임워크 기반 콘텐츠 구현, 버그 11건 수정 |
| [TOGU: Planet Survivors](https://github.com/Hunobas/Planet) | Unreal 5.4 | 2025.04~06 | 1명 | 오브젝트 풀링, 데이터 기반 설계, 아키텍처 전체 설계 |

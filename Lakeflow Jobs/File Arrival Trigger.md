## 1. 개요

**File Arrival Trigger**는 외부 스토리지(S3, Azure Blob, GCS 등)에  
**새로운 파일이 도착했을 때 자동으로 Job을 실행**하는 기능입니다.

👉 특징
- 이벤트 기반 실행 (Event-driven)
- 불규칙한 데이터 유입에 적합
- 스케줄 기반보다 효율적

> 데이터가 "언제 올지 모를 때" 사용하는 트리거

---

## 2. 동작 방식

- 새로운 파일을 **약 1분 주기로 탐지**
- 클라우드 스토리지 성능에 따라 지연 가능
- 추가 비용 없음 (스토리지 파일 조회 비용만 발생) :contentReference[oaicite:0]{index=0}

### 모니터링 대상
- Unity Catalog 외부 위치 (External Location)
- Volume 경로
- 특정 Sub-path도 가능

---

## 3. 주요 설정 옵션

### 3.1 Minimum time between triggers
- 이전 실행 종료 후 **다음 실행까지 최소 대기 시간**
- 너무 잦은 실행 방지

👉 사용 예:
- 파일이 매우 자주 들어올 때 실행 빈도 제한

---

### 3.2 Wait after last change
- 파일 도착 후 **일정 시간 기다린 뒤 실행**
- 그 사이 파일이 추가되면 타이머 리셋

👉 사용 예:
- 여러 파일이 한 번에 들어오는 배치 처리

---

## 4. 설정 방법

1. Job 선택
2. **Schedules & Triggers → Add trigger**
3. Type: `File arrival` 선택
4. Storage 경로 입력
5. 옵션 설정 후 저장 :contentReference[oaicite:1]{index=1}

---

## 5. 파일 처리 방식 (중요)

### 5.1 Auto Loader 사용 (권장)

- 신규 파일을 **증분 방식으로 처리**
- **Exactly-once 보장**
- Delta Table 적재에 적합 :contentReference[oaicite:2]{index=2}

👉 핵심 옵션
- `cloudFiles`
- `checkpointLocation`
- `availableNow = True`

---

### 5.2 foreachBatch 사용

- 커스텀 로직 처리 가능
- 단점:
  - **at-least-once 보장**

👉 사용 예:
- 파일 URL 기반 커스텀 처리

---

## 6. 운영 시 주의사항

### 6.1 경로 설계 중요

- Sub-path 사용 시:
  - 상위 경로 변경 영향 받음
  - 메타데이터 증가 → 성능 저하 가능

👉 해결 방법
- 별도 Volume 생성 후 해당 경로만 모니터링 :contentReference[oaicite:3]{index=3}

---

### 6.2 파일 변경도 트리거됨

- 기존 파일 수정 시:
  - "새 파일"로 인식 가능

👉 권장
- Immutable 파일 사용
- Auto Loader로 상태 관리

---

### 6.3 파일 이벤트 미사용 시 제한

| 항목 | 제한 |
|------|------|
| Job 수 | 최대 50개 |
| 파일 수 | 최대 10,000개 |

👉 Sub-path 기준으로 적용됨 :contentReference[oaicite:4]{index=4}

---

### 6.4 존재하지 않는 경로

- S3/GCS에서 경로가 없어도:
  - 에러 발생 ❌
  - 단순히 실행 안 됨

👉 정상 동작 :contentReference[oaicite:5]{index=5}

---

## 7. 언제 사용해야 하는가

### 사용 추천
- 파일 도착 시 바로 처리해야 하는 경우
- 데이터 도착 시점이 불규칙한 경우
- 비용 최적화가 필요한 경우

---

### 사용 비추천
- 파일이 매우 자주 도착 (→ Continuous 고려)
- 정해진 시간에만 처리하면 되는 경우 (→ Scheduled)

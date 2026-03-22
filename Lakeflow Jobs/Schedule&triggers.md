# # Databricks Jobs Trigger 정리

## 1. 개요
Databricks의 **Job Trigger**는 작업(Job)을 **자동으로 실행시키는 조건**을 정의하는 기능입니다.  

다음과 같은 방식으로 실행을 트리거할 수 있습니다:

- 시간 기반 스케줄
- 파일 도착 이벤트
- 테이블 업데이트 이벤트
- 지속 실행 (Continuous)
- 수동 실행 또는 외부 도구 호출

---

## 2. Trigger 유형

| Trigger 유형 | 설명 |
|-------------|------|
| **Scheduled** | 특정 시간/주기(예: 매일 2시)에 실행 |
| **Table update** | 소스 테이블이 업데이트되면 실행 |
| **File arrival** | 지정된 스토리지 위치에 파일이 도착하면 실행 |
| **Continuous** | 작업이 끝나면 즉시 다시 실행 (항상 실행 상태 유지) |
| **Manual (None)** | 사용자가 직접 실행하거나 API/외부 툴로 실행 |

---

## 3. 주요 특징

### 3.1 동시 실행 제한
- 기본적으로 **하나의 Job 실행만 활성 상태 유지**
- 설정을 통해 **동시 실행 수 증가 가능**
- 제한 초과 시 실행은 **스킵됨**

---

### 3.2 File Arrival Trigger 특징
- 파일 도착 여부를 **약 1분 단위로 체크**
- S3, Azure Blob, GCS 등 외부 스토리지 지원
- 불규칙 데이터 도착 처리에 적합

---

## 4. Trigger 설정 방법

1. Job 화면에서 작업 선택
2. **Schedules & Triggers** 섹션 이동
3. **Add trigger** 클릭
4. Trigger 유형 선택
5. 세부 옵션 설정 후 저장

---

## 5. Pause / Resume (중요)

Trigger는 일시 중지 및 재개 가능

### Pause
- 현재 실행 중인 Job은 계속 실행
- 새로운 실행은 발생하지 않음

### Resume
- 기존 설정 그대로 다시 실행 시작
- Continuous Trigger의 경우:
  - 기존 실행이 끝난 뒤 다음 실행 시작

---

## 6. 핵심 정리

- Trigger는 **Job 실행 조건 정의**
- 이벤트 기반 + 시간 기반 모두 지원

### 대표 사용 패턴
- 배치 처리: **Scheduled**
- 데이터 이벤트 기반: **File arrival / Table update**
- 스트리밍/상시 처리: **Continuous**
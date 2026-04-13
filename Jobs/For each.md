# 🚀 Databricks Workflows: For each 루프 활용 가이드

데이터브릭스 워크플로우(Workflows)의 **`For each` 태스크**는 복잡한 병렬 처리 코드를 작성하지 않고도, UI 상에서 동시성(Concurrency)과 동적 파라미터 주입을 우아하게 처리할 수 있는 강력한 기능입니다.

## 1. 💡 For each 기능이란?
단일 태스크(노트북, Python 스크립트 등)를 **JSON 배열(Array of Dictionaries)**의 각 요소(Row)에 대해 반복적으로 실행하는 기능입니다. 

과거에는 `concurrent.futures`를 이용해 파이썬 코드 레벨에서 멀티스레딩을 직접 구현해야 했지만, `For each`를 사용하면 워크플로우 UI만으로 병렬 처리 파이프라인을 구축할 수 있습니다.

**주요 활용 시나리오:**
* 여러 테이블에 대한 동일한 전처리 로직 반복
* 특정 조건에 맞는 사용자 목록에 대해 개별 분석 Job 실행
* 날짜 배열을 받아 각 일자별 데이터 파티션 처리

---

## 2. ⚠️ 핵심 제약 사항 (설계 전 필수 확인)
`For each`를 파이프라인에 도입하기 전에 다음 두 가지를 반드시 고려해야 합니다.

1. **최대 100회 반복 제한 (Max Iterations)**
   * `For each` 루프는 **최대 100번**까지만 반복 실행을 지원합니다.
   * 처리해야 할 데이터(배열의 크기)가 100건을 초과한다면, 이 기능 대신 PySpark의 병렬 처리 로직을 사용하여 하나의 Job 안에서 처리하는 구조로 설계해야 합니다.
2. **동시성 제한 설정 (Maximum Concurrent Runs)**
   * 반복되는 태스크(Job B)의 설정에서 **Maximum concurrent runs** 값을 반드시 늘려주어야 합니다.
   * 기본값인 1로 설정되어 있으면 병렬 실행이 되지 않고 대기(Queue) 상태에 빠지거나 실패하게 됩니다.

---

## 3. 🧪 서로 다른 Job 간의 연결 테스트 방법

> **시나리오:** 완전히 분리된 두 개의 Job(Job A, Job B)을 연결하여 테스트합니다. (Task Value는 같은 Job 내에서만 공유 가능하므로, Databricks SDK와 Job Parameter를 활용합니다.)

### 3.1. Job A: 데이터 생성 및 Job B 트리거 (Python)
파이썬 노트북에 작성할 코드입니다.

```python
from databricks.sdk import WorkspaceClient
import json

# 1. 간단한 샘플 데이터 리스트 생성
test_data = [
    {"user_id": 101, "item": "Notebook", "amount": 2},
    {"user_id": 102, "item": "Mouse", "amount": 5},
    {"user_id": 103, "item": "Keyboard", "amount": 1}
]
```

# 2. JSON 문자열로 직렬화 (Job Parameter는 문자열만 지원)
json_payload = json.dumps(test_data)

# 3. Databricks SDK를 사용하여 Job B 호출
w = WorkspaceClient()
JOB_B_ID = 1234567890 # 대상 Job ID 입력

run_response = w.jobs.run_now(
    job_id=JOB_B_ID,
    job_parameters={"input_rows_array": json_payload}
)
print(f"Job B 트리거 완료! Run ID: {run_response.bind()['run_id']}")

### 3.2. Job B: 워크플로우 설정 (UI)
Job B 화면에서 아래 순서대로 설정합니다.
# 1. Job Parameter 정의: Job B 화면 우측 Job details > Edit parameters에서 Key: input_rows_array, Value: [] 추가

# 2. For each 태스크 생성:
- Type: For each
- Inputs: {{job.parameters.input_rows_array}}

# 3.내부 작업 태스크 매핑: For each 태스크 안에 실제 작업용 노트북 태스크를 추가하고, 하단 Parameters 설정에 매핑
- target_id : {{input.user_id}}
- target_item : {{input.item}}

### 3.3. Job B: 작업 실행 결과 확인 (Python)
```python
# 1. 태스크 파라미터 가져오기
user_id = dbutils.widgets.get("target_id")
item = dbutils.widgets.get("target_item")

# 2. 결과 출력 (Job 실행 결과 로그에서 확인)
print(f"처리 사용자: {user_id}")
print(f"주문 아이템: {item}")
```

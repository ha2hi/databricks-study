# Job
데이터브릭스에서는 Job을 통해 개발한 노트북을 배치 작업으로 실행할 수 있다.  
이 때 UI로 의존성을 정의할 수 있기 때문에 편하긴 하지만 반복된 Task가 많을 때 한땀한땀 Task를 추가하는건 번거롭다.  
이러한 경우 Databricks SDK for Python으로 구현하면 편하다.  

# Create Job(Databricks SDK for Python)
```
from databricks.sdk import WorkspaceClient
from databricks.sdk.service import jobs

w = WorkspaceClient()

# 1. 메타데이터 정의 (테이블명 및 골드 테이블 소스)
silver_tables = ["users", "orders", "products"]
gold_definitions = {
    "sales_report": ["orders", "products"], # orders, users, products가 끝나야 실행
    "user_engagement": ["users"] # users, events가 끝나야 실행
}

# 2. 기존 Job 삭제
job_name = "MY_JOB"
jobs_list = w.jobs.list(name=job_name)
job_ids = []

for j in jobs_list:
    if j.settings.name == job_name:
        job_ids.append(j.job_id)
        
for j in job_ids:
    w.jobs.delete(job_id=j)


tasks = []
# 3. Bronze -> Silver 태스크 생성 (병렬)
for table in silver_tables:
    tasks.append(jobs.Task(
    task_key=f"silver_{table}",
    depends_on=[jobs.TaskDependency(task_key=f"bronze_{table}")],
    notebook_task=jobs.NotebookTask(
    notebook_path="/ETL/Silver_Process",
    base_parameters={"table_name": table} # 노트북 내에서 데이터 체크 로직 포함
    )
    ))

# 4. Gold 태스크 생성 (의존성 부여)
for gold_table, sources in gold_definitions.items():
    tasks.append(jobs.Task(
    task_key=f"gold_{gold_table}",
    depends_on=[jobs.TaskDependency(task_key=f"silver_{src}") for src in sources],
    notebook_task=jobs.NotebookTask(
    notebook_path="/ETL/Gold_Process",
    base_parameters={"gold_name": gold_table}
    )
    ))

# 5. Job 생성
job = w.jobs.create(name=job_name, tasks=tasks)
```
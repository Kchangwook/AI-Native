# 스펙 저장 구조

## 위치와 역할

대상 프로젝트 루트의 `doc/spec/`를 사용한다. 설계·개발에 필요한 첫 기록 시 없으면 생성한다. 조회만 할 때는 생성하지 않는다. YAML은 AI가 필요한 정보만 읽도록 구조화하되 무의미한 축약으로 정보나 근거를 잃지 않는다.

```text
doc/spec/
├── index.yaml
├── features/
│   ├── <feature-id>.yaml
│   └── <domain>/
│       ├── index.yaml
│       └── <feature-id>.yaml
├── drafts/
│   └── <change-id>/
│       ├── project.yaml      # 빈 프로젝트의 기본 설정이 필요한 경우
│       ├── design.yaml
│       ├── plan.yaml
│       └── tasks/
│           └── <task-id>.yaml # task별 상세 설계·확인 상태
└── progress.yaml
```

처음부터 모든 도메인 디렉토리를 만들지 않는다. 관련 기능이 늘어 조회 범위를 나누는 이점이 생기면 도메인별로 분리한다. 공통 상태와 프로젝트 구성은 프로젝트 색인에, 도메인 공통 규칙은 도메인 색인에, 상세 동작은 기능 스펙에 한 번만 기록한다. 색인에 상세 스펙을 복제하지 않는다.

## 조회용 색인

프로젝트 색인은 목적, 확정된 기술·FE/BE 구성, 도메인 또는 기능의 ID·짧은 설명·경로를 제공한다. 도메인 색인은 해당 도메인의 역할, 공통 규칙, 기능의 ID·설명·경로와 관계를 제공한다. 작은 프로젝트에서는 도메인 색인 없이 기능을 직접 가리켜도 된다.

경로는 `doc/spec/` 기준으로 통일한다. 기능과 작업은 파일 경로와 독립적인 고유 ID를 가진다. 아래는 프로젝트 색인의 형식 예시다. 표시된 내용은 실제 프로젝트의 확정 사실로 대체한다.

```yaml
schema_version: 1
project:
  id: todo-app
  purpose: 개인 할 일 관리
  components:
    FE:
      stack: [React]
      entrypoint: src/main.tsx
    BE:
      stack: [FastAPI, SQLite]
      entrypoint: server/main.py
entries:
  - id: todo
    kind: domain
    summary: 할 일 관리
    path: features/todo/index.yaml
```

새 프로젝트에서는 최종 확인 전까지 이 색인을 만들지 않고 구성 정보를 설계 초안에서 관리한다. 실제 확정된 구성이 아니면 예시 값을 복사하지 않는다.

## 기능 스펙

기능당 다음 정보를 저장한다. FE/BE 정보는 해당 계층이 필요한 경우에만 포함한다. 정보가 없으면 추측하지 말고 명시적인 미확인 상태를 사용한다.

```yaml
schema_version: 1
id: todo-create
domain: todo
revision: 1
purpose: 새 할 일 등록
scope:
  included: [제목을 입력하여 할 일 저장]
  excluded: [반복 일정]
user_flow:
  - 제목 입력
  - 등록 요청
  - 저장 결과 표시
rules:
  - id: title-required
    description: 공백만 있는 제목은 거절한다
acceptance_criteria:
  - 정상 입력은 저장된다
  - 잘못된 입력은 오류를 표시한다
dependencies: []
FE:
  routes: [/todos]
  behavior: [등록 요청 중 중복 제출을 막는다]
  source_paths: [src/features/todo/CreateTodo.tsx]
BE:
  apis:
    - method: POST
      path: /todos
      request:
        title: string
      response:
        status: 201
        body: {id: string, title: string, completed: boolean}
      errors:
        - condition: 제목이 비어 있음
          status: 422
  data:
    entity: Todo
    fields: {id: string, title: string, completed: boolean}
  source_paths: [server/todo.py]
design_status: approved
implementation_status: completed
confirmed_change: todo-initial
```

API·데이터 형식은 실제 계약의 필수 여부, 제약, 오류 등 후속 설계에 필요한 정보를 담도록 확장한다. 예시의 스택·경로·응답 코드는 기본 요구사항이 아니다. 관련 소스 위치는 개발에서 확인한 것을 남긴다. 설계에서 이를 채우기 위해 코드를 읽지 않는다.

초안은 `design_status: draft | awaiting_confirmation | approved`, `implementation_status: not_started | in_progress | awaiting_confirmation | completed`를 구분한다. 기능의 기존 확정 ID는 변경 시에도 유지한다. 설계 승인만으로 구현 완료로 표기하지 않는다. 기반 화면·개발 환경도 후속 작업에 필요한 구성과 상태를 확정 스펙에 남긴다.

## 최종 확정과 재구성

최종 확인은 `progress.yaml`에서 특정 변경·설계·계획 버전에 연결한다. 확인 전에는 기존 확정 스펙과 조회용 색인을 유지한다. 여러 작업으로 구성된 요청은 모든 작업 및 전체 검증의 최종 확인 후 함께 반영한다. 부분 반영이 필요하면 사용자와 확정 범위를 먼저 합의한다.

최종 확인 후에만 다음을 수행한다.

1. 현재 확정 스펙이 초안의 기준 버전에서 바뀌었는지 확인한다. 바뀌었다면 덮어쓰지 않고 충돌을 정리해 영향 있는 내용을 다시 확인받는다.
2. 확인받은 초안에서 기능 스펙과 프로젝트 구성의 확정 내용을 반영한다.
3. 필요한 도메인 디렉토리 분리와 색인 갱신을 같은 반영 작업으로 수행한다. ID를 유지하고 경로 참조를 모두 갱신한다.
4. 색인 경로, ID 중복, 기능 의존 관계와 누락을 확인한다. 초안이나 미확정 기능을 조회용 색인에 넣지 않는다.
5. 확정 기록과 진행 상태를 완료 처리한다. 초안은 `published`로 표시해 승인 이력을 보존하되 일반 조회에서 제외한다.

파일 반영이 중단되면 진행 상태의 `publication`에 반영 대상·완료된 파일·남은 파일을 남긴다. 재개 시 일관성을 복원하고, 모두 반영하기 전까지 게시 완료로 보고하지 않는다. 자동 정리는 관련 스펙의 범위 안에서 수행하며 사용자 자료를 임의로 삭제하지 않는다.

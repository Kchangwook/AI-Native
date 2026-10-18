# 작업 분해

승인된 설계와 관련 확정 스펙을 기준으로 계획한다. 코드와 설정을 탐색하지 않는다. 설계가 없거나 승인되지 않았다면 [설계](design.md) 단계부터 수행한다.

## 분해 기준

- 먼저 기능별로 그룹을 만들고, 각 기능 안에 FE/BE 하위 작업을 둔다. FE 전체, BE 전체를 최상위 그룹으로 나누지 않는다.
- 화면이나 서버 등 선행 기반이 없다고 설계에서 확인됐다면 기반 구성 또는 기본 화면을 먼저 배치한다. 스펙에 없다는 이유만으로 미구현으로 단정하지 않는다.
- FE 또는 BE만 필요한 기능은 해당 작업만 만든다. 통합 검증이 필요하면 기능 그룹 안에 연결 검증 작업을 둘 수 있다.
- 기능의 독립된 동작을 구분한다. 추가와 조회처럼 서로 다른 동작은 원칙적으로 각각 기능으로 기록하고 의존 관계로 연결한다. 하나의 사용자 동작을 인위적으로 잘게 쪼개지는 않는다.
- FE/BE 실행 순서는 의존 관계로 정한다. 항상 BE부터 수행하도록 고정하지 않는다.
- 각 작업은 개별적으로 검증·설명·사용자 확인이 가능한 크기로 잡는다.

## 작업 기록

`drafts/<change-id>/plan.yaml`에 기능 그룹과 하위 작업을 기록한다.

```yaml
change_id: todo-initial
design_revision: 1
revision: 1
approval: pending
features:
  - id: todo-create
    goal: 새 할 일을 등록할 수 있다
    acceptance_criteria:
      - 등록한 내용이 저장된다
    tasks:
      - id: todo-create-be
        layer: BE
        scope: 할 일 저장과 추가 API
        depends_on: []
        acceptance_criteria:
          - 정상 입력은 저장하고 결과를 반환한다
          - 빈 제목은 설계에 정한 오류로 거절한다
        verification:
          - 정상·빈 제목 요청의 결과를 확인한다
      - id: todo-create-fe
        layer: FE
        scope: 입력 UI와 추가 API 연결
        depends_on: [todo-create-be]
        acceptance_criteria:
          - 저장 결과와 실패를 화면에 표시한다
        verification:
          - 정상 등록과 오류 표시 흐름을 확인한다
execution_order: [todo-create-be, todo-create-fe]
```

예시는 형식 참고이며 실제 기능과 완료 기준은 승인된 설계에서 가져온다. 기능 ID는 확정 스펙과 동일하게 유지한다. 작업 상태와 실제 검증 결과는 [진행 상태](progress.md)에 기록해 계획과 중복 관리하지 않는다.

## 계획 확인

순환 의존, 빠진 선행 작업, 설계 완료 기준 누락을 확인한다. 설계 자체를 바꿔야 하면 설계 재확인부터 진행한다.

기능별 FE/BE 작업, 실행 순서, 각 확인 지점을 사용자에게 제시하고 계획 확인 대기로 저장한다. 승인 전에는 코드를 읽거나 구현하지 않는다. 승인받은 계획의 버전을 기록한 뒤 전체 구현 요청 범위에 따라 [개발](development.md)을 시작한다. 계획만 요청했으면 종료한다.

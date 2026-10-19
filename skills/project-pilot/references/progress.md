# 진행 상태와 확인

`doc/spec/progress.yaml`은 현재 작업과 마지막 사용자 확인 지점의 기록이다. 설계와 계획 원문은 `drafts/<change-id>/`에 두고 진행 파일에서는 경로·버전으로 연결한다. 조회만으로 새 파일을 만들거나 상태를 변경하지 않는다.

## 기록 형식

```yaml
schema_version: 1
active_change: todo-initial
changes:
  todo-initial:
    request_scope: full_pipeline
    phase: development
    run_state: active
    design: {path: drafts/todo-initial/design.yaml, revision: 1}
    plan: {path: drafts/todo-initial/plan.yaml, revision: 1}
    base_spec_revisions: {}
    approvals:
      design: {status: approved, revision: 1, evidence: 사용자가 설계안을 확인하고 진행에 동의함}
      plan: {status: approved, revision: 1, evidence: 사용자가 작업 계획을 확인하고 진행에 동의함}
      final: {status: pending}
    session_policy: per_task
    workspace_path: /absolute/path/to/project
    chat_creation_authorization: {status: pending}
    handoff: null
    current_task: todo-create-be
    tasks:
      todo-create-be:
        status: in_progress
        chat_id: null
        design: {path: drafts/todo-initial/tasks/todo-create-be.yaml, revision: 1}
        design_confirmation: {status: approved, revision: 1, evidence: 사용자가 task 상세 설계를 확인하고 진행에 동의함}
        completed: [할 일 저장 로직 구현]
        remaining: [추가 API 구현, 검증과 리뷰]
        next_action: 추가 API 구현
        changed_files: [server/todo.py]
        verification: []
        review: {status: pending, findings: []}
        confirmation: {status: pending}
    last_confirmation: {kind: task_design, task_id: todo-create-be, revision: 1}
    awaiting: null
    publication: {status: not_started, completed_paths: [], remaining_paths: []}
```

`request_scope`은 `design_only`, `plan_only`, `full_pipeline` 등 실제 요청 범위를 기록한다. `phase`는 `project_setup`, `design`, `planning`, `task_design`, `development`, `final_verification`, `publication`, `completed`로 구분한다. `run_state`는 `active`, `awaiting_confirmation`, `paused`, `blocked`, `completed`로 구분한다. 중단은 진행하던 `phase`와 작업 상태를 지우지 않는다.

빈 프로젝트에서는 `project_setup`에 `drafts/<change-id>/project.yaml` 경로와 버전을 연결하고, `approvals.project_setup`에 해당 버전의 승인 여부를 기록한다. 기본 설정 확인 대기 상태도 설계 확인 대기와 동일하게 보존·재개한다.

작업 상태는 `not_started`, `designing`, `awaiting_design_confirmation`, `in_progress`, `awaiting_confirmation`, `completed`, `blocked`를 사용한다. `completed`는 검증·리뷰·사용자 확인이 끝난 작업에만 사용한다. 검증에는 수행 방법·결과·미수행 사유를 남긴다.

task의 `design_confirmation`은 구현 전 상세 설계 승인, `confirmation`은 개발 결과 승인을 뜻한다. 각각 문서 버전과 연결해 기록하며 서로 대체하지 않는다. 상세 설계 대기 시 `phase: task_design`, `run_state: awaiting_confirmation`, task 상태는 `awaiting_design_confirmation`을 사용한다. `awaiting`은 `{kind: task_design, task_id: todo-create-be, revision: 1}`처럼 구분한다. 승인받아 개발을 시작할 때 `phase: development`와 `status: in_progress`로 변경한다.

## 채팅 간 인계

[task별 새 채팅](task-chat.md)에 따라 task마다 `chat_id`를 연결한다. 확인 가능한 실제 ID만 기록하며 모르면 `null`로 유지한다. `chat_creation_authorization`에는 사용자의 새 채팅 생성 요청·승인 근거를 기록한다. 설치된 지침 자체를 승인 근거로 적지 않는다.

`handoff`에는 `change_id`, `task_id`, `from_chat_id`, `to_chat_id`, `status`, `next_action`을 필요에 따라 기록한다. 상태는 `pending_creation`, `created`, `accepted`, `manual_required`, `failed`로 구분한다. 인계 대기 시 원래 채팅은 구현을 멈추며 task 상태를 완료나 진행 중으로 임의 변경하지 않는다. 새 채팅이 인계를 수락한 뒤에는 원래 채팅이 해당 변경의 상태를 갱신하지 않는다.

새 채팅은 저장된 승인 버전과 담당 task를 확인해 인계를 수락한다. 기존 task 채팅이 있으면 중복 생성 대신 그 채팅으로 안내한다. 사용자 응답 없이 인계·채팅 생성 성공을 설계 승인으로 처리하지 않는다. 마지막 task 채팅에서 전체 검증과 최종 확인을 수행한다.

## 갱신 시점

- 단계 및 작업 시작 시 현재 지점과 다음 행동을 저장한다.
- 의미 있는 구현·검증·리뷰가 끝날 때 완료한 일과 남은 일을 갱신한다.
- 사용자 확인 요청 전에 `awaiting`에 단계 또는 작업 ID와 대상 버전을 남긴다.
- 실제 사용자 응답 후 승인·수정 요청과 확인 대상을 기록한다. 날짜를 기록하면 실제 확인한 시간을 사용한다.
- 중단 요청 시 최신 상태를 저장하고 멈춘다. 다른 업무 요청이 들어오면 기존 진행 상황을 보존한다.

저장된 완료 기록, 도구 성공, 침묵은 사용자 승인이 아니다. 대기 중 결과를 직접 가리키는 “ㅇㅇ”, “ㅇㅋ”, “진행해”는 명확한 문맥에서 해당 결과 승인으로 처리할 수 있다. 단순 질문·조회·아이디어 요청은 승인으로 처리하지 않는다. 대상이 불분명하면 무엇을 승인하는지 확인한다.

## 수정과 재개

수정 시 변경된 기능 설계·계획·task 상세 설계 버전을 올리고 영향받는 승인과 작업만 재확인 대상으로 표시한다. 승인 증거와 실제 구현 이력은 지우지 않는다. 재작업이 필요한 항목은 `remaining`에 명시한다.

재개할 때 진행 파일과 연결된 현재 초안만 먼저 읽는다. 현재 작업, 완료한 일, 남은 일, 승인 대기 여부를 간단히 알린다. 대기 중이면 사용자 확인 없이 다음 작업을 시작하지 않는다. 개발 재개에서는 기록된 관련 코드와 변경 상태를 확인하고 이어서 수행한다. 작업 상태가 `in_progress`라는 이유만으로 이미 수행한 변경을 반복하지 않는다.

파일이 없거나 손상됐다면 승인과 완료를 추측해 복구하지 않는다. 확인 가능한 초안·사용자 대화를 근거로 부족한 상태를 질문한다. 여러 변경이 기록돼 있으면 명시된 변경 또는 `active_change`를 사용하고, 불명확하면 어느 변경을 재개할지 묻는다.

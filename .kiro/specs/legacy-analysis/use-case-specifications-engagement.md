# Use-Case Specifications - 커뮤니티 참여 컨텍스트

## 개요

이 문서는 커뮤니티 참여(Community Engagement) 컨텍스트의 Use-Case 상세 명세를 Unified Process 표준에 따라 작성합니다.

---

## UC-E1: 좋아요 표시

### Brief Description
독자가 마음에 드는 기사에 좋아요를 표시하는 과정

### Actors
- **Primary**: 독자 (Reader)
- **Secondary**: 시스템 (System)

### Preconditions
- 독자가 로그인되어 있어야 함
- 좋아요를 표시할 기사가 존재해야 함

### Postconditions
- **Success**: 좋아요가 기록됨, 기사의 좋아요 수가 증가함
- **Failure**: 좋아요 실패, 오류 메시지 표시

### Basic Flow
1. 독자가 기사를 조회한다
2. 독자가 좋아요 버튼을 클릭한다
3. 시스템이 기사를 조회한다
4. 시스템이 중복 좋아요 여부를 확인한다
5. 시스템이 좋아요를 생성한다:
   - 기사 식별자
   - 독자 식별자
6. 시스템이 좋아요를 저장한다
7. 시스템이 기사의 좋아요 수를 재집계한다
8. 시스템이 업데이트된 기사 정보를 반환한다 (favorited: true, favoritesCount: 증가)
9. Use-Case 종료

### Alternative Flows

#### AF1: 기사가 존재하지 않음
- **Trigger**: Step 3에서 기사를 찾을 수 없는 경우
- **Flow**:
  1. 시스템이 "기사를 찾을 수 없습니다" 오류 메시지를 표시한다
  2. Use-Case 종료

#### AF2: 이미 좋아요 표시함
- **Trigger**: Step 4에서 독자가 이미 해당 기사에 좋아요를 표시한 경우
- **Flow**:
  1. 시스템이 중복 좋아요를 무시한다 (오류 없음)
  2. 시스템이 현재 기사 정보를 반환한다 (favorited: true)
  3. Use-Case 종료

### Business Rules
- **BR-E1.1**: 독자는 동일한 기사에 한 번만 좋아요를 표시할 수 있음
- **BR-E1.2**: 중복 좋아요는 무시됨 (오류 발생하지 않음)
- **BR-E1.3**: 좋아요는 언제든지 취소할 수 있음

### Special Requirements
- 없음

### Related Use Cases
- **Includes**: UC-M2 (로그인)
- **Extends**: UC-P4 (기사 조회)
- **Related**: UC-E2 (좋아요 취소)

### Notes
- **프로세스 참조**: business-process-flows.md의 "프로세스 2: 독자의 기사 소비 및 반응 여정"

---

## UC-E2: 좋아요 취소

### Brief Description
독자가 이전에 표시한 좋아요를 취소하는 과정

### Actors
- **Primary**: 독자 (Reader)
- **Secondary**: 시스템 (System)

### Preconditions
- 독자가 로그인되어 있어야 함
- 좋아요를 취소할 기사가 존재해야 함
- 독자가 해당 기사에 좋아요를 표시한 상태여야 함

### Postconditions
- **Success**: 좋아요가 삭제됨, 기사의 좋아요 수가 감소함
- **Failure**: 취소 실패, 오류 메시지 표시

### Basic Flow
1. 독자가 기사를 조회한다
2. 독자가 좋아요 취소 버튼을 클릭한다
3. 시스템이 기사를 조회한다
4. 시스템이 좋아요를 조회한다
5. 시스템이 좋아요를 삭제한다
6. 시스템이 기사의 좋아요 수를 재집계한다
7. 시스템이 업데이트된 기사 정보를 반환한다 (favorited: false, favoritesCount: 감소)
8. Use-Case 종료

### Alternative Flows

#### AF1: 기사가 존재하지 않음
- **Trigger**: Step 3에서 기사를 찾을 수 없는 경우
- **Flow**:
  1. 시스템이 "기사를 찾을 수 없습니다" 오류 메시지를 표시한다
  2. Use-Case 종료

#### AF2: 좋아요를 표시하지 않음
- **Trigger**: Step 4에서 좋아요를 찾을 수 없는 경우
- **Flow**:
  1. 시스템이 삭제를 무시한다 (오류 없음)
  2. 시스템이 현재 기사 정보를 반환한다 (favorited: false)
  3. Use-Case 종료

### Business Rules
- **BR-E2.1**: 좋아요를 표시하지 않은 기사는 취소할 수 없음
- **BR-E2.2**: 존재하지 않는 좋아요 취소는 무시됨 (오류 발생하지 않음)

### Special Requirements
- 없음

### Related Use Cases
- **Includes**: UC-M2 (로그인)
- **Extends**: UC-P4 (기사 조회)
- **Related**: UC-E1 (좋아요 표시)

---

## UC-E3: 댓글 작성

### Brief Description
독자가 기사에 대한 의견이나 질문을 댓글로 작성하는 과정

### Actors
- **Primary**: 독자 (Reader)
- **Secondary**: 시스템 (System)

### Preconditions
- 독자가 로그인되어 있어야 함
- 댓글을 작성할 기사가 존재해야 함

### Postconditions
- **Success**: 댓글이 기록됨, 기사에 댓글이 추가됨
- **Failure**: 작성 실패, 오류 메시지 표시

### Basic Flow
1. 독자가 기사를 조회한다
2. 독자가 댓글 입력란에 내용을 작성한다
3. 독자가 댓글 작성 버튼을 클릭한다
4. 시스템이 기사를 조회한다
5. 시스템이 댓글 내용의 유효성을 검증한다
6. 시스템이 댓글을 생성한다:
   - 댓글 내용
   - 독자 식별자
   - 기사 식별자
   - 작성 시각 (현재 시각)
7. 시스템이 댓글을 저장한다
8. 시스템이 작성된 댓글 정보를 반환한다
9. Use-Case 종료

### Alternative Flows

#### AF1: 기사가 존재하지 않음
- **Trigger**: Step 4에서 기사를 찾을 수 없는 경우
- **Flow**:
  1. 시스템이 "기사를 찾을 수 없습니다" 오류 메시지를 표시한다
  2. Use-Case 종료

#### AF2: 댓글 내용이 비어있음
- **Trigger**: Step 5에서 댓글 내용이 비어있는 경우
- **Flow**:
  1. 시스템이 "댓글 내용을 입력해주세요" 오류 메시지를 표시한다
  2. Step 2로 돌아간다

### Business Rules
- **BR-E3.1**: 댓글은 작성 후 수정할 수 없음 (불변)
- **BR-E3.2**: 댓글 내용은 필수임 (빈 댓글 불가)
- **BR-E3.3**: 댓글은 작성자 또는 기사 저자만 삭제할 수 있음

### Special Requirements
- 없음

### Related Use Cases
- **Includes**: UC-M2 (로그인)
- **Extends**: UC-P4 (기사 조회)
- **Related**: UC-E4 (댓글 삭제)

### Notes
- **프로세스 참조**: business-process-flows.md의 "프로세스 2: 독자의 기사 소비 및 반응 여정"

---

## UC-E4: 댓글 삭제

### Brief Description
댓글 작성자 또는 기사 저자가 부적절한 댓글을 삭제하는 과정

### Actors
- **Primary**: 독자/저자 (Reader/Author)
- **Secondary**: 시스템 (System)

### Preconditions
- 사용자가 로그인되어 있어야 함
- 삭제하려는 댓글이 존재해야 함
- 사용자가 댓글 작성자 또는 기사 저자여야 함

### Postconditions
- **Success**: 댓글이 삭제됨
- **Failure**: 삭제 실패, 오류 메시지 표시

### Basic Flow
1. 사용자가 기사를 조회한다
2. 사용자가 특정 댓글의 삭제 버튼을 클릭한다
3. 시스템이 기사를 조회한다
4. 시스템이 댓글을 조회한다
5. 시스템이 삭제 권한을 확인한다:
   - 사용자가 댓글 작성자인가?
   - 또는 사용자가 기사 저자인가?
6. 시스템이 댓글을 삭제한다
7. 시스템이 성공 응답을 반환한다
8. Use-Case 종료

### Alternative Flows

#### AF1: 기사가 존재하지 않음
- **Trigger**: Step 3에서 기사를 찾을 수 없는 경우
- **Flow**:
  1. 시스템이 "기사를 찾을 수 없습니다" 오류 메시지를 표시한다
  2. Use-Case 종료

#### AF2: 댓글이 존재하지 않음
- **Trigger**: Step 4에서 댓글을 찾을 수 없는 경우
- **Flow**:
  1. 시스템이 "댓글을 찾을 수 없습니다" 오류 메시지를 표시한다
  2. Use-Case 종료

#### AF3: 권한 없음 (댓글 작성자도 기사 저자도 아님)
- **Trigger**: Step 5에서 사용자가 댓글 작성자도 기사 저자도 아닌 경우
- **Flow**:
  1. 시스템이 "삭제 권한이 없습니다" 오류 메시지를 표시한다
  2. Use-Case 종료

#### AF4: 댓글 작성자가 삭제
- **Trigger**: Step 5에서 사용자가 댓글 작성자인 경우
- **Flow**:
  1. 시스템이 권한을 승인한다
  2. Basic Flow를 계속 진행한다

#### AF5: 기사 저자가 삭제 (커뮤니티 관리)
- **Trigger**: Step 5에서 사용자가 기사 저자인 경우
- **Flow**:
  1. 시스템이 권한을 승인한다 (기사 저자는 자신의 기사에 달린 모든 댓글 삭제 가능)
  2. Basic Flow를 계속 진행한다

### Business Rules
- **BR-E4.1**: 댓글 작성자는 자신의 댓글을 삭제할 수 있음
- **BR-E4.2**: 기사 저자는 자신의 기사에 달린 모든 댓글을 삭제할 수 있음 (커뮤니티 관리 권한)
- **BR-E4.3**: 그 외의 사용자는 댓글을 삭제할 수 없음
- **BR-E4.4**: 삭제된 댓글은 복구할 수 없음

### Special Requirements
- 없음

### Related Use Cases
- **Includes**: UC-M2 (로그인)
- **Extends**: UC-P4 (기사 조회)
- **Related**: UC-E3 (댓글 작성)

### Notes
- **결정 테이블 참조**: business-logic-analysis.md의 "로직 2-2: 댓글 삭제 권한"
- **프로세스 참조**: business-process-flows.md의 "프로세스 4: 커뮤니티 관리 - 댓글 삭제 여정"

### Issues
- **발견된 이슈**: 플랫폼 관리자가 부적절한 댓글을 삭제할 수 있는 권한이 없음
- **개선 기회**: 신고 기능 추가 필요

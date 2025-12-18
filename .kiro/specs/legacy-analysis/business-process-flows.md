# 비즈니스 프로세스 흐름 시각화

## 개요

이 문서는 RealWorld 플랫폼의 핵심 비즈니스 프로세스를 End-to-End로 시각화합니다. 각 흐름도는 비즈니스 이벤트와 활동 중심으로 구성되어 있으며, Product Owner, 개발자, 비즈니스 분석가, 도메인 전문가가 함께 검토할 수 있도록 유비쿼터스 언어를 사용합니다.

---

## 프로세스 1: 저자의 기사 발행 여정

**비즈니스 목적**: 저자가 자신의 생각을 기사로 작성하여 독자들에게 공유하는 전체 과정

**참여 컨텍스트**: 회원 → 출판 → 콘텐츠 큐레이션

```mermaid
flowchart TD
    Start([저자가 기사 작성 시작]) --> CheckAuth{회원 인증 완료?}
    
    CheckAuth -->|아니오| Login[회원 로그인]
    CheckAuth -->|예| WriteArticle[기사 작성]
    Login --> WriteArticle
    
    subgraph BC1[회원 컨텍스트]
        Login
        AuthToken[인증 토큰 발급]
        Login --> AuthToken
    end
    
    subgraph BC2[출판 컨텍스트]
        WriteArticle[기사 정보 입력<br/>제목, 설명, 본문, 태그]
        ValidateArticle{제목 중복 확인}
        GenerateSlug[슬러그 자동 생성<br/>제목 → URL 식별자]
        CreateArticle[기사 발행]
        ArticleCreated[기사 발행 완료]
        
        WriteArticle --> ValidateArticle
        ValidateArticle -->|중복됨| DuplicateError[오류: 동일 제목 존재]
        ValidateArticle -->|고유함| GenerateSlug
        GenerateSlug --> CreateArticle
        CreateArticle --> ArticleCreated
    end
    
    subgraph BC3[콘텐츠 큐레이션 컨텍스트]
        IndexArticle[기사 색인 등록]
        UpdateTagList[태그 목록 갱신]
        NotifyFollowers[팔로워 피드에 추가]
        
        ArticleCreated --> IndexArticle
        IndexArticle --> UpdateTagList
        UpdateTagList --> NotifyFollowers
    end
    
    NotifyFollowers --> End([기사가 독자에게 노출됨])
    DuplicateError --> WriteArticle
    
    style Start fill:#e1f5e1
    style End fill:#e1f5e1
    style DuplicateError fill:#ffe1e1
```

**데이터 흐름**:
- 회원 → 출판: 저자 식별자, 인증 토큰
- 출판 → 큐레이션: 기사 식별자, 태그 목록, 저자 식별자

**주요 결정 지점**:
1. **회원 인증 확인**: 로그인하지 않은 사용자는 기사를 작성할 수 없음
2. **제목 중복 확인**: 동일한 제목의 기사가 이미 존재하는지 검증

**비즈니스 규칙**:
- 슬러그는 제목으로부터 자동 생성됨 (소문자 변환, 특수문자 제거)
- 태그는 중복 없이 저장됨
- 발행 즉시 팔로워의 피드에 노출됨

---

## 프로세스 2: 독자의 기사 소비 및 반응 여정

**비즈니스 목적**: 독자가 관심 있는 기사를 발견하고, 읽고, 반응하는 전체 과정

**참여 컨텍스트**: 콘텐츠 큐레이션 → 커뮤니티 참여 → 회원

```mermaid
flowchart TD
    Start([독자가 플랫폼 방문]) --> BrowseMethod{콘텐츠 발견 방법}
    
    subgraph BC4[콘텐츠 큐레이션 컨텍스트]
        BrowseMethod -->|피드 보기| ViewFeed[팔로우한 저자의<br/>최신 기사 목록]
        BrowseMethod -->|태그 검색| SearchByTag[특정 태그로<br/>기사 필터링]
        BrowseMethod -->|저자 검색| SearchByAuthor[특정 저자의<br/>기사 목록]
        
        ViewFeed --> ArticleList[기사 목록 표시]
        SearchByTag --> ArticleList
        SearchByAuthor --> ArticleList
    end
    
    ArticleList --> SelectArticle[기사 선택]
    
    subgraph BC2[출판 컨텍스트]
        SelectArticle --> ReadArticle[기사 읽기<br/>제목, 본문, 태그]
    end
    
    ReadArticle --> ReactChoice{반응 선택}
    
    subgraph BC3[커뮤니티 참여 컨텍스트]
        ReactChoice -->|좋아요| CheckFavorited{이미 좋아요<br/>표시했는가?}
        CheckFavorited -->|아니오| AddFavorite[좋아요 표시]
        CheckFavorited -->|예| AlreadyFavorited[이미 좋아요 상태]
        
        ReactChoice -->|댓글 작성| WriteComment[댓글 내용 입력]
        WriteComment --> PostComment[댓글 게시]
        
        ReactChoice -->|저자 팔로우| CheckFollowing{이미 팔로우<br/>중인가?}
        
        AddFavorite --> FavoriteCountUp[좋아요 수 증가]
        PostComment --> CommentAdded[댓글 추가 완료]
    end
    
    subgraph BC1[회원 컨텍스트]
        CheckFollowing -->|아니오| FollowAuthor[저자 팔로우]
        CheckFollowing -->|예| AlreadyFollowing[이미 팔로우 중]
        FollowAuthor --> FollowAdded[팔로우 관계 형성]
    end
    
    ReactChoice -->|반응 없음| End
    FavoriteCountUp --> End
    AlreadyFavorited --> End
    CommentAdded --> End
    FollowAdded --> End
    AlreadyFollowing --> End
    
    End([독자의 활동 완료])
    
    style Start fill:#e1f5e1
    style End fill:#e1f5e1
```

**데이터 흐름**:
- 큐레이션 → 출판: 기사 식별자
- 출판 → 커뮤니티 참여: 기사 식별자, 저자 식별자
- 커뮤니티 참여 → 회원: 독자 식별자, 저자 식별자

**주요 결정 지점**:
1. **콘텐츠 발견 방법**: 피드, 태그 검색, 저자 검색 중 선택
2. **좋아요 중복 확인**: 이미 좋아요한 기사인지 확인
3. **팔로우 중복 확인**: 이미 팔로우 중인 저자인지 확인

**비즈니스 규칙**:
- 동일한 기사에 중복 좋아요 불가
- 동일한 저자를 중복 팔로우 불가
- 댓글은 작성 후 수정 불가 (삭제만 가능)

---

## 프로세스 3: 저자의 기사 수정 여정

**비즈니스 목적**: 저자가 발행한 기사의 내용을 수정하는 과정

**참여 컨텍스트**: 회원 → 출판

```mermaid
flowchart TD
    Start([저자가 기사 수정 시작]) --> FindArticle[기사 조회<br/>슬러그로 검색]
    
    FindArticle --> ArticleExists{기사 존재?}
    ArticleExists -->|아니오| NotFoundError[오류: 기사를 찾을 수 없음]
    ArticleExists -->|예| CheckOwnership
    
    subgraph BC1[회원 컨텍스트]
        CheckOwnership{저자 본인인가?}
    end
    
    CheckOwnership -->|아니오| AuthError[오류: 수정 권한 없음]
    CheckOwnership -->|예| EditArticle
    
    subgraph BC2[출판 컨텍스트]
        EditArticle[수정할 정보 입력<br/>제목, 설명, 본문]
        DetermineChanges{어떤 항목을<br/>수정하는가?}
        
        EditArticle --> DetermineChanges
        
        DetermineChanges -->|제목 포함| UpdateTitle[제목 변경<br/>+ 슬러그 재생성]
        DetermineChanges -->|제목 제외| KeepTitle[제목 유지<br/>+ 슬러그 유지]
        
        UpdateTitle --> UpdateOtherFields[설명/본문 변경<br/>제공된 항목만]
        KeepTitle --> UpdateOtherFields
        
        UpdateOtherFields --> UpdateTimestamp[수정 시각 갱신]
        UpdateTimestamp --> SaveArticle[기사 저장]
        SaveArticle --> ArticleUpdated[기사 수정 완료]
    end
    
    ArticleUpdated --> End([수정된 기사 표시])
    NotFoundError --> End
    AuthError --> End
    
    style Start fill:#e1f5e1
    style End fill:#e1f5e1
    style NotFoundError fill:#ffe1e1
    style AuthError fill:#ffe1e1
```

**데이터 흐름**:
- 회원 → 출판: 저자 식별자, 인증 토큰
- 출판 내부: 기사 식별자, 수정할 필드 목록

**주요 결정 지점**:
1. **기사 존재 확인**: 슬러그로 기사를 찾을 수 있는가?
2. **소유권 확인**: 요청자가 기사 저자 본인인가?
3. **수정 항목 결정**: 제목을 수정하는가? (슬러그 재생성 여부 결정)

**비즈니스 규칙** (결정 테이블 3-2 참조):
- 제공된 항목만 수정되고 나머지는 기존 값 유지
- 제목을 수정하면 슬러그도 자동으로 재생성됨
- 어떤 항목이든 수정되면 수정 시각이 현재 시각으로 갱신됨

**잠재적 이슈**:
- 제목 변경 시 슬러그(URL)가 변경되어 기존 링크가 깨질 수 있음

---

## 프로세스 4: 커뮤니티 관리 - 댓글 삭제 여정

**비즈니스 목적**: 부적절한 댓글을 삭제하여 건전한 커뮤니티를 유지하는 과정

**참여 컨텍스트**: 회원 → 출판 → 커뮤니티 참여

```mermaid
flowchart TD
    Start([댓글 삭제 요청]) --> FindArticle[기사 조회<br/>슬러그로 검색]
    
    FindArticle --> ArticleExists{기사 존재?}
    ArticleExists -->|아니오| NotFoundError1[오류: 기사를 찾을 수 없음]
    ArticleExists -->|예| FindComment
    
    subgraph BC2[출판 컨텍스트]
        FindComment[댓글 조회<br/>댓글 ID로 검색]
    end
    
    FindComment --> CommentExists{댓글 존재?}
    CommentExists -->|아니오| NotFoundError2[오류: 댓글을 찾을 수 없음]
    CommentExists -->|예| CheckDeleteAuth
    
    subgraph BC1[회원 컨텍스트]
        CheckDeleteAuth{삭제 권한 확인}
        IsCommentAuthor{댓글 작성자<br/>본인인가?}
        IsArticleAuthor{기사 저자<br/>본인인가?}
        
        CheckDeleteAuth --> IsCommentAuthor
        IsCommentAuthor -->|예| HasPermission[권한 있음]
        IsCommentAuthor -->|아니오| IsArticleAuthor
        IsArticleAuthor -->|예| HasPermission
        IsArticleAuthor -->|아니오| NoPermission[권한 없음]
    end
    
    HasPermission --> DeleteComment
    NoPermission --> AuthError[오류: 삭제 권한 없음]
    
    subgraph BC3[커뮤니티 참여 컨텍스트]
        DeleteComment[댓글 삭제]
        CommentDeleted[댓글 삭제 완료]
        DeleteComment --> CommentDeleted
    end
    
    CommentDeleted --> End([댓글이 제거됨])
    NotFoundError1 --> End
    NotFoundError2 --> End
    AuthError --> End
    
    style Start fill:#e1f5e1
    style End fill:#e1f5e1
    style NotFoundError1 fill:#ffe1e1
    style NotFoundError2 fill:#ffe1e1
    style AuthError fill:#ffe1e1
```

**데이터 흐름**:
- 회원 → 출판: 요청자 식별자, 기사 슬러그
- 출판 → 커뮤니티 참여: 기사 식별자, 댓글 식별자
- 커뮤니티 참여 → 회원: 댓글 작성자 식별자, 기사 저자 식별자

**주요 결정 지점**:
1. **기사 존재 확인**: 슬러그로 기사를 찾을 수 있는가?
2. **댓글 존재 확인**: 해당 기사에 댓글이 존재하는가?
3. **삭제 권한 확인** (결정 테이블 2-2 참조):
   - 댓글 작성자 본인인가? → 권한 있음
   - 기사 저자 본인인가? → 권한 있음
   - 둘 다 아니면 → 권한 없음

**비즈니스 규칙**:
- 댓글 작성자는 자신의 댓글을 삭제할 수 있음
- 기사 저자는 자신의 기사에 달린 모든 댓글을 삭제할 수 있음 (커뮤니티 관리 권한)
- 그 외의 회원은 댓글을 삭제할 수 없음

**발견된 이슈**:
- 플랫폼 관리자가 부적절한 댓글을 삭제할 수 있는 권한이 없음
- 신고 기능이 없어 부적절한 댓글을 발견해도 대응 방법이 제한적

---

## 프로세스 5: 회원 정보 수정 여정

**비즈니스 목적**: 회원이 자신의 프로필 정보를 업데이트하는 과정

**참여 컨텍스트**: 회원

```mermaid
flowchart TD
    Start([회원이 정보 수정 시작]) --> InputChanges[수정할 정보 입력<br/>이메일, 사용자명, 비밀번호,<br/>자기소개, 프로필 사진]
    
    subgraph BC1[회원 컨텍스트]
        InputChanges --> ValidateUniqueness{고유성 검증}
        
        CheckEmail{입력된 이메일이<br/>다른 회원이 사용 중?}
        CheckUsername{입력된 사용자명이<br/>다른 회원이 사용 중?}
        
        ValidateUniqueness --> CheckEmail
        ValidateUniqueness --> CheckUsername
        
        CheckEmail -->|예, 중복됨| EmailError[오류: 이메일 이미 존재]
        CheckEmail -->|아니오 또는<br/>본인 것| EmailValid[이메일 유효]
        
        CheckUsername -->|예, 중복됨| UsernameError[오류: 사용자명 이미 존재]
        CheckUsername -->|아니오 또는<br/>본인 것| UsernameValid[사용자명 유효]
        
        EmailValid --> BothValid{모두 유효?}
        UsernameValid --> BothValid
        
        BothValid -->|예| ApplyChanges[변경 사항 적용<br/>제공된 항목만 수정]
        BothValid -->|아니오| ValidationFailed
        
        EmailError --> ValidationFailed[검증 실패]
        UsernameError --> ValidationFailed
        
        ApplyChanges --> EncryptPassword{비밀번호<br/>변경되었는가?}
        EncryptPassword -->|예| HashPassword[비밀번호 암호화]
        EncryptPassword -->|아니오| SaveUser
        HashPassword --> SaveUser[회원 정보 저장]
        
        SaveUser --> UpdateComplete[정보 수정 완료]
    end
    
    UpdateComplete --> End([수정된 정보 표시])
    ValidationFailed --> InputChanges
    
    style Start fill:#e1f5e1
    style End fill:#e1f5e1
    style ValidationFailed fill:#ffe1e1
```

**데이터 흐름**:
- 회원 내부: 회원 식별자, 수정할 필드 목록, 검증 결과

**주요 결정 지점**:
1. **이메일 고유성 확인** (결정 테이블 1 참조):
   - 다른 회원이 사용 중인가? → 오류
   - 본인의 기존 이메일인가? → 허용
   - 사용 중이지 않은가? → 허용
2. **사용자명 고유성 확인**:
   - 다른 회원이 사용 중인가? → 오류
   - 본인의 기존 사용자명인가? → 허용
   - 사용 중이지 않은가? → 허용
3. **비밀번호 변경 여부**: 비밀번호가 제공되었는가? → 암호화 필요

**비즈니스 규칙** (결정 테이블 1, 3-1 참조):
- 이메일과 사용자명은 플랫폼 전체에서 유일해야 함
- 자신의 기존 값으로 재설정하는 것은 허용됨
- 제공된 항목만 수정되고 나머지는 기존 값 유지
- 비밀번호는 암호화되어 저장됨

---

## 프로세스 6: 개인화된 피드 생성

**비즈니스 목적**: 독자가 팔로우한 저자들의 최신 기사를 시간순으로 제공하는 과정

**참여 컨텍스트**: 회원 → 콘텐츠 큐레이션 → 출판

```mermaid
flowchart TD
    Start([독자가 피드 요청]) --> CheckAuth{회원 인증<br/>완료?}
    
    CheckAuth -->|아니오| AuthError[오류: 로그인 필요]
    CheckAuth -->|예| GetFollowing
    
    subgraph BC1[회원 컨텍스트]
        GetFollowing[팔로우 중인<br/>저자 목록 조회]
        FollowingList[저자 식별자 목록]
        GetFollowing --> FollowingList
    end
    
    FollowingList --> HasFollowing{팔로우 중인<br/>저자가 있는가?}
    
    HasFollowing -->|아니오| EmptyFeed[빈 피드 반환]
    HasFollowing -->|예| QueryArticles
    
    subgraph BC4[콘텐츠 큐레이션 컨텍스트]
        QueryArticles[저자 목록으로<br/>기사 검색]
        SortByDate[발행 시각 기준<br/>최신순 정렬]
        ApplyPagination[페이지네이션 적용<br/>커서 기반]
        
        QueryArticles --> SortByDate
        SortByDate --> ApplyPagination
    end
    
    ApplyPagination --> EnrichData
    
    subgraph BC2[출판 컨텍스트]
        EnrichData[기사 상세 정보<br/>추가 조회]
    end
    
    subgraph BC3[커뮤니티 참여 컨텍스트]
        AddEngagement[좋아요 수 및<br/>독자의 좋아요 여부 추가]
    end
    
    EnrichData --> AddEngagement
    AddEngagement --> FeedReady[피드 생성 완료]
    EmptyFeed --> End
    FeedReady --> End([피드 표시])
    AuthError --> End
    
    style Start fill:#e1f5e1
    style End fill:#e1f5e1
    style AuthError fill:#ffe1e1
```

**데이터 흐름**:
- 회원 → 큐레이션: 독자 식별자, 팔로우 중인 저자 목록
- 큐레이션 → 출판: 기사 식별자 목록
- 출판 → 커뮤니티 참여: 기사 식별자 목록
- 커뮤니티 참여 → 독자: 기사 목록 + 좋아요 정보

**주요 결정 지점**:
1. **회원 인증 확인**: 로그인한 회원만 개인화된 피드를 볼 수 있음
2. **팔로우 여부 확인**: 팔로우 중인 저자가 없으면 빈 피드 반환

**비즈니스 규칙**:
- 피드는 팔로우한 저자들의 기사만 포함
- 최신 기사가 먼저 표시됨 (발행 시각 역순)
- 커서 기반 페이지네이션으로 무한 스크롤 지원
- 각 기사에 좋아요 수와 독자의 좋아요 여부가 표시됨

---

## 컨텍스트 간 상호작용 요약

### 주요 데이터 흐름

```mermaid
graph LR
    subgraph BC1[회원]
        User[회원 정보]
        Follow[팔로우 관계]
    end
    
    subgraph BC2[출판]
        Article[기사]
        Tag[태그]
    end
    
    subgraph BC3[커뮤니티 참여]
        Favorite[좋아요]
        Comment[댓글]
    end
    
    subgraph BC4[콘텐츠 큐레이션]
        Feed[피드]
        Search[검색]
    end
    
    User -->|저자 식별자| Article
    User -->|독자 식별자| Favorite
    User -->|독자 식별자| Comment
    User -->|팔로우 목록| Feed
    
    Article -->|기사 식별자| Favorite
    Article -->|기사 식별자| Comment
    Article -->|기사 + 태그| Search
    
    Follow -->|저자 목록| Feed
    Favorite -->|좋아요 정보| Search
    Tag -->|태그 목록| Search
    
    Article -->|기사 목록| Feed
```

### 비즈니스 이벤트 흐름

| 이벤트 | 발생 컨텍스트 | 영향받는 컨텍스트 | 데이터 전달 |
|--------|-------------|-----------------|-----------|
| 회원 가입 완료 | 회원 | - | 회원 식별자 |
| 기사 발행 완료 | 출판 | 콘텐츠 큐레이션 | 기사 식별자, 태그 목록, 저자 식별자 |
| 좋아요 표시 | 커뮤니티 참여 | 출판 (좋아요 수 증가) | 기사 식별자, 독자 식별자 |
| 댓글 작성 | 커뮤니티 참여 | 출판 (댓글 수 증가) | 기사 식별자, 댓글 내용, 독자 식별자 |
| 팔로우 형성 | 회원 | 콘텐츠 큐레이션 (피드 구성 변경) | 독자 식별자, 저자 식별자 |

---

## 발견된 비즈니스 기회 및 개선 사항

### 1. 커뮤니티 관리 강화
**현재 상태**: 기사 저자만 댓글을 삭제할 수 있음
**개선 기회**:
- 플랫폼 관리자 역할 추가
- 신고 기능 추가 (부적절한 콘텐츠 신고)
- 회원 정지/차단 기능

### 2. 콘텐츠 버전 관리
**현재 상태**: 기사 수정 시 이전 버전이 사라짐
**개선 기회**:
- 기사 수정 이력 저장
- 이전 버전으로 되돌리기 기능
- 수정 내역 표시

### 3. 슬러그 안정성
**현재 상태**: 제목 변경 시 슬러그(URL)가 변경됨
**개선 기회**:
- 슬러그를 불변으로 유지
- 또는 이전 슬러그로의 리다이렉션 지원

### 4. 알림 시스템
**현재 상태**: 알림 기능 없음
**개선 기회**:
- 새 댓글 알림
- 새 팔로워 알림
- 좋아요 알림

### 5. 검색 고도화
**현재 상태**: 태그, 저자, 좋아요 기반 필터링만 가능
**개선 기회**:
- 전문 검색 (제목, 본문 내용 검색)
- 복합 필터 (태그 + 기간 + 인기도)
- 추천 알고리즘

---

## 다음 단계

이 분석 결과를 바탕으로 다음 작업을 수행할 수 있습니다:

1. **현대화 우선순위 결정**: 어떤 프로세스를 먼저 개선할 것인가?
2. **마이크로서비스 분리 전략**: 바운디드 컨텍스트를 독립 서비스로 분리할 것인가?
3. **이벤트 기반 아키텍처 도입**: 컨텍스트 간 통신을 이벤트로 전환할 것인가?
4. **API 설계 개선**: RESTful API를 GraphQL로 통합할 것인가?

---
description: 현재 브랜치의 커밋 메시지를 분석하여 GitLab MR URL 생성 및 브라우저 오픈
---

# Create GitLab Merge Request

현재 브랜치의 커밋 메시지를 분석하여 GitLab Merge Request URL을 생성하고 브라우저에서 엽니다.

## Step 1: Git Repository 확인

Git 저장소인지 확인합니다:

```bash
git rev-parse --git-dir
```

Git 저장소가 아니면: "Git 저장소가 아닙니다." 출력 후 중단

## Step 2: Git Remote에서 GitLab 정보 추출

원격 저장소 URL을 가져옵니다:

```bash
git remote get-url origin
```

### URL 파싱

**HTTPS 형식:**
`https://gitlab.gum3.io/nft-project/market-service-web.git`
→ `GITLAB_URL=https://gitlab.gum3.io`, `PROJECT_PATH=nft-project/market-service-web`

**SSH 형식:**
`git@gitlab.gum3.io:nft-project/market-service-web.git`
→ `GITLAB_URL=https://gitlab.gum3.io`, `PROJECT_PATH=nft-project/market-service-web`

GitLab URL을 추출할 수 없으면: "GitLab remote를 찾을 수 없습니다." 출력 후 중단

## Step 3: 현재 브랜치 확인

```bash
git branch --show-current
```

결과를 `SOURCE_BRANCH`로 저장

Detached HEAD 상태면: "현재 브랜치가 없습니다. 브랜치를 체크아웃해주세요." 출력 후 중단

## Step 4: Target 브랜치 선택

사용자에게 질문:
"MR을 생성할 타겟 브랜치를 입력해주세요 (기본값: develop):"

- 사용자가 입력하면 해당 브랜치 사용
- 입력하지 않으면 "develop" 사용

타겟 브랜치가 존재하는지 확인:
```bash
git rev-parse --verify origin/<TARGET_BRANCH>
```

존재하지 않으면: "타겟 브랜치가 존재하지 않습니다." 출력 후 중단

## Step 5: 커밋 개수 확인 및 메시지 가져오기

커밋 개수 확인:
```bash
git rev-list --count origin/<TARGET_BRANCH>..HEAD
```

50개 초과 시: "⚠️ 커밋이 50개를 초과합니다. 최근 50개만 분석합니다." 경고 표시

커밋 메시지 가져오기 (최대 50개):
```bash
git log origin/<TARGET_BRANCH>..HEAD --pretty=format:"%s%n%b%n---" --reverse -50
```

커밋이 없으면: "타겟 브랜치와 동일합니다. MR을 생성할 변경사항이 없습니다." 출력 후 중단

## Step 6: 현재 브랜치 푸시

원격 저장소로 현재 브랜치를 푸시합니다:

```bash
git push -u origin HEAD
```

푸시 실패 시: 에러 메시지 표시 후 중단

## Step 7: 커밋 분석 및 중요도 결정

모든 커밋 메시지(제목 + 본문)를 분석하여 중요도를 결정합니다.

### 중요도 기준

**🔴 (상) - 다음 중 하나라도 해당:**
- 아키텍처/구조 변경
- Breaking Changes
- 보안 관련 변경
- DB 스키마 변경
- 대규모 리팩토링
- 주요 라이브러리 업데이트

**🟠 (중) - 다음 중 하나라도 해당:**
- 신규 기능 추가 (feat)
- API 변경
- 주요 로직 수정
- 여러 모듈 변경
- 핵심 기능 버그 수정

**🟢 (하) - 모두 해당:**
- 텍스트/문구 변경
- UI 스타일만 변경
- 문서 업데이트
- 오타 수정
- 단순 리팩토링

**기본값:** 🟠 (애매하면 중간)

## Step 8: MR 제목 생성

형식: `<이모지> [<타입>] <간결한 설명>`

예시:
- `🔴 [refactor] 인증 시스템 아키텍처 변경`
- `🟠 [feat] 주문내역 UI 개선 및 컴포넌트 리팩토링`
- `🟢 [docs] README 설치 가이드 업데이트`

### 규칙
- **50자 이내**
- **이모지만 사용** (상/중/하 텍스트 제외)
- **타입은 대표 타입 하나만**
- **한글로 간결하게**

## Step 9: MR Body 생성

```markdown
## 무엇을 변경했나요?
[2-3문장으로 MR의 목적과 배경 설명]

## 주요 변경 사항
- [커밋 분석 기반 변경사항 1]
- [커밋 분석 기반 변경사항 2]
- [커밋 분석 기반 변경사항 3]

## 리뷰 포인트
- [리뷰어가 중점적으로 봐야 할 부분 1]
- [리뷰어가 중점적으로 봐야 할 부분 2]
```

### Body 작성 지침
- **무엇을 변경했나요?**: 비즈니스 가치나 해결한 문제 중심
- **주요 변경 사항**: 커밋 메시지를 기반으로 구체적 변경사항 나열 (3-5개)
- **리뷰 포인트**: 리스크 있는 부분, 테스트 필요한 부분 (2-4개)

## Step 10: GitLab MR URL 생성

URL 형식:
```
https://<GITLAB_URL>/<PROJECT_PATH>/-/merge_requests/new?merge_request[source_branch]=<SOURCE_BRANCH>&merge_request[target_branch]=<TARGET_BRANCH>&merge_request[title]=<URL_ENCODED_TITLE>&merge_request[description]=<URL_ENCODED_BODY>
```

### URL 인코딩 규칙
- 한글 → UTF-8 퍼센트 인코딩
- 줄바꿈 → `%0A`
- 공백 → `%20`
- `[` → `%5B`, `]` → `%5D`
- `#` → `%23`
- 이모지 → UTF-8 퍼센트 인코딩 (예: 🟠 → `%F0%9F%9F%A0`)

## Step 11: 브라우저 오픈

OS를 감지하여 적절한 명령으로 브라우저를 엽니다:

```bash
# macOS
open "<URL>"

# Linux
xdg-open "<URL>"
```

## 출력 형식

### 성공 시:
```
✓ 현재 브랜치를 원격 저장소로 푸시 완료
✓ GitLab MR URL 생성 완료!

MR 제목: 🟠 [feat] 주문내역 UI 개선 및 컴포넌트 리팩토링

브라우저에서 MR 페이지가 열립니다...
```

### 푸시 실패 시:
```
✗ 푸시 실패: <에러 메시지>
MR 생성을 중단합니다.
```

## 제약사항

1. **절대 포함하지 않음**: "Generated with Claude Code"
2. **코드 변경사항 분석 안함**: 커밋 메시지만 분석
3. **최대 50개 커밋만 분석**
4. **GitLab API 사용 안함**: URL 방식으로 MR 생성 페이지 오픈
5. **한글 사용**: 타입을 제외한 모든 텍스트는 한글

## 에지 케이스 처리

| 상황 | 처리 |
|------|------|
| Git 저장소 아님 | "Git 저장소가 아닙니다." |
| Remote가 없음 | "원격 저장소가 설정되지 않았습니다." |
| GitLab URL 아님 | "GitLab remote를 찾을 수 없습니다." |
| Detached HEAD | "현재 브랜치가 없습니다." |
| Target 브랜치 없음 | "타겟 브랜치가 존재하지 않습니다." |
| 커밋 없음 | "타겟 브랜치와 동일합니다." |
| 커밋 50개 초과 | 경고 + 최근 50개만 분석 |
| 푸시 실패 | 에러 메시지 표시 후 중단 |

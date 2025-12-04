# create-mr

GitLab 커밋 및 Merge Request 생성을 자동화하는 Claude Code 플러그인입니다.

## 기능

### `/commit` - 한국어 커밋 메시지 생성 및 푸시

스테이징된 변경사항을 분석하여 한국어 컨벤셔널 커밋 메시지를 자동 생성하고, 커밋 후 원격 저장소로 푸시합니다.

### `/mr` - GitLab Merge Request 생성

현재 브랜치의 커밋 메시지를 분석하여 GitLab Merge Request URL을 생성하고 브라우저에서 엽니다.

## 설치

```bash
# 1. 마켓플레이스 추가
/plugin marketplace add JoonSukCho/create-mr

# 2. 플러그인 설치
/plugin install create-mr@create-mr-marketplace
```

### 로컬 설치 (개발용)

```bash
/plugin install /path/to/create-mr
```

## 사용법

### /commit

1. 변경사항을 스테이징합니다:
   ```bash
   git add <파일>
   ```

2. Claude Code에서 `/commit` 명령 실행

3. 자동으로 커밋 메시지 생성 → 커밋 → 푸시가 진행됩니다

**출력 예시:**
```
✓ 커밋 생성 완료: a1b2c3d
  feat(인증): 소셜 로그인 기능 추가
✓ 원격 저장소로 푸시 완료
```

### /mr

1. 작업 브랜치에서 `/mr` 명령 실행

2. 타겟 브랜치 선택 (기본값: develop)

3. 자동으로 MR URL이 생성되고 브라우저에서 열립니다

**출력 예시:**
```
✓ 현재 브랜치를 원격 저장소로 푸시 완료
✓ GitLab MR URL 생성 완료!

MR 제목: 🟠 [feat] 주문내역 UI 개선 및 컴포넌트 리팩토링

브라우저에서 MR 페이지가 열립니다...
```

## 커밋 메시지 규칙

### 형식
```
타입(스코프): 주제

본문 (복잡한 변경사항인 경우에만)
```

### 타입

| 타입 | 설명 |
|------|------|
| feat | 새로운 기능 추가 |
| fix | 버그 수정 |
| docs | 문서 변경 |
| style | 코드 스타일/포맷 변경 |
| refactor | 코드 리팩토링 |
| test | 테스트 추가/수정 |
| perf | 성능 개선 |
| build | 빌드 시스템/외부 의존성 변경 |
| ci | CI 설정 변경 |
| chore | 기타 자잘한 수정 |

### 예시
```
feat(인증): 소셜 로그인 기능 추가
fix(결제): 결제 실패 시 에러 처리 개선
docs(README): 설치 가이드 업데이트
```

## MR 규칙

### 제목 형식
```
<중요도 이모지> [<타입>] <간결한 설명>
```

### 중요도

| 이모지 | 중요도 | 기준 |
|--------|--------|------|
| 🔴 | 상 | 아키텍처 변경, Breaking Changes, 보안, DB 스키마 |
| 🟠 | 중 | 신규 기능, API 변경, 주요 로직 수정 |
| 🟢 | 하 | 텍스트 변경, UI 스타일, 문서, 오타 |

### 본문 템플릿
```markdown
## 무엇을 변경했나요?
[2-3문장 요약]

## 주요 변경 사항
- [변경사항 1]
- [변경사항 2]
- [변경사항 3]

## 리뷰 포인트
- [리뷰 포인트 1]
- [리뷰 포인트 2]
```

## 요구사항

- Git 2.0 이상
- macOS 또는 Linux (브라우저 자동 오픈용)
- GitLab 원격 저장소

## 문제 해결

### "스테이징된 변경사항이 없습니다"
→ `git add <파일>`로 먼저 파일을 스테이징하세요.

### "푸시 실패: no upstream branch"
→ 다음 명령으로 upstream을 설정하세요:
```bash
git push -u origin <branch-name>
```

### "GitLab remote를 찾을 수 없습니다"
→ origin remote가 GitLab URL인지 확인하세요:
```bash
git remote -v
```

### "타겟 브랜치가 존재하지 않습니다"
→ 원격 저장소를 최신으로 업데이트하세요:
```bash
git fetch origin
```

## 라이선스

MIT

# awesome-kr-oss-sponsors - 설계 문서

- 작성일: 2026-05-05
- 상태: 초안 (사용자 검토 대기)

## 1. 개요

한국 OSS 생태계를 후원하는 기업/단체를 공개적으로 기록하고 인정(recognition)하기 위한 큐레이션 프로젝트. 후원의 가치 기준은 **금액 규모가 아니라 장기적 꾸준함(longest streak)**에 둔다.

리스트는 사람이 직접 손으로 쓰는 마크다운이 아니라, 구조화된 TOML 데이터(`data/`)를 단일 진실의 원천(source of truth)으로 두고 Python 빌드 스크립트가 마크다운 출력물을 자동 생성하는 방식으로 운영한다.

## 2. 인정하는 후원의 범위

다음 세 가지를 인정한다. 우선순위는 D > B > A.

- **D. 이벤트 후원 (최우선)**: 한국 OSS 커뮤니티가 운영하는 컨퍼런스 및 미트업의 유료 스폰서십. 자사 행사(예: DEVIEW, if(kakao))는 마케팅이므로 제외.
- **B. 고용을 통한 간접 후원**: 직원이 업무 시간의 일부 또는 전부를 OSS 메인테이닝에 쓰도록 허용한 경우, 또는 OSS 메인테이너를 명시적으로 채용한 경우. 회사가 커뮤니티에 OSS 메인테이너 공개 채용 공고를 낸 경우도 OSS 친화 신호로 추적한다.
- **A. 직접 금전 후원**: GitHub Sponsors, OpenCollective, 직접 송금/계약 등.

본 프로젝트는 인프라 제공(C)과 자사 OSS 공개(E)는 다루지 않는다.

## 3. 데이터 스코프

### 3.1 시드 컨퍼런스

다음 세 개의 커뮤니티 기반 컨퍼런스만 시드 데이터로 작성한다. 인프콘은 인프런(주식회사) 주최라 커뮤니티 행사가 아니므로 제외한다.

- 파이콘 한국 (PyCon Korea)
- FEConf
- JSConf Korea (행사 종료 상태일 가능성, 데이터 작성 단계에서 확인)

### 3.2 시드 시작 연도

**2021년부터** 5년치 데이터로 시작한다. streak 의미가 살아남는 최소 분량.

### 3.3 미트업 지원

스키마와 빌드 로직 차원에서 미트업도 1급 시민으로 다룬다. 단, 시드 데이터는 컨퍼런스 위주로 시작하고, 미트업 시드는 별도 PR로 추가한다. 미트업도 동일한 기준(유료 스폰서십, 커뮤니티 기반)을 적용한다.

## 4. 데이터 모델

### 4.1 디렉토리 구조

```
data/
  languages.toml                       # 언어 슬러그 화이트리스트 (단일 파일)
  events/
    conferences/
      pycon-kr/
        meta.toml
        2021.toml
        2022.toml
        ...
      feconf/
      jsconf-kr/
    meetups/
      (스키마는 동일, 시드는 비움)
  companies/
    toss.toml
    naver.toml
    kakao.toml
    ...
  maintainers/
    <slug>.toml                        # 메인테이너 1인 또는 채용 공고 1건당 파일
```

### 4.2 events/.../meta.toml

```toml
slug = "pycon-kr"
name = "PyCon Korea"
name_ko = "파이콘 한국"
type = "conference"           # "conference" | "meetup"
languages = ["python"]        # data/languages.toml 의 slug 값만 허용
homepage = "https://pycon.kr"
organizer = "PyCon Korea Preparatory Committee"
self_hosted_by = ""           # 자사 행사면 회사 slug. 빈 값이면 커뮤니티 행사
ended = ""                    # 행사 종료된 경우 마지막 개최 연도, 예: "2019"
```

### 4.3 events/.../YYYY.toml

```toml
year = 2024
held = true                   # 그 해 미개최면 false. 갭 무시 규칙용
event_url = "https://2024.pycon.kr"

[[sponsors]]
company = "toss"              # data/companies/<slug>.toml 의 slug 참조
tier = "diamond"              # 자유 라벨 (검증/정렬에 사용 안 함)
paid = true                   # streak 계산은 paid=true 만 카운트
note = ""

  [[sponsors.sources]]
  url = "https://2024.pycon.kr/sponsors"
  type = "official_event_page"
  archive_url = "https://web.archive.org/web/20241015.../sponsors"

  [[sponsors.sources]]
  url = "https://x.com/PyConKR/status/..."
  type = "social_media"
```

`source.type` enum:

- `official_event_page`
- `event_archive`
- `meetup_platform` (meetup.com, festa.io 등)
- `sponsor_announcement`
- `social_media`
- `press`
- `photo_evidence`
- `slide_deck`
- `other`

검증 규칙:

- 모든 `[[sponsors]]`는 최소 1개 이상의 source 필수
- `archive_url`은 권장 (필수 아님). `event_archive` type이면 url == archive_url
- 같은 회사가 같은 행사 같은 해에 중복 등장 금지
- `paid=true`인데 `tier`가 비어있으면 워닝

### 4.4 companies/<slug>.toml

```toml
slug = "toss"
name = "Toss"
name_ko = "토스"
homepage = "https://toss.im"
careers = "https://toss.im/career"
github = "https://github.com/toss"
aliases = ["Toss", "토스", "비바리퍼블리카", "Viva Republica", "(주)비바리퍼블리카"]
tracked_since = "2021"        # 회사 단위 streak 계산 시작 연도. 시드 누락과 진짜 미후원 구분용
dissolved = ""                # 합병/폐업한 경우 종료 연도, 예: "2023"
note_md = """
필요한 경우 정성적 설명. 예: 2020년부터 OSS 메인테이너의 업무 시간 사용을 정책적으로 허용.
"""
```

회사 정규화 규칙:

- 그룹 단위로 한 슬러그에 통합한다(자회사도 모기업으로). 예: 토스페이먼츠, 비바리퍼블리카는 모두 `toss` 슬러그.
- `aliases` 배열에 행사 페이지에서 등장하는 모든 표기를 나열한다(빌드 시 매칭 보조용).
- 데이터 입력은 슬러그만 허용. alias 문자열을 직접 events 파일에 쓸 수 없다.
- 두 회사가 같은 alias를 가지면 검증 실패.

### 4.5 maintainers/<slug>.toml

채용된 메인테이너와 회사가 낸 OSS 메인테이너 채용 공고를 같은 디렉토리에서 추적한다.

```toml
slug = "hong-gildong-some-project"     # 또는 "toss-2024-oss-job-posting"
kind = "employed"                      # "employed" | "job_posting"
company = "toss"                       # 회사 슬러그 참조
maintainer = "Hong Gildong"            # employed 일 때만
project = "some-project"               # 메인테이닝하는 OSS 프로젝트
project_url = "https://github.com/..."
since = "2021"                         # employed 시작 연도, 또는 채용 공고 게시 연도
posting_url = ""                       # job_posting 일 때 공고 URL

[[sources]]
url = "..."
type = "sponsor_announcement"          # 회사 블로그/PR이 가장 흔함
```

검증:

- `company` 슬러그가 `data/companies/`에 존재해야 한다.
- `kind=employed`면 `maintainer`와 `project` 필수.
- `kind=job_posting`이면 `posting_url` 필수.
- `[[sources]]` 최소 1개 필수.

### 4.6 data/languages.toml

언어 슬러그를 자유롭게 두면 표기가 흩어지므로 단일 화이트리스트로 강제한다.

```toml
[[language]]
slug = "python"
name = "Python"
aliases = ["py"]

[[language]]
slug = "javascript"
name = "JavaScript"
aliases = ["js"]

[[language]]
slug = "typescript"
name = "TypeScript"
aliases = ["ts"]

[[language]]
slug = "csharp"           # 슬러그에는 #/+ 등 특수문자 금지. 표시는 name 사용
name = "C#"
aliases = ["cs", "c-sharp"]

[[language]]
slug = "cpp"
name = "C++"
aliases = ["c++", "cplusplus"]

[[language]]
slug = "fsharp"
name = "F#"

[[language]]
slug = "objective-c"
name = "Objective-C"
aliases = ["objc"]

[[language]]
slug = "go"
name = "Go"
aliases = ["golang"]

# kotlin, java, rust, ruby, swift, php, scala, elixir, dart, lua, r, julia, zig 등 추가
```

규칙:

- 슬러그: 소문자 + ASCII 영문/숫자/하이픈만. 특수문자는 풀어 표기.
- events `meta.toml`의 `languages` 배열은 슬러그만 허용. alias는 검색/제안용.
- 새 언어 추가는 `languages.toml` 업데이트 PR과 같이 진행. 미정의 슬러그 사용 PR은 자동 차단.

## 5. streak 계산 규칙

streak은 두 가지 모두 보여준다. 컨퍼런스 단위 streak이 메인이고 회사 단위는 보조 인디케이터다.

### 5.1 공통 원칙

- `paid=true`인 후원만 카운트한다.
- `held=false`인 해(코로나 등으로 미개최)는 갭 무시(시퀀스에서 빠짐). 즉 "전년 후원 → 미개최 → 다다음 해 후원"이면 streak 유지.
- `self_hosted_by`가 비어있지 않은 행사(자사 행사)는 streak에서 제외.
- tier 변동은 streak에 영향 주지 않는다(Gold → Silver도 streak 유지). 본질이 "지속성"이기 때문.
- 회사가 alias 통합되기 전 다른 이름으로 후원했어도 빌드 시 정규화되어 streak에 합산.
- `dissolved` 회사는 dissolved 연도까지만 카운트.

### 5.2 컨퍼런스 단위 streak

특정 (회사, 행사) 페어에 대해 그 행사에서의 후원 연속 길이.

```
1. 행사의 모든 YYYY.toml 중 held=true 인 해만 정렬
2. 회사가 paid=true 로 등장한 해 집합을 구한다
3. held 시퀀스 위에서 sponsored가 연속된 부분 중 가장 긴 길이가 streak
4. 가장 최근까지 이어지면 active streak, 아니면 past streak
```

### 5.3 회사 단위 streak

회사가 한국 OSS 행사 중 어느 곳이든 매년 1개 이상 후원했는지 기준의 streak.

```
1. 모든 비-자사 행사 × 모든 해를 합쳐, 회사가 후원한 연도 집합을 구한다
2. 회사 TOML의 tracked_since 부터 현재 연도까지 시퀀스에서 연속 길이 계산
3. 그 해 한국 OSS 행사가 모두 미개최면 갭 무시
```

`tracked_since`는 시드 데이터 누락과 진짜 미후원을 구분하기 위함. 누군가 과거 데이터를 더 거슬러 추가하면 같은 PR에서 `tracked_since`도 갱신한다.

### 5.4 현재 연도 처리

"현재 연도"는 빌드가 실행된 시점의 UTC 연도로 정의한다(CLI에서 `--as-of YYYY`로 override 가능, 테스트와 결정론적 빌드용). 빌드 시점에 그 해 데이터가 입력되지 않았다면 streak는 깨지지 않는다(아직 입력 대기 중일 수 있음). 단, `docs/data-quality.md`에 "올해 OO 행사 데이터 미입력" 경고를 자동 표시한다.

### 5.5 종료된 행사

`meta.toml`의 `ended`가 채워진 행사는 ended 연도에서 streak가 frozen된다. README 매트릭스에 "🏁 ended" 라벨로 구분.

### 5.6 Hall of Fame (README 상단)

- **Longest active streak by event**: 컨퍼런스 단위 streak Top 10. 동률은 회사 슬러그 가나다(영문)순.
- **Longest active streak by company**: 회사 단위 streak Top 10.
- **Most events sponsored this year**: 올해 후원 행사 수 Top 10.

종료된 streak("Past legends")은 별도 섹션으로 두지 않고 회사 페이지 안에서만 표시한다.

### 5.7 매트릭스 표시

각 회사 섹션에 다음 매트릭스를 자동 생성한다(별 ⭐ 형식).

```
                  '21  '22  '23  '24  '25
파이콘 한국         ⭐   ⭐   ⭐   ⭐   ⭐    streak: 5y active
FEConf            -    ⭐   ⭐   ⭐   ⭐    streak: 4y active
JSConf Korea      -    -    -    -    -    🏁 ended (2019)
```

기호:

- `⭐` = 그 해 후원 (paid=true)
- `-` = 그 해 미후원
- `?` = 그 해 데이터 미입력 (data-quality.md에서 action item으로 노출)

tier는 매트릭스에 표시하지 않는다(streak 본질이 꾸준함이고 tier 변동을 무시하기로 했기 때문). 단 `tier` 필드 자체는 데이터로 보존한다 - 회사 페이지의 인용/노트, by-event 페이지, 향후 추가될 수 있는 분석에서 활용 가능하기 때문.

## 6. 빌드 파이프라인

### 6.1 언어/도구

- Python 3.12+
- uv로 환경 관리, `pyproject.toml`로 의존성 명시
- 의존성:
    - `tomllib` (표준 라이브러리)
    - `pydantic` v2 - 스키마 검증, 친절한 에러 메시지
    - `jinja2` - 마크다운 템플릿
    - `rich` - CLI 에러 출력 (파일/라인 강조, 색상)
    - `typer` - CLI 진입점
    - `httpx` - source URL 생존 검사 (`check-links`에서만 사용)
    - 테스트: `pytest`

### 6.2 패키지 구조

```
src/sponsors/
  __init__.py
  models.py              # Pydantic 모델
  loader.py              # TOML 로딩 + 참조 무결성 검사
  streaks.py             # streak 계산 (순수 함수)
  build.py               # 마크다운 생성
  cli.py                 # Typer 진입점
  templates/
    README.md.j2
    by_language.md.j2
    by_event.md.j2
    company_section.md.j2
    maintainers.md.j2
    data_quality.md.j2
tests/
  test_models.py
  test_loader.py
  test_streaks.py
  test_build.py
  fixtures/              # 테스트용 미니 데이터셋
pyproject.toml
```

### 6.3 CLI 명령

```bash
uv run sponsors validate           # 스키마 + 참조 무결성 검사
uv run sponsors build              # README.md, docs/by-language/*.md 등 생성
uv run sponsors check-links        # source URL 생존 검사 (느림, 별도 실행)
uv run sponsors stats              # streak 등 통계만 출력 (디버그용)
```

### 6.4 Validation 규칙

검증 항목:

1. 스키마 검증 (Pydantic)
2. 회사 슬러그 참조 무결성 (events → companies)
3. 메인테이너에서 참조하는 회사 슬러그 존재 여부
4. `[[sponsors.sources]]` 최소 1개
5. 같은 회사가 같은 해 같은 행사에 중복 등장 안 함
6. alias 충돌 (두 회사가 같은 alias를 갖지 않음)
7. `paid=true`인데 `tier` 비어있으면 워닝
8. `self_hosted_by` 회사 슬러그가 실제 존재
9. events `languages` 슬러그가 `data/languages.toml`에 정의되어 있음

에러 메시지 형식 (Pydantic + rich로 출력):

```
ValidationError: 2 errors
  data/events/conferences/pycon-kr/2024.toml:
    sponsors[0].sources: List should have at least 1 item, not 0
    sponsors[2].company: 'tos' references unknown company (did you mean 'toss'?)
```

- 파일 경로 + 필드 경로 + 이유 + (가능하면) "did you mean" 제안 (`difflib` 또는 `rapidfuzz`)
- validate가 실패하면 build는 실행하지 않는다.

### 6.5 출력물

빌드 결과물(`README.md` 등)은 GitHub Actions가 자동 커밋한다. 기여자는 빌드 환경 구성 없이 `data/`만 수정하면 된다.

```
README.md                          # 메인: hall of fame + 기업별 섹션
docs/by-language/python.md
docs/by-language/javascript.md
docs/by-language/<lang>.md         # 발견된 언어만큼 자동 생성
docs/by-event/pycon-kr.md          # 행사별 후원사 연도 매트릭스 (역방향 조회)
docs/by-event/feconf.md
docs/maintainers.md                # 채용된 OSS 메인테이너 + 채용 공고
docs/data-quality.md               # 누락 source, 미입력 행사 연도 등 리포트
```

README.md 구조:

1. 한 줄 소개 + 인정 기준 요약
2. Hall of Fame (자동 생성, 5.6 참조)
3. By Company (메인): 슬러그 정렬. 각 회사: 한 줄 소개 + 매트릭스 + 채용한 메인테이너 요약 + 인용
4. Indexes: by-language, by-event, maintainers 링크
5. Contributing 짧은 안내 (CONTRIBUTING.md 링크)

빌드는 결정론적이어야 한다(같은 입력 → 같은 출력). 정렬 순서, 시간 의존성 등을 고정하여 `git diff`가 데이터 변경에만 반응하도록 한다.

## 7. 기여 워크플로우

### 7.1 PR 흐름

1. fork + 브랜치 생성
2. `data/` 아래 TOML 파일만 수정/추가 (코드/빌드 환경 설치 불필요)
3. (선택) 로컬 검증: `uv run sponsors validate`
4. PR 생성 - 템플릿이 출처 링크와 변경 종류를 강제
5. CI가 자동 validate. 실패 시 머지 차단
6. 메인테이너 리뷰 + 머지
7. 머지 직후 GitHub Actions 봇이 README 빌드 + 후속 커밋 자동 push

### 7.2 PR 템플릿 (`.github/PULL_REQUEST_TEMPLATE.md`)

```markdown
## 변경 종류 (해당하는 것에 X)
- [ ] 새 회사 추가
- [ ] 새 이벤트 추가 (컨퍼런스/미트업)
- [ ] 후원 정보 추가 (특정 연도)
- [ ] 메인테이너 채용 정보 추가
- [ ] 데이터 정정/오류 수정
- [ ] 코드/빌드 변경

## 요약
<무엇을, 왜>

## 출처
<후원 사실을 입증할 수 있는 URL 1개 이상 - 행사 페이지, archive.org, 사진, 공식 발표 등>

## 체크리스트
- [ ] `uv run sponsors validate` 통과 (또는 CI에서 통과 확인)
- [ ] 모든 [[sponsors]]에 sources가 1개 이상
- [ ] 새 회사 추가 시 aliases와 homepage 채움
```

출처는 최소 1개 필수. 더 엄격하게(2개 이상, archive.org 동반) 가지는 않는다.

### 7.3 Issue 템플릿 (`.github/ISSUE_TEMPLATE/`)

PR을 만들기 어려운 사용자가 데이터를 제보할 수 있도록 폼 기반 issue 템플릿을 제공한다.

- `add-company.yml` - 회사 추가 요청
- `add-event-year.yml` - 특정 연도 후원 정보 제보
- `data-correction.yml` - 데이터 오류 신고

### 7.4 GitHub Actions 워크플로우 (`.github/workflows/`)

1. **`validate.yml`** - PR 및 push마다
    - `uv sync` → `uv run sponsors validate`
    - 실패 시 PR comment에 에러 위치 + 메시지 자동 게시

2. **`build.yml`** - main 머지 시
    - `uv sync` → `uv run sponsors build`
    - `git diff` 변경 있으면 `github-actions[bot]` 작성자로 후속 커밋 + push
    - 커밋 메시지에 `[skip ci]` 포함하여 무한 루프 방지
    - 워크플로우 가드: `actor != 'github-actions[bot]'`

3. **`check-links.yml`** - 매주 월요일 + 수동 (`workflow_dispatch`)
    - `uv run sponsors check-links` 실행
    - 죽은 링크 발견 시 워크플로우 실패만 발생시키고 사람이 보러 옴 (자동 issue 생성하지 않음)

4. **`data-quality.yml`** - 매월 1일
    - 누락된 source, 미입력 행사 연도, archive.org 누락 등 리포트
    - `docs/data-quality.md` 갱신 + 봇 커밋

### 7.5 거버넌스

- 초기에는 단독 메인테이너(@darjeeling) 머지 권한
- `MAINTAINERS.md` 파일을 두고 향후 공동 메인테이너 추가 시 명시
- 초기에는 CODEOWNERS 두지 않음

### 7.6 라이선스

- **데이터** (`data/`): CC BY 4.0
- **코드** (`src/`, 빌드 스크립트): MIT

데이터를 다른 곳에서 가져다 쓸 때 출처가 표시되어야 인정 목적과 부합한다.

### 7.7 GitHub Pages

당분간 사용하지 않는다. 마크다운만으로도 GitHub UI에서 충분히 읽힌다. 필요해지면 추후 별도 PR로 추가.

## 8. 비목표 / 추후 과제

다음은 본 단계에서 다루지 않으며 별도 후속 작업으로 둔다.

- 인프라/리소스 후원(C 범주) 추적
- 자사 OSS 공개(E 범주) 추적
- 기업의 후원 금액 정량화 (데이터 입수 어려움 + "장기성" 가치 기준에 부적합)
- GitHub Sponsors 자동 크롤링 (직접 후원은 우선순위 낮음)
- 회사 단위 정확한 자회사/지주회사 분리 (그룹 단위 통합 채택)
- GitHub Pages 사이트
- 다국어 README (한국어 단일)

## 9. 결정 사항 요약

| 항목 | 결정 |
|---|---|
| 목적 | 인정(recognition) 중심 |
| 후원 범위 | A(직접 금전), B(고용), D(이벤트). 우선순위 D > B > A |
| 1차 그룹화 | 기업별 (메인) + 언어별 인덱스 (세컨드, Python 우선) |
| 데이터 형식 | TOML, 빌드 시 마크다운 자동 생성 |
| 이벤트 종류 | 컨퍼런스 + 미트업 둘 다 1급 시민 |
| 시드 컨퍼런스 | 파이콘 한국, FEConf, JSConf Korea (커뮤니티 행사만) |
| 시드 시작 연도 | 2021 |
| 회사 정규화 | 그룹 단위 통합 + aliases 배열 |
| 출처 요구 | sources 배열, 최소 1개 필수, type enum |
| streak | 컨퍼런스 + 회사 단위 둘 다, paid=true만, 갭 무시, tier 변동 무시 |
| 매트릭스 표시 | 별 ⭐ / 미후원 - / 미입력 ? |
| Past streak | 회사 페이지 안에서만 표시 |
| 빌드 도구 | Python 3.12+, uv, pydantic, jinja2, rich, typer |
| Validation | PR 단위 자동, 실패 시 머지 차단 |
| 빌드 결과물 | github-actions[bot]이 머지 후 자동 커밋 |
| 죽은 링크 | 워크플로우 실패만, 자동 issue 생성 안 함 |
| 라이선스 | 데이터 CC BY 4.0 + 코드 MIT |

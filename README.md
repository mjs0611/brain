# brain

Git + 마크다운 세컨드 브레인. 노트를 쌓고, 임베딩으로 관련 노트를 꺼내고, Claude가 종합합니다.

## 새 머신에 옮기기

```bash
git clone git@github.com:mjs0611/brain.git ~/brain
git clone git@github.com:mjs0611/brain-vault.git ~/brain/vault   # private
cd ~/brain && bin/brain index
```

끝입니다. `bin/brain`은 `uv run` 셔뱅이라 의존성(fastembed·numpy·pyyaml)과 임베딩 모델을 첫 실행 때 알아서 받습니다. `index`가 pre-commit 훅(`core.hooksPath`)도 같이 걸어줍니다.

전제: [uv](https://docs.astral.sh/uv/), `git`, `rg`(ripgrep).

> **`~/Documents` 밑에 두지 마세요.** iCloud Drive가 심볼릭 링크로 걸려 있어 파일이 offload되면 0바이트로 읽힙니다. 이 저장소는 iCloud 밖(`~/brain`)에 있어야 합니다.

## 구조

```
~/brain/                  public repo (mjs0611/brain)
├── notes/                원자적 노트. 플랫, 파일명 = <id>-<slug>.md
├── sources/              출처 메타(책·아티클). 선택적
├── templates/note.md
├── bin/brain             CLI 단일 파일
├── .githooks/pre-commit  vault 누출 차단
├── .brain/               임베딩 인덱스. 로컬 산출물, gitignore
└── vault/                private repo (mjs0611/brain-vault), 부모가 gitignore
    ├── inbox/            모든 캡처가 떨어지는 곳
    ├── notes/
    └── sources/
```

**폴더가 아니라 태그로 분류합니다.** 원자적 노트는 여러 주제에 동시에 속하므로 폴더 트리는 반드시 어긋납니다. 파일명이 id 접두사라 `ls notes/`가 곧 시간순입니다.

## 명령

| 명령 | 하는 일 |
|---|---|
| `brain capture "텍스트"` | `vault/inbox/`에 타임스탬프 파일로 저장. stdin 파이프도 가능 |
| `brain refine --list` | 미처리 inbox 목록 |
| `brain refine <파일> --title … --visibility … --body -` | id·frontmatter 발급, 올바른 repo에 배치, inbox 소진 |
| `brain search <질의> [--tag T] [--scope public\|vault]` | ripgrep 키워드 검색 |
| `brain index [--rebuild]` | 증분 임베딩 → `.brain/index.npz` |
| `brain recall <질의> [-k 8] [--min-score 0.8]` | 의미 검색. 관련 노트 전문 출력 |
| `brain doctor [--staged]` | visibility↔위치 불변식 검증 |

## 리마커블 필기 가져오기

macOS 데스크톱 앱이 기기 xochitl 트리의 **전체 사본**을 로컬에 갖고 있습니다. 클라우드 API·USB·SSH 전부 불필요합니다.

```bash
bin/rm-export --list                   # 필기 있는 노트북 목록
bin/rm-export "Masters of SCALE"       # 페이지 순서대로 PNG
```

`.rm`(v6) → `rmc`로 SVG → `qlmanage`로 PNG. 필기가 그대로 읽히므로 **OCR도 기기의 Convert to Text도 필요 없습니다.** 출력은 `.brain/rm/<폴더>/<이름>/NNN.png` (gitignore).

- 페이지 순서는 `.content`의 `cPages.pages[].id`에서 옵니다. **디렉터리 나열 순서는 UUID 순이라 페이지 순서가 아닙니다**
- 기기 폴더가 곧 visibility 힌트입니다. `toss` · `Diary` · `Personal`은 **vault**, `Books` · `Readings` · `Math` · `Physics`는 대체로 public
- 렌더 실패 페이지는 건너뛰고 stderr에 남깁니다 (rmscene이 최신 포맷 일부를 아직 못 읽음)

## 두 저장소

| | public `brain` | private `brain-vault` |
|---|---|---|
| 담는 것 | 일반 지식 노트 | 회사 내부 정보, 개인사, 금융, **모든 미정제 캡처** |
| 판정 | 나머지 전부 | **애매하면 무조건 이쪽** |

### 누출 방어 3중

1. **캡처가 애초에 private에 떨어집니다.** 정제 전 원문이 public 워킹 트리에 존재한 적이 없으므로 실수로 커밋될 대상 자체가 없습니다
2. `vault/`는 public repo에서 gitignore — git이 순회조차 하지 않습니다
3. pre-commit 훅이 staged된 `.md`를 검사해 `visibility: vault`나 `vault/` 경로가 잡히면 커밋을 중단합니다 (`git add -f` 우회 대비)

`brain doctor`는 반대 방향도 봅니다. vault repo에 있는데 `visibility: public`인 노트도 실패입니다.

## 노트 규격

```yaml
---
id: 20260824T1147-3f9a     # 정렬 가능·충돌 안전. 파일명 접두사와 동일
created: 2026-08-24         # KST
source_type: book           # book | article | conversation | remarkable | thought
source: "사피엔스 / 유발 하라리, p.212"
tags: [조직설계, 인지]
visibility: public          # public | vault — 실제 위치와 항상 일치
related: [20260812T0930-b1c2]
---
```

`title` 필드는 없습니다. H1이 제목이고, 제목은 주제가 아니라 **주장 한 문장**입니다.

본문에서 `>` 블록쿼트는 **원문 그대로**, 나머지 문단은 **내 해석**입니다. 커스텀 문법이 아니라 마크다운 네이티브라 GitHub·Claude 앱·Obsidian 어디서나 그대로 읽힙니다.

## 임베딩

`intfloat/multilingual-e5-large` (1024차원)를 fastembed(ONNX)로 **로컬 실행**합니다. API 비용 0, 데몬 불필요, torch 불필요. 모델 약 1.1GB를 최초 1회 받아 `~/.cache/`에 둡니다.

로컬을 고른 결정적 이유는 비용이 아니라 vault입니다. 회사 내부 정보와 개인 금융 기록을 외부 API에 태우지 않습니다.

벡터는 `.npz` 하나에 정규화해 저장하고 검색은 numpy 내적 브루트포스입니다. 3천 노트 × 1024차원 = 12MB, 10ms 미만. 벡터 DB는 5만 노트를 넘으면 그때 붙입니다.

## Claude 앱에서

public repo는 GitHub 커넥터로 그냥 읽힙니다. vault는 private repo 접근 권한이 필요합니다.
`recall`(임베딩 검색)은 로컬 명령이라 **Claude Code에서만** 동작합니다. 앱에서는 파일 브라우징과 키워드 검색까지입니다.

## 자동화 (맥미니 이전 후)

`index`는 증분·멱등이라 cron에 그대로 걸립니다. `refine --list`가 미처리 건수를 stderr로 뱉으므로 정제 리마인드도 여기에 붙습니다.

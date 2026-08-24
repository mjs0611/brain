# brain — 세컨드 브레인

이 repo는 원자적 지식 노트 저장소입니다. 코드가 아니라 노트가 본체입니다.

## 저장소 구조

- `notes/` — public 노트. 1노트 = 1인사이트, 본문 300~800자, 플랫 구조(폴더 분류 없음, 태그로 분류)
- `sources/` — 출처 단위 메타(책·아티클). 선택적
- `vault/` — **private repo**(`brain-vault`)가 중첩돼 있고 부모가 gitignore 중. 회사 내부정보·개인사·금융은 전부 여기
- `vault/inbox/` — 정제 전 캡처. **모든 캡처는 항상 여기로 떨어집니다**(기본값 private)
- `bin/brain` — CLI. `uv run` 셔뱅이라 별도 설치 불필요

## 불변식

노트의 `visibility` 값과 실제 위치는 **항상** 일치합니다. `visibility: vault`인 노트는 `vault/notes/`에만 존재합니다. `brain doctor`가 검증하고 pre-commit 훅이 강제합니다. 이 규칙을 우회하지 마세요.

## June이 "~에 대해 뭐 알고 있었지" 류로 물으면

1. `bin/brain recall "<질의>" -k 8` 실행
2. 출력된 노트 **전문**을 읽고 **종합·재해석**해서 답변. 노트 목록을 그대로 나열하지 말 것
3. 노트 간 긴장·모순이 보이면 그걸 먼저 짚기. 그게 이 시스템의 존재 이유입니다
4. 인용한 노트는 id로 밝히기 (`20260824T1147-3f9a`)
5. recall이 비면 `bin/brain search`로 키워드 폴백

## 새 노트를 만들 때

`refine`은 배관만 담당합니다 — id 발급, frontmatter, 올바른 repo에 배치, inbox 소진. **원자화와 해석은 Claude가 합니다.**

```bash
bin/brain refine --list                      # 미처리 inbox 확인
bin/brain refine vault/inbox/20260824-143022.md \
  --title "허구를 공유해야 150명 넘는 협력이 생긴다" \
  --visibility public --source-type book \
  --source "사피엔스 / 유발 하라리, p.212" \
  --tags "조직설계,인지" --body - <<'EOF'
> 대규모 인간 협력은 모두 공통의 신화에 뿌리를 둔다.
> — 사피엔스, p.212

**해석.** ...
EOF
```

캡처 하나가 인사이트 두 개면 노트 두 개로 쪼갭니다. `--keep`으로 inbox 파일을 남긴 뒤 두 번째 노트를 만들고, 마지막 노트에서만 소진하세요.

**visibility 판정:** 회사 업무 내부 정보 / 개인사 / 금융은 **애매하면 무조건 `vault`**. 판단이 서지 않으면 vault입니다.

## 리마커블 필기에서 노트 만들 때

`bin/rm-export "<노트북 이름>"`으로 PNG를 뽑고 Read로 직접 읽습니다. 필기가 그대로 읽히므로 OCR을 붙이지 마세요.

기기 폴더가 visibility 1차 힌트입니다 — `toss`(회사 업무) · `Diary` · `Personal`은 **vault**. 단 폴더는 힌트일 뿐이고, `Books` 안에 있어도 내용이 회사 건이면 vault입니다.

⚠️ 리마커블 필기는 **이미 June이 압축한 결과**입니다. 원문 인용이 아니라 June의 요약이므로, 블록쿼트에 넣을 때 원저자 말인 것처럼 쓰지 마세요. 출처는 `source_type: remarkable`로 하고 책 이름은 `source`에 적습니다.

## 본문 포맷

`>` 블록쿼트 = 원문 그대로. 나머지 문단 = 내 해석. 이 구분을 섞지 마세요.
제목(H1)은 주제가 아니라 **주장 한 문장**입니다. "인지혁명"이 아니라 "허구를 공유해야 150명 넘는 협력이 생긴다".

## 노트를 고친 뒤

`bin/brain index` — 증분이라 바뀐 노트만 다시 임베딩합니다.

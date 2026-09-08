# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

이 저장소는 코드가 아니라 **Claude Code Agent Skill 모음**이다. 마크다운뿐이고
빌드·테스트·린트·패키지 매니저가 없다. 찾지 마라.

## 이 저장소의 스킬은 여기서 동작하지 않는다

스킬은 소비하는 쪽 디렉터리에 놓여야 활성화된다. 여기서 `SKILL.md` 를 고쳐도
어디의 동작도 바뀌지 않는다 — 사본이 이미 배포돼 있다면 그 사본이 낡는다.

| 배포 위치 | 용도 |
|---|---|
| `<대상 저장소>/.claude/skills/<name>/` | 팀 공유. 그 저장소에 커밋된다 |
| `~/.claude/skills/<name>/` | 개인용. 공유되지 않는다 |

`code-review-pr` 의 사본이 `~/dev/devgrr/eland/brand/.claude/skills/` 에 있고
아직 커밋되지 않았다. 여기를 고치면 그쪽도 같이 봐야 한다.

## 세 스킬은 리뷰 사다리다

같은 일을 셋으로 나눈 게 아니라, **변경이 어느 단계에 있느냐**로 나뉜다. 각각
다른 상류 리뷰어를 감싼다.

| 스킬 | 대상 | 감싸는 것 |
|---|---|---|
| `code-review-before-commit` | 커밋 전 워킹 트리 (staged + unstaged + untracked) | `pr-review-toolkit:review-pr` |
| `code-review-add-feature` | 커밋된 기능 범위 `BASE..HEAD` | `superpowers:requesting-code-review` |
| `code-review-pr` | 올라온 PR 번호 | 빌트인 `code-review` |

## 세 스킬이 공유하는 설계 — 새 스킬도 이걸 따른다

리뷰 자체를 새로 만들지 않는다. 상류 도구를 부르고, **그 도구가 놓치는 두 가지만
보탠다.** 그게 래퍼의 존재 이유다.

1. **범위 교정.** 상류의 기본 범위는 대개 틀리다. `review-pr` 은 `git diff` 라
   스테이징·untracked 를 놓치고, `requesting-code-review` 는 SHA 범위 밖 워킹 트리를
   못 보고, 빌트인 `code-review` 는 target 을 검증하지 않아 문장을 넘기면 에러 없이
   엉뚱한 범위를 리뷰한다. 각 스킬 1단계가 이걸 다룬다.
2. **프로젝트 규칙 축.** 상류는 일반적인 버그만 본다. 빌드도 린트도 테스트도 잡지
   않는 규칙(테스트가 어디 살아야 하는지, `okf/` 를 같은 변경에서 갱신했는지)은
   리뷰가 안 잡으면 아무도 안 잡는다.

**규칙 내용을 SKILL.md 에 복사하지 마라.** 대상 저장소 `CLAUDE.md` 의 절 이름만
가리킨다. 복사본은 원본이 바뀔 때 조용히 어긋나고, 어긋난 줄 아무도 모른다.
`code-review-pr` 의 축 표가 그 형태다 — 절 이름과 "diff 에 이게 있으면 본다" 뿐이다.

그 밖의 공통 규약:

- **보고만 한다.** 파일 수정·커밋·PR 코멘트 등록은 사용자가 명시적으로 요청할 때만.
  외부에 보이는 행위와 리뷰는 다른 결정이다.
- 상류 에이전트 리포트를 이어 붙이지 않는다. **심각도 표 하나**로 합친다.
- 위치는 `파일:줄`. "인증 로직 부근" 은 위치가 아니다.
- 사용자 인덱스를 건드리지 않는다. `git add -N` / `git add` / `git stash` 로 범위를
  맞추려 하지 말고 물어본다.

## SKILL.md 형식

- frontmatter 는 `name`, `description` 둘뿐이다.
- `description` 이 유일한 트리거 수단이다. 무엇을 하는지 + 실제 호출 문구
  ("PR 리뷰해줘", "커밋 전에 리뷰") + 언제 쓰면 안 되는지를 넣는다. 스킬은
  과소 트리거되는 쪽으로 치우치므로 다소 밀어붙이는 문장이 맞다.
- 본문은 한국어 명령형. 규칙마다 **왜** 그런지를 붙인다 — 이유 없는 규칙은 지켜지지 않는다.
- 마지막은 `## 하지 말 것`. 조용히 틀리는 실패 모드를 적는 자리다.

## 커밋

`<type>: <한글 subject>` — type 접두사만 영어. 스킬 하나가 커밋 하나다.

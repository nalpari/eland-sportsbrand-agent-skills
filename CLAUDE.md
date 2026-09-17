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

## 네 스킬은 리뷰 사다리다

같은 일을 넷으로 나눈 게 아니라, **변경이 어느 단계에 있느냐**로 나뉜다.

| 스킬 | 대상 | 리뷰 주체 |
|---|---|---|
| `code-review-before-commit` | 커밋 전 워킹 트리 (staged + unstaged + untracked) | `pr-review-toolkit:review-pr` |
| `code-review-add-feature` | 커밋된 기능 범위 `BASE..HEAD` | `superpowers:requesting-code-review` |
| `code-review-pr` | 올라온 PR 번호 | 빌트인 `code-review` |
| `code-review-final` | 머지 직전 PR (브랜치를 체크아웃한다) | 감싸지 않는다. Opus 서브에이전트 3개를 직접 띄운다 |

앞의 셋은 여기서 만든 **래퍼**고, `code-review-final` 은 외부 저장소에서 그대로
가져온 것이다. 규약이 다르니 섞어 읽지 마라 (아래 예외 절).

`code-review-pr` 과 `code-review-final` 은 둘 다 PR 이 대상이니 description 의 호출
문구를 겹치게 쓰지 않는다 — "PR 리뷰해줘"/"#123 리뷰" 는 `code-review-pr`,
"머지해도 되는지 봐줘"/적대적 리뷰/코멘트 등록은 `code-review-final` 이 가져간다.
`code-review-final` 은 `disable-model-invocation: true` 여서 모델 자동 트리거 대상이
아니고, 겹치는 문구가 생기면 충돌이 아니라 죽은 트리거가 된다. 새 PR 리뷰 스킬을
더할 때도 이 경계를 따른다.

## 래퍼 셋이 공유하는 설계 — 새 래퍼도 이걸 따른다

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

### code-review-final 은 이 규약을 따르지 않는다

`nalpari/interplug-team-agent-skills` 의 `ip-code-review-claude` 를 가져온 것이다
(`name` 필드만 바꿨다). 전제가 달라서 위 규약에 맞추려 들면 스킬이 망가진다.

| 위 규약 | code-review-final |
|---|---|
| 상류 도구를 감싼다 | 감싸지 않는다. Opus 서브에이전트 3개를 한 메세지에 동시에 띄운다 |
| 프로젝트 규칙 축을 얹는다 | 없다. 대상 저장소 `CLAUDE.md` 를 읽지 않는다 |
| 보고만 한다 | 브랜치를 체크아웃하고, PR 코멘트 등록이 산출물이다 |
| 심각도 표 하나 | 머지 블로커만 남기고 major/minor 는 버린다 |
| 모델이 description 으로 고른다 | `disable-model-invocation: true` — 사용자가 `/code-review-final` 로만 건다 |

마지막 두 줄은 여기서 수동으로 얹은 개입이다. 원본 `ip-code-review-claude` 의
description 은 "PR 리뷰해줘" 를 `code-review-pr` 과 겹치게 쓰고, frontmatter 에 이
필드가 없다. **원본을 다시 가져오면 두 가지가 모두 돌아온다** — description 충돌이
재발하니 재-가져온 뒤 이 절대로 다시 갈라놓아라.

원본이 갱신돼도 여기로 자동으로 따라오지 않는다. 다시 가져와야 한다.

## grilling 은 리뷰 스킬이 아니고, 아래 형식도 따르지 않는다

`mattpocock/skills` 의 `skills/productivity/grilling/SKILL.md` 를 커밋 `170ad48`
(2026-07-13) 시점 그대로 가져왔다. 한 글자도 바꾸지 않았다.

- 본문이 영어고 `## 하지 말 것` 절이 없다. **한국어로 옮기거나 절을 보태지 마라.**
  원본과 `diff` 로 대조할 수 있어야 버전을 확인하고 다시 가져올 수 있다.
- 원본 최신판(라운드 방식, 한 라운드에 질문 여러 개)으로 올리지 마라. 한 번에 하나씩
  묻는 이 버전을 일부러 고정한 것이다.
- 확인은 원본 저장소에서
  `git show 170ad48:skills/productivity/grilling/SKILL.md | diff - grilling/SKILL.md`.

## SKILL.md 형식

- frontmatter 는 `name`, `description` 둘뿐이다. 유일한 예외는 `code-review-final` 의
  `disable-model-invocation: true` — 아래 "code-review-final 은 이 규약을 따르지 않는다" 절 참조.
- `description` 이 유일한 트리거 수단이다. 무엇을 하는지 + 실제 호출 문구
  ("PR 리뷰해줘", "커밋 전에 리뷰") + 언제 쓰면 안 되는지를 넣는다. 스킬은
  과소 트리거되는 쪽으로 치우치므로 다소 밀어붙이는 문장이 맞다.
- 본문은 한국어 명령형. 규칙마다 **왜** 그런지를 붙인다 — 이유 없는 규칙은 지켜지지 않는다.
- `disable-model-invocation: true` 는 "사용자가 슬래시로 치는 관문" 류에만 붙인다.
  모델 자동 트리거를 원천 차단하므로, 모델이 먼저 뜨는 게 맞는 스킬에 붙이면 죽은 스킬이 된다.
- 마지막은 `## 하지 말 것`. 조용히 틀리는 실패 모드를 적는 자리다.

## 커밋

`<type>: <한글 subject>` — type 접두사만 영어. 스킬 하나가 커밋 하나다.

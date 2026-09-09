# eland-sportsbrand-agent-skills

변경이 어느 단계에 있느냐에 따라 갈라지는 **Claude Code 코드리뷰 스킬 4종**.
마크다운뿐이고 빌드할 것이 없다.

## 무엇인가

Claude Code 에는 이미 여러 리뷰 도구가 있다. 문제는 두 가지다.

1. **기본 범위가 대개 틀리다.** `git diff` 로 범위를 잡는 도구는 이미 `git add` 한
   파일과 아직 추적 안 되는 신규 파일을 놓친다. 커밋 직전에는 그 둘이 변경의
   대부분이다.
2. **저장소 고유 규칙을 안 본다.** 테스트가 어느 레이어에 살아야 하는지, 문서 번들을
   같은 변경에서 갱신했는지 — 빌드도 린트도 테스트도 안 잡는다. 리뷰가 놓치면
   아무도 안 잡는다.

앞의 세 스킬은 리뷰를 새로 만들지 않는다. 기존 도구를 부르고 저 두 가지만 보탠다.
네 번째 `code-review-final` 만 다르다 — 머지 직전 관문이라 자체 서브에이전트로
적대적 리뷰를 돌리고 결과를 PR 코멘트로 남긴다.

## 설치

디렉터리째 복사하면 끝이다.

```bash
# 팀 공유 — 대상 저장소에 커밋된다
mkdir -p <대상 저장소>/.claude/skills
cp -R code-review-pr <대상 저장소>/.claude/skills/

# 개인용 — 공유되지 않는다
cp -R code-review-before-commit ~/.claude/skills/
```

## 어느 스킬을 언제

| 단계 | 스킬 | 호출 예 |
|---|---|---|
| 아직 커밋 안 함 | `code-review-before-commit` | "커밋 전에 리뷰해줘" |
| 기능 하나 완성, 커밋됨 | `code-review-add-feature` | "기능 구현 완료했어 리뷰해줘" |
| PR 올라감 | `code-review-pr` | "#123 리뷰해줘" |
| 머지 직전 | `code-review-final` | "머지해도 되는지 봐줘" |

> `code-review-pr` 과 `code-review-final` 은 둘 다 PR 이 대상이고 호출 문구가 겹친다
> ("PR 리뷰해줘"). 한 저장소에 같이 설치하면 어느 쪽이 뜰지 예측이 안 되니, 둘 중
> 하나만 설치하거나 `description` 의 호출 문구를 갈라 놓아라.

### code-review-before-commit

워킹 트리 전체 — 스테이징·언스테이징·untracked 신규 파일 — 를 대상으로
`pr-review-toolkit:review-pr` 의 에이전트들을 병렬로 돌린다. 범위를 먼저 확정해
넘기므로 `git add` 한 파일이 리뷰에서 빠지지 않는다. 인덱스는 건드리지 않는다.

### code-review-add-feature

`superpowers:requesting-code-review` 로 리뷰어를 띄우되, 리뷰어 프롬프트에 대상
저장소의 테스트 정책과 문서 번들 동시 갱신 규칙을 체크리스트로 박아 넣는다.
범위는 `merge-base ... HEAD`. 커밋 안 된 변경이 있으면 멈추고 묻는다 — SHA 범위
밖이라 리뷰에서 통째로 빠지기 때문이다.

### code-review-pr

빌트인 `code-review` 를 PR 번호로 부르고, 결과에 저장소 규칙 축별 판정을 붙여
표로 낸다. 팀원 전원이 같은 기준으로 PR 을 보게 하는 것이 목적이다.
`ultra` 는 부르지 않고, `--comment`/`--fix` 는 요청받았을 때만 붙인다.

### code-review-final

PR 번호와 브랜치를 받아 **체크아웃한 뒤**, Opus 서브에이전트 3개를 정확성 /
보안·데이터 무결성 / 통합·회귀 관점으로 동시에 띄워 적대적으로 공격한다. 리드는
직접 리뷰하지 않고 그 결과에서 **머지 블로커만** 걸러 해결 방안을 붙이고, 표를
사용자에게 보여준 뒤 PR 코멘트로 등록한다. major·minor·취향 지적은 버린다.

`nalpari/interplug-team-agent-skills` 의 `ip-code-review-claude` 를 가져온 것이다.
원본이 갱신돼도 자동으로 따라오지 않는다.

## 전제

| 스킬 | 필요한 것 |
|---|---|
| `code-review-before-commit` | `pr-review-toolkit` 플러그인 |
| `code-review-add-feature` | `superpowers` 플러그인 |
| `code-review-pr` | 빌트인 `code-review`, `gh` CLI |
| `code-review-final` | `gh` CLI, Opus 서브에이전트를 띄울 수 있는 세션 |

앞의 셋은 리뷰 결과를 **보고만 한다.** 파일 수정·커밋·PR 코멘트 등록은 명시적으로
요청했을 때만 한다.

`code-review-final` 은 예외다. **브랜치를 체크아웃하고** (워킹 트리가 더러우면 멈추고
알린다) **PR 코멘트를 등록한다.** 코멘트는 팀 전체에 보인다. 리뷰만 보고 싶으면
`code-review-pr` 을 써라.

## 다른 저장소에 옮길 때

규칙 축은 대상 저장소의 `CLAUDE.md` 절 이름을 가리킨다. 규칙 내용을 SKILL.md 에
복사하지 않기 때문에 원본이 바뀌어도 어긋나지 않지만, **절 이름이 다른 저장소에
옮기면 그 표를 고쳐야 한다.** `code-review-pr` 의 축 표와 `code-review-add-feature`
의 규칙 블록이 그 자리다.

대상 저장소에 `CLAUDE.md` 가 없으면 규칙 축은 그냥 건너뛰고, 남는 것은 범위 교정과
표 정리다. 그것만으로도 쓸모는 있지만 이 스킬들의 절반이다.

`code-review-final` 은 규칙 축이 없어서 이 작업이 필요 없다. 어느 저장소에 놓든
그대로 동작한다.

## 기여

설계 원칙과 새 스킬을 추가할 때의 규약은 [CLAUDE.md](CLAUDE.md) 에 있다.

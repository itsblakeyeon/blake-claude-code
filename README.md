# blake-claude-code

Blake's personal Claude Code plugin.

## Skills

| Skill | Description | Usage |
|-------|-------------|-------|
| `job-match` | 채용공고 크롤링 → 프로필 매칭 → 솔직한 추천 | `/blake-claude-code:job-match <URL>` |

## Agents

| Agent | Description |
|-------|-------------|
| `job-analyzer` | JD 분석 및 프로필 매칭 전문 에이전트 |

## Installation

```bash
# Local install (from plugin directory)
claude plugin install ~/blake-claude-code
```

## Adding New Skills

`skills/` 아래에 폴더를 만들고 `SKILL.md`를 추가하면 됨:

```
skills/
├── job-match/SKILL.md
├── resume-custom/SKILL.md    # 예: 이력서 커스터마이징
└── new-skill/SKILL.md        # 아무거나 추가 가능
```

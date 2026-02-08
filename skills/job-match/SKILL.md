---
name: job-match
description: 채용공고 크롤링 → 프로필 매칭 → 솔직한 추천
---

# Job Match

채용 페이지 URL을 받아서 전체 공고를 크롤링하고, 내 프로필과 매칭하여 가장 핏한 공고를 추천한다.

## Usage

```
/blake-claude-code:job-match <채용페이지 URL>
```

## 실행 프로세스

### Step 1: 프로필 로드

아래 파일을 읽어서 후보자 프로필을 파악한다:

1. **이력서** — `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/Default/03.Projects/job/templates/이력서_en.md`
2. **SOURCE (성향/강점)** — `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/Default/03.Projects/job/references/self/01.SOURCE.md`
3. **TARGET (비전/목표)** — `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/Default/03.Projects/job/references/self/03.TARGET.md`

핵심 요약:
- Blake Yeon (연준현), PM/BizDev/Growth, ~5년 경력
- 현직: 화이트큐브 PM (차량 구독 서비스, 10개월)
- 창업: 롸잇 공동창업 (프리랜서 매칭, 2년 8개월)
- 에이전시: BAT 그로스 마케터/파트 리드 (1.5년)
- 보험: 삼성화재 영업 (1년)
- 스킬: Claude Code, Supabase, Vercel, Webflow, Zapier, SQL, Python, Mixpanel, Google/Meta/Naver Ads
- 학력: 고려대 체육교육+패션디자인머천다이징
- 언어: **한국어 네이티브, 영어 비즈니스 레벨 미달** (영어 유창 필수 포지션은 현실적으로 어려움)
- 목표: 네임벨류 확보 → 창업 레버리지

### Step 2: 채용 페이지 크롤링

1. URL을 WebFetch로 시도
2. 동적 페이지(SPA/무한스크롤)인 경우:
   - 소스에서 API 엔드포인트 탐색
   - pagination 파라미터 파악 (firstIndex, page, offset 등)
   - totalRows/totalCount 확인하여 전체 수량 파악
   - API 직접 호출로 전체 목록 수집
3. API 접근 불가 시:
   - WebSearch로 `site:{도메인}` 검색
   - 서드파티(LinkedIn, Glassdoor, Outscal 등)에서 보충
4. **전체 공고 수를 유저가 알려준 수와 대조하여 누락 없는지 확인**

### Step 3: 1차 필터링

전체 공고에서 아래 기준으로 빠르게 걸러낸다:

| 필터 | 기준 |
|------|------|
| 인턴/학생/신입 | 제외 (경력 5년+) |
| 순수 엔지니어링 | 제외 (SWE, ML Engineer 등) |
| 경력 10년+ 필수 | 제외 (자격 미달) |
| 영어 유창 필수 (Minimum) | **경고 표시** — 현실적으로 어려움 |
| PhD 필수 | 제외 |

### Step 4: 상세 JD 크롤링

1차 통과한 공고의 상세 JD를 크롤링한다.
- 개별 공고 페이지 WebFetch 시도
- 안 되면 WebSearch로 공고 제목 검색하여 스니펫/서드파티에서 확보

### Step 5: 자격요건 중심 매칭

각 공고마다 아래 테이블 작성:

| 요건 | JD 요구사항 | Blake 경험 | 판단 |
|------|------------|-----------|------|
| 최소 경력 | X년 | 5년+ | ✅/❌ |
| 필수 스킬 | ... | ... | ✅/⚠️/❌ |
| 언어 | ... | 영어 비즈니스 미달 | ✅/❌ |
| 기타 | ... | ... | ... |

**솔직함 원칙:**
- "유사 경험" ≠ "동일 경험" → 구분해서 표기 (⚠️)
- 바이사이드(광고 집행) ≠ 셀사이드(광고 판매)
- "할 수 있을 것 같다" 금지 → 실제로 한 경험만 매칭
- 우대사항은 우대사항으로, 필수와 분리 판단

### Step 6: 계약 형태 확인

- 공고 타이틀에 "Fixed-Term Contract", "계약직", "FTC" 등 표시 확인
- 정규직/계약직 여부 명시

### Step 7: 최종 추천

**매칭 스코어 (10점):**
- 자격요건 충족: 4점
- 경험 직결성: 3점
- 커리어 방향 정합 (네임벨류/창업 레버리지): 2점
- 성장 가치: 1점

**출력 형식:**

1. 전체 공고 요약 테이블
2. Top 3 상세 분석 (자격요건 매칭 + 솔직한 포인트 + 링크)
3. 탈락 공고 요약 (공고명 + 탈락 사유)
4. 결론 (솔직한 총평)

## Examples

```
/blake-claude-code:job-match https://recruit.navercorp.com/rcrt/list.do?...
```

→ 네이버 전체 공고 크롤링 → 28개 중 현실적 후보 3개 추천

```
/blake-claude-code:job-match https://www.coupang.jobs/en/jobs/?...
```

→ 쿠팡 전체 공고 크롤링 → 핏한 공고 추천

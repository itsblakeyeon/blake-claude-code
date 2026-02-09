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

1. **이력서(한글)** — `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/Default/03.Projects/job/templates/이력서_kr.md`
2. **이력서(영문)** — `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/Default/03.Projects/job/templates/이력서_en.md`
3. **SOURCE (성향/강점)** — `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/Default/03.Projects/job/references/self/01.SOURCE.md`
4. **TARGET (비전/목표)** — `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/Default/03.Projects/job/references/self/03.TARGET.md`
5. **이력서 가이드** — `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/Default/03.Projects/job/CLAUDE.md`

**경로에 공백 있음 → Bash 사용 시 항상 따옴표로 감싸기**

핵심 요약:
- Blake Yeon (연준현), PM/BizDev/Growth, 경력 ~6년 (2019.07~)
- 현직: 화이트큐브 PM & BizDev (차량 구독 서비스 '패러데이', 2025.05~)
- 창업: 롸잇 공동창업 (프리랜서 매칭 '원포인트', 3년, MVP→PMF→시드투자→BEP)
- 에이전시: BAT 그로스 마케터/파트 리드 (1.5년)
- 보험: 삼성화재 영업 (1년)
- 스킬: Claude Code, Supabase, Vercel, Webflow, Zapier, SQL, Python, Mixpanel, Google/Meta/Naver Ads
- 학력: 고려대 체육교육+패션디자인머천다이징
- 언어: **한국어 네이티브, 영어 비즈니스 레벨 미달** (영어 유창 필수 포지션은 현실적으로 어려움)
- 목표: 네임벨류 확보 → 창업 레버리지

### Step 2: 전체 제목 수집

**핵심: 전체 공고 제목+URL을 먼저 전부 수집한다. 키워드 검색이 아님.**

키워드 검색은 누락이 생긴다 (예: 223건 중 키워드 매칭 152건만 나옴 = 71건 미확인).
반드시 페이지네이션 전체를 순회해서 모든 제목을 수집해야 한다.

#### 크롤링 순서

1. **WebFetch 시도** → 대부분 Cloudflare/봇차단으로 403 실패
2. **Playwright 비헤드리스 브라우저** (주력 방법):
   ```python
   from playwright.sync_api import sync_playwright
   browser = p.chromium.launch(
       headless=False,  # 반드시 False (Cloudflare 우회)
       args=['--disable-blink-features=AutomationControlled']
   )
   context = browser.new_context(
       user_agent='Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) ...',
       viewport={'width': 1920, 'height': 1080}, locale='ko-KR'
   )
   page = context.new_page()
   page.add_init_script("Object.defineProperty(navigator, 'webdriver', {get: () => undefined});")
   ```
   - 페이지네이션 전체 순회 (page=1,2,3...)
   - Cloudflare 챌린지 시 `time.sleep(5~10)` 후 재시도
   - 각 페이지에서 공고 제목+URL 추출 (정규식 or querySelector)
3. **API 탐색** (보조): 페이지 소스에서 API 엔드포인트 발견 시 직접 호출
4. **WebSearch 보충** (최후수단): `site:{도메인}` 검색

**수집 완료 후 반드시 유저가 알려준 총 공고 수와 대조하여 누락 확인.**

### Step 3: 제목 기반 1차 필터링

전체 공고에서 **제목만 보고** 빠르게 걸러낸다:

| 제외 대상 | 예시 |
|-----------|------|
| 순수 엔지니어링 | Backend/Frontend/Mobile/ML/Infra/SRE/DBA |
| 디자이너 | Product Designer, UX Designer |
| Data Scientist/Engineer | 분석 '전담'직 (PM이 아닌 것) |
| TPM | Technical Program Manager |
| 법무/컴플라이언스 | Legal, Compliance, 사내변호사 |
| 재무/회계/세무 | Finance, Tax, Accounting |
| HR/채용/노무 | Recruiting, Employee Relations |
| 보안/인프라 | Security Engineer, Cloud Infra |
| 경력 10년+ 필수 | 자격 미달 |
| 계약직+직무 불일치 | 계약직이면서 매칭도 안 되는 경우 |
| 물류/현장/설비 | 물류센터, 설비보전 |

**필터링 결과를 유저에게 보고:** "전체 N건 중 X건 제외, Y건 남음. 제외 카테고리별 건수: ..."

유저 확인 후 다음 단계 진행.

### Step 4: 상세 JD 크롤링

1차 통과한 공고만 상세 페이지 크롤링한다.
- Playwright 동일 방식 사용
- HTML에서 텍스트 추출 (태그 제거, unescape)

### Step 5: 자격요건 중심 매칭

각 공고마다 아래 테이블 작성:

| 요건 | JD 요구사항 | Blake 경험 | 판단 |
|------|------------|-----------|------|
| 최소 경력 | X년 | ~6년 | ✅/❌ |
| 필수 스킬 | ... | ... | ✅/⚠️/❌ |
| 언어 | ... | 영어 비즈니스 미달 | ✅/❌ |
| 기타 | ... | ... | ... |

**솔직함 원칙:**
- "유사 경험" ≠ "동일 경험" → 구분해서 표기 (⚠️)
- 바이사이드(광고 집행) ≠ 셀사이드(광고 판매)
- "할 수 있을 것 같다" 금지 → 실제로 한 경험만 매칭
- 우대사항은 우대사항으로, 필수와 분리 판단
- **JD에 명시된 요건만으로 판단한다. JD에 없는 요건을 추측/가정하지 않는다.**
  - 회사가 금융회사라는 이유만으로 금융 도메인 경험을 요구사항으로 가정하지 않는다
  - JD에 "금융 경험 필수"라고 안 써 있으면 금융 경험 부족을 감점 사유로 쓰지 않는다
  - JD에 "영어 유창 필수"라고 안 써 있으면 영어를 감점 사유로 쓰지 않는다
  - "이런 분이면 더 좋아요" (우대사항)는 필수가 아니다. 없어도 감점하지 않는다
  - 채점은 쿠팡/토스 등 회사 간 동일 스케일을 유지해야 한다

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

1. 전체 공고 요약 테이블 (점수순 정렬)
2. Top 5 상세 분석 (자격요건 매칭 + 솔직한 포인트 + 링크)
3. 탈락 공고 요약 (공고명 + 탈락 사유)
4. 지원 전략 제안 (우선순위 + 이력서 강조점)

### Step 8: 결과 저장

분석 결과를 job 프로젝트 폴더에 저장:
- `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/Default/03.Projects/job/customized/{회사명}/all_titles.json`
- `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/Default/03.Projects/job/customized/{회사명}/matched_jds.json`
- `~/Library/Mobile Documents/iCloud~md~obsidian/Documents/Default/03.Projects/job/customized/{회사명}/analysis.md`

## Examples

```
/blake-claude-code:job-match https://www.coupang.jobs/kr/jobs/?location=Seoul
```

→ 쿠팡 서울 전체 222건 제목 수집 → 26건 필터링 → 상세 JD 크롤링 → 매칭 분석 → TOP 5 추천

---
name: RSA_report
description: >
  표준 문서 양식(HTML 템플릿)을 골라 placeholder를 채워주는 보고서 작성 스킬.
  사용자가 "문서를 작성해줘", "보고서 써줘", "report 양식으로", "RSA report",
  "/RSA_report" 라고 하면 사용하세요. 사용 가능한 양식 목록을 보여주고, 고른
  템플릿의 {{...}} 자리를 채워 현재 작업 폴더에 HTML 문서를 생성합니다.
---

# RSA_report — 양식 기반 문서 작성

사용자가 고른 HTML 양식의 `{{...}}` placeholder를 채워 완성 문서를 만든다.
양식은 이 스킬의 `templates/` 폴더에 있는 `.html` 파일들이며, 사용자가 자유롭게
추가/수정/삭제한다. 스킬 코드는 양식을 건드리지 않는다.

## 절차

### 1. 양식 선택 (즉시 — 도구 호출 없이)
**아래 `양식 카탈로그`는 이 SKILL.md 안에 있고, `/RSA_report` 호출 시 이미 모델
컨텍스트에 로드되어 있다. 따라서 파일을 스캔하거나 읽지 말고, 곧바로
`AskUserQuestion` 으로 카탈로그의 양식들을 제시한다 (label=name, description=desc).**
도구 왕복을 1회 줄여 선택 메뉴가 즉시 뜨게 하는 것이 핵심이다. 메뉴를 띄우기 전에
어떤 파일도 읽거나 편집하지 않는다.

선택된 양식의 `file` 을 기억해 3단계에서 해당 HTML 만 읽는다.
양식이 1개뿐이면 그것을 쓰되 한 번만 확인한다.

#### 양식 카탈로그
| file | name | desc | naming |
|---|---|---|---|
| `L2_개발_Report.html` | [L2] 개발 Report | RSA Team L2 표준 5단 구성(요약/IO정의/개발내용/개발결과/ToDo) 개발 보고서. 개발 결과 지표는 로직/요구사항에 따라 가변. Confluence 업로드용. | `L2_{{YY.MM.DD}}_{{프로젝트명}}.html` |
| `L3_개발_Report.html` | [L3] 개발 Report | RSA Team L3 표준 6단 구성(요약/데이터취득/IO정의/개발내용/개발결과/ToDo) 개발 보고서. 개발 결과 지표는 로직/요구사항에 따라 가변. Confluence 업로드용. | `L3_{{YY.MM.DD}}_{{프로젝트명}}.html` |
| `L3_성능_Report.html` | [L3] 성능 Report | RSA Team L3 SW 성능 Test/검증 보고서. 4단 구성(Test Dataset / Final Performance / Test Conditions / Work Schedule). Confluence 업로드용. | `L3_{{YY.MM.DD}}_{{프로젝트명}}_Test.html` |
| `회의록_Minutes.html` | [General] Meeting Minutes | RSA Team 회의록. 회의록 로그(누적 표) + 개별 회의록(개요/안건/논의/결정사항/액션아이템/다음회의) 구성. Confluence 업로드용. | `Minutes_{{YY.MM.DD}}_{{회의명}}.html` |

(이 카탈로그가 실제 `templates/` 폴더와 어긋나 보일 때 — 예: 사용자가 "방금 추가한
양식이 안 보인다" 고 하면 — **그때만** 아래 명령으로 스캔해 보정한다. 평상시에는
스캔하지 않는다. `${CLAUDE_PLUGIN_ROOT}` 가 플러그인 설치 루트다(자동 설정):

    for f in "${CLAUDE_PLUGIN_ROOT}/skills/RSA_report/templates/"*.html; do echo "### $f"; head -n 5 "$f"; done

`${CLAUDE_PLUGIN_ROOT}` 가 비어 있으면 Glob 도구로
`**/skills/RSA_report/templates/*.html` 를 찾는다.)

### 2. 추가 입력 확인 (skip 가능)
양식 선택 직후, `AskUserQuestion` 으로 "추가로 문서에 반영할 내용/자료가
있는지" 한 번 묻는다. 옵션에 **"없음 (바로 진행)"** 같은 skip 항목을 첫 번째로
넣어, 사용자가 없다고 하면 곧장 다음 단계로 넘어간다.
사용자가 내용(메모, 수치, 로그 경로, 참고 링크 등)을 주면 그 자료를 최우선으로
반영해 placeholder를 채운다.

### 3. 내용 수집
선택한 템플릿 파일을 읽어 `{{...}}` placeholder를 모두 추출한다.
- 대화에서 이미 받은 정보로 채울 수 있는 것은 채운다.
- 비어 있는 핵심 항목(제목 / 날짜 / 프로젝트명 / 작성자 등)은 사용자에게 묻는다.
  한꺼번에 받기 위해 필요한 항목 목록을 정리해 한 번에 질문하는 것을 우선한다.
- 날짜류 placeholder(`{{YY.MM.DD}}`, `{{날짜}}` 등)는 오늘 날짜로 기본 채움.
- 작성자 기본값은 Liam (tglee@bitsensing.com), 소속 bitsensing / RSA Team.

#### 결과 지표는 고정하지 말고 물어본다
개발/검증 보고서의 "결과" 표 컬럼(지표)은 로직 / 요구사항에 따라 매번 다르다
(Alignment 는 GT/Mean/Std/|Mean-GT|/KPI, Detection 은 Precision/Recall/F1 등).
양식에 예시 컬럼이 들어 있더라도 그대로 쓰지 말고, 작성 전에
**"이번 보고서의 결과 지표(표 컬럼)를 무엇으로 할지"** 를 사용자에게 한 번 묻는다.
- 사용자가 지표를 주면 그 지표로 컬럼 헤더를 구성하고 행을 채운다.
- 사용자가 모르거나 "예시대로"라고 하면 양식의 예시 컬럼을 유지한다.
- 양식 내 `결과 지표 정의` callout 의 예시 텍스트는 실제 지표로 교체하거나 삭제한다.

### 4. 작성
placeholder를 실제 값으로 치환해 완성한다.
- 필요 없는 섹션 / 표 행은 통째로 삭제한다.
- 반복되는 항목(세션별 상세 등)은 해당 행/블록을 복제해 채운다.
- 강조는 `<strong>` 또는 `.highlight`, 경로/파일명/코드는 `<code>`,
  결론/주의/긍정 박스는 `.callout` / `.callout.warn` / `.callout.ok` 사용.
- 본문 어조는 간결한 명사형 종결("~함", "~확인", "~완료") 또는 정중한 평서문 중
  하나로 일관되게. 수치는 단위 명시(deg, m, m/s, scan 등).

### 5. 저장
**`/report`를 호출한 현재 작업 폴더**에 저장한다.
파일명은 템플릿 주석의 `naming` 규칙을 따른다
(예: `L3_{{YY.MM.DD}}_{{프로젝트명}}.html`).
`naming`이 없으면 `{양식 name}_{YYYY.MM.DD}.html` 형태로 한다.
저장한 절대경로를 사용자에게 알린다.

## 양식 추가 방법 (사용자 안내용)
양식 1개를 추가/수정/삭제할 때는 아래 3가지를 **한 commit 에서 같이** 처리한다.
(선택 메뉴는 위 `양식 카탈로그` 표만 보므로, 표를 갱신하지 않으면 새 양식이
메뉴에 안 나온다.)

1. `templates/` 폴더에 HTML 파일을 넣는다. 파일 맨 위에 아래 주석을 단다
   (자가 문서화 + 카탈로그가 깨졌을 때의 스캔 보정용):

        <!-- REPORT-TEMPLATE
          name: 표시될 양식 이름
          desc: 한 줄 설명
          naming: 파일명규칙_{{YY.MM.DD}}.html
        -->

   채울 자리는 본문 어디든 `{{항목명}}` 으로 표시한다.

2. **위 `### 1. 양식 선택` 의 `양식 카탈로그` 표에 행을 추가**한다
   (file / name / desc / naming). 이게 메뉴에 바로 반영되는 부분이다.

3. `.claude-plugin/plugin.json` 의 `version` 을 올린다(예: 1.0.0 -> 1.0.1).
   버전이 그대로면 팀원의 `/plugin` 업데이트가 변경을 못 받을 수 있다.

세 가지를 commit & push 하면, 팀원은 `/plugin` 에서 업데이트로 받는다.

## 규칙
- 소스/HTML 주석은 ASCII만 사용(non-ASCII 주석 금지). 본문 한국어는 무방.
- Confluence 업로드가 필요하면 MCP `updateConfluencePage`(contentFormat: html),
  cloudId 는 bitsensing.atlassian.net 사용.

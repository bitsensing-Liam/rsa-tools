# bitsensing RSA Team — Claude Code 플러그인 마켓플레이스

RSA Team 내부에서 사용하는 Claude Code 플러그인 모음입니다.

## 설치 방법 (팀원용)

Claude Code 에서 아래 두 명령만 실행하면 됩니다.

```
/plugin marketplace add bitsensing-Liam/rsa-tools
/plugin install rsa-report@bitsensing-rsa
```

또는 `/plugin` 만 입력해 인터랙티브 UI 에서 마켓플레이스를 추가하고 설치할 수 있습니다.

설치 후 Claude Code 를 재시작하면 `/RSA_report` 또는 "보고서 써줘" 등으로 스킬이 동작합니다.

## 포함 플러그인

| 플러그인 | 설명 |
|---|---|
| `rsa-report` | 표준 보고서 양식(HTML)을 골라 placeholder 를 채워 문서를 생성하는 스킬. L2/L3 개발·성능·외부업무 양식 포함. Confluence 업로드 지원. |

## 양식 추가/수정

각 양식은 `rsa-report/skills/RSA_report/templates/` 폴더의 `.html` 파일입니다.
새 양식을 추가하려면 그 폴더에 HTML 파일을 넣고, 파일 맨 위에 아래 주석을 답니다.

```
<!-- REPORT-TEMPLATE
  name: 표시될 양식 이름
  desc: 한 줄 설명
  naming: 파일명규칙_{{YY.MM.DD}}.html
-->
```

채울 자리는 본문 어디든 `{{항목명}}` 으로 표시합니다.
양식을 추가/수정한 뒤 commit & push 하면, 팀원은 `/plugin` 에서 업데이트로 받을 수 있습니다.

## 구조

```
rsa-tools/                          (마켓플레이스 레포)
├── .claude-plugin/
│   └── marketplace.json            (플러그인 목록)
└── rsa-report/                     (플러그인)
    ├── .claude-plugin/
    │   └── plugin.json             (플러그인 매니페스트)
    └── skills/
        └── RSA_report/
            ├── SKILL.md
            └── templates/*.html
```

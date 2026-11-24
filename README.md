# page-source-check

AI 코딩 에이전트(Claude Code, Codex)용 웹사이트 외부 노출 점검 스킬. URL 하나로 클라이언트에 배포된 자산과 응답 헤더를 수집해 정보 노출·개인정보보호법·보안 설정 결함을 찾고, 코드 레벨 수정과 콘솔 조치 순서를 제시합니다.

```
이 웹사이트 점검해줘 https://example.com
```

## 점검 범위

| 영역 | 항목 |
|---|---|
| Information Disclosure | HTML/CSS/JS 주석 속 작업 메모, 하드코딩된 PII(휴대폰·주민번호·개인 메일), 내부 경로·스테이징 URL, 공개 Source Map 및 `sourcesContent` |
| Secret Exposure | OpenAI·Anthropic·Stripe·AWS·GitHub·Slack 키 패턴, Supabase JWT role 디코딩(`anon` vs `service_role`), `sb_secret_` |
| Transport | HTTPS 적용, HTTP→HTTPS 리다이렉트, Mixed Content |
| Security Headers | HSTS, CSP, X-Content-Type-Options, Referrer-Policy, X-Frame-Options / `frame-ancestors`, 세션 쿠키 `Secure`·`HttpOnly`·`SameSite` |
| Dependencies | 알려진 취약 버전의 jQuery(<3.5), Bootstrap(<3.4.1), AngularJS 1.x, Lodash(<4.17.21) |
| Compliance (KR PIPA) | 연락처 수집 폼의 수집·이용 동의 고지(제15조), 개인정보 처리방침 링크(제30조), 제3자 트래킹 스크립트 목록 |
| Public Files | robots.txt의 비표준 관리 경로 |
| `--own` (자기 소유 사이트 한정) | `.env`·`.git` 등 설정 파일 노출, anon key로 Supabase PostgREST 테이블 조회 가능 여부(RLS), Firebase RTDB 공개 규칙 |

SPA·AI 빌더(Lovable, Bolt, v0) 사이트를 위해 HTML 외에 동일 출처 JS 번들까지 크롤링합니다(페이지 20, 에셋 40).

## 출력

- 심각도(높음/중간/확인/낮음)별 findings 표: 위치(file:line), 마스킹된 증거, 영향
- 조치 주체 구분: 에이전트가 수정(코드) / 사용자가 콘솔에서 조치(키 rotation, RLS, Firebase rules)
- 수정 요청 시: 원본 백업 → src 수정 → 재스캔 0건 확인 → 시각적 회귀 확인. 배포는 하지 않음

## 설치

```
이 저장소의 skills/page-source-check 폴더를 ~/.claude/skills/ 에 설치해 줘: https://github.com/joobz-web/page-source-check
```
Codex는 `~/.codex/skills/`(또는 프로젝트 `.agents/skills/`)에 같은 폴더를 둡니다. 의존성: Python 3 표준 라이브러리.

직접 실행: `python3 skills/page-source-check/scripts/scan.py <url|dir> [--own]`

## 한계

인증 이후 화면, 서버 사이드 로직(인가 우회 등), 비-Supabase/Firebase 백엔드, 남의 사이트 DB 권한은 보지 않습니다. 침투 테스트를 대체하지 않습니다.

## 사용 원칙

타인 사이트는 공개 응답만 수집합니다. `--own`과 수정 기능은 소유했거나 점검 권한을 받은 사이트에만 쓰세요.

---
MIT · 이효율 (Threads @leehyoyoul)

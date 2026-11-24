# page-source-check

AI 코딩 에이전트(Claude Code, Codex)용 웹사이트 외부 노출 점검 스킬. 주소 하나로 브라우저에 배포된 파일과 응답 헤더를 수집해 정보 노출, 개인정보보호법 위반, 보안 설정 결함을 찾고, 코드 수정과 관리 화면 조치 순서를 제시합니다.

```
이 웹사이트 점검해줘 https://example.com
```

## 점검 범위

| 영역 | 항목 |
|---|---|
| 정보 노출 | HTML·CSS·JS 주석 속 작업 메모, 코드에 직접 들어간 개인정보(휴대폰·주민번호·개인 메일), 내부 경로·스테이징 주소, 공개된 소스맵과 그 안의 원본 코드(`sourcesContent`) |
| 비밀키 노출 | OpenAI·Anthropic·Stripe·AWS·GitHub·Slack 키 형식, Supabase 키의 역할 판별(JWT 해독으로 공개용 `anon`과 관리자용 `service_role` 구분), `sb_secret_` |
| 전송 구간 보안 | HTTPS 적용, HTTP→HTTPS 자동 전환, 혼합 콘텐츠 |
| 보안 헤더 | HSTS, CSP, X-Content-Type-Options, Referrer-Policy, 클릭재킹 방지(X-Frame-Options·`frame-ancestors`), 세션 쿠키 속성(`Secure`·`HttpOnly`·`SameSite`) |
| 취약 라이브러리 | 알려진 취약점이 있는 버전의 jQuery(<3.5), Bootstrap(<3.4.1), AngularJS 1.x, Lodash(<4.17.21) |
| 개인정보보호법 준수 | 연락처 수집 폼의 수집·이용 동의 고지(제15조), 개인정보 처리방침 게시(제30조), 처리방침에 빠진 외부 추적 서비스(위탁·국외 이전) |
| 공개 파일 | robots.txt에 적힌 관리자 경로 |
| `--own` (자기 소유 사이트 한정) | `.env`·`.git` 등 설정 파일 노출, 공개 키로 Supabase 테이블이 조회되는지(행 수준 보안 RLS), Firebase 실시간 DB 공개 규칙 |

단일 페이지 앱과 AI 빌더(Lovable, Bolt, v0)로 만든 사이트는 내용이 JS에 있어서, HTML 외에 같은 사이트의 JS 묶음 파일까지 수집합니다(페이지 20, 파일 40).

## 출력

- 심각도(높음/중간/확인/낮음)별 점검 결과 표: 위치(파일:줄), 가린 증거, 영향
- 조치 주체 구분: 에이전트가 코드 수정 / 사용자가 관리 화면에서 조치(키 교체, RLS, Firebase 규칙)
- 수정 요청 시: 원본 백업 → src 수정 → 재스캔 0건 확인 → 시각적 회귀 확인. 배포는 하지 않음

## 설치

```
이 저장소의 skills/page-source-check 폴더를 ~/.claude/skills/ 에 설치해 줘: https://github.com/joobz-web/page-source-check
```
Codex는 `~/.codex/skills/`(또는 프로젝트 `.agents/skills/`)에 같은 폴더를 둡니다. 의존성: Python 3 표준 라이브러리.

직접 실행: `python3 skills/page-source-check/scripts/scan.py <url|dir> [--own]`

## 한계

로그인 이후 화면, 서버 쪽 로직(권한 우회 등), Supabase·Firebase 외 백엔드, 남의 사이트 DB 권한은 보지 않습니다. 침투 테스트를 대체하지 않습니다.

## 사용 원칙

타인 사이트는 공개 응답만 수집합니다. `--own`과 수정 기능은 소유했거나 점검 권한을 받은 사이트에만 쓰세요.

---
MIT · 이효율 (Threads @leehyoyoul)

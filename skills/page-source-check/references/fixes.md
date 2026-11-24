# 수정 가이드

점검 결과의 항목별로 어떻게 고치는지 적었다. 스킬이 직접 고치는 것과 사람이 콘솔에서 해야 하는 것을 나눈다.

## A. 에이전트가 코드에서 고치는 것 (요청 시에만)

| 항목 | 조치 |
|---|---|
| 작업 메모 주석 | 원본 코드(src)에서 주석 삭제. 문자열 안 `//`(예: `https://`)는 건드리지 않는다 |
| 테스트 고객 데이터·주석 속 번호 | 삭제 |
| 소스맵 공개 | Vite `build.sourcemap: false` / Next.js `productionBrowserSourceMaps: false` / CRA `GENERATE_SOURCEMAP=false` |
| 화면 코드 속 비밀키 | 코드에서 빼고 서버 환경 변수로. 화면 코드에서 서버 키를 쓰던 호출은 서버 함수(Edge Function 등)로 옮기는 안을 사용자에게 보여 주고 확인받는다 |
| 보안 헤더 없음 | 아래 B의 호스팅별 설정 파일을 만든다 |
| 폼 동의 문구 없음 | 아래 C의 문구와 체크박스를 제출 버튼 위에 넣고, 체크 안 하면 제출 안 되게 한다 |
| 처리방침 링크 없음 | 푸터에 `/privacy` 링크를 넣고, 내용 초안을 만든다(최종은 사용자 확인) |
| 오래된 라이브러리 | 최신 버전으로 올리고 화면이 그대로인지 확인 |
| HTTP 혼합 콘텐츠 | `http://` 주소를 `https://`로 |
| robots.txt의 관리자 주소 | 줄 삭제. 관리자 페이지는 로그인으로 막는다 |

## B. 보안 헤더 (호스팅별)

Netlify `netlify.toml`:
```toml
[[headers]]
  for = "/*"
  [headers.values]
    Strict-Transport-Security = "max-age=31536000; includeSubDomains"
    X-Content-Type-Options = "nosniff"
    Referrer-Policy = "strict-origin-when-cross-origin"
    X-Frame-Options = "DENY"
    Content-Security-Policy = "frame-ancestors 'none'"
```

Vercel `vercel.json`:
```json
{ "headers": [{ "source": "/(.*)", "headers": [
  { "key": "Strict-Transport-Security", "value": "max-age=31536000; includeSubDomains" },
  { "key": "X-Content-Type-Options", "value": "nosniff" },
  { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
  { "key": "X-Frame-Options", "value": "DENY" },
  { "key": "Content-Security-Policy", "value": "frame-ancestors 'none'" }
]}]}
```

Cloudflare Pages·Netlify `_headers`:
```
/*
  Strict-Transport-Security: max-age=31536000; includeSubDomains
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  X-Frame-Options: DENY
```

CSP는 `frame-ancestors`만 먼저 넣는다. `script-src`까지 넣으면 분석 도구·채팅 위젯이 막힐 수 있으니, 쓰는 외부 서비스 목록(점검 결과의 '외부 서비스')을 보고 사용자와 함께 정한다.

## C. 수집·이용 동의 고지 (개인정보보호법 제15조)

```
[필수] 개인정보 수집·이용 동의
- 수집 항목: [이름, 연락처, 문의 내용]
- 이용 목적: [상담 신청 확인과 회신]
- 보유 기간: [상담 종료 후 1년, 이후 바로 파기]
- 동의를 거부할 수 있으며, 거부하면 상담 신청이 어렵습니다.
□ 위 내용에 동의합니다
```
마케팅 문자·메일을 보낼 거면 [선택] 마케팅 수신 동의를 따로 받는다. 최종 문구는 법률 검토를 권한다.

## D. 사이트 운영자가 관리 화면에서 할 일

| 항목 | 관리 화면 | 절차 |
|---|---|---|
| Supabase 표가 로그인 없이 읽힘 | Supabase 대시보드 | Advisors → Security Advisor에서 'RLS Disabled in Public' 확인 → Database → Policies → 그 표의 Enable RLS → 확인 창 Enable RLS. 켜면 정책이 없는 표는 아무도 못 읽고 못 쓴다. 문의 폼이 저장돼야 하면 아래 SQL로 '저장만 허용' 정책을 만든다 |
| Firebase DB가 열려 있음 | Firebase 콘솔 → Realtime Database → 규칙 | `".read": "auth != null"` 이상으로 바꾸고 게시 |
| 비밀키 노출 | 키를 발급한 곳(OpenAI, Supabase 등) | 새 키 발급 → 사이트에 넣기 → 확인 → 옛 키 삭제. 모르는 사용 기록이 있으면 옛 키부터 바로 삭제 |
| HTTPS 강제 이동 | 호스팅 설정 | Netlify: Domain management → HTTPS / Vercel: 기본 적용 / 카페24·아임웹: 보안서버(SSL) 설정 |
| 관리자 비밀번호 기본값 | 관리자 페이지 | 바로 변경, 2단계 인증 켜기 |

문의 폼용 Supabase 정책(저장만 허용, 읽기는 막음):
```sql
alter table public.<표 이름> enable row level security;
create policy "누구나 문의 저장만" on public.<표 이름>
  for insert to anon with check (true);
```
읽기는 대시보드나 서버(서비스 키)로만 한다.

## E. 검증

1. 빌드 결과 폴더를 `scan.py <폴더>`로 다시 점검해 0개인지 본다.
2. 배포 뒤 `scan.py <주소> --own`으로 다시 점검한다.
3. 고치기 전후 화면을 스크린샷으로 비교한다.

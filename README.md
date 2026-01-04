# SCL (Student Cloud Lab) 웹사이트 보안 분석 결과 보고서

본 보고서는 `https://scl.up.railway.app/` 웹사이트에 대해 수행된 보안 분석 결과

## 분석 개요
- **대상**: [scl.up.railway.app](https://scl.up.railway.app/)
- **기술 스택**: Next.js, Tailwind CSS, Railway Hosting
- **분석 범위**: 공개적으로 접근 가능한 페이지, 응답 헤더, 클라이언트 측 스크립트 및 API 엔드포인트

## 주요 점검 결과 및 권고 사항

### 1. 보안 헤더 미설정 (Security Headers Missing)
분석 결과, 웹 서버의 응답 헤더에서 필수적인 보안 헤더들이 누락된 것을 확인

| 헤더 이름 | 상태 | 위험 요소 |
| :--- | :--- | :--- |
| `Content-Security-Policy` (CSP) | **누락** | 크로스 사이트 스크립팅(XSS) 공격에 취약할 수 있음 |
| `Strict-Transport-Security` (HSTS) | **누락** | HTTPS 강제 접속이 보장되지 않아 중간자 공격(MITM) 위험이 있음 |
| `X-Frame-Options` | **누락** | 클릭재킹(Clickjacking) 공격을 통해 사용자 조작을 유도 가능 |
| `X-Content-Type-Options` | **누락** | MIME 스니핑을 통한 악성 스크립트 실행 위험이 있음 |

> [!IMPORTANT]
> **권고:** `next.config.js` 또는 서버 설정을 통해 해당 보안 헤더들을 추가하여 웹 브라우저 수준에서의 보안을 강화해야됨.

### 2. 관리자 페이지 리다이렉션 설정 (Redirection Behavior)
관리자 전용 페이지(`/dashboard/admin/*`) 접근 시 권한이 없는 경우, 일반적으로 로그인 페이지로 리다이렉트되어야 하나, 현재는 일반 사용자 대시보드(`/dashboard`)로 리다이렉트되고 있음

- **관찰 결과**: 미인증 상태에서 `/dashboard/admin` 접속 시 `/dashboard`로 이동함
- **잠재적 위험**: 공격자가 리다이렉션 과정에서 찰나의 순간에 노출될 수 있는 관리자 화면의 일부 정보를 가로채거나(Information Leakage), 설정 오류로 인해 접근 통제가 우회될 가능성이 높음

> [!TIP]
> **권고:** 미인증 또는 권한 미달 사용자는 `/login` 페이지로 명확히 리다이렉트하고, 서버 측에서도 해당 경로에 대한 엄격한 권한 검증을 수행해야됨

### 3. 정보 노출 (Information Disclosure)
- `X-Powered-By: Next.js` 헤더가 노출, 이는 공격자에게 사용 중인 기술 스택에 관한 정보를 제공하여 특정 프레임워크 버전의 취약점을 노리는 기초 정보가 될 수 있음

> [!NOTE]
> **권고:** `next.config.js`에서 `poweredByHeader: false` 설정을 통해 해당 헤더를 제거하는 것이 좋을 거 같음

## 결론
`scl.up.railway.app`은 최신 프레임워크인 Next.js를 사용하여 구축되었으나, 기본적인 서버 측 보안 헤더 설정 및 접근 통제의 정교함이 부족한 상태암
특히 보안 헤더 설정은 가장 쉽고 빠르게 적용할 수 있는 강력한 방어 수단이므로 최우선적으로 보완할 것을 권장

# Claude Code 개발 환경 세팅 가이드

새 PC에서 Claude Code 서브에이전트 + MCP 서버를 동일하게 구성하는 가이드입니다.

---

## 사전 준비 (필수 설치)

| 도구 | 설치 명령 | 확인 |
|------|-----------|------|
| Node.js 18+ | https://nodejs.org 에서 LTS 설치 | `node -v` |
| Git | https://git-scm.com 에서 설치 | `git --version` |
| Claude Code | `npm install -g @anthropic-ai/claude-code` | `claude --version` |
| GitHub CLI | https://cli.github.com 에서 설치 | `gh --version` |

---

## 1단계 — GitHub 로그인

```powershell
gh auth login
```

- **GitHub.com** 선택
- **HTTPS** 선택
- **Login with a web browser** 선택 → 브라우저 인증 완료

---

## 2단계 — 서브에이전트 설치 (VoltAgent 100개+ 컬렉션)

```powershell
# agents 폴더 생성
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\agents"

# VoltAgent 클론
git clone https://github.com/VoltAgent/awesome-claude-code-subagents.git "$env:TEMP\volt-agents"

# 에이전트 파일 복사
Copy-Item "$env:TEMP\volt-agents\agents\*" "$env:USERPROFILE\.claude\agents\" -Recurse -Force

# 확인
Get-ChildItem "$env:USERPROFILE\.claude\agents\" | Measure-Object | Select-Object -ExpandProperty Count
```

> 설치 후 Claude Code 재시작 시 자동으로 에이전트가 인식됩니다.

---

## 3단계 — MCP 서버 등록

### Playwright (브라우저 자동화 / E2E 테스트)

```powershell
claude mcp add playwright npx -- -y @playwright/mcp@latest
```

### GitHub (PR·이슈·코드리뷰 자동화)

```powershell
claude mcp add github npx -- -y @modelcontextprotocol/server-github
```

> **GitHub MCP 토큰 설정** (선택 — API 제한 해제용)
>
> 1. https://github.com/settings/tokens 에서 `repo`, `read:org` 권한으로 토큰 생성
> 2. 아래 명령으로 재등록:
>
> ```powershell
> claude mcp remove github
> claude mcp add github -e GITHUB_PERSONAL_ACCESS_TOKEN=ghp_여기에토큰 npx -- -y @modelcontextprotocol/server-github
> ```

### Figma (디자인 → 코드 변환)

```powershell
# Figma 토큰: https://www.figma.com/developers/api#access-tokens 에서 발급
claude mcp add figma npx -- -y figma-mcp --figma-token=YOUR_FIGMA_TOKEN
```

### Supabase (DB / Auth 관리)

```powershell
claude mcp add supabase npx -- -y @supabase/mcp-server-supabase --project-url=YOUR_SUPABASE_URL --anon-key=YOUR_ANON_KEY
```

---

## 4단계 — 등록 확인

```powershell
claude mcp list
```

정상 출력 예시:
```
playwright: npx -y @playwright/mcp@latest         - ✓ Connected
github: npx -y @modelcontextprotocol/server-github - ✓ Connected
```

---

## 5단계 — Claude Code 전역 설정 (선택)

`C:\Users\[사용자명]\.claude\settings.json` 파일을 아래 내용으로 생성:

```json
{
  "language": "Korean",
  "theme": "dark",
  "model": "sonnet",
  "effortLevel": "medium",
  "statusLine": {
    "type": "command",
    "command": "npx -y ccstatusline@latest",
    "padding": 0,
    "refreshInterval": 10
  }
}
```

---

## 한 번에 실행하는 스크립트

위 모든 단계를 한 번에 실행:

```powershell
# 1. agents 폴더 준비
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\agents"

# 2. VoltAgent 서브에이전트 설치
git clone https://github.com/VoltAgent/awesome-claude-code-subagents.git "$env:TEMP\volt-agents"
Copy-Item "$env:TEMP\volt-agents\agents\*" "$env:USERPROFILE\.claude\agents\" -Recurse -Force

# 3. MCP 서버 등록
claude mcp add playwright npx -- -y @playwright/mcp@latest
claude mcp add github npx -- -y @modelcontextprotocol/server-github

# 4. 확인
claude mcp list
Write-Host "✅ 설정 완료" -ForegroundColor Green
```

---

## 참고 레포지토리

| 이름 | URL | 용도 |
|------|-----|------|
| VoltAgent 서브에이전트 | https://github.com/VoltAgent/awesome-claude-code-subagents | 100개+ 에이전트 컬렉션 |
| Playwright MCP | https://github.com/microsoft/playwright-mcp | E2E 테스트, 브라우저 자동화 |
| GitHub MCP | https://github.com/github/github-mcp-server | PR·이슈·리뷰 자동화 |
| Figma MCP | https://github.com/GLips/Figma-Context-MCP | 디자인→코드 변환 |
| Supabase MCP | https://github.com/supabase-community/supabase-mcp | DB·Auth 관리 |
| lst97 서브에이전트 | https://github.com/lst97/claude-code-sub-agents | Next.js/React 특화 33개 |

# Claude Code 개발 환경 세팅 가이드 (macOS)

새 맥북에서 Claude Code 서브에이전트 + MCP 서버를 동일하게 구성하는 가이드입니다.

---

## 사전 준비 (필수 설치)

| 도구 | 설치 명령 | 확인 |
|------|-----------|------|
| Homebrew | `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"` | `brew --version` |
| Node.js 18+ | `brew install node` | `node -v` |
| Git | `brew install git` | `git --version` |
| Claude Code | `npm install -g @anthropic-ai/claude-code` | `claude --version` |
| GitHub CLI | `brew install gh` | `gh --version` |

---

## 1단계 — GitHub 로그인

```bash
gh auth login
```

- **GitHub.com** 선택
- **HTTPS** 선택
- **Login with a web browser** 선택 → 브라우저 인증 완료

---

## 2단계 — 서브에이전트 설치 (VoltAgent 100개+ 컬렉션)

```bash
# agents 폴더 생성
mkdir -p ~/.claude/agents

# VoltAgent 클론
git clone https://github.com/VoltAgent/awesome-claude-code-subagents.git /tmp/volt-agents

# 에이전트 파일 복사
cp -r /tmp/volt-agents/agents/* ~/.claude/agents/

# 확인 (설치된 에이전트 수)
ls ~/.claude/agents/ | wc -l
```

> 설치 후 Claude Code 재시작 시 자동으로 에이전트가 인식됩니다.

---

## 3단계 — MCP 서버 등록

### Playwright (브라우저 자동화 / E2E 테스트)

```bash
claude mcp add playwright npx -- -y @playwright/mcp@latest
```

### GitHub (PR·이슈·코드리뷰 자동화)

```bash
claude mcp add github npx -- -y @modelcontextprotocol/server-github
```

> **GitHub MCP 토큰 설정** (선택 — API 제한 해제용)
>
> 1. https://github.com/settings/tokens 에서 `repo`, `read:org` 권한으로 토큰 생성
> 2. 아래 명령으로 재등록:
>
> ```bash
> claude mcp remove github
> claude mcp add github -e GITHUB_PERSONAL_ACCESS_TOKEN=ghp_여기에토큰 npx -- -y @modelcontextprotocol/server-github
> ```

### Figma (디자인 → 코드 변환)

```bash
# Figma 토큰: https://www.figma.com/developers/api#access-tokens 에서 발급
claude mcp add figma npx -- -y figma-mcp --figma-token=YOUR_FIGMA_TOKEN
```

### Supabase (DB / Auth 관리)

```bash
claude mcp add supabase npx -- -y @supabase/mcp-server-supabase \
  --project-url=YOUR_SUPABASE_URL \
  --anon-key=YOUR_ANON_KEY
```

---

## 4단계 — 등록 확인

```bash
claude mcp list
```

정상 출력 예시:
```
playwright: npx -y @playwright/mcp@latest          - ✓ Connected
github: npx -y @modelcontextprotocol/server-github  - ✓ Connected
```

---

## 5단계 — Claude Code 전역 설정 (선택)

`~/.claude/settings.json` 파일을 아래 내용으로 생성:

```bash
cat > ~/.claude/settings.json << 'EOF'
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
EOF
```

---

## 한 번에 실행하는 스크립트

위 모든 단계를 한 번에 실행:

```bash
#!/bin/bash
set -e

echo "🚀 Claude Code 개발 환경 세팅 시작..."

# 1. Homebrew 필수 도구 설치
brew install node git gh 2>/dev/null || true

# 2. Claude Code 설치
npm install -g @anthropic-ai/claude-code

# 3. agents 폴더 준비
mkdir -p ~/.claude/agents

# 4. VoltAgent 서브에이전트 설치
git clone https://github.com/VoltAgent/awesome-claude-code-subagents.git /tmp/volt-agents
cp -r /tmp/volt-agents/agents/* ~/.claude/agents/
echo "✅ 서브에이전트 $(ls ~/.claude/agents/ | wc -l | tr -d ' ')개 설치 완료"

# 5. MCP 서버 등록
claude mcp add playwright npx -- -y @playwright/mcp@latest
claude mcp add github npx -- -y @modelcontextprotocol/server-github

# 6. 확인
echo ""
echo "✅ 설치 완료! MCP 서버 상태:"
claude mcp list
```

스크립트 저장 후 실행:

```bash
chmod +x setup.sh && ./setup.sh
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

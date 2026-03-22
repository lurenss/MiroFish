# Using MiroFish with Claude (Anthropic)

MiroFish supports any OpenAI-compatible LLM API. This guide covers three ways to use Claude as the LLM backend.

## Option A: CLIProxyAPI + Claude Max Subscription (No Extra Cost)

Use your existing Claude Pro/Max subscription via a local proxy.

> **Note:** Anthropic's ToS prohibits using subscription OAuth tokens in third-party apps. Use at your own discretion.

### Prerequisites

- Active Claude Pro ($20/mo) or Max ($100/mo) subscription
- macOS or Linux

### Setup

```bash
# 1. Install CLIProxyAPI
brew install cliproxyapi          # macOS
# or: curl -fsSL https://raw.githubusercontent.com/brokechubb/cliproxyapi-installer/refs/heads/master/cliproxyapi-installer | bash  # Linux

# 2. Authenticate with your Claude account
cliproxyapi --claude-login

# 3. Fix config for local use (disable API key requirement)
# Edit /opt/homebrew/etc/cliproxyapi.conf (macOS) or ~/.cli-proxy-api/config.yaml (Linux):
#   host: "127.0.0.1"
#   api-keys: []

# 4. Start the proxy
brew services start cliproxyapi   # macOS
# or: systemctl --user enable --now cliproxyapi.service  # Linux

# 5. Verify it works
curl http://localhost:8317/v1/models
```

### .env Configuration

```env
LLM_API_KEY=not-needed
LLM_BASE_URL=http://localhost:8317/v1
LLM_MODEL_NAME=claude-sonnet-4-6
```

### Available Models

| Model | ID | Best For |
|-------|-----|----------|
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | Best balance of speed/quality |
| Claude Opus 4.6 | `claude-opus-4-6` | Highest quality, slower |
| Claude Haiku 4.5 | `claude-haiku-4-5-20251001` | Cheapest, fastest (good for boost) |
| Claude Sonnet 4.5 | `claude-sonnet-4-5-20250929` | Previous generation |

---

## Option B: Anthropic API (Pay-as-you-go)

Use a separate Anthropic API key with prepaid credits.

```bash
# Get API key from https://console.anthropic.com
```

```env
LLM_API_KEY=sk-ant-api03-your-key-here
LLM_BASE_URL=https://api.anthropic.com/v1/
LLM_MODEL_NAME=claude-sonnet-4-6
```

---

## Option C: OpenRouter (Multi-provider Gateway)

Use OpenRouter for Claude access with automatic failover and competitive pricing.

```bash
# Get API key from https://openrouter.ai
```

```env
LLM_API_KEY=sk-or-v1-your-key-here
LLM_BASE_URL=https://openrouter.ai/api/v1
LLM_MODEL_NAME=anthropic/claude-sonnet-4-6
```

---

## Dual-Model Strategy (Recommended for Cost Savings)

Use a powerful model for analysis and a cheaper model for the mass agent simulation:

```env
# Primary: Claude Sonnet for analysis, reports, ontology
LLM_API_KEY=not-needed
LLM_BASE_URL=http://localhost:8317/v1
LLM_MODEL_NAME=claude-sonnet-4-6

# Boost: Claude Haiku for mass agent simulation (thousands of calls)
LLM_BOOST_API_KEY=not-needed
LLM_BOOST_BASE_URL=http://localhost:8317/v1
LLM_BOOST_MODEL_NAME=claude-haiku-4-5-20251001
```

---

## Cost Considerations

MiroFish simulations are LLM-heavy. A typical simulation with 35 agents × 72 rounds = ~2,500+ API calls.

| Model | Cost per simulation (est.) |
|-------|---------------------------|
| Claude Haiku 4.5 | ~$0.50-2 |
| Claude Sonnet 4.6 | ~$5-20 |
| Claude Opus 4.6 | ~$50-200 |
| CLIProxyAPI (Max plan) | $0 (included in subscription) |

**Recommendation:** Start with < 40 rounds and ~20 agents to gauge consumption.

---

## Zep Cloud Setup

MiroFish requires Zep Cloud for agent memory and knowledge graphs.

1. Sign up at [app.getzep.com](https://app.getzep.com) (free tier available)
2. Create a project
3. Copy the API key
4. Add to `.env`: `ZEP_API_KEY=z_your_key_here`

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `Missing API key` from proxy | Set `api-keys: []` in cliproxyapi config and restart |
| Rate limiting (429 errors) | Switch to Haiku for simulation, reduce rounds/agents |
| Proxy not starting | Check `brew services list` or `systemctl --user status cliproxyapi` |
| JSON parse errors | Some Claude responses may differ from OpenAI format — the code has retry logic built in |

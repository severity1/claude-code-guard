# claude-code-guard Technical Specification

## Architecture

```
User Prompt --> Claude Code --> Hooks --> claude-code-guard --> Decision
                                              |
                                              v
                                         [Pattern DB]
                                              |
                                              v
                                         [ML Analysis] (optional)
                                              |
                                              v
                                         [Response Handler]
```

## Component Design

| Component | Language | Responsibility |
|-----------|----------|----------------|
| CLI wrapper | Bash | Parse args, invoke binary |
| Core binary | Go | Hook handling, pattern matching |
| ML analyzer | Go | claude-code-sdk-go integration |
| Config loader | Go | YAML parsing, validation |
| Pattern engine | Go | Regex/AST threat detection |
| Response handler | Go | Block/confirm/scrub/remediate |

## Hook Coverage

| Hook | Trigger | Guard Action |
|------|---------|--------------|
| UserPromptSubmit | Before prompt sent | Scan for injection patterns |
| PreToolUse | Before tool execution | Validate tool args, check allowlists |
| PostToolUse | After tool completes | Scan output for leaked secrets |
| SubagentStop | Agent completion | Audit trail, session summary |

## Hook Configuration

```yaml
# .guard/config.yaml
version: "1.0"
mode: confirm  # block | confirm | log
ml_provider: ollama  # ollama | anthropic
ml_model: llama3.2

allowlist:
  domains:
    - github.com
    - api.anthropic.com
  paths:
    - /tmp
    - ${PROJECT_ROOT}

threats:
  network_exfil: confirm
  credential_access: block
  shell_injection: block
  file_attacks: confirm
  prompt_injection: confirm
  privilege_escalation: block
  persistence: block
  reconnaissance: log
```

## Claude Code Integration

### hooks.json

```json
{
  "hooks": [
    {
      "event": "UserPromptSubmit",
      "command": "claude-code-guard hook prompt"
    },
    {
      "event": "PreToolUse",
      "command": "claude-code-guard hook pre-tool"
    },
    {
      "event": "PostToolUse",
      "command": "claude-code-guard hook post-tool"
    },
    {
      "event": "SubagentStop",
      "command": "claude-code-guard hook subagent-stop"
    }
  ]
}
```

## Project Structure

```
claude-code-guard/
├── cmd/
│   └── claude-code-guard/
│       └── main.go
├── internal/
│   ├── config/
│   │   └── config.go
│   ├── hooks/
│   │   ├── prompt.go
│   │   ├── pretool.go
│   │   ├── posttool.go
│   │   └── subagent.go
│   ├── patterns/
│   │   ├── patterns.go
│   │   └── patterns.yaml
│   ├── ml/
│   │   ├── analyzer.go
│   │   ├── ollama.go
│   │   └── anthropic.go
│   └── response/
│       └── handler.go
├── scripts/
│   └── claude-code-guard.sh
├── spec/
│   ├── PRD.md
│   └── SPEC.md
├── go.mod
├── go.sum
└── README.md
```

## CLI Interface

```
claude-code-guard <command> [options]

Commands:
  init          Initialize guard in project
  status        Show configuration status
  disable       Disable hooks temporarily
  enable        Re-enable hooks
  allowlist     Manage allowlists
  audit         View security events
  hook          Handle hook events (internal)

Global Flags:
  --config      Path to config file
  --verbose     Enable debug output
  --json        Output in JSON format
```

## ML Analysis Flow

1. Hook receives tool call context
2. Pattern engine performs fast regex check
3. If inconclusive, spawn ML analysis:
   - Format context as prompt
   - Call Ollama/Anthropic via SDK
   - Parse threat assessment response
4. Combine pattern + ML scores
5. Apply configured response action

## Build Matrix

| OS | Arch | Binary |
|----|------|--------|
| linux | amd64 | claude-code-guard-linux-amd64 |
| linux | arm64 | claude-code-guard-linux-arm64 |
| darwin | amd64 | claude-code-guard-darwin-amd64 |
| darwin | arm64 | claude-code-guard-darwin-arm64 |
| windows | amd64 | claude-code-guard-windows-amd64.exe |

## Dependencies

| Package | Purpose |
|---------|---------|
| github.com/spf13/cobra | CLI framework |
| github.com/spf13/viper | Config management |
| gopkg.in/yaml.v3 | YAML parsing |
| github.com/anthropics/claude-code-sdk-go | Claude API |
| github.com/ollama/ollama | Ollama client |

## Security Considerations

| Concern | Mitigation |
|---------|------------|
| Config tampering | Validate config checksums |
| Binary replacement | Signed releases |
| Credential logging | Never log sensitive values |
| Prompt leakage | Redact before ML analysis |

## Performance Targets

| Operation | Target |
|-----------|--------|
| Pattern match | < 10ms |
| Config load | < 5ms |
| ML analysis | < 2000ms |
| Hook overhead | < 100ms (no ML) |

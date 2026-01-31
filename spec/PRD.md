# claude-code-guard PRD

## Problem Statement

Claude Code executes arbitrary tool calls (bash, file writes, API requests) based on user prompts. Malicious or accidental prompts can lead to:
- Data exfiltration via network calls
- Credential theft from environment/files
- System compromise through shell injection
- Unauthorized file modifications

Users need a security layer that intercepts and validates Claude Code operations before execution.

## Goals

| Goal | Description |
|------|-------------|
| Prevent exfiltration | Block unauthorized data transfers |
| Protect credentials | Detect and scrub secrets before transmission |
| Validate operations | Confirm dangerous commands with user |
| ML-powered analysis | Use LLM for context-aware threat detection |
| Zero friction setup | Single command activation |

## Non-Goals

- Replacing Claude Code's built-in safety
- Obfuscating source code (open source project)
- Supporting non-Claude Code AI tools
- Real-time network traffic monitoring
- Sandboxing/containerization

## User Stories

| As a... | I want to... | So that... |
|---------|--------------|------------|
| Developer | Block curl to unknown hosts | My env vars stay private |
| Enterprise user | Audit all file operations | I have compliance logs |
| Security researcher | Analyze prompt patterns | I can identify new threats |
| Team lead | Configure allowed domains | Teams have safe defaults |

## Success Criteria

| Metric | Target |
|--------|--------|
| False positive rate | < 5% |
| Latency overhead | < 100ms per hook |
| Setup time | < 1 minute |
| Hook coverage | All 4 Claude Code hooks |

## Threat Categories

| ID | Category | Examples |
|----|----------|----------|
| T1 | Network exfiltration | curl, wget, nc to external hosts |
| T2 | Credential access | Reading .env, AWS credentials, SSH keys |
| T3 | Shell injection | Command substitution, eval, pipes to sh |
| T4 | File system attacks | Writing to sensitive paths, symlink abuse |
| T5 | Prompt injection | Hidden instructions in file content |
| T6 | Privilege escalation | sudo, chmod 777, setuid |
| T7 | Persistence | Cron jobs, shell rc modifications |
| T8 | Reconnaissance | System enumeration, user discovery |

## Response Actions

| Action | Behavior |
|--------|----------|
| block | Reject operation, return error to Claude |
| confirm | Pause and prompt user for approval |
| scrub | Remove sensitive data, allow operation |
| remediate | Modify command to safe alternative |
| log | Allow but record for audit |

## Activation Model

- Opt-in via `/guard:init` slash command
- Creates `.guard/config.yaml` in project root
- Installs hooks in Claude Code settings
- Default: confirm mode for all threats

## Slash Commands

| Command | Description |
|---------|-------------|
| /guard:init | Initialize guard in current project |
| /guard:status | Show current configuration |
| /guard:disable | Temporarily disable all hooks |
| /guard:enable | Re-enable hooks |
| /guard:allowlist | Manage allowed domains/paths |
| /guard:audit | View recent security events |

## Future Roadmap

| Version | Features |
|---------|----------|
| v1.1 | Team policy sync, remote allowlists |
| v1.2 | Web dashboard for audit logs |
| v1.3 | Custom rule DSL |
| v2.0 | Multi-agent coordination protection |

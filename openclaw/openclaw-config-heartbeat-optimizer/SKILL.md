---
name: openclaw-config-heartbeat-optimizer
description: Optimize OpenClaw configuration for token efficiency, cost reduction, and best practices. Use when user asks about config optimization, token usage, heartbeat settings, or wants to reduce API costs.
---

# OpenClaw Config Heartbeat Optimizer

This skill helps optimize OpenClaw configuration for token efficiency and cost reduction.

## When to Use

- User asks about token usage or costs
- User wants to optimize heartbeat settings
- User asks about config best practices
- Reviewing configuration for efficiency improvements
- Setting up new OpenClaw instances

## Core Optimizations

### 1. Heartbeat Token Optimization

Heartbeat runs can burn significant tokens if not configured properly.

**Default (expensive):**
- Full context: ~100K tokens/run
- 24 runs/day = ~2.4M tokens/day

**Optimized (95%+ savings):**
```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "1h",              // Run every hour (not 30m)
        timeoutSeconds: 120,      // 2 min timeout
        lightContext: true,       // Keep only HEARTBEAT.md from bootstrap
        isolatedSession: true,    // Fresh session, no history
      }
    }
  }
}
```

**Result:** ~2-5K tokens/run = ~50-120K tokens/day (95%+ reduction)

### 2. Service Warning Suppression

If using nvm or version managers intentionally, suppress doctor warnings:

```json5
{
  gateway: {
    doctor: {
      suppressServiceNodeWarnings: true
    }
  }
}
```

### 3. Model Selection

Use appropriate models for different tasks:
- Primary: Efficient model for日常 tasks
- Consider cheaper alternatives for heartbeat/model that doesn't need full reasoning

## Commands

```bash
# Check current heartbeat config
openclaw config get agents.defaults.heartbeat

# Apply optimizations
openclaw config set agents.defaults.heartbeat.every "1h"
openclaw config set agents.defaults.heartbeat.timeoutSeconds 120
openclaw config set agents.defaults.heartbeat.lightContext true
openclaw config set agents.defaults.heartbeat.isolatedSession true

# Restart to apply
openclaw gateway restart

# Verify
openclaw config get agents.defaults.heartbeat
openclaw gateway status
```

## Token Impact Calculator

| Setting | Before | After | Savings |
|---------|--------|-------|---------|
| Interval | 30m | 1h | 50% fewer runs |
| Context | Full | lightContext | ~95% fewer tokens/run |
| Session | History | isolated | ~95% fewer tokens/run |
| **Total/day** | ~2.4M | ~50-100K | **~95%+** |

## Related Config Paths

- `agents.defaults.heartbeat.every` - Interval (0m to disable)
- `agents.defaults.heartbeat.timeoutSeconds` - Max turn time
- `agents.defaults.heartbeat.lightContext` - Minimal bootstrap context
- `agents.defaults.heartbeat.isolatedSession` - Fresh session per run
- `agents.defaults.heartbeat.model` - Override model for heartbeat

## Safety Notes

- Always backup config before changes (auto-done by `openclaw config set`)
- Test changes in non-critical environments first
- Monitor first few heartbeat runs after optimization
- `lightContext: true` means heartbeat won't see full workspace context
- `isolatedSession: true` means no conversation history between heartbeat runs

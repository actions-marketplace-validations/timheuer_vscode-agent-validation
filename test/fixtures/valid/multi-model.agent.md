---
description: Agent with multiple model fallbacks for availability
model:
  - claude-sonnet-4
  - gpt-4
  - claude-3.5-sonnet
tools:
  - read_file
  - grep_search
---

# Multi-Model Agent

This agent specifies multiple models in priority order.

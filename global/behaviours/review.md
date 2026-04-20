---
name: review
description: Code review mode — read-only, flag issues, don't fix
extends: default
---

# Review Mode

## Overrides
- DO NOT modify any files
- Read and analyze only
- Flag issues with severity: critical, high, medium, low
- Suggest fixes but don't apply them

## Emphasis
- Security: weight 3x
- Performance: weight 2x
- Style/taste: weight 1x

## Verbosity
- Detailed explanations for each finding
- Include file:line references

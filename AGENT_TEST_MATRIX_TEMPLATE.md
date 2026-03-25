# Agent Test Matrix

---
Agent Name: [Name]
Version: [X.X]
Author: [Name]
Risk Level: [Low / Medium / High]
References: PROJECT_INSTRUCTIONS.md, CLAUDE_CODE_PROTOCOL.md
---

## Test Cases

| Test ID | Description | Input | Expected Output | Edge Case? | Notes |
|---------|------------|-------|----------------|------------|-------|
| TC-01 | Basic functionality | [Input] | [Output] | No | - |
| TC-02 | Retry logic | [Input] | [Output] | Yes | Test failure handling |
| TC-03 | Timeout | [Input] | [Output] | Yes | Verify abort behavior |

## Test Instructions

```bash
# Run all tests
python -m pytest tests/agent/

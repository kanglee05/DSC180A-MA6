---
name: seed-fault
description: Introduce one specified pipeline fault and capture the verification failure.
---

Seed this fault: $ARGUMENTS

1. Inspect the current diff and identify the smallest source edit that introduces the requested fault.
2. Make the edit with the Edit tool so the project's PostToolUse hook runs. Leave the tests unchanged.
3. Stop after the edit and show the hook message verbatim. If no hook message appeared, say so.
4. When asked to continue, show the source diff and run the specific test that should detect this fault. Preserve the actual failure output.
5. Wait for me to request the repair. Then reverse only the seeded edit and rerun the same test.

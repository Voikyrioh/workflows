# Known Issues & Bugs

## Bugs

- [01M5B3BHSRCSEPPADJJ3NV2BR0](01M5B3BHSRCSEPPADJJ3NV2BR0.md) — npm 401 : token passé seulement en build-arg, Dockerfile en secret buildx — 2026-10-20

## Reporting & Tracking

When a bug is discovered:

1. Create an issue on GitHub with workflow name and reproduction steps.
2. Document it here with reference: `FIX:ULID` (ticket ID).

### Bug Report Template

```markdown
## FIX:ULID-xxx — [Short description]

**Severity**: Critical | High | Medium | Low  
**Workflow**: ci-release.yml | deploy-app.yml | discord-notify.yml  
**Trigger**: [How to reproduce]

### Root Cause
[If investigated]

### Impact
[Which apps/repos affected]

### Fix
[Link to PR or commit]
```


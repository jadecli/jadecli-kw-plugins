# Sprint M9: Mobile — "We work on mobile"

**Branch**: `sprint/M9-mobile`
**Tag on merge**: `v0.9.0`
**Depends on**: M5 (v0.5.0), M8 (v0.8.0)
**Brand**: jadecli works end-to-end on iOS

## Task Queue

All tasks are `cowork:` — this is the mobile validation sprint. Every task must be performed from a phone or tablet.

```sql
INSERT INTO task_queue (sprint, task_id, task_type, title, depends_on, status)
VALUES
  ('M9', 'CW-M9-01', 'cowork',   'Cowork app invokes KW skill',           NULL,       'queued'),
  ('M9', 'CW-M9-02', 'cowork',   'Slack @Claude creates session',         NULL,       'queued'),
  ('M9', 'CW-M9-03', 'cowork',   'Dispatch sends task → Desktop runs',    NULL,       'queued'),
  ('M9', 'CW-M9-04', 'cowork',   'Remote Control drives local session',   NULL,       'queued'),
  ('M9', 'CW-M9-05', 'cowork',   'Permission relay from mobile',          'CW-M9-04', 'queued'),
  ('M9', 'CW-M9-06', 'cowork',   'Notification reaches phone',            'CW-M9-03', 'queued'),
  ('M9', 'CW-M9-07', 'cowork',   'Session history syncs to mobile',       'CW-M9-02', 'queued'),
  ('M9', 'RS-M9-08', 'research', 'Document mobile-specific limitations',  NULL,       'queued'),
  ('M9', 'CW-M9-09', 'cowork',   'Telegram channel reply from mobile',    NULL,       'queued'),
  ('M9', 'CD-M9-10', 'code',     'M9 milestone tests pass',              'ALL',       'queued');
```

## DSPy Signatures

```python
class MobileE2ETest(dspy.Signature):
    """End-to-end mobile validation for a specific integration."""
    integration: str = dspy.InputField(desc="cowork | slack | dispatch | rc | channel")
    device: str = dspy.InputField(desc="iOS | Android")
    task_description: str = dspy.InputField()
    session_created: bool = dspy.OutputField()
    response_received: bool = dspy.OutputField()
    latency_seconds: float = dspy.OutputField()
    screenshot_path: str = dspy.OutputField(default=None)
```

## Tasks

### CW-M9-01: Cowork skill invocation
**Device**: iOS Claude app (Cowork)
**Action**: Open Cowork → invoke `data-engineering:ast-index`
**PASS**: Skill returns valid output visible in Cowork UI

### CW-M9-02: Slack → Code from phone
**Device**: iOS Slack app
**Action**: @Claude in channel with coding task
**PASS**: Claude Code web session created, "View Session" button in Slack thread

### CW-M9-03: Dispatch → Desktop
**Device**: iOS Claude app
**Action**: Send task from mobile → Desktop picks it up
**PASS**: Desktop session spawns, task executes, completion visible on phone

### CW-M9-04: Remote Control
**Device**: iOS browser (claude.ai/code) or Claude app
**Action**: `claude remote-control` on machine → connect from phone → send message
**PASS**: Message sent from phone executes on local machine

### CW-M9-05: Permission relay
**Depends on**: CW-M9-04
**Action**: During Remote Control, Claude hits a permission prompt → approve from phone
**PASS**: Tool execution continues after mobile approval

### CW-M9-06: Notification delivery
**Depends on**: CW-M9-03
**PASS**: iOS notification appears when Dispatch task completes

### CW-M9-07: Session history sync
**Depends on**: CW-M9-02
**PASS**: Session created via Slack visible in Claude app session list

### RS-M9-08: Document mobile limitations
**Type**: `research:` | **Owner**: integration-engineering
**Action**: Document known mobile constraints (network drops, session timeouts, etc.)
**PASS**: `docs/mobile-limitations.md` exists with ≥5 documented limitations

### CW-M9-09: Telegram from mobile
**Device**: iOS Telegram app
**Action**: Send message to jadecli Telegram bot → reply received
**PASS**: Reply appears in Telegram within 60 seconds

### CD-M9-10: M9 tests
**PASS**: `npx vitest run tests/milestones/m9-mobile.test.ts` exits 0 (automated checks + manual test log)

## Sprint Gate

All 10 green → merge → tag `v0.9.0` → M10 unblocks.

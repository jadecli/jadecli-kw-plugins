# Sprint M5: Integrations — "We work everywhere"

**Branch**: `sprint/M5-integrations`
**Tag on merge**: `v0.5.0`
**Depends on**: M2 (v0.2.0)
**Brand**: jadecli works from your phone

## Task Queue

```sql
INSERT INTO task_queue (sprint, task_id, task_type, title, depends_on, status)
VALUES
  ('M5', 'CD-M5-01', 'code',     'Install Telegram channel plugin',       NULL,       'queued'),
  ('M5', 'CD-M5-02', 'code',     'Install Discord channel plugin',        NULL,       'queued'),
  ('M5', 'CD-M5-03', 'code',     'Configure iMessage channel',            NULL,       'queued'),
  ('M5', 'CW-M5-04', 'cowork',   'Pair Telegram bot + test message',      'CD-M5-01', 'queued'),
  ('M5', 'CW-M5-05', 'cowork',   'Pair Discord bot + test message',       'CD-M5-02', 'queued'),
  ('M5', 'CW-M5-06', 'cowork',   'Test iMessage self-text reply',         'CD-M5-03', 'queued'),
  ('M5', 'CW-M5-07', 'cowork',   'Remote Control session from mobile',    NULL,       'queued'),
  ('M5', 'CW-M5-08', 'cowork',   'Dispatch task from Claude mobile app',  NULL,       'queued'),
  ('M5', 'CD-M5-09', 'code',     'Scheduled task fires (/loop 1m test)',  NULL,       'queued'),
  ('M5', 'CD-M5-10', 'code',     'M5 milestone tests pass',              'ALL',       'queued');
```

## DSPy Signatures

```python
class PairChannel(dspy.Signature):
    """Pair a channel plugin with sender allowlist."""
    channel: str = dspy.InputField(desc="telegram | discord | imessage")
    bot_token: str = dspy.InputField(desc="Bot token (Telegram/Discord) or None (iMessage)")
    pairing_code: str = dspy.OutputField(desc="Code returned by bot")
    paired: bool = dspy.OutputField()
    allowlist_policy: str = dspy.OutputField(default="allowlist")

class TestRemoteControl(dspy.Signature):
    """Verify Remote Control session accessible from mobile."""
    session_url: str = dspy.InputField()
    mobile_connected: bool = dspy.OutputField()
    round_trip_ms: float = dspy.OutputField()
```

## Tasks

### CD-M5-01 through CD-M5-03: Install channel plugins (parallel)
**Type**: `code:` | **Owner**: integration-engineering
**PASS (each)**: `claude plugin list | grep -q "channel-name.*enabled"`

### CW-M5-04: Pair Telegram
**Type**: `cowork:` | **Owner**: integration-engineering
**Action**: Send message to bot → get pairing code → `/telegram:access pair <code>` → set allowlist
**PASS**: `/telegram:access` shows allowlisted sender

### CW-M5-05: Pair Discord
**Type**: `cowork:` | Same pattern as Telegram
**PASS**: `/discord:access` shows allowlisted sender

### CW-M5-06: iMessage self-text
**Type**: `cowork:` | **Owner**: integration-engineering
**PASS**: Self-text → Claude reply appears in Messages within 60 seconds

### CW-M5-07: Remote Control from mobile
**Type**: `cowork:` | **Owner**: integration-engineering
**Action**: `claude remote-control` → scan QR → drive from phone
**PASS**: Session appears in claude.ai/code with green status dot

### CW-M5-08: Dispatch from mobile
**Type**: `cowork:` | **Owner**: integration-engineering
**PASS**: Mobile app task → Desktop session spawns and shows in session list

### CD-M5-09: Scheduled task fires
**Type**: `code:` | **Owner**: data-engineering
**PASS**: `/loop 1m echo PASS` → output appears within 2 minutes

### CD-M5-10: M5 tests
**PASS**: `npx vitest run tests/milestones/m5-integrations.test.ts` exits 0

## Sprint Gate

All 10 green → merge → tag `v0.5.0` → M9 unblocks (with M8).

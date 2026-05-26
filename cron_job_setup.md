# Cron Job Setup for Marketing Script (Two-Step Workflow)

Since the workflow has **two sequential steps** with a human review in between, a standard cron job can't handle that automatically. Here are your options.

---

## The Problem

Cron jobs are "fire and forget" — they can't **wait for human approval** between steps. You need a different approach.

---

## Best Solution: Two Separate Cron Jobs + A Flag File

The script generates, a human reviews, then manually "approves" — and the second cron checks for that approval.

### How it works:

**Cron Job 1 — Generate (runs 1st & 15th at 9 AM):**
```
0 9 1,15 * * /usr/bin/python3 /home/usdicom/marketing.py --generate
```

**Cron Job 2 — Send (runs 1st & 15th at a later time, e.g. 2 PM):**
```
0 14 1,15 * * [ -f /home/usdicom/approved.flag ] && /usr/bin/python3 /home/usdicom/marketing.py --send && rm /home/usdicom/approved.flag
```

### Your review workflow becomes:

1. **9 AM** → Script auto-generates
2. **You review it**
3. If approved, run: `touch /home/usdicom/approved.flag`
4. **2 PM** → Cron checks for the flag, sends the email, then deletes the flag
5. If no flag exists, it skips silently

---

## Alternative: Just Keep `--send` Manual

If the review window is unpredictable, only automate the generate step:

```
0 9 1,15 * * /usr/bin/python3 /home/usdicom/marketing.py --generate
```

Then manually run `python3 marketing.py --send` after reviewing — this is the **simplest and safest** approach if timing isn't critical.

---

## Recommendation

| Approach | Best For |
|----------|----------|
| Flag file method | You always review within a predictable window |
| Manual `--send` | Review time varies, safety is priority |

The **flag file method** gives you the most automation while keeping human control.

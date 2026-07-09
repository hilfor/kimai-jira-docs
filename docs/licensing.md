# License

JiraBundle is a paid plugin. Kimai time tracking always runs unlicensed, but the plugin's **own
Jira features — worklog sync, import, and the live issue lookup — only run with a valid license
key.** This page covers where the key comes from, the two ways to set it, and exactly what happens
when a subscription lapses or the key can't be checked.

## Where the key comes from

You receive the license key by email when you purchase the subscription — the same delivery as the
release ZIP. It is a signed token (`v1.<payload>.<signature>`); paste it verbatim, including the
`v1.` prefix.

## Setting the key

There are two ways to provide it, and they have a **deliberate precedence**:

1. **Environment variable `JIRA_LICENSE_KEY`** — for container and automated deploys. **This wins.**
   If it is set and non-empty, the plugin uses it and ignores the system-configuration value.
2. **System configuration** — as an admin, open **System → Settings → Jira** and paste the key into
   the **license key** field. Used whenever `JIRA_LICENSE_KEY` is unset or empty.

!!! note "The environment variable overrides the settings field"
    If `JIRA_LICENSE_KEY` is set, editing the key under **System → Settings → Jira** has no effect —
    the env var is read first and short-circuits the lookup. On a containerised deploy, manage the
    key through the env var and leave the settings field blank to avoid confusion.

## Offline verification

The key is verified **entirely offline**. It carries an Ed25519 signature that the plugin checks
against an embedded public key — no call to any server is needed to *run* the plugin. An
air-gapped install with a valid, unexpired key works with no outbound network at all (subject to
the offline-staleness clock below).

## The grace window: what stops, and when

When a license expires, is revoked, or goes stale (see below), Jira features do **not** cut off
immediately. There is a **14-day grace window**: features keep working, and Kimai surfaces an
escalating banner counting down the days remaining. The banner reads:

> Jira subscription lapsed — sync keeps working for *N* more day(s), then Jira features are
> disabled. Renew and update the license key under System settings.

Once the 14 days pass, the plugin's Jira features switch **off**:

- **Worklog sync** (inline on timesheet save, and the `kimai:jira:sync` reconciler) pauses. Pending
  worklogs are **not lost** — they stay queued and drain once a valid key is active again.
- **Import** (`kimai:jira:import`) is disabled and exits without importing.
- **Live issue lookup** (the advisory issue-key validation on the timesheet form) goes quiet — it
  returns nothing, exactly like an unreachable Jira, and never blocks saving a timesheet.

**Kimai core, timesheet entry, and all existing data are never touched.** This gate only ever
decides whether the plugin's own Jira behaviour runs.

If no key is configured at all (rather than an expired one), the same features are off, and the
banner instead prompts you to paste the key you received on purchase.

## The daily revocation heartbeat rides `kimai:jira:sync`

Verification is offline, but the plugin still needs a way to notice a **revoked** subscription (a
refund or chargeback). It does this with a once-per-day **revocation heartbeat** that runs from the
`kimai:jira:sync` cron. The heartbeat is **fail-open**: only an explicit "revoked/inactive" answer
from the licensing service starts the grace clock. An unreachable endpoint, a non-2xx response, or
unparsable JSON is recorded as "unknown" and never disables a working instance.

!!! warning "Schedule `kimai:jira:sync`, or your paid install degrades on its own"
    The `kimai:jira:sync` cron is the **only** thing that resets the offline-staleness clock (below).
    The plugin's live features — inline worklog sync on timesheet save, issue lookup — work *without*
    the cron, so it is easy to leave it unscheduled. But then a paid, online install still degrades
    its Jira features after roughly **44 days**, computed as **30 days offline-staleness + 14 days
    grace** — because it never checks in. **Schedule the cron** (see [Configure → Cron](configure.md)),
    or set `JIRA_LICENSE_OFFLINE_GRACE_DAYS=0` to disable the offline clock entirely.

## The offline-staleness clock (air-gapped installs)

Fail-open on its own would let a copy that permanently blocks the licensing host evade revocation
forever. To bound that, the plugin runs an **offline-staleness clock**: after **30 days** with no
successful contact with the licensing service, staleness starts the same 14-day grace-then-disable
window. That is where the ~44-day figure comes from — **30 (stale) + 14 (grace)**.

- The clock is anchored to when *this install* first checked in (its first-seen time), **not** to
  when the key was issued — so restoring from a backup with an older key never starts you
  already-lapsed.
- **Any successful daily heartbeat resets it**, so a normally-connected install never trips it.
- The window is set by **`JIRA_LICENSE_OFFLINE_GRACE_DAYS`** (default `30`):
    - **raise it** for intermittently-connected sites, or
    - set it to **`0` to disable the offline clock entirely** for a legitimately air-gapped install.

!!! note "Air-gapped installs"
    On a deliberately offline install, set `JIRA_LICENSE_OFFLINE_GRACE_DAYS=0`. The signed key stays
    authoritative for its full lifetime, and the offline clock never applies. You will not receive
    revocation updates — that is the trade for running fully decoupled from the licensing service.

## Renewal and expiry

- **Renew** by updating the key — paste the new key under **System → Settings → Jira**, or update
  the `JIRA_LICENSE_KEY` env var. On the next `kimai:jira:sync` run the heartbeat re-checks the
  subscription and re-activates features; any queued worklogs then drain.
- A **prolonged outage of the licensing service itself** (~44 days: 30 stale + 14 grace) will
  eventually disable Jira features on otherwise-healthy, online, paid installs. Kimai and your data
  stay untouched, and the plugin **self-heals** on the next successful check. If that trade-off
  matters — air-gapped, deliberately offline, or you want to fully decouple from vendor-side
  availability — set `JIRA_LICENSE_OFFLINE_GRACE_DAYS=0`.

If Jira features stop unexpectedly, see
[Troubleshooting → Jira features stopped working](features/troubleshooting.md#jira-features-stopped-working).

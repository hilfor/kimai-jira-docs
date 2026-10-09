# License

JiraBundle is a paid plugin. Kimai time tracking always runs unlicensed, but the plugin's **own
Jira features — worklog sync, import, and the live issue lookup — only run with a valid license
key.** This page covers how to buy the license, where the key comes from, the two ways to set it,
and exactly what happens when a subscription lapses or the key can't be checked.

## Buy a license

JiraBundle is sold as a **49 €/year subscription**. Purchase it here:
**[Buy JiraBundle — 49 €/year](https://buy.stripe.com/00wbJ32Bhfyo0AQevdefC01)**.

After checkout, the signed license key arrives by email. Paste it in Kimai under
**System → Settings → Jira** — [Setting the key](#setting-the-key) below covers both ways to
provide it and which one wins.

JiraBundle is also listed on the
[Kimai marketplace](https://www.kimai.org/en/store/jira-sync.html); you can buy it there or through
the checkout link above.

## Where the key comes from

You receive the license key by email when you purchase the subscription — the same delivery as the
release ZIP. It is a signed token (`v1.<payload>.<signature>`); paste it verbatim, including the
`v1.` prefix.

A trial start also emails a key, valid until the trial ends. After each renewal charge succeeds, a
new key arrives by email, and the daily license check fetches a renewed key automatically when it
can reach the licensing service (see [Renewal and expiry](#renewal-and-expiry)). A failed renewal
charge sends no key.

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

A renewal key the plugin fetched itself takes precedence over both while the configured key has a
valid signature, even after the configured key expires. A key you paste for the same subscription
with a later expiry wins over the fetched one. See [Renewal and expiry](#renewal-and-expiry).

## Offline verification

The key is verified **entirely offline**. It carries an Ed25519 signature that the plugin checks
against an embedded public key — no call to any server is needed to *run* the plugin. An
air-gapped install with a valid, unexpired key works with no outbound network at all (subject to
the offline-staleness clock below).

## The grace window: what stops, and when

When a license expires, is revoked, or goes stale (see below), Jira features do **not** cut off
immediately. There is a **14-day grace window**: features keep working, and Kimai surfaces an
escalating banner counting down the days remaining. The banner names the cause:

**The key expired.** Its expiry date has passed and no renewed key has arrived yet:

> Your Jira license key expired on *date*. Jira sync keeps working for *N* more day(s). While the
> daily license check (kimai:jira:sync) runs, the renewal key is fetched automatically after
> payment; otherwise paste the key from the renewal email under System settings.

**The subscription is inactive.** The licensing service reports a failed payment or a cancelled
subscription:

> The licensing service reports this Jira subscription as inactive (payment failed or subscription
> cancelled). Jira sync keeps working for *N* more day(s). After the payment succeeds, the next
> daily license check restores it. See System settings.

**The license check is stale.** Kimai has not reached the licensing service within the
[offline-staleness window](#the-offline-staleness-clock-air-gapped-installs):

> Kimai has not reached the Jira licensing service since *date*. Jira sync keeps working for *N*
> more day(s). Check that kimai:jira:sync runs and can reach the licensing host.

Each banner ends with the address of the settings page.

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
unparsable JSON is recorded as "unknown" and never disables a working instance. The same check
fetches renewal keys (see [Renewal and expiry](#renewal-and-expiry)).

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
    The plugin cannot fetch renewal keys either, so paste the renewal key from the email at each
    renewal.

## Renewal and expiry

- **Renewal is automatic** while `kimai:jira:sync` runs and can reach the licensing service. After
  the renewal charge succeeds, the daily license check fetches the new key from the licensing
  service, verifies it (signature, same customer, later expiry), stores it, and uses it from then
  on instead of the configured key. No paste is needed.
- The automatic fetch needs a licensing host that uses `https`, or `JIRA_ALLOW_INSECURE_URL` to be
  set. The default host uses `https`, so this matters only if you override `JIRA_LICENSE_HOST`.
  `JIRA_ALLOW_INSECURE_URL` also disables the Jira server URL check. Over
  plain `http` the daily check still runs as a revocation check, but it does not send the key and
  fetches no renewal key.
- **Paste the key from the renewal email** under **System → Settings → Jira**, or update the
  `JIRA_LICENSE_KEY` env var, when the cron does not run or cannot reach the licensing service
  (air-gapped with `JIRA_LICENSE_OFFLINE_GRACE_DAYS=0`, or behind a firewall), the licensing host
  uses plain `http`, or the old key expired more than 30 days ago. A newer key pasted by hand wins
  over a fetched one. On the next `kimai:jira:sync` run the heartbeat re-checks the subscription and
  re-activates features; any queued worklogs then drain.
- A **failed renewal charge** sends no key. The licensing service reports the subscription as
  inactive and the grace window starts; once the payment succeeds, the next daily check restores
  it.
- Clearing the configured key still unlicenses the install, and a fetched key never replaces a
  configured key whose signature does not verify.
- A **prolonged outage of the licensing service itself** (~44 days: 30 stale + 14 grace) will
  eventually disable Jira features on otherwise-healthy, online, paid installs. Kimai and your data
  stay untouched, and the plugin **self-heals** on the next successful check. If that trade-off
  matters — air-gapped, deliberately offline, or you want to fully decouple from vendor-side
  availability — set `JIRA_LICENSE_OFFLINE_GRACE_DAYS=0`.

If Jira features stop unexpectedly, see [Troubleshooting → Jira features stopped
working](features/troubleshooting.md#jira-features-stopped-working).

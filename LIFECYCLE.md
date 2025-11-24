## Lifecycle statuses

The system assigns each proxy a **lifecycle status** based on protocol checks, uptime, latency, validation results and deeper probe/heavy tests. The most important “healthy” tiers are:

- `active_low`
- `active_medium`
- `active_high`

Anything in `dead_stage_*` or `deactivated` is considered unusable and is not part of the normal export sets.

Below is what the three `active_*` states mean in practice.

---

### `active_high`

> “Premium / best quality” pool.

A proxy in `active_high` is one of the best-performing ones in the system, with very strict requirements:

- **Uptime & latency (from `performance_data`):**
  - Uptime typically **≥ 99.8%**
  - Latency **> 0 ms and < 500 ms**
  - Performance score **≥ 85**
- **Sample quality:**
  - Has a solid history of successful checks (not just one lucky sample).
- **Telemetry gating (must be “good” and fresh):**
  - Latest **probe** result is `ok` and not older than **7 days**
  - Latest **heavy** result is `ok` and not older than **7 days**
- **Validation:**
  - No repeated validation failures (validation_fail_count < 3)
- **Tasks & treatment:**
  - When a proxy becomes `active_high`, the system:
    - Clears out old scheduled tasks
    - Schedules fresh `geo`, `uptime` and (if needed) `capability` checks
    - May add extra “burst” uptime tasks to keep the performance view very fresh
  - `active_high` proxies may be checked more aggressively and are good candidates for premium / paid export lists.

If a proxy fails the probe/heavy gating (missing or stale telemetry), it is **capped down to `active_medium`** until it proves itself again.

---

### `active_medium`

> “Good / recommended default” pool.

`active_medium` proxies are solid and generally reliable, but don’t quite hit the strict bar for `active_high`:

- **Uptime & latency (from `performance_data`):**
  - Uptime typically **≥ 99.0%**
  - Latency **> 0 ms and < 1500 ms**
  - Performance score **≥ 70**
  - At least **3 successful** samples
- **Telemetry gating:**
  - Latest **probe**: `ok` and ≤ **7 days** old
  - Latest **light** check: `ok` and ≤ **7 days** old
  - If probe/light telemetry is missing, stale, or not `ok`, the proxy cannot stay in `active_medium` and will be **pushed down to `active_low`**.
- **Validation:**
  - Repeated validation failures (validation_fail_count ≥ 3) will force the proxy into `active_low`.
- **Tasks & treatment:**
  - On entering `active_medium`, we again reset tasks and enqueue fresh `geo`, `uptime` and (if needed) `capability` checks, with optional extra (burst) uptime checks.
  - `active_medium` is the main pool of “good” proxies that are fine for most use cases.

In short: `active_medium` proxies are **high-uptime**, reasonably fast, and have **recent, successful probe + light checks**, but may be missing the stricter heavy-check guarantees or absolute top-tier performance of `active_high`.

---

### `active_low`

> “Usable but degraded / under observation” pool.

`active_low` is where proxies end up when they are still technically alive and usable, but show weaker or problematic metrics:

- **Performance downgrade conditions (any of these can push to `active_low`):**
  - Uptime **< 90%**, or
  - Latency **> 2000 ms**, or
  - Performance score **< 40**
- **Validation failures:**
  - If a proxy is already in any `active_*` state and accumulates **≥ 3 validation failure(s)**, it is forced into `active_low`.
- **Gating fallout:**
  - If a proxy *could* be `active_medium` based on basic metrics, but does **not** have:
    - recent `ok` **probe** and **light** telemetry, or
    - enough successful samples,
    it will **fall back to `active_low`** instead.
- **Early-life protection:**
  - Newly discovered proxies (`active_new`) are protected from immediate demotion:
    - With too few samples, the system may keep them `active_new` instead of dropping them to `active_low` right away.
- **Tasks & treatment:**
  - `active_low` proxies still get `uptime`, `geo` and other checks, but are considered **lower priority / lower quality**.
  - They’re candidates for either recovery (if metrics improve) or eventual demotion into the `dead_stage_*` / `deactivated` states if they accumulate too many protocol failures.

In short: `active_low` proxies **work**, but they are slow, flaky, or suspicious in some way. They remain in the exports but should not be treated as top quality.

---

### How this ties into the exported files

The exporter writes these lifecycle-based lists:

- `lifecycle/active_low.{txt,json}`
- `lifecycle/active_medium.{txt,json}`
- `lifecycle/active_high.{txt,json}`

Each JSON entry contains:
- The proxy endpoint (`ip`, `port`, `protocol`)
- Current `lifecycle_status`
- Latest protocol & uptime info
- Performance metrics (uptime%, latency, performance score)
- Geo / ASN info
- Anonymity level and first/last seen timestamps

**Important:**  
These lifecycle lists **do not respect** the `EXPORT_UPTIME_WINDOW_MINUTES` cutoff – they show the current lifecycle classification, even if the last uptime check is older than the main “freshness” window used for other exports.

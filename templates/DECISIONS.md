---
status: active
updated: YYYY-MM-DD
---

# DECISIONS — <project name>

<!-- Append-only. Newest first. Never edit past entries except their `status` line.
     Entry target: 3–8 lines. Write the conclusion, the reason, what it replaced. -->

## YYYY-MM-DD — <short decision title>

- **Status:** active
- **Decision:** <what is now true>
- **Because:** <the one or two reasons that actually drove it>
- **Replaces:** <prior approach, or "n/a">
- **Consequence:** <what this commits us to / rules out>

<!-- Example:

## 2026-06-30 — Cloudinary for publish-time media hosting

- **Status:** active
- **Decision:** All publish-time media URLs come from Cloudinary.
- **Because:** PUBLIC_MEDIA_BASE_URL required self-hosted uptime we don't want to own.
- **Replaces:** PUBLIC_MEDIA_BASE_URL static hosting.
- **Consequence:** Cloudinary account is a publish dependency; render step gains an upload stage.
-->

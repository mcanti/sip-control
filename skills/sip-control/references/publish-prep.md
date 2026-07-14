# SIP publish preparation notes

- Publish path requires env-var-only examples.
- Never hardcode the instance `SIP_BASE_URL`; use placeholders in README/SKILL.md.
- Subscription/install pattern: via tap repo → subscribers run `hermes skills install owner/skills/sip-control`.
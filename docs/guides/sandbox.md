# Sandbox vs Live API Keys

LexgoSign sandbox is an **API key stage**, not a separate host or environment.

Use a **sandbox** key while integrating. Switch to a **live** key when you need legally binding signatures for real recipients.

## Same base URL

Sandbox and live keys both call production API hosts:

```
https://api.lexgo.cl/api/v1
```

!!! note "Gateway path"
    Externally, `api.lexgo.cl` strips the `/api` prefix. Call
    `https://api.lexgo.cl/v1/...`.

!!! danger "There is no sandbox-api host"
    `https://sandbox-api.lexgo.cl` does **not** exist. Always use
    `https://api.lexgo.cl` and select sandbox or live via your API key.

Confirm the stage of your key with `POST /api/v1/test` — the response includes
`"environment": "sandbox"` or `"environment": "live"`.

## Sandbox restrictions

| Behavior | Sandbox | Live |
|----------|---------|------|
| Base URL | `api.lexgo.cl` | `api.lexgo.cl` |
| Recipient emails | Only enterprise **team members** | Any valid email |
| Document watermark | `NON-BINDING DOCUMENT` on every page | None |
| Legal validity | Not legally binding | Legally binding |
| `QUALIFIED_CHILE` signature type | Not allowed (use `INTERNATIONAL`) | Allowed |
| Usage / billing reports | Excluded | Included |

### Recipient email allowlist

With a sandbox key, every recipient email must belong to a user on your
enterprise team. Otherwise create/send fails with:

```text
The email of the recipient '…' is not in the sandbox allowed email list: …
```

**How to add an allowed email**

1. Sign in to [app.lexgo.cl](https://app.lexgo.cl)
2. Open **Equipos** in the sidebar (`/teams`)
3. Click **Acciones** → **Crear miembro**
4. Invite or create the user with the email you will use as a sandbox recipient

Only emails listed under your organization members can be used as sandbox
recipients.

## Sandbox vs staging

Do not confuse sandbox keys with Lexgo's staging environment:

| Concept | What it is | Host |
|---------|------------|------|
| **Sandbox key** | Test stage of your production API key | `api.lexgo.cl` |
| **Staging** | Separate Lexgo deployment for Lexgo QA | `api-beta.lexgo.cl` |

Integrators almost always want a **sandbox key** on `api.lexgo.cl`, not staging.

## Recommended test flow

1. Create a **sandbox** API key in Lexgo (**Settings** → **API Keys**)
2. Add test recipients as team members under **Equipos**
3. Create and send an envelope against `https://api.lexgo.cl`
4. Open signing links / complete the flow with those team emails
5. When ready for production, create a **live** key and use real recipient emails

## Related

- [Authentication](../getting-started/authentication.md) — API keys and the `/test` endpoint
- [Quick Start](../getting-started/quick-start.md) — first envelope
- [Envelopes API](../api/envelopes.md) — create and send

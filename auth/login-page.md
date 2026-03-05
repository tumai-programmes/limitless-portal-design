# Login Page — Layout Specification

Design spec for the portal login page after removing the dev-mode bypass.

## Target Layout

The login page uses the `SingleCard` layout — a centered card on a light background with the Limitless Modus brand header.

```
┌──────────────────────────────────────────────────┐
│                                                  │
│              LIMITLESS MODUS                     │
│                                                  │
│              Sign In                             │
│              Sign in to your account             │
│                                                  │
│   ┌──────────────────────────────────────────┐   │
│   │  🪟  Sign in with Microsoft              │   │
│   └──────────────────────────────────────────┘   │
│                                                  │
│   ┌──────────────────────────────────────────┐   │
│   │  G   Sign in with Google (coming soon)   │   │
│   └──────────────────────────────────────────┘   │
│                                                  │
│   Use your organisation's account to sign in.    │
│                                                  │
└──────────────────────────────────────────────────┘

        Limitless Modus — AI-Native Enterprise
               Transformation
```

## Elements

### Brand Header
- **"LIMITLESS MODUS"** — brand name in caps, spaced tracking, brand colour
- Part of the `SingleCard` layout (already exists)

### Title & Subtitle
- **"Sign In"** — h2 heading
- **"Sign in to your account"** — subtitle in muted text
- Both provided by the `SingleCard` layout via route meta `title` and `description`

### Microsoft SSO Button (Primary)
- Full-width button with Microsoft 4-colour logo SVG on the left
- Text: "Sign in with Microsoft"
- Style: white/surface background, border, medium font weight
- On hover: subtle shadow
- On click: `window.location.href = '/auth/login'` (redirect to Azure)

### Google SSO Button (Disabled — Phase 2)
- Full-width button with Google "G" logo on the left
- Text: "Sign in with Google"
- Style: same as Microsoft button but greyed out / reduced opacity
- Cursor: `not-allowed`
- Tooltip or small label: "Coming soon"
- Not clickable until Google OAuth is implemented

### Help Text
- Below the buttons: "Use your organisation's account to sign in."
- Small, muted text

### Footer
- "Limitless Modus — AI-Native Enterprise Transformation"
- Already part of the `SingleCard` layout

## Error States

The login page handles errors via URL query parameter `?error=<code>`:

| Error Code | Message Shown | Cause |
|------------|--------------|-------|
| `sso_failed` | "Sign-in was cancelled or failed. Please try again." | Azure returned an error (user cancelled, consent denied) |
| `token_exchange_failed` | "Authentication failed. Please try again." | Backend couldn't exchange code for tokens |
| `token_parse_failed` | "Authentication failed. Please try again." | Backend couldn't parse the id_token |
| `no_email` | "Your account does not have an email address. Please contact your administrator." | Azure id_token missing email claim |
| `not_registered` | "Your account is not registered for this portal. Please contact your consultant." | Email not found in users or participants table |
| `db_error` | "A system error occurred. Please try again later." | Database lookup failed |

Error messages appear as a red alert box above the SSO buttons.

## Loading State

While the Microsoft redirect is happening (after button click), show a brief loading indicator or disable the button to prevent double-clicks. The redirect is near-instant so a simple `pointer-events: none` on click is sufficient.

## Responsive Behaviour

- The `SingleCard` layout already handles centering and responsive width
- On mobile, the card expands to near-full-width with padding
- SSO buttons should be full-width within the card at all breakpoints

## Removed Elements

The following are removed from the current login page:

- "Development Mode" badge
- Email input field
- "Sign In as Consultant" button
- "Sign In as Participant" button
- Divider ("or")
- Dev-mode conditional rendering

---

*Part of the [Auth Design](README.md) — [Portal Design](../README.md).*

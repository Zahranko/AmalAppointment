# Al Amal — Appointment request (public form)

The anonymous appointment request page (`احجز موعدك`), hosted on its own
subdomain as a single self-contained `index.html` — no build step, no
dependencies. It is built from the patient feedback page
(`D:\AmalFeedback`) and shares its app shell, keyboard handling, zoom guard,
bottom-sheet pickers and country-code list; read that README for why those
work the way they do.

What a patient sends: full name, phone number with country code, the
**department** they want an appointment with, how they heard about the
hospital, and optional free-text details. Both lists come from the API — the
departments are the hospital's shared department list (the same one the
feedback page shows), and the referral sources are maintained by an Admin in
the console under *Settings → Appointments*.

The department is not just a label: it decides whose inbox the request lands
in (`/ap/inbox` in the console) and which managers are emailed about it — the
department's `AManager`s. It replaced an appointment-only "procedure" list and
a hand-maintained list of notification addresses on 2026-09-16.

## Differences from the feedback page

- **Laptop and desktop fill the window.** From 960px wide the page is a
  full-height split screen — the hospital panel on one side, the form on the
  other — instead of a floating card capped at 760px tall. Fields sit in two
  columns (name | phone, department | how you heard) so the whole form fits
  above the submit bar on a 1366×768 laptop without scrolling; a
  `max-height:760px` rule tightens the spacing for exactly that screen.
  Phones and tablets (<960px) keep the feedback page's layouts unchanged.
- **Mouse and keyboard.** Hover states on fields and options; opening a picker
  focuses its search box (or the current option) when the device has a fine
  pointer; ↑/↓ walk the options and Enter picks; Ctrl/⌘+Enter submits from any
  field (it works, but is deliberately not advertised under the button — the
  submit button sits centred and alone on that row). None of this runs on
  touch devices, where focusing a search box would throw the keyboard over
  the list.
- **No reference number.** Nothing is looked up by the patient afterwards; the
  success screen recaps the name, the number that will be called and the
  department.

## The two things that must be set

**1. `API_BASE`** near the top of the `<script>` block — the AlAmalBusiness API
origin (the same value the feedback page uses).

**2. CORS** — this subdomain must be in the API's `Cors:AllowedOrigins`.
`https://appointment.alamalhospitaljo.com` is already listed; if the page is
hosted anywhere else, add that origin instead.

## Endpoints it calls

Both are on `PublicAppointmentController`, rate limited with the feedback form's
`PublicFormLimit` (10 requests/minute per IP).

| | |
|---|---|
| `GET /api/public/appointment/options` | `{ departments: [{id, name}], referralSources: [{id, name}] }`, active entries only |
| `POST /api/public/appointment` | `{ fullName, phoneCountryCode, phoneNumber, departmentId, referralSourceId, details }` → `{ success, error, createdDate }` |

## Deploying

Same as the feedback page: Hostinger Git deployment of this repository into the
subdomain's document root, with `.htaccess` blocking `README.md` and `.git`.

## Local preview

```
npx serve .
```

Without a reachable API the pickers read `تعذّر التحميل — اضغط لإعادة المحاولة`
— that is the expected offline behaviour.

# Ezynomad.com — Project Backup (May 26, 2026)

A snapshot of the codebase covering the brand migration, payment integration,
email pipeline, SEO foundation, careers page, password strength rules, and
the multiple-room-photos / two-column property page features shipped to date.

## How the project is organized on the live server

```
~/Desktop/easynomad-project/         (root, your Mac)
├── index.html                       (the entire frontend, single file ~390 KB)
├── netlify.toml                     (Netlify config: functions dir + redirects)
├── robots.txt                       (search engine crawler hints)
├── netlify-functions/               (serverless backend)
│   ├── invite-user.js               (admin → invite a new partner via Supabase)
│   ├── razorpay-create-order.js     (online payment: create order)
│   ├── razorpay-verify-payment.js   (online payment: verify signature)
│   ├── send-booking-emails.js       (customer + host booking confirmations)
│   ├── send-cancellation-email.js   (cancellation email)
│   ├── send-feedback-email.js       (in-app feedback → support inbox)
│   ├── send-referral-email.js       (invite email to the referee)
│   ├── send-referrer-confirmation.js (thank-you email to the referrer)
│   ├── send-admin-referral-notification.js (NEW: notify support@ of each referral)
│   ├── send-careers-application.js  (NEW: job application emails to join@)
│   ├── send-referral-payout-email.js (payout confirmation email)
│   ├── sitemap.js                   (dynamic /sitemap.xml from the database)
│   └── package.json                 (function dependencies)
└── sql/                             (Supabase migrations 01-31, run in order)
```

To restore from this backup: copy index.html, netlify.toml and robots.txt
into the project root, the .js files into netlify-functions/, and (only if
rebuilding the database) the .sql files run in numeric order on Supabase.

## Infrastructure NOT in this backup, must be reconfigured separately

These run in third-party services and need to be set up at the dashboards:

- Supabase project (database, auth, storage). 2FA on the owner account.
- Resend domain verification for ezynomad.com
- Netlify env vars: RESEND_API_KEY, SUPABASE_URL, SUPABASE_SERVICE_KEY,
  FROM_EMAIL=support@ezynomad.com, SUPPORT_EMAIL=support@ezynomad.com,
  SITE_URL=https://ezynomad.com, RAZORPAY_KEY_ID, RAZORPAY_KEY_SECRET
- Netlify domain config: easynomad.world (primary), ezynomad.com (alias)
- GoDaddy DNS for ezynomad.com (A record, MX, single merged SPF, DKIM, DMARC,
  Google verification, Resend verification)
- Razorpay live keys
- Google OAuth + custom SMTP for Supabase Auth (smtp.resend.com:465,
  user=resend, password=current Resend API key, sender=support@ezynomad.com)
- Google Search Console: easynomad.world verified via HTML meta tag,
  sitemap submitted
- Google Workspace: support@ezynomad.com and join@ezynomad.com mailboxes
- Instagram: @ezynomad

## What was working as of this backup

- Booking flow (guest + logged-in, online + pay-at-property)
- Property listing form with multiple room types and up to 4 photos per room
- Two-column property detail page (wide, sticky booking card)
- Per-property referrals with $5 bounty
- Open referral form (no login required, auto-link on signup)
- Cancellation flow with policy enforcement
- Admin console (properties, bookings, referrals, feedback)
- Email pipeline (Resend, end-to-end, with SPF/DKIM/DMARC correct)
- Strong password rules on signup and reset
- Sitemap submitted to Google
- Careers page at /careers with 3 open roles and resume upload to join@

## Key technical decisions and patterns

- USD is the canonical stored currency in the price_usd column; INR display
  via Razorpay conversion at checkout
- Customer bookings work both logged-in and guest (customer_id nullable)
- Room photos: up to 4 per room in photo_urls (jsonb array); legacy photo_url
  retained as first-photo fallback
- Referrals: referrer_id nullable until they register (linkPendingReferrals
  auto-links on signup by email match)
- Email sender: support@ezynomad.com (Resend verified)
- Online payment via Razorpay; pay-at-property is the always-on fallback
- /sitemap.xml is a function output, cached headers set to no-cache,
  generates 6 static + N active-property URLs
- Lightbox supports keyboard navigation (←/→/Esc), used everywhere photos
  appear

## What was still pending

- ezynomad.com SSL certificate (Let's Encrypt rate-limit, self-resolves
  within 24-48 hours)
- Active customer acquisition / distribution (partner outreach ongoing)
- Stage 2 referral notifications (status-change emails, not just submission)
- Blog infrastructure

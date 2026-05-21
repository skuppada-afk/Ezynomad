# Easynomad.world - Code Backup

**Backup date**: May 13, 2026
**Live site**: https://easynomad.world
**Tech stack**: Single-page HTML/JS frontend on Netlify + Supabase backend

## What's in this backup

```
easynomad_backup_2026-05-13/
├── README.md                           ← you are here
├── frontend/
│   ├── index.html                      ← the entire app (one file, ~2200 lines)
│   └── netlify.toml                    ← Netlify routing config
├── sql/                                ← run in this exact order on a fresh Supabase project
│   ├── 01_initial_schema.sql           ← profiles, properties, rooms, bookings, payouts, site_content
│   ├── 02_photos.sql                   ← photos column + storage bucket + policies
│   ├── 03_reviews.sql                  ← reviews table with RLS
│   └── 04_profile_and_deactivation.sql ← phone field + soft-delete flags
└── netlify-functions/                  ← Stripe serverless functions (NOT deployed, parked until India approval)
    ├── create-payment-intent.js
    ├── stripe-webhook.js
    ├── connect-stripe.js
    └── package.json
```

## Current architecture

**Frontend**: One static HTML file deployed via Netlify Drop. All UI, routing, and business logic in pure JS (no build step, no React).

**Backend**: Supabase Postgres database with row-level security. Auth via Supabase Auth (email/password). Storage via Supabase Storage (property photos in public bucket `property-photos`).

**Payments**: Stripe wired in code but disabled (India invitation pending). Currently demo mode - bookings save to DB but no money moves. When Stripe is approved, deploy the netlify-functions/ folder and replace the placeholder keys in CONFIG.

## Features built (as of backup date)

**Customer (no console, just public site)**:
- Browse properties on homepage with photo cards
- Search by city
- View property detail with photo gallery + lightbox
- Multi-currency display (USD, VND, IDR, INR) with live FX rates
- Sign up / sign in / sign out
- Reserve dates, complete demo checkout
- "My Bookings" page showing all stays
- Leave 1-5 star review with comment after a confirmed booking
- Edit own profile (name, email, phone, password)

**Partner console** (/partner):
- Dashboard with stats (properties, bookings, upcoming, net earnings)
- List new property with multi-photo upload, edit existing properties
- View all bookings across their properties
- Availability tab (placeholder, not yet built)
- Payouts tab showing gross/fee/net (Stripe Connect parked until India approval)

**Admin console** (/admin):
- Platform overview (users, properties, bookings, GMV, platform revenue)
- All Properties tab: edit, suspend, deactivate (soft delete)
- All Bookings tab: full audit
- Users tab: deactivate/reactivate any user (blocks login if deactivated)
- Payouts tab: gross/fee/net by booking
- Site Content tab: edit homepage hero copy

**Reviews system**:
- Only confirmed-booking customers can write a review (one per booking, enforced by RLS)
- Customers can edit their own reviews
- Admins can hide abusive reviews
- Average rating shown on property detail page

## How to restore from this backup

### If you lost everything (worst case)

1. **Supabase**: create new project, run SQL files in order (01, 02, 03, 04). Copy your project URL and anon key.
2. **Code**: open `frontend/index.html` in a code editor. Find the CONFIG block near the bottom. Replace SUPABASE_URL and SUPABASE_ANON_KEY with your new project's values.
3. **Netlify**: drag `frontend/index.html` to a new Netlify site. Connect your domain.
4. **Auth setup**: Supabase > Authentication > URL Configuration. Set Site URL to your domain. Add redirect URLs.
5. **First admin**: sign up on your site, then run in SQL Editor: `update public.profiles set role = 'admin' where email = 'your-email';`

Total recovery time: 30-45 minutes from scratch.

### If you just want to edit a feature

1. Edit `frontend/index.html`
2. Drag to Netlify Deploys drop zone
3. Done in 30 seconds

## Payment gateway status (as of May 13, 2026)

- **Stripe India**: applied, NOT invited. Stripe responded that they're "working on a solution for India" with no timeline. Code ready in `netlify-functions/` but not deployed.
- **Backup options** to evaluate when ready:
  - **Razorpay Route** — most likely fit. Marketplace-native, supports split payments to multiple property owners (like Stripe Connect). UPI/wallets native.
  - **Cashfree Payouts** — alternative, slightly cheaper fees.
  - **PayU** — older, heavier integration. Last resort.
- **Decision rule**: don't pick a gateway in isolation. Pick when a real partner says "how do I get paid?" Their preferred method (UPI, bank transfer) drives the choice.

## Known gaps / unbuilt features

These are intentional. Build only when a real user asks:

1. **Payments**: demo mode only. Bookings save to DB but no money moves. See "Payment gateway status" above.
2. **Email notifications**: no booking confirmation emails sent yet. Will need Resend or Postmark when ready.
3. **Password reset UI**: Supabase sends recovery email but there's no "set new password" screen on the site.
4. **Image compression**: photos upload at full size. Slow on mobile, especially in India.
5. **Availability calendar enforcement**: bookings can technically overlap on the same room.
6. **Cancellation flow**: no UI for customer to cancel a booking.
7. **Refund flow**: would happen manually in Stripe Dashboard once payments are live.
8. **Multi-photo reordering**: you can add/remove photos but can't drag to reorder. First photo is always cover.
9. **Hotel response to reviews**: owners can't reply to reviews yet.

## Supabase config that's NOT in code (set manually in Supabase dashboard)

These are settings on your Supabase project that need to be configured separately when restoring:

1. **Authentication > URL Configuration**:
   - Site URL: `https://easynomad.world`
   - Redirect URLs: `https://easynomad.world`, `https://easynomad.world/**`

2. **Authentication > Providers > Email**:
   - Confirm email: OFF (for testing). Turn ON before public launch.

3. **Storage**: bucket `property-photos` is created via SQL migration #2, no manual config needed.

## Environment variables (for when Stripe is wired)

When you eventually enable Stripe and deploy the netlify-functions, set these in Netlify > Site configuration > Environment variables:

```
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_SERVICE_ROLE_KEY=eyJ...  (Supabase API page > "secret keys")
STRIPE_SECRET_KEY=sk_test_...  (start with test mode)
STRIPE_WEBHOOK_SECRET=whsec_...  (from Stripe Dashboard webhook)
SITE_URL=https://easynomad.world
```

Never put SUPABASE_SERVICE_ROLE_KEY in `index.html`. Server-side only.

## Security notes

1. The Supabase anon key is intentionally embedded in `index.html`. This is safe because of Row-Level Security policies. The anon key alone can't bypass RLS.
2. Service role key has been rotated as of this backup (after a prior exposure incident). Never share it again.
3. All write operations are gated by RLS policies. Check `01_initial_schema.sql` for the current policies.

## File counts

- **index.html**: ~2200 lines (HTML + CSS + JS in one file by design)
- **Total SQL migrations**: 4 files, ~250 lines combined
- **Netlify functions**: 3 functions, ~150 lines combined (not deployed yet)

## When you grow beyond this

If you ever outgrow the single-HTML approach (likely at 100+ daily users), the migration path is:
1. Move to Next.js or similar React framework
2. Re-deploy on Vercel or keep on Netlify (Netlify supports Next.js)
3. Supabase stays as-is (no migration needed, just point new app at same DB)
4. Photos stay as-is (Supabase Storage unchanged)

You can do this without touching production data. Build new app in parallel, switch DNS when ready.

## Contact / handoff notes for future-you or a developer

The app is designed to be hand-off-able to any junior developer. There's no build step, no framework lock-in, no exotic dependencies. Anyone who can read HTML and JavaScript can maintain this. Modifying the app means: edit index.html, save, drag to Netlify, done.

The "boring" stack was intentional. Don't let anyone "modernize" it until you have real users telling you it's slow.

---

Backup created during initial build phase. Live site has 2 test properties, ~5 test users, and 1 test booking as of this date.

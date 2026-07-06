# EPIC Sales Portal

Guidance for working in this repo. Read this before changing anything.

## What this is

A single page sales portal for EPIC hotels. It is one static HTML file with all
markup, CSS, and JavaScript inline. There is no build step and no framework. The
backend is Supabase (auth, a PostgREST data API, and storage for avatars).

Live site: https://portvisory.com
Host: Netlify, which auto deploys on every push to `main`.

## The two file mirror rule

The app ships as two files that MUST stay byte identical:

* `index.html`
* `portal.html`

`portal.html` is a copy of `index.html`. After any edit to `index.html`, copy it
over `portal.html` before committing, and confirm they are identical. A commit
where the two files differ is a bug.

## Writing conventions

* No dashes or tildes anywhere in user facing copy or in this documentation.
  Write "sign in" not the hyphenated form, "one click" not the hyphenated form.
  Underscores are fine in code, table names, and ids.
* Keep JavaScript in the same terse inline style already in the file (short
  helper functions, `var`, no external libraries beyond the Supabase CDN script).
* Line endings are LF. `.gitattributes` enforces this so Windows editors do not
  rewrite every line to CRLF. If you ever see a diff that touches thousands of
  lines, it is a line ending problem, not real change.

## Authentication

Auth is invite based. A person only gets access if their email is listed in the
`admins` or `hotel_members` tables, and row level security gates all data.

Sign in options on the login card:

1. Email and password (`sb.auth.signInWithPassword`). This is the default.
2. Forgot password link (`sb.auth.resetPasswordForEmail`) that emails a reset
   link back to the same page. Existing users who never had a password use this
   to set one for the first time.
3. Email me a sign in link instead (`sb.auth.signInWithOtp` with
   `shouldCreateUser:false`, so unknown emails cannot self provision an account).

The password reset landing is handled inside the same file. When the page loads
from a recovery link (detected via the `PASSWORD_RECOVERY` auth event and a
`type=recovery` check on the URL hash), a reset card appears asking for a new
password (`sb.auth.updateUser`). The `RECOVERY` flag prevents the app from
loading behind the reset card.

## Data model (Supabase)

Tables read by the client:

* `admins` (email) : EPIC admins, full access.
* `hotel_members` (email, hotel_slug, role) : per hotel membership and role.
* `hotels` (slug, name, manager, source, has_agency360, sales_coverage).
* `profiles` (email, display_name, avatar_path, title) : upserted by the user.

Storage bucket `avatars` holds profile photos.

The anon key is embedded in the client. That is expected for Supabase; row level
security is what protects the data, so never rely on the client to hide anything.

## Supabase settings to keep in mind

These live in the Supabase dashboard, not in this repo, but the auth flows depend
on them:

* Email provider must stay enabled.
* Auth URL configuration must allowlist the site URLs, including the
  `portal.html` path, or reset and magic links fall back to the site URL.
* For reliable delivery, configure a custom SMTP provider. The built in email is
  rate limited and often lands in spam.
* The Reset Password email template should be branded and point at the site.

## Feature flags

Near the top of the script, just after the Supabase client is created:

* `SHOW_REVMAX` : controls the RevMax commentaries tab and the revenue view.
  It is `false` for portvisory, because EPIC gets that revenue picture from
  another report. When `false` the RevMax sidebar tab is hidden, the two
  revenue Home cards (Revenue management and Revenue management advisory) are
  filtered out, and any navigation into that view falls back to Weekly sync.
  For the bredysolutions clone, set it to `true` to restore the whole revenue
  section. Nothing is deleted, so this is a one line change.

## Deploy and verify

1. Edit `index.html`, then copy it over `portal.html`.
2. Commit both files (plus any repo files like this one).
3. Push to `main`. Netlify deploys within a minute.
4. Verify on https://portvisory.com and https://portvisory.com/portal.html.

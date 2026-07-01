# Spec: Login and Logout

## Overview
Implement session-based authentication so registered users can sign in and out of Spendly. This step upgrades the existing stub `GET /login` route into a full `GET`/`POST` route that verifies credentials against the `users` table and starts a Flask session, and upgrades the `GET /logout` stub to clear that session. This is the second half of the auth flow started in Step 02 (Registration) and is a prerequisite for any page that needs to know "who is the current user" (e.g. Step 04 Profile, Step 07-09 Expenses).

## Depends on
- Step 01 — Database setup (`users` table, `get_db()`)
- Step 02 — Registration (`create_user()`, users can already be created with hashed passwords)

## Routes
- `GET /login` — render login form — public (already exists as stub, upgrade it)
- `POST /login` — verify email/password, start session, redirect to a logged-in page — public
- `GET /logout` — clear session, redirect to landing page — logged-in (redirects to `/login` if no active session)

## Database changes
No new tables or columns. The existing `users` table (id, name, email, password_hash, created_at) covers all requirements.

A new DB helper must be added to `database/db.py`:
- `get_user_by_email(email)` — returns the user row (including `password_hash`) for the given email, or `None` if no match.

## Templates
- **Modify**: `templates/login.html`
  - Change the form `action` from the hardcoded `/login` string to `url_for('login')`
  - Add a block to display a flashed error message (e.g. "Invalid email or password") — follow the same pattern used in `register.html`, replacing the unused `{% if error %}` block
  - Keep all existing visual design
- **Modify**: `templates/base.html`
  - Nav links must reflect session state: show "Sign in" / "Get started" when logged out (current behavior), show a "Sign out" link (`url_for('logout')`) when a session is active
  - Nav must not hardcode `/login` or `/register` URLs where session logic is added — use `url_for()` for anything new

## Files to change
- `app.py` — upgrade `login()` to handle `GET` and `POST` with session logic; upgrade `logout()` to clear the session and redirect
- `database/db.py` — add `get_user_by_email()` helper
- `templates/login.html` — wire up form action and flash message display
- `templates/base.html` — make nav links session-aware

## Files to create
None.

## New dependencies
No new dependencies. Uses Flask's built-in `session`, `flash`, `redirect`, `url_for` and `werkzeug.security.check_password_hash` (already installed).

## Rules for implementation
- No SQLAlchemy or ORMs
- Parameterised queries only — never use f-strings in SQL
- Verify passwords with `werkzeug.security.check_password_hash` — never compare plaintext
- Use Flask's `session` object to store only the logged-in user's `id` (e.g. `session["user_id"]`) — never store the password or password hash in the session
- Server-side validation must check:
  1. Both email and password fields are non-empty
  2. Email exists in `users`
  3. Password matches the stored hash
- On any validation failure, re-render the form with a flashed error message — do not redirect, and do not reveal whether the email or the password was wrong (generic "Invalid email or password" message)
- On success, store `user_id` in the session and redirect to `url_for('landing')` (no dashboard/profile page exists yet)
- `GET /logout` must clear the session (`session.clear()` or `session.pop("user_id", None)`) and redirect to `url_for('landing')`
- Use `abort(405)` if an unsupported HTTP method reaches `/login`
- All templates extend `base.html`
- Use CSS variables — never hardcode hex values
- Use `url_for()` for every internal link — never hardcode URLs

## Definition of done
- [ ] `GET /login` renders the login form without errors
- [ ] Submitting valid credentials for an existing user logs them in and redirects to the landing page
- [ ] Submitting an unknown email re-renders the form with "Invalid email or password", no session created
- [ ] Submitting a known email with the wrong password re-renders the form with "Invalid email or password", no session created
- [ ] Submitting with an empty email or password re-renders the form with a validation error
- [ ] After a successful login, the navbar shows a "Sign out" link instead of "Sign in" / "Get started"
- [ ] Visiting `/logout` while logged in clears the session and redirects to the landing page, navbar reverts to logged-out state
- [ ] Visiting `/logout` while already logged out does not error — redirects cleanly

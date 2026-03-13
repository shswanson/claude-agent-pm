# Batch 1: Login Form + Protected Content (P1)

| Field        | Value                                            |
|--------------|--------------------------------------------------|
| **Batch**    | 1                                                |
| **Repo**     | `app-theme`                                      |
| **Priority** | P1 — users need accounts before launch           |
| **Status**   | Ready                                            |
| **Files**    | `inc/auth.php` (new), `inc/template-tags.php`, `assets/js/auth.js` (new), `style.css` |

---

## Problem

The site has no user-facing login. Members need to log in to access gated content (subscriber-only articles, event registrations). WordPress has native auth but no frontend UI — only `wp-login.php`, which doesn't match the site design.

## Intent

Logged-out users see a styled login form that matches the site design. After logging in, they can access gated content inline. Failed logins return them to the same page with an error message, not to `wp-login.php`.

## Architecture

Two new files (`auth.php`, `auth.js`) plus minimal additions to existing files. Uses WordPress native sessions — no custom auth system. The login form is a reusable partial that can be placed in templates or called via shortcode.

---

## Task 1: Login/Logout Form Partial

**File:** `inc/auth.php` (new)

Create a login form using the existing `.form-field` BEM pattern from the newsletter signup form.

<new_code file="inc/auth.php">
```php
<?php
/**
 * User authentication: login form, session handling, protected content.
 */

/**
 * Render login/logout form.
 *
 * @param string $redirect URL to redirect after login. Default: current page.
 */
function mysite_login_form( $redirect = '' ) {
    if ( is_user_logged_in() ) {
        $user = wp_get_current_user();
        ?>
        <div class="auth-form auth-form--logged-in">
            <p>Signed in as <?php echo esc_html( $user->display_name ); ?></p>
            <a href="<?php echo esc_url( wp_logout_url( $redirect ?: home_url() ) ); ?>"
               class="auth-form__logout">Sign Out</a>
        </div>
        <?php
        return;
    }

    if ( ! $redirect ) {
        $redirect = ( is_ssl() ? 'https://' : 'http://' ) . $_SERVER['HTTP_HOST'] . $_SERVER['REQUEST_URI'];
    }

    $error = isset( $_GET['login'] ) && $_GET['login'] === 'failed';
    ?>
    <form class="auth-form" method="post" action="<?php echo esc_url( site_url( 'wp-login.php' ) ); ?>">
        <?php if ( $error ) : ?>
            <p class="auth-form__error">Invalid username or password.</p>
        <?php endif; ?>

        <div class="form-field">
            <label class="form-field__label" for="auth-user">Email or Username</label>
            <input class="form-field__input" type="text" name="log" id="auth-user" required>
        </div>

        <div class="form-field">
            <label class="form-field__label" for="auth-pass">Password</label>
            <input class="form-field__input" type="password" name="pwd" id="auth-pass" required>
        </div>

        <input type="hidden" name="redirect_to" value="<?php echo esc_attr( $redirect ); ?>">
        <?php wp_nonce_field( 'mysite_login', '_mysite_nonce' ); ?>

        <button type="submit" class="auth-form__submit btn btn--primary">Sign In</button>

        <p class="auth-form__meta">
            <a href="<?php echo esc_url( wp_lostpassword_url() ); ?>">Forgot password?</a>
        </p>
    </form>
    <?php
}
```
</new_code>

<notes>
- Uses `wp_nonce_field()` for CSRF protection
- Error message escapes output — no XSS via URL params
- Redirect URL uses `esc_attr()` — prevent open redirect injection
- Follows existing BEM naming (`.form-field__label`, `.form-field__input`) from newsletter form
</notes>

---

## Task 2: Failed Login Redirect

**File:** `inc/auth.php` (same file, add below the form function)

WordPress redirects failed logins to `wp-login.php` by default. Redirect back to the referring page with an error flag instead.

<new_code file="inc/auth.php">
```php
/**
 * Redirect failed logins back to the referring page.
 */
add_action( 'wp_login_failed', function( $username ) {
    $referrer = wp_get_referer();
    if ( $referrer && ! strpos( $referrer, 'wp-login.php' ) ) {
        wp_safe_redirect( add_query_arg( 'login', 'failed', $referrer ) );
        exit;
    }
} );
```
</new_code>

---

## Task 3: Protected Content Shortcode

**File:** `inc/auth.php` (same file, add below)

<new_code file="inc/auth.php">
```php
/**
 * [members_only] shortcode — content only visible to logged-in users.
 *
 * Usage: [members_only]This content is for members only.[/members_only]
 */
add_shortcode( 'members_only', function( $atts, $content = '' ) {
    if ( is_user_logged_in() ) {
        return do_shortcode( $content );
    }
    return '<div class="members-gate">'
        . '<p>This content is available to members. <a href="' . esc_url( wp_login_url( get_permalink() ) ) . '">Sign in</a> to view.</p>'
        . '</div>';
} );
```
</new_code>

---

## Task 4: Include auth.php

**File:** `inc/template-tags.php`

Add one line at the top of the file, after the existing includes:

<current_code file="inc/template-tags.php" line="3">
```php
require_once get_template_directory() . '/inc/seo.php';
```
</current_code>

<new_code file="inc/template-tags.php">
```php
require_once get_template_directory() . '/inc/auth.php';
```
</new_code>

---

<do_not_touch>
## What NOT to Touch

- `functions.php` — the include chain goes through `template-tags.php`, not `functions.php` directly
- `wp-login.php` — we're overriding behavior via hooks, not editing core files
- Any existing form styling — the auth form reuses `.form-field` and `.btn` classes already defined
</do_not_touch>

---

<verification>
## Verification Checklist

- [ ] `inc/auth.php` exists with `mysite_login_form()`, failed login redirect, and shortcode
- [ ] `inc/template-tags.php` includes `auth.php`
- [ ] `php -l inc/auth.php` passes (no syntax errors)
- [ ] `php -l inc/template-tags.php` passes
- [ ] Only 2 files changed/created: `inc/auth.php` (new), `inc/template-tags.php` (1 line added)

### Verification commands:

```bash
# Syntax
php -l inc/auth.php
php -l inc/template-tags.php

# Form renders on staging
curl -s 'https://staging.example.com/' | grep -c 'auth-form'

# Nonce present
curl -s 'https://staging.example.com/' | grep -c '_mysite_nonce'

# Shortcode renders gate for logged-out users
curl -s 'https://staging.example.com/test-page-with-shortcode/' | grep -c 'members-gate'
```
</verification>

---

## Commit Message

```
Add frontend login form, failed-login redirect, and members-only shortcode

- mysite_login_form() partial using existing .form-field BEM pattern
- Failed login redirects back to referring page (not wp-login.php)
- [members_only] shortcode for gated content
- CSRF protected via wp_nonce_field, output escaped
```

---

## Risks & Mitigations

- **Open redirect via `redirect_to` param**: Mitigated — WordPress's `wp_safe_redirect()` only allows same-host redirects. The form uses `esc_attr()` on the redirect value.
- **XSS via `?login=failed` param**: Mitigated — we check for exact string match (`=== 'failed'`), not echoing the param value.
- **Brute force login**: Not addressed in this batch. Rate limiting should be handled at the infrastructure level (WAF, fail2ban, or plugin). Log as P2 follow-up.

# Security guardrails (plugin work)

Use this file when making security fixes or when handling any input/output.

## Nonces + permissions

- Nonces help prevent CSRF, not authorization.
- Always pair nonces with capability checks (`current_user_can()` or a more specific capability).

Upstream reference:

- <https://developer.wordpress.org/apis/security/nonces/>

## Authorization: capabilities only, never request data

Decide who may do or see something with a capability check on the current user, and nothing else.

- Use `current_user_can()` with the most specific capability that fits (`edit_products`, `edit_post`, `manage_options`).
- Never base an access decision on data the client sends: the `Referer` header, `wp_get_raw_referer()` (it reads `$_REQUEST['_wp_http_referer']` before the header, so a plain query parameter sets it), other headers, cookies other than the WordPress auth cookie, query arguments or POST fields.
- `is_admin()` is not a permission check. It is true for every `admin-ajax.php` request, including unauthenticated ones, and only says which screen context is loading.
- `wp_doing_ajax()` is not a context check either. WooCommerce defines `DOING_AJAX` for every front-end `?wc-ajax=` request.
- When behavior differs between admin and storefront requests, gate the privileged branch on a capability, so a forged request falls through to the unprivileged behavior.

```php
// Wrong: the visitor controls the referer.
if ( wp_doing_ajax() ) {
    return ! str_starts_with( (string) wp_get_raw_referer(), admin_url() );
}

// Right: only users who can edit products get the privileged view.
if ( wp_doing_ajax() ) {
    return ! current_user_can( 'edit_products' );
}
```

## Sanitization and escaping

Golden rule:

- sanitize/validate on input, escape on output.

Practical rules:

- never process the entire `$_POST` / `$_GET` array; read explicit keys
- use `wp_unslash()` before sanitizing when needed
- use prepared statements for SQL; avoid interpolating user input into queries

## Output escaping contexts

Escape at output, using the function that matches the context:

- HTML text: `esc_html()`
- HTML attribute: `esc_attr()`
- URL: `esc_url()`
- textarea: `esc_textarea()`
- JavaScript strings: `esc_js()`

For JSON data, prefer `wp_json_encode()` and pass data through WordPress script APIs such as `wp_add_inline_script()` or `wp_localize_script()`.

For user-provided or third-party HTML (feeds, API responses, AI output), restrict allowed markup with `wp_kses_post()` or `wp_kses()` before it is saved or output. Do not rely on core's kses filters running at save time: they are off for users with `unfiltered_html`, including when such a user triggers a background job inline.

## AJAX handlers

- For `wp_ajax_*`, verify the nonce and check capabilities.
- For `wp_ajax_nopriv_*`, assume unauthenticated, attacker-controlled traffic.
- Return JSON with `wp_send_json_success()` or `wp_send_json_error()`.
- Do not expose private data from AJAX responses. Filters that change what a visitor sees (prices, stock, hidden fields) must stay in force on AJAX requests unless a capability check says otherwise.

## REST routes

- Every `register_rest_route()` needs a `permission_callback`. Use a capability check there; `__return_true` only for data that is genuinely public, and say so in a comment.
- Validate and sanitize arguments with `args` (`validate_callback`, `sanitize_callback`, `type`, `enum`) rather than inside the handler.
- Return `WP_Error` with a status code for failures; never echo or `die` from a REST handler.
- Public endpoints should be cheap: cache expensive results in a transient so they cannot be used to amplify load.

Upstream reference:

- <https://developer.wordpress.org/rest-api/extending-the-rest-api/adding-custom-endpoints/#permissions-callback>

Common review guidance:

- <https://developer.wordpress.org/plugins/wordpress-org/detailed-plugin-guidelines/>

## The canonical quartet: nonce + capability + sanitize + escape

Every admin form handler needs all four. Example (PSR-4, `src/Admin/FeedbackHandler.php`):

```php
<?php

namespace EightyFourEM\ExamplePlugin\Admin;

class FeedbackHandler {

    public function register(): void {
        add_action( 'admin_post_84em_save_feedback', [ $this, 'handle' ] );
    }

    public function handle(): void {
        // 1. Nonce (CSRF).
        if ( ! isset( $_POST['_wpnonce'] ) || ! wp_verify_nonce( sanitize_key( wp_unslash( $_POST['_wpnonce'] ) ), '84em_save_feedback' ) ) {
            wp_die( 'Security check failed.', '', [ 'response' => 403 ] );
        }

        // 2. Capability (authorization).
        if ( ! current_user_can( 'manage_options' ) ) {
            wp_die( 'Insufficient permissions.', '', [ 'response' => 403 ] );
        }

        // 3. Sanitize input (explicit keys, unslash first, type-appropriate sanitize_*).
        $subject = isset( $_POST['subject'] ) ? sanitize_text_field( wp_unslash( $_POST['subject'] ) ) : '';
        $email   = isset( $_POST['email'] ) ? sanitize_email( wp_unslash( $_POST['email'] ) ) : '';

        update_option( '84em_feedback_subject', $subject );
        update_option( '84em_feedback_email', $email );

        wp_safe_redirect( add_query_arg( 'updated', '1', admin_url( 'options-general.php?page=84em-feedback' ) ) );
        exit;
    }
}
```

The matching form, with the nonce field and output escaping (4. escape on output with the context-appropriate `esc_*`):

```php
<form action="<?php echo esc_url( admin_url( 'admin-post.php' ) ); ?>" method="post">
    <input type="hidden" name="action" value="84em_save_feedback">
    <?php wp_nonce_field( '84em_save_feedback' ); ?>
    <label for="subject"><?php echo esc_html__( 'Subject', 'example-plugin' ); ?></label>
    <input type="text" id="subject" name="subject" value="<?php echo esc_attr( get_option( '84em_feedback_subject', '' ) ); ?>">
    <?php submit_button(); ?>
</form>
```

Context matters for escaping: see "Output escaping contexts" above.

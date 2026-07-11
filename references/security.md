# Security guardrails (plugin work)

Use this file when making security fixes or when handling any input/output.

## Nonces + permissions

- Nonces help prevent CSRF, not authorization.
- Always pair nonces with capability checks (`current_user_can()` or a more specific capability).

Upstream reference:

- https://developer.wordpress.org/apis/security/nonces/

## Sanitization and escaping

Golden rule:

- sanitize/validate on input, escape on output.

Practical rules:

- never process the entire `$_POST` / `$_GET` array; read explicit keys
- use `wp_unslash()` before sanitizing when needed
- use prepared statements for SQL; avoid interpolating user input into queries

Common review guidance:

- https://developer.wordpress.org/plugins/wordpress-org/detailed-plugin-guidelines/

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

Context matters for escaping: `esc_html()` for element content, `esc_attr()` inside attributes, `esc_url()` for URLs, `esc_js()` for inline JS strings, `wp_kses_post()` when limited HTML is allowed.


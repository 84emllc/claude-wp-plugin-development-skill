# Settings API (admin options)

Use this file when adding settings pages or storing user-configurable options.

Core APIs:

- `register_setting()`
- `add_settings_section()`
- `add_settings_field()`

Upstream references:

- Settings API overview: https://developer.wordpress.org/plugins/settings/settings-api/
- Register settings: https://developer.wordpress.org/plugins/settings/registration/
- Add settings fields: https://developer.wordpress.org/plugins/settings/settings-fields/

Practical guardrails:

- Use `sanitize_callback` to validate/sanitize data.
- Use capability checks (commonly `manage_options`) for settings screens and saves.
- Escape values on output (`esc_attr`, `esc_html`, etc.).

## Complete example

One option, one section, one field, wired end to end (PSR-4, `src/Admin/Settings.php`):

```php
<?php

namespace EightyFourEM\ExamplePlugin\Admin;

class Settings {

    public function register(): void {
        add_action( 'admin_menu', [ $this, 'add_page' ] );
        add_action( 'admin_init', [ $this, 'register_settings' ] );
    }

    public function add_page(): void {
        add_options_page( 'Example Settings', 'Example', 'manage_options', '84em-example', [ $this, 'render_page' ] );
    }

    public function register_settings(): void {
        register_setting( '84em_example_group', '84em_example_api_key', [
            'type'              => 'string',
            'sanitize_callback' => [ $this, 'sanitize_api_key' ],
            'default'           => '',
        ] );

        add_settings_section( '84em_example_main', 'API Configuration', '__return_false', '84em-example' );

        add_settings_field( '84em_example_api_key', 'API Key', [ $this, 'render_api_key_field' ], '84em-example', '84em_example_main' );
    }

    public function sanitize_api_key( $value ): string {
        $value = sanitize_text_field( (string) $value );

        if ( '' !== $value && ! preg_match( '/^[A-Za-z0-9_\-]{16,64}$/', $value ) ) {
            add_settings_error( '84em_example_api_key', 'invalid_key', 'API key format is invalid; previous value kept.' );
            return (string) get_option( '84em_example_api_key', '' );
        }

        return $value;
    }

    public function render_api_key_field(): void {
        printf(
            '<input type="text" class="regular-text" id="84em_example_api_key" name="84em_example_api_key" value="%s">',
            esc_attr( get_option( '84em_example_api_key', '' ) )
        );
    }

    public function render_page(): void {
        if ( ! current_user_can( 'manage_options' ) ) {
            return;
        }
        ?>
        <div class="wrap">
            <h1><?php echo esc_html( get_admin_page_title() ); ?></h1>
            <form action="options.php" method="post">
                <?php
                settings_fields( '84em_example_group' );
                do_settings_sections( '84em-example' );
                submit_button();
                ?>
            </form>
        </div>
        <?php
    }
}
```

Wiring notes: the group passed to `settings_fields()` must match the first argument of `register_setting()`; the page slug passed to `do_settings_sections()` must match the page argument of `add_settings_section()`/`add_settings_field()`. `options.php` handles the nonce and save; your `sanitize_callback` is the validation gate.


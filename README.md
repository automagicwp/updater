
## WordPress Plugin Updater

A simple and lightweight updater for WordPress plugins hosted on [AutomagicWP](https://www.automagicwp.com). It hooks into WordPress's built-in update mechanism to check for new versions and display the standard update notice in the admin area.

> [!IMPORTANT]
> This updater requires your plugin to be hosted on AutomagicWP.com. Your plugin header must include `Update URI: https://www.automagicwp.com`.

## Minimum requirements

- PHP: 8.0 or later
- WordPress: 6.0 or later
- Access to your plugin's source code.
- Optional: Composer for managing PHP dependencies

## Installation

### Via Composer

```bash
composer require automagicwp/updater
```

### Manual

Copy the file in `src/` into your plugin and require it:

```php
require_once 'path/to/PluginUpdater.php';
```

## Usage

Instantiate `PluginUpdater` inside your plugin. The defaults point at `automagicwp.com` so you only need to provide `file` and `id` for the standard setup:

```php
<?php
/**
 * Plugin Name: My Plugin
 * Version:     1.0.0
 * Update URI:  https://www.automagicwp.com
 */

if ( ! defined( 'ABSPATH' ) ) {
    exit;
}

require_once __DIR__ . '/vendor/autoload.php';

use AutomagicWP\Updater\PluginUpdater;

add_action( 'init', function () {
    new PluginUpdater( array(
        'file'   => __FILE__,
        'id'     => 'plugin_abc123',   // Plugin ID from the AutomagicWP dashboard
        'secret' => 'your-api-key',    // Optional: required for private plugins
    ) );
} );
```

### All options

| Option | Required | Default | Description |
|--------|----------|---------|-------------|
| `file` | Yes | — | Path to the main plugin file (`__FILE__`) |
| `id` | Yes | — | Plugin ID from the AutomagicWP dashboard |
| `secret` | No | — | API key or license key. Required for private plugins and white-label branding. |
| `api_url` | No | `https://www.automagicwp.com/api/v1/plugin/update` | Update check endpoint |
| `config_url` | No | `https://www.automagicwp.com/api/v1/plugin/config` | Branding config endpoint |
| `hostname` | No | `automagicwp.com` | Must match the `Update URI` header |
| `telemetry` | No | `true` | Send anonymous site info (WP version, PHP version) with update checks |

Replace `plugin_abc123` with the Plugin ID shown in your AutomagicWP dashboard under **Settings**.

---

## White-label branding

Agencies distributing a single plugin to multiple client sites can create **licenses** in the AutomagicWP dashboard (Settings → White-label Licenses). Each license has:

- Its own API key (revocable per site)
- **Client name** — shown in the plugin admin UI instead of the canonical plugin name
- **Primary colour / foreground colour** — brand colours for the admin UI
- **Metadata** — free-form JSON for any extra fields (logo URL, support email, feature flags, etc.)

### Fetching branding config

Pass the license key as the `secret` option, then call `get_config()` anywhere in your plugin:

```php
$updater = new PluginUpdater( array(
    'file'   => __FILE__,
    'id'     => 'plugin_abc123',
    'secret' => get_option( 'my_plugin_license_key' ),
) );

$config = $updater->get_config();

if ( $config ) {
    $client_name   = $config->clientName            ?? 'My Plugin';
    $primary_color = $config->primaryColor          ?? '#2563eb';
    $fg_color      = $config->primaryForegroundColor ?? '#ffffff';

    // Access free-form metadata
    $logo_url      = $config->metadata->logo_url      ?? '';
    $support_email = $config->metadata->support_email ?? '';
    $report_from   = $config->metadata->report_from   ?? '';
    $report_footer = $config->metadata->report_footer ?? '';

    // Feature flags stored in metadata
    $features      = (array) ( $config->metadata->features ?? array() );
    $multi_country = $features['multi_country'] ?? false;
}
```

### Response shape

```json
{
  "clientName": "Acme Corp",
  "primaryColor": "#1A56DB",
  "primaryForegroundColor": "#ffffff",
  "metadata": {
    "logo_url": "https://acme.com/logo.svg",
    "support_email": "help@acme.com",
    "report_from": "Acme SEO <noreply@acme.com>",
    "report_footer": "Powered by Acme · acme.com",
    "features": {
      "multi_country": true,
      "vehicle_compatibility": false,
      "content_creation": false
    },
    "countries": {
      "NL": { "weight": 1.0, "frequency": "daily" },
      "DE": { "weight": 0.8, "frequency": "weekly" }
    }
  }
}
```

All fields are nullable — always provide defaults in your plugin code.

### Caching

`get_config()` caches the response in a WordPress transient for **5 minutes** to avoid a remote call on every admin page load. To force a fresh fetch (e.g. after saving a settings page):

```php
$config = $updater->get_config( force: true );
```

### Providing a license key settings field

Your plugin's settings page should let the site owner paste their license key:

```php
add_action( 'admin_init', function () {
    register_setting( 'my_plugin_settings', 'my_plugin_license_key' );

    add_settings_field(
        'license_key',
        'License Key',
        function () {
            $key = get_option( 'my_plugin_license_key', '' );
            echo '<input type="password" name="my_plugin_license_key"
                         value="' . esc_attr( $key ) . '" class="regular-text">';
        },
        'my-plugin',
        'my_plugin_license_section'
    );
} );
```

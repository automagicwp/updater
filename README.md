
## WordPress Plugin Updater

A simple and lightweight updater for WordPress plugins hosted on [AutomagicWP](https://automagicwp.com). It hooks into WordPress's built-in update mechanism to check for new versions and display the standard update notice in the admin area.

> [!IMPORTANT]
> This updater requires your plugin to be hosted on AutomagicWP.com. Your plugin header must include `Update URI: https://automagicwp.com`.

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
 * Update URI:  https://automagicwp.com
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
| `secret` | No | — | API key for private plugins (omit for public plugins) |
| `api_url` | No | `https://automagicwp.com/api/v1/plugin/update` | API endpoint |
| `hostname` | No | `automagicwp.com` | Must match the `Update URI` header |
| `telemetry` | No | `true` | Send anonymous site info (WP version, PHP version) with update checks |

Replace `plugin_abc123` with the Plugin ID shown in your AutomagicWP dashboard under **Settings**.

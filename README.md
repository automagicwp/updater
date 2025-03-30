
## WordPress Plugin Updater

This is a simple plugin updater for WordPress plugins. It allows you to check for updates from the WPUpdateHub.com API and display a notice in the WordPress admin area when an update is available.

> [!IMPORTANT]
> This plugin updater is designed to work with plugins hosted on WPUpdateHub.com. However you can use this to integrate the updater into your own plugin. Then all you need is to create a backend endpoint to check and serve the update.

## Minimum requirements

- PHP: 8.0 or later
- WordPress: 6.0 or later
- Access to your plugin's source code.
- Optional: Composer for managing PHP dependencies

## Installation

To integrate this updater into your plugin, you need to require it via Composer or copy the files into your plugin.

### Via Composer

```bash
composer require wplatest/updater
```

### Manual

Copy the file in `src` directory into your plugin. Then require the file in your plugin.

```php
require_once 'path/to/updater.php';
```

## Usage

Instantiate the `PluginUpdater` class within your plugin, providing it with the necessary configuration options. Here's a basic example you can adjust according to your needs:

```php
<?php

/**
 * Plugin Name:       Example Plugin
 *
 * @package ExamplePlugin
 */

if ( ! defined( 'ABSPATH' ) ) {
	exit;
}

// Include the Composer autoloader in your plugin.
require_once 'vendor/autoload.php';

use WpLatest\Updater\PluginUpdater;

/**
 * Include the updater class.
 */
class ExamplePlugin {

	/**
	 * Base constructor.
	 */
	public function __construct() {
		add_action( 'init', array( $this, 'initialise' ) );
	}

	/**
	 * Initialize the updater.
	 */
	public function initialise() {
		$options = array(
			'file'      => __FILE__,
			'id'        => 'your-plugin-id',
			'secret'	=> 'your-plugin-secret',
			// Optional configuration, you don't need to set these if you're using the wpupdatehub.com API.
			'api_url'   => 'https://api.yoursite.com',
			// corresponds with the "Update URI" field in your plugin's header.
			'hostname'  => 'yoursite.com',
			'telemetry' => true,
		);

    	new PluginUpdater( $options );
	}
}

$wp_latest_test_plugin = new ExamplePlugin();
```

Replace `'your-plugin-id'` with the actual ID provided to you by WPUpdateHub.com for your plugin.
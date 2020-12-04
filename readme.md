# Endurance WordPress Module Loader
This loader instantiates Endurance WordPress Modules inside our WordPress Plugins.

* What are modules?
* Creating/registering modules
* Installing from our Satis
* Local development notes

## Endurance WordPress Modules
Endurance WordPress Modules are PHP packages intended to be installed in WordPress Plugins via composer from our satis registry. They were first used in the Mojo Marketplace WordPress Plugin and later the Bluehost WordPress Plugin.

Modules are essentially WordPress Plugins for reuse in Endurance products.

Modules can be required/forced, optional, hidden and can be toggled by code and (sometimes) by users.

## Creating & Registering a Module

Modules will eventually be created from templates, but for now here are some key things to know.

* Modules should contain a `bootstrap.php` file that is autoloaded by Composer. Functionality should load from `/inc`.
* Modules are loaded on the `init` hook with a priority of `10`.
* Module registration should tap the `after_setup_theme` hook.
* In `mojoness/mojo-marketplace-wp-plugin` two constants exist for tapping the Plugin's path and base URL: `MM_BASE_DIR` and `MM_BASE_URL`. Modules are in `/vendor`.
* In `bluehost/bluehost-wordpress-plugin` the equivalents are `BLUEHOST_PLUGIN_DIR` and `BLUEHOST_PLUGIN_URL`. Modules are in `/vendor`.

### `eig_register_module()`

This global function should be run during `after_theme_setup`.

```php
/**
 * @param array $args
 *
 * $args = array(
 *     'name'     => (string) **required** slug used for internal reference, like a CPT.
 *     'label     => (string) **required** i18n display label
 *     'callback' => (callable) **required** executed when module is active
 *     'isActive' => (boolean) **optional, default: false** whether module is forced active by default (can be overriden if !isHidden)
 *     'isHidden' => (boolean) **optional, default: false** whether module can be toggled in UI.
 * )
 */
eig_register_module( $args );
```

Example `bootstrap.php`:
```php
<?php

if ( function_exists( 'add_action' ) ) {
	add_action( 'after_setup_theme', 'eig_module_spam_prevention_register' );
}

/**
 * Register the spam prevention module
 */
function eig_module_spam_prevention_register() {
	eig_register_module( array(
		'name'     => 'spam-prevention',
		'label'    => __( 'Spam Prevention', 'endurance' ),
		'callback' => 'eig_module_spam_prevention_load',
		'isActive' => true,
	) );
}

/**
 * Load the spam prevention module
 */
function eig_module_spam_prevention_load() {
	require dirname( __FILE__ ) . '/spam-prevention.php';
}
```

### Installing from our Satis

Our modules are sourced from our 3rd-party package repository (Satis).

#### 1. Make sure to register our repository in the `composer.json`
```json
{
  "repositories": [
    {
      "type": "composer",
      "url": "https://bluehost.github.io/satis/",
      "only": [
        "bluehost/*",
        "mojoness/*",
        "endurance/*"
      ]
    }
  ]
}
```

#### 2. `composer require [satis-package-identifier]`

#### 3. `composer install`

### Local Development

When working on modules locally:

1. Clone the module repository somewhere on the filesystem, i.e. `/wp-content/modules/endurance-wp-module-awesome`.
2. Modify the sourcing-plugin's `composer.json` to reference that directory as a `path` in the repositories array (above the satis reference).
```json
{
  "repositories": [
    {
      "type": "path",
      "url": "../../modules/endurance-wp-module-awesome",
      "options": {
        "symlink": true
      }
    },
    {
      "type": "composer",
      "url": "https://bluehost.github.io/satis/",
      "only": [
        "bluehost/*",
        "mojoness/*",
        "endurance/*"
      ]
    }
  ]
}
```
3. In the `require` declaration, change the version to `@dev`
4. `composer install`



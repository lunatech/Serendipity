# Serendipity Codebase Reference for Agents

This document provides essential information about the Serendipity codebase structure, coding standards, and key systems to help agents understand and work with the code effectively.

## Table of Contents
1. [Project Overview](#project-overview)
2. [Directory Structure](#directory-structure)
3. [Coding Standards](#coding-standards)
4. [Template Loading System](#template-loading-system)
5. [Core Architecture Patterns](#core-architecture-patterns)
6. [Key Files Reference](#key-files-reference)

---

## Project Overview

**Serendipity (s9y)** is a mature, reliable PHP-powered weblog engine and CMS. It's been in development since 2002 and emphasizes security, extensibility, and ease of use.

### Technology Stack
- **Language**: PHP 7.4+ (designed for PHP 8.0+)
- **Template Engine**: Smarty 5.1
- **Database**: Abstracted SQL (supports MySQL/MariaDB, PostgreSQL, SQLite)
- **Package Manager**: Composer
- **Vendor Directory**: `bundled-libs/`

### Key Dependencies
```json
{
  "smarty/smarty": "^5.1",
  "katzgrau/klogger": "^1.0.0",
  "masterminds/html5": "^2.9",
  "simplepie/simplepie": "^1.8",
  "pear/http_request2": "^2.5"
}
```

---

## Directory Structure

### Root Level Entry Points
```
/
├── index.php                    # Frontend entry point
├── serendipity_admin.php        # Admin panel entry point
├── serendipity_config.inc.php   # Core configuration
├── comment.php                  # Comment/trackback handler
├── rss.php                      # RSS feed handler
└── composer.json                # Dependency management
```

### Main Directories

#### `/include/` - Core Libraries
All core functionality is split into modular include files (~21,400 lines of PHP total):

```
include/
├── admin/                       # Admin panel modules
├── db/                          # Database abstraction layer
│   ├── mysqli.inc.php
│   ├── postgres.inc.php
│   ├── pdo-sqlite.inc.php
│   └── sqlrelay.inc.php
├── tpl/                         # Configuration templates (for installer, .htaccess)
├── functions.inc.php            # General utilities (50 KB)
├── functions_config.inc.php     # Configuration management (93 KB)
├── functions_entries.inc.php    # Blog entry operations (82 KB)
├── functions_images.inc.php     # Image/media handling (149 KB)
├── functions_comments.inc.php   # Comment management (54 KB)
├── functions_smarty.inc.php     # Smarty integration & template plugins (55 KB)
├── functions_permalinks.inc.php # URL/permalink handling (30 KB)
├── functions_trackbacks.inc.php # Trackback/pingback handling (37 KB)
├── functions_installer.inc.php  # Installation system (47 KB)
├── plugin_api.inc.php           # Plugin architecture (71 KB)
├── serendipity_smarty_class.inc.php  # Smarty wrapper class
├── template_api.inc.php         # Template API
└── compat.inc.php               # PHP compatibility layer (17 KB)
```

#### `/templates/` - Theme Directory
Each theme is a self-contained directory with templates and assets:

```
templates/
├── default/                     # Basic default theme
├── 2k11/                        # Modern default theme
├── clean-blog/                  # Clean blog design
├── bootstrap4/                  # Bootstrap-based theme
├── timeline/                    # Timeline layout
├── skeleton/                    # Minimal theme
└── [other themes]/
    ├── index.tpl                # Main layout template
    ├── entries.tpl              # Blog entries display
    ├── comments.tpl             # Comments section
    ├── commentform.tpl          # Comment submission form
    ├── sidebar.tpl              # Sidebar plugins
    ├── trackbacks.tpl           # Trackbacks display
    ├── entries_archives.tpl     # Archive view
    ├── feed_*.tpl               # RSS/Atom feed templates
    ├── plugin_*.tpl             # Plugin-specific templates
    ├── admin/                   # Admin panel templates
    ├── img/                     # Theme images
    └── js/                      # Theme JavaScript
```

#### `/templates_c/` - Smarty Compilation Cache
Smarty compiled templates are cached here. Safe to delete (will be regenerated).

#### `/plugins/` - Bundled Plugins
30+ plugins providing extended functionality (sidebar widgets, entry features, etc.)

#### `/bundled-libs/` - Vendor Libraries
Composer-managed dependencies (Smarty, SimplePie, PEAR, etc.)

#### `/lang/` - Internationalization
Language files for 28+ languages, organized by language code.

---

## Coding Standards

### Function Naming Convention
All functions use the `serendipity_` prefix:

```php
// General functions
serendipity_addAuthor($name)
serendipity_fetchEntries($args)
serendipity_getTemplateFile($file)
serendipity_specialchars($str)

// Database functions (serendipity_db_ prefix)
serendipity_db_query($sql)
serendipity_db_escape_string($str)
serendipity_db_limit_sql($limit)
serendipity_db_insert_id()

// Smarty plugin functions (serendipity_smarty_ prefix)
serendipity_smarty_fetchPrintEntries($params, $smarty)
serendipity_smarty_printSidebar($params, $smarty)
serendipity_smarty_hookPlugin($params, $smarty)
serendipity_smarty_fetch($block, $file)
```

### Class Naming Convention
PascalCase with `Serendipity_` or `serendipity_` prefix:

```php
class Serendipity_Smarty extends \Smarty\Smarty
class Serendipity_Smarty_Security_Policy extends \Smarty\Security
class serendipity_plugin_recententries extends serendipity_plugin
class serendipity_smarty_emulator
```

### Constant Naming Convention
UPPERCASE with project scope prefix:

```php
define('IN_serendipity', true)           // Security guard constant
define('S9Y_FRAMEWORK', true)
define('S9Y_INCLUDE_PATH', dirname(__FILE__) . '/')
define('S9Y_TEMPLATE_FALLBACK', $path)
define('S9Y_TEMPLATE_USERDEFAULT', $path)
define('PATH_SMARTY_COMPILE', 'templates_c')
define('USERLEVEL_ADMIN', 255)
```

### Variable Naming Convention
- **camelCase** for local and instance variables
- **$serendipity['key']** for global state (main superglobal array)

```php
// Global state array
$serendipity['version'] = '2.6-beta2'
$serendipity['smarty']                  // Smarty instance
$serendipity['template']                // Current frontend theme
$serendipity['template_backend']        // Backend/admin theme
$serendipity['templatePath']            // Relative path to templates/
$serendipity['dbPrefix']                // Database table prefix
$serendipity['production']              // 'debug' or empty for production

// Local variables
$dateformat = $config['dateformat'];
$templateFile = $serendipity['templatePath'] . $directory . $file;
$numberOfComments = count($comments);
```

### Code Style

#### Indentation & Spacing
- **4 spaces** (no tabs)
- Space after control structures: `if (condition)`, `foreach($array as $key)`
- Opening braces on same line: `function name() {`

#### Documentation
All functions should have PHPDoc comments:

```php
/**
 * Fetch a list of trackbacks for an entry
 *
 * @access public
 * @param   int     $id         The ID of the entry
 * @param   string  $limit      How many trackbacks to show
 * @param   boolean $showAll    If true, also non-approved trackbacks will be shown
 * @return  array               Array of trackback objects
 */
function &serendipity_fetchTrackbacks($id, $limit = null, $showAll = false) {
    // Implementation
}
```

#### File Ending
Files end with vim settings:
```php
/* vim: set sts=4 ts=4 expandtab : */
```

---

## Template Loading System

### Overview
Serendipity uses **Smarty 5.1** with a custom wrapper class that implements:
- **Theme fallback chain** - Multiple themes checked in priority order
- **Security policy** - Custom security wrapper for template rendering
- **Caching** - Compiled template cache in `/templates_c/`
- **Plugin integration** - Custom Smarty functions and modifiers

### Template Loading Workflow

#### 1. Configuration Phase
**File**: `serendipity_config.inc.php` (lines 145-157)

Defines template paths and directories:
```php
@define('S9Y_TEMPLATE_FALLBACK', $serendipity['serendipityPath'] . 'templates/default');
@define('S9Y_TEMPLATE_USERDEFAULT', $serendipity['serendipityPath'] . 'templates/' . $serendipity['template']);
@define('S9Y_TEMPLATE_SECUREDIR', $serendipity['serendipityPath'] . 'templates');
@define('PATH_SMARTY_COMPILE', 'templates_c');

// Theme selection
$serendipity['defaultTemplate'] = '2k11';      // Frontend default
$serendipity['template_backend'] = '2k11';     // Backend/admin default
$serendipity['template_engine'] = '';          // Fallback themes (optional)
```

#### 2. Smarty Instance Creation
**File**: `include/functions_smarty.inc.php`

The `serendipity_smarty_init()` function:
1. Loads Smarty from `bundled-libs/`
2. Creates `Serendipity_Smarty` singleton instance
3. Enables custom security policy
4. Registers all custom modifiers and functions

```php
function serendipity_smarty_init() {
    global $serendipity;

    // Load Smarty
    require_once SMARTY_DIR . 'Smarty.class.php';
    require_once S9Y_INCLUDE_PATH . 'serendipity_smarty_class.inc.php';

    // Create instance
    $serendipity['smarty'] = Serendipity_Smarty::getInstance();

    // Enable security
    $serendipity['smarty']->enableSecurity('Serendipity_Smarty_Security_Policy');

    // Register plugins (modifiers and functions)
    // ... register calls here
}
```

#### 3. Smarty Wrapper Configuration
**File**: `include/serendipity_smarty_class.inc.php` (lines 70-196)

The `Serendipity_Smarty` class constructor builds the template directory search chain:

```php
class Serendipity_Smarty extends \Smarty\Smarty {
    private function setParams() {
        global $serendipity;

        $template_dirs = array();

        // 1. Current selected theme
        $template_dirs[] = $serendipity['serendipityPath'] .
                          $serendipity['templatePath'] .
                          $serendipity['template'];

        // 2. Template engine fallback themes (if configured)
        if ($template_engine = serendipity_get_config_var('template_engine')) {
            $engines = explode(',', $template_engine);
            foreach($engines as $engine) {
                $template_dirs[] = $serendipity['serendipityPath'] .
                                  $serendipity['templatePath'] .
                                  trim($engine);
            }
        }

        // 3. Default theme fallback
        $template_dirs[] = $serendipity['serendipityPath'] .
                          $serendipity['templatePath'] .
                          $serendipity['defaultTemplate'];

        // 4. Backend theme
        $template_dirs[] = $serendipity['serendipityPath'] .
                          $serendipity['templatePath'] .
                          $serendipity['template_backend'];

        // 5. Root templates directory
        $template_dirs[] = S9Y_TEMPLATE_SECUREDIR;

        // 6. Plugin directories
        $template_dirs[] = $serendipity['serendipityPath'] . 'plugins';

        $this->setTemplateDir($template_dirs);
        $this->setCompileDir($serendipity['serendipityPath'] . PATH_SMARTY_COMPILE);

        // Smarty behavior
        $this->compile_check = true;    // Check if templates changed
        $this->use_sub_dirs = true;     // Create cache subdirectories

        // Debug mode
        if ($serendipity['production'] === 'debug') {
            $this->force_compile = true;
            $this->caching = false;
            $this->debugging = true;
        }
    }
}
```

**Template Directory Search Order (Priority)**:
1. Current selected frontend theme (e.g., `clean-blog/`)
2. Template engine fallback themes (if configured)
3. Default theme (`2k11/`)
4. Backend theme
5. Root templates directory
6. Plugin directories

#### 4. Template File Resolution
**File**: `include/functions_config.inc.php` (lines 262-288)

The `serendipity_getTemplateFile($file, $key, $force_frontend_fallback)` function:

```php
function serendipity_getTemplateFile($file, $key = 'serendipityHTTPPath', $force_frontend_fallback = false) {
    global $serendipity;

    $directories = array();

    // Determine if frontend or backend
    if ((! defined('IN_serendipity_admin')) || $force_frontend_fallback) {
        $directories[] = $serendipity['template'] . '/';      // Frontend theme
    } else {
        $directories[] = $serendipity['template_backend'] . '/'; // Backend theme
    }

    // Add fallbacks
    $directories[] = $serendipity['template_engine'] . '/';
    $directories[] = $serendipity['defaultTemplate'] . '/';
    $directories = array_unique($directories);

    // Search in order
    foreach ($directories as $directory) {
        $templateFile = $serendipity['templatePath'] . $directory . $file;

        if (file_exists($serendipity['serendipityPath'] . $templateFile)) {
            return ($serendipity[$key] ?? '') . $templateFile;
        }
    }

    return false;
}
```

#### 5. Template Rendering
**File**: `index.php` or `serendipity_admin.php`

Final step - assign variables and render:

```php
// Assign template variables
$serendipity['smarty']->assign(array(
    'blogTitle'          => $serendipity['blogTitle'],
    'blogDescription'    => $serendipity['blogDescription'],
    'entries'            => $entries_array,
    'comments'           => $comments_array,
    'CONTENT'            => $main_content,
    'SIDEBAR'            => $sidebar_content,
));

// Render template
$serendipity['smarty']->display(
    serendipity_getTemplateFile($serendipity['smarty_file'])
);
```

### Smarty Registered Functions & Modifiers

**Functions** (template tags):
```php
// Usage: {serendipity_printSidebar side='left'}
serendipity_smarty_printSidebar($params, $smarty)

// Usage: {serendipity_hookPlugin hook='frontend_header'}
serendipity_smarty_hookPlugin($params, $smarty)

// Usage: {serendipity_printComments entry_id=$entry.id}
serendipity_smarty_printComments($params, $smarty)

// Usage: {serendipity_printEntries entries=$entries}
serendipity_smarty_fetchPrintEntries($params, $smarty)
```

**Modifiers** (line filters):
```php
// Usage: {$date|formatTime}
serendipity_smarty_formatTime($date, $format)

// Usage: {$filename|makeFilename}
serendipity_makeFilename($filename)

// Usage: {$value|emptyPrefix}
serendipity_emptyPrefix($value)
```

### Template Fetch API

**File**: `include/functions_smarty.inc.php`

```php
/**
 * Fetch a template file and assign to a template variable
 *
 * @param string $block      Variable name to assign to
 * @param string $file       Template filename
 * @param bool   $echo       If true, echo instead of return
 * @param bool   $force_frontend If true, use frontend fallback chain
 * @return string            Parsed template output
 */
function serendipity_smarty_fetch($block, $file, $echo = false, $force_frontend = false) {
    global $serendipity;

    $output = $serendipity['smarty']->fetch(
        'file:' . serendipity_getTemplateFile($file, 'serendipityPath', $force_frontend)
    );

    $serendipity['smarty']->assignByRef($block, $output);

    return $output;
}
```

Example usage:
```php
// Fetch and render trackbacks template, assign to {$TRACKBACKS}
serendipity_smarty_fetch('TRACKBACKS', 'trackbacks.tpl');

// Now in template, use: {$TRACKBACKS}
```

### Security Policy

**File**: `include/serendipity_smarty_class.inc.php` (lines 12-24)

```php
class Serendipity_Smarty_Security_Policy extends \Smarty\Security {
    /**
     * Allow fetch() and include calls to pull .tpl files from any directory
     * Enables symlinked plugin directories outside main s9y path
     */
    public function isTrustedResourceDir($path, $isConfig = NULL) {
        return true;
    }
}
```

Enabled with:
```php
$serendipity['smarty']->enableSecurity('Serendipity_Smarty_Security_Policy');
```

### Caching Behavior

```php
$this->compile_check = true;      // Always check template modification time
$this->use_sub_dirs = true;       // Create subdirectories in templates_c/
$this->config_overwrite = true;   // Overwrite Smarty config values

// Debug mode (if $serendipity['production'] === 'debug')
$this->force_compile = true;      // Force recompile every request (no cache)
$this->caching = false;           // No output caching in debug
$this->debugging = true;          // Enable Smarty debugging
```

---

## Core Architecture Patterns

### 1. Singleton Pattern (Smarty)
```php
class Serendipity_Smarty {
    public static function getInstance($newInstance = null) {
        static $instance = null;
        if(isset($newInstance)) $instance = $newInstance;
        if($instance == null) $instance = new Serendipity_Smarty();
        return $instance;
    }
}

// Global usage
$serendipity['smarty'] = Serendipity_Smarty::getInstance();
```

### 2. Global State Array
All configuration and runtime state stored in `$serendipity`:

```php
$serendipity = array(
    'version'               => '2.6-beta2',
    'template'              => 'clean-blog',      // Frontend theme
    'template_backend'      => '2k11',            // Admin theme
    'templatePath'          => 'templates/',
    'serendipityPath'       => '/var/www/html/s9y/',
    'serendipityHTTPPath'   => 'http://example.com/s9y/',
    'blogTitle'             => 'My Blog',
    'blogDescription'       => 'A blog about...',
    'smarty'                => $smarty_instance,
    'smarty_file'           => 'index.tpl',       // Template to render
    'viewtype'              => 'start',           // View mode (start, entry, plugin, etc)
    'production'            => '',                // '' = production, 'debug' = debug mode
    'dbPrefix'              => 's9y_',            // Database table prefix
    // ... many more settings
);
```

### 3. Plugin Hook System
Plugins register event handlers that are triggered at specific points:

```php
// Register hook
serendipity_plugin_api::registerPlugin('frontend_header', 'plugin_name');

// Trigger hook (in template)
{serendipity_hookPlugin hook='frontend_header'}

// Or in PHP
serendipity_hookPlugin('frontend_header');
```

### 4. Database Abstraction Layer
Multiple database drivers in `/include/db/`:
- `mysqli.inc.php` - MySQL/MariaDB
- `postgres.inc.php` - PostgreSQL
- `pdo-sqlite.inc.php` - SQLite
- `sqlrelay.inc.php` - SQL Relay

All access through functions like:
```php
serendipity_db_query($sql)
serendipity_db_escape_string($str)
serendipity_db_limit_sql($limit)
serendipity_db_insert_id()
```

---

## Key Files Reference

| File | Purpose | Lines |
|------|---------|-------|
| `index.php` | Frontend entry point | 127 |
| `serendipity_admin.php` | Admin panel entry point | 262 |
| `serendipity_config.inc.php` | Core configuration | 507 |
| `comment.php` | Comment/trackback handler | 251 |
| `rss.php` | RSS feed handler | 320 |
| `include/functions.inc.php` | General utilities | ~50 KB |
| `include/functions_config.inc.php` | Configuration management | ~93 KB |
| `include/functions_entries.inc.php` | Entry/post operations | ~82 KB |
| `include/functions_smarty.inc.php` | Smarty integration | ~55 KB |
| `include/functions_images.inc.php` | Image/media handling | ~149 KB |
| `include/functions_comments.inc.php` | Comment operations | ~54 KB |
| `include/functions_permalinks.inc.php` | URL routing | ~30 KB |
| `include/functions_installer.inc.php` | Installation wizard | ~47 KB |
| `include/plugin_api.inc.php` | Plugin system | ~71 KB |
| `include/serendipity_smarty_class.inc.php` | Smarty wrapper class | Core |
| `include/template_api.inc.php` | Template API | Core |
| `include/compat.inc.php` | PHP compatibility | ~17 KB |
| `composer.json` | Dependency management | - |
| `templates/{theme}/` | Theme directory | Multiple |
| `plugins/` | Bundled plugins | 30+ |
| `lang/` | Language files | 28+ langs |

---

## Quick Reference: Common Tasks

### To add a new template variable:
1. Assign in PHP: `$serendipity['smarty']->assign('varName', $value);`
2. Use in template: `{$varName}`

### To add a new Smarty function:
1. Create function: `function serendipity_smarty_myFunc($params, $smarty) { ... }`
2. Register in `serendipity_smarty_init()`: `$smarty->registerPlugin('function', 'myFunc', 'serendipity_smarty_myFunc');`
3. Use in template: `{serendipity_myFunc param1=$value}`

### To add a new Smarty modifier:
1. Create function: `function serendipity_myModifier($value, $param) { ... }`
2. Register in `serendipity_smarty_init()`: `$smarty->registerPlugin('modifier', 'myModifier', 'serendipity_myModifier');`
3. Use in template: `{$variable|myModifier:$param}`

### To add a new database query:
1. Use abstracted functions: `serendipity_db_query($sql)`
2. Always escape user input: `serendipity_db_escape_string($input)`
3. Use limit helper: `serendipity_db_limit_sql($limit)`

### To override a template:
1. Copy from default: `templates/2k11/entries.tpl`
2. Paste into theme: `templates/myTheme/entries.tpl`
3. Modify as needed - will use theme version instead of fallback

---

**Last Updated**: February 2026
**Project Version**: 2.6-beta2
**Documentation Version**: 1.0

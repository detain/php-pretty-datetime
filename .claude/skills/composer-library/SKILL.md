---
name: composer-library
description: Adds `detain/php-pretty-datetime` as a Composer dependency and sets up PSR-4 autoload per `composer.json`. Use when user says 'install this library', 'add to composer', 'require in my project', or asks how to include `PrettyDateTime\` namespace. Do NOT use for modifying the library's own `composer.json`.
---
# composer-library

## Critical

- Target `composer.json` is the **consumer project's** file — never edit the library's own `composer.json` at `vendor/detain/php-pretty-datetime/composer.json`.
- Requires PHP `>= 7.1` in the consumer project (library uses `public const` syntax). Verify `"php"` constraint before adding.
- The namespace is `PrettyDateTime\` (with trailing backslash in JSON: `"PrettyDateTime\\"`). A typo here silently breaks autoloading with no clear error.

## Instructions

1. **Add the dependency.** Run from the consumer project root:
   ```bash
   composer require detain/php-pretty-datetime
   ```
   Verify `vendor/detain/php-pretty-datetime/src/PrettyDateTime.php` exists before proceeding.

2. **Confirm autoload entry.** Open the consumer project's `composer.json` and verify Composer added (or manually add) under `"autoload"`:
   ```json
   {
       "require": {
           "detain/php-pretty-datetime": "^1.0"
       },
       "autoload": {
           "psr-4": {
               "YourApp\\": "src/"
           }
       }
   }
   ```
   The library registers its own `"PrettyDateTime\\": "src/"` mapping inside its own `composer.json` — you do **not** add that to your project's autoload block.

3. **Regenerate autoloader** (required after any manual `composer.json` edit):
   ```bash
   composer dump-autoload
   ```
   Verify `vendor/autoload.php` exists and `vendor/composer/autoload_psr4.php` contains `'PrettyDateTime\\'`.

4. **Use in PHP code.** Import and call the static `parse()` method:
   ```php
   <?php
   require_once __DIR__ . '/vendor/autoload.php';

   use PrettyDateTime\PrettyDateTime;

   // Past date
   echo PrettyDateTime::parse(new DateTime('- 2 hours'));   // 2 hours ago

   // Future date
   echo PrettyDateTime::parse(new DateTime('+ 1 minute'));  // In 1 minute

   // Custom reference point
   $ref = new DateTime('1991-05-18 00:00:00 UTC');
   $dt  = new DateTime('1991-05-17 23:59:59 UTC');
   echo PrettyDateTime::parse($dt, $ref);  // Yesterday
   ```
   Verify output matches expected string before shipping.

## Examples

**User says:** "Add php-pretty-datetime to my project and show me how to display a post timestamp."

**Actions taken:**
1. `composer require detain/php-pretty-datetime` — installs to `vendor/`
2. Confirmed `vendor/autoload.php` includes `PrettyDateTime\\` mapping
3. Added to controller:
   ```php
   use PrettyDateTime\PrettyDateTime;

   $posted = new DateTime($post['created_at']);
   $post['time_ago'] = PrettyDateTime::parse($posted); // e.g. "3 days ago"
   ```

**Result:** `$post['time_ago']` returns a human-readable string like `"3 days ago"` or `"In 2 hours"`.

## Common Issues

- **`Class 'PrettyDateTime\PrettyDateTime' not found`** after `composer require`:
  1. Run `composer dump-autoload`
  2. Check `vendor/composer/autoload_psr4.php` for `'PrettyDateTime\\'` key
  3. Confirm `require_once __DIR__ . '/vendor/autoload.php'` appears before the `use` statement

- **`composer require` resolves nothing / version conflict`**:
  Check your `composer.json` `"php"` constraint — it must allow `>= 7.1`. Change `"php": ">=7.1"` if needed, then retry.

- **`parse()` returns wrong string for timezone-sensitive dates`**:
  Pass an explicit `$reference` DateTime with a matching timezone instead of relying on the `new DateTime('now')` default. The library inherits the timezone from `$dateTime` when no reference is given.

- **`PHP Fatal error: Uncaught Error: Cannot use 'public' as constant modifier`** (PHP < 7.1):
  Upgrade to PHP 7.1+. The library uses `public const` which is not available in PHP 5.x/7.0.
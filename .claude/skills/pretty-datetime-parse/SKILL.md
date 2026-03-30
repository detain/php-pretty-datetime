---
name: pretty-datetime-parse
description: Demonstrates correct usage of PrettyDateTime::parse() from src/PrettyDateTime.php, including the optional reference DateTime argument. Use when user asks 'how do I use this library', 'format a date', 'show relative time', 'Yesterday/Tomorrow output', or 'moments ago'. Covers past, future, same-day, and cross-midnight edge cases. Do NOT use for modifying the library internals or adding new time units.
---
# pretty-datetime-parse

## Critical

- `parse()` is the **only public API** — never call `prettyFormat()` directly (it is `private static`).
- Both arguments must be `\DateTime` instances. Passing strings or timestamps directly will cause a type error.
- Day-boundary detection (`Yesterday`/`Tomorrow`) is based on calendar date comparison (`Y/m/d`), not raw seconds. A 1-second difference across midnight triggers `Yesterday`/`Tomorrow`, not `Moments ago`.
- Omitting `$reference` defaults to `new \DateTime('now')` in the same timezone as `$dateTime`. Always pass an explicit `$reference` in tests to avoid time-dependent failures.

## Instructions

1. **Install the library.**
   Verify `detain/php-pretty-datetime` is in `composer.json` and run `composer install`.
   Confirm `src/PrettyDateTime.php` exists before proceeding.

2. **Import the class** at the top of your file:
   ```php
   use PrettyDateTime\PrettyDateTime;
   ```

3. **Call `parse()` with one argument** (relative to now):
   ```php
   $result = PrettyDateTime::parse(new \DateTime('now'));          // Moments ago
   $result = PrettyDateTime::parse(new \DateTime('- 59 minute')); // 59 minutes ago
   $result = PrettyDateTime::parse(new \DateTime('+ 1 minute'));  // In 1 minute
   ```

4. **Call `parse()` with an explicit `$reference`** for deterministic output (required in tests and batch jobs):
   ```php
   $ref = new \DateTime('1991-05-18 23:59:59 UTC');
   $dt  = new \DateTime('1991-05-18 22:59:59 UTC');
   echo PrettyDateTime::parse($dt, $ref); // 1 hour ago
   ```

5. **Cross-midnight edge cases** — use `clone` + `modify()` to stay within the same reference day:
   ```php
   // Yesterday: 1 second before midnight of the reference day
   $midnight = new \DateTime('1991-05-18 00:00:00 UTC');
   $dt = clone $midnight;
   $dt->modify('- 1 second');
   echo PrettyDateTime::parse($dt, $midnight); // Yesterday

   // Tomorrow: 1 second after 23:59:59 of the reference day
   $beforeMidnight = new \DateTime('1991-05-18 23:59:59 UTC');
   $dt = clone $beforeMidnight;
   $dt->modify('+ 1 second');
   echo PrettyDateTime::parse($dt, $beforeMidnight); // Tomorrow
   ```

6. **Verify output** against the full output map:
   | Condition | Output |
   |---|---|
   | Same day, 0–59 s in past | `Moments ago` |
   | Same day, 0–59 s in future | `Seconds from now` |
   | Same day, 1–59 min | `1 minute ago` / `In 1 minute` |
   | Same day, 1–23 hr | `1 hour ago` / `In 1 hour` |
   | Calendar day before reference | `Yesterday` |
   | Calendar day after reference | `Tomorrow` |
   | 2–7 days | `N days ago` / `In N days` |
   | 2–5 weeks | `N weeks ago` / `In N weeks` |
   | 2–11 months | `N months ago` / `In N months` |
   | 1+ years | `N year(s) ago` / `In N year(s)` |

## Examples

**User says:** "How do I show that a post was created 3 hours ago?"

**Actions taken:**
```php
use PrettyDateTime\PrettyDateTime;

$createdAt = new \DateTime('- 3 hour'); // stored timestamp
echo PrettyDateTime::parse($createdAt); // outputs: 3 hours ago
```

**User says:** "Show relative time between two specific historical dates."

**Actions taken:**
```php
use PrettyDateTime\PrettyDateTime;

$ref = new \DateTime('1991-05-18 00:00:00 UTC');
$dt  = new \DateTime('1991-05-17 23:59:59 UTC');
echo PrettyDateTime::parse($dt, $ref); // Yesterday
```

## Common Issues

- **`Call to undefined method PrettyDateTime::prettyFormat()`** — you called the private helper directly. Use `PrettyDateTime::parse()` only.
- **Always returns `Moments ago` in tests** — you omitted `$reference`. The default is `now`, so a fixed past `$dateTime` returns the correct string in production but drifts in tests. Fix: always pass an explicit `$reference` in test assertions.
- **Expected `Yesterday` but got `1 day ago`** — the `$dateTime` and `$reference` are on the same calendar day or more than 1 day apart. `Yesterday`/`Tomorrow` require exactly adjacent calendar dates (`Y/m/d` comparison). Use `clone $reference; ->modify('- 1 second')` from a midnight reference to guarantee the cross-day boundary.
- **`Class 'PrettyDateTime\PrettyDateTime' not found`** — autoloader not loaded. Add `require __DIR__ . '/../src/PrettyDateTime.php';` or ensure `composer install` was run and the autoloader is included via `require 'vendor/autoload.php';`.
- **Wrong plural form (e.g. `1 minutes ago`)** — you are not using `parse()`; the pluralization logic inside `prettyFormat()` only fires when `floor(abs($difference)) > 1`. If you built a custom format string, delete it and delegate to `parse()`.
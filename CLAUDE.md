# php-pretty-datetime

Generates human-readable strings from PHP `DateTime` objects — past (`2 years ago`) and future (`In 1 minute`).

## Commands

```bash
phpunit                  # run all tests from project root
composer install         # install deps
```

## Architecture

- **Library entry**: `src/PrettyDateTime.php` · namespace `PrettyDateTime\` · PSR-4 via `composer.json`
- **Single class**: `PrettyDateTime::parse(\DateTime $dateTime, \DateTime $reference = null)` → string
- **Time constants** (seconds): `MINUTE=60` · `HOUR=3600` · `DAY=86400` · `WEEK=604800` · `MONTH=2628000` · `YEAR=31536000`
- **Tests**: `tests/PrettyDateTimeTest.php` · PHPUnit data providers for past/future ranges
- **Config**: `phpunit.xml.dist` · CI via `.travis.yml`

## Usage Pattern

```php
use PrettyDateTime\PrettyDateTime;

PrettyDateTime::parse(new DateTime('now'));              // Moments ago
PrettyDateTime::parse(new DateTime('+ 1 minute'));       // In 1 minute
PrettyDateTime::parse(new DateTime('- 59 minute'));      // 59 minutes ago

// Optional reference DateTime (defaults to DateTime('now'))
$ref = new DateTime('1991-05-18 00:00:00 UTC');
$dt  = new DateTime('1991-05-17 23:59:59 UTC');
PrettyDateTime::parse($dt, $ref);  // Yesterday
```

## Test Pattern

Data providers in `tests/PrettyDateTimeTest.php` generate ranges:

```php
public function pastDateTimesAndStrings() {
    return [
        ['- 1 minute', '1 minute ago'],
        ['- 2 minute', '2 minutes ago'],
    ];
}
/** @dataProvider pastDateTimesAndStrings */
public function testDateTimesInThePast($timeAgo, $prettyString) {
    $dateTime = clone $this->beforeMidnight;
    $dateTime->modify($timeAgo);
    $this->assertEquals($prettyString, PrettyDateTime::parse($dateTime, $this->beforeMidnight));
}
```

## Conventions

- All logic lives in `src/PrettyDateTime.php` — no additional files
- `prettyFormat()` is private static; only `parse()` is public API
- Day boundary detection uses `Y/m/d` format string comparison, not raw diff
- Pluralization: append `s` when `floor(abs($difference)) > 1`
- PHP `>= 7.1` required (`public const` syntax)
- MIT License — see `LICENSE.txt`

<!-- caliber:managed:pre-commit -->
## Before Committing

**IMPORTANT:** Before every git commit, you MUST ensure Caliber syncs agent configs with the latest code changes.

First, check if the pre-commit hook is already installed:
```bash
grep -q "caliber" .git/hooks/pre-commit 2>/dev/null && echo "hook-active" || echo "no-hook"
```

- If **hook-active**: the hook handles sync automatically — just commit normally. Tell the user: "Caliber will sync your agent configs automatically via the pre-commit hook."
- If **no-hook**: run Caliber manually before committing:
  1. Tell the user: "Caliber: Syncing agent configs with your latest changes..."
  2. Run: `caliber refresh && git add CLAUDE.md .claude/ .cursor/ .cursorrules .github/copilot-instructions.md .github/instructions/ AGENTS.md CALIBER_LEARNINGS.md 2>/dev/null`
  3. After it completes, briefly tell the user what Caliber updated. Then proceed with the commit.

If `caliber` is not found, tell the user: "This project uses Caliber for agent config sync. Run /setup-caliber to get set up."
<!-- /caliber:managed:pre-commit -->

<!-- caliber:managed:learnings -->
## Session Learnings

Read `CALIBER_LEARNINGS.md` for patterns and anti-patterns learned from previous sessions.
These are auto-extracted from real tool usage — treat them as project-specific rules.
<!-- /caliber:managed:learnings -->

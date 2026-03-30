---
name: phpunit-data-provider
description: Generates PHPUnit test methods with @dataProvider following the pattern in tests/PrettyDateTimeTest.php. Use when user says 'add test', 'write a test case', 'test this range', or adds new time thresholds to src/PrettyDateTime.php. Covers setUp fixtures, data array generation loops, and assertEquals assertions. Do NOT use for integration or functional tests.
---
# PHPUnit Data Provider

## Critical

- The test class extends `PHPUnit_Framework_TestCase` (not `TestCase`) — match this exactly.
- `require __DIR__ . '/../src/PrettyDateTime.php'` must be the first line; there is no autoloader in the test file.
- Provider methods must be `public` and return a plain `array` of `[$modifier, $expectedString]` pairs.
- Always use `clone $this->beforeMidnight` (past tests) or `clone $this->midnight` (future tests) — never `new DateTime('now')` in data-driven tests, as that creates timing-dependent tests.
- Pluralization rule: unit gets `s` appended when `floor(abs($difference)) > 1`. Match output strings exactly: `'1 minute ago'`, `'2 minutes ago'`, `'In 1 hour'`, `'In 2 hours'`.

## Instructions

1. **Verify the test file exists** at `tests/PrettyDateTimeTest.php`. All new tests go in this file — do not create additional test files.

2. **Check the `setUp` fixtures** (lines 9–15). Two reference points are always available:
   ```php
   $this->midnight       = new DateTime('1991-05-18 00:00:00 UTC'); // use for future tests
   $this->beforeMidnight = new DateTime('1991-05-18 23:59:59 UTC'); // use for past tests
   ```
   Use `clone` on these — never mutate them directly.

3. **Write the data provider method** (public, no annotation):
   ```php
   public function myRangeDataProvider()
   {
       $testData = [
           ['- 1 unit', '1 unit ago'],       // singular baseline
           [sprintf('- %d second', PrettyDateTime::CONSTANT), 'boundary label'],
       ];

       for ($i = 2; $i <= MAX; $i++) {
           $seconds = PrettyDateTime::CONSTANT * $i;
           array_push($testData, ["- $seconds second", "$i units ago"]);
       }

       return $testData;
   }
   ```
   - Use `PrettyDateTime::MINUTE/HOUR/DAY/WEEK/MONTH/YEAR` constants for second-based offsets.
   - For week offsets, compute `$days = 7 * $i` and use `"- $days day"` as the modifier (see lines 68–70).
   - For month/year offsets, add `PrettyDateTime::HOUR` to month-based seconds to push clearly past the boundary (see line 74).

4. **Write the test method** with the `@dataProvider` annotation:
   ```php
   /**
    * @dataProvider myRangeDataProvider
    */
   public function testMyRange($modifier, $prettyString)
   {
       $dateTime = clone $this->beforeMidnight; // or $this->midnight for future
       $dateTime->modify($modifier);
       $prettyDateTime = PrettyDateTime::parse($dateTime, $this->beforeMidnight);
       $this->assertEquals($prettyString, $prettyDateTime);
   }
   ```
   - `assertEquals($expected, $actual)` — expected string is always the first argument.
   - Always pass the reference fixture as the second argument to `PrettyDateTime::parse()`, never omit it in tests.

5. **Run the test suite** from the project root:
   ```bash
   phpunit
   ```
   All existing tests must still pass. A new provider failure prints the failing `[$modifier, $expectedString]` pair — fix the expected string to match `PrettyDateTime::prettyFormat()` output.

## Examples

**User says:** "Add tests for the 2–5 week range in the past"

**Actions taken:**
1. Read `tests/PrettyDateTimeTest.php` — confirm `$this->beforeMidnight` fixture exists.
2. Add provider (mirrors lines 66–70 of existing file):
   ```php
   public function pastWeeksAndStrings()
   {
       $testData = [];
       for ($i = 2; $i <= 5; $i++) {
           $days = 7 * $i;
           array_push($testData, ["- $days day", "$i weeks ago"]);
       }
       return $testData;
   }
   ```
3. Add test method:
   ```php
   /** @dataProvider pastWeeksAndStrings */
   public function testDateTimesInPastWeeks($timeAgo, $prettyString)
   {
       $dateTime = clone $this->beforeMidnight;
       $dateTime->modify($timeAgo);
       $this->assertEquals($prettyString, PrettyDateTime::parse($dateTime, $this->beforeMidnight));
   }
   ```
4. Run `phpunit` — all tests pass.

**Result:** 4 new data-driven assertions covering 2–5 weeks in the past.

## Common Issues

- **`Call to undefined method PHPUnit_Framework_TestCase::...`**: You used a PHPUnit 8+ method (`expectException`, etc.) not available in the version pinned in `composer.json`. Check `composer.json` for the PHPUnit version constraint and use only compatible assertions.
- **Off-by-one on boundary (e.g. `'2 months ago'` returns `'1 month ago'`)**:  Month-boundary offsets sit right on the threshold — add `PrettyDateTime::HOUR` to the seconds offset (see line 74): `$seconds = PrettyDateTime::MONTH * $i + PrettyDateTime::HOUR;`.
- **`Fatal error: Class 'PrettyDateTime\PrettyDateTime' not found`**: The `require` at the top of the test file is missing or path is wrong. Verify line 3: `require __DIR__ . '/../src/PrettyDateTime.php';`.
- **Test mutates fixture, later tests fail**: You called `$this->beforeMidnight->modify(...)` directly. Always `clone` first: `$dateTime = clone $this->beforeMidnight;`.
- **Singular/plural string mismatch (`'1 minutes ago'` vs `'1 minute ago'`)**:  `prettyFormat()` only appends `s` when `floor(abs($difference)) > 1`. Singular baselines must use `'1 unit ago'` / `'In 1 unit'`, never `'1 units ago'`.
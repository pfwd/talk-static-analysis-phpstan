---
theme: uncover
class:
  - lead
  - invert
size: 16:9
footer: "Peter Fisher BSc MBCS [howtocodewell.net](https://howtocodewell.net) [@howToCodeWell](https://twitter.com/howtocodewell) [@pfwd](https://twitter.com/pfwd)"
---

# Code with confidence using PHPStan

---

1. What does code confidence mean to me
2. What is static analysis
3. How do we install/run/configure PHPStan
4. How to increase code confidence using PHPStan

---

# $ whoami = Peter Fisher

- PHP Contractor from the UK
- Host of the How To Code Well 
  -- Podcast [howtocodewell.fm](https://howtocodewell.fm)
  -- YouTube channel [youtube.com/howtocodewell](https://youtube.com/howtocdewell)
  -- Twitch live coders team [howtocodewell.net/live](https://howtocodewell.net/live)
  -- Tutorials and courses [howtocodewell.net](https://howtocodewell.net)

---

# Get the slides

[https://github.com/pfwd/talk-static-analysis-phpstan](https://github.com/pfwd/talk-static-analysis-phpstan)

---

# There are two types of projects that every programmer deals with during their career

---

# New projects

<!--
- Start from scratch, no code (yet)
- No architectural decisions have been made (yet)
- No frameworks or library’s chosen (yet)
- No bugs (yet)
- No end users (yet)
- Shopping list of new requirements
-->

---

# Legacy projects

<!--
- Could have a spaghetti code base
- Could have Mixture of frameworks and library's
- Could be out of date code
- Could be using an older version of PHP
- Incoming change requests
- Could have no tests
- Lots of known bugs
- Lots of unknown bugs
- Lots of end users. Some are complaining
- Performance issues
- Security concerns
- Could have a low confidence that an upgrade will work
-->

---

# The dream

---

# New projects

Start clean, continue clean whilst building up confidence with the code

<!--
- Be aware of known issues before deployment
- Gain visual feedback on what code needs to be fixed
- Spot potential gotchas in the new architecture
- Create a CI that reports errors
- Ensure the team follows the same rules
-->
---

# Legacy projects

Quickly identify issues whilst building up confidence with the code

<!--
- Be aware of known issues before deployment
- Clean up code smells
- Enforce coding standards and rules in the CI
- Have confidence with the codebase
- Standardize the codebase
-->

---

# Code confidence

The dreaded 3am phone call on Saturday after the Friday production deployment 
VS 
Knowing your code gets checked before it touches production

---

# How do we get there

---

# Add Static Analysis to your toolbox

<!--
- Static Analysis is just one of many tools
- PHP mess detection
- PHP code sniffers
- PHP unit tests
-->

---

# Static Analysis: What is it?

---

# From Wikipedia

"Static program analysis is the analysis of computer software performed without executing any programs,

in contrast with dynamic analysis,

which is performed on programs during their execution"

---

# I still understand?

Static analysis will search code for non coding compliance without the need for code execution.

---

# I still don't understand?

- It tests the code without running it = SPEED
- It compares the code against a given set of rules
- It tells you which file and line doesn't conform to the given rules

---

# Why you should care

<!--
- Catch errors before they happen
- It's fast
- Consistently clean code base
- Happy devs
-->

---

## PHPStan has entered the chat

- [phpstan.org](https://phpstan.org/)
- [Ondřej Mirtes](https://github.com/ondrejmirtes)
- [@OndrejMirtes](https://twitter.com/OndrejMirtes)

---

## PHPStan

- Has an online editor
- Is free and open source
- Has pro paid features

---

## Pro features

- Web UI
- Watch mode
- Interactive fixer mode

Checkout [phpstan.org](https://phpstan.org/) for more details

---

# How to install

```bash
$ composer require --dev phpstan/phpstan
```

<!--
- PHPStan should be a dev dependency.  Don't install it in production
-->
---

# Your first run

```bash
$ ./vendor/bin/phpstan analyse src
```

<!--
- You can provide multiple folders or files to analyse in the command
-->

---

# When things go well

```bash
root@768e64cf6e00:/var/www/html# ./vendor/bin/phpstan analyse src

 293/293 [▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓] 100%

 [OK] No errors
```

---

# Catching errors

```bash
root@768e64cf6e00:/var/www/html# ./vendor/bin/phpstan analyse src
 293/293 [▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓] 100%

 ------ -------------------------------------------------------
  Line   Downloader/CodeDownloader.php
 ------ -------------------------------------------------------
  84     Method App\Downloader\CodeDownloader::getFilename() 
         should return string but returns string|null.
 ------ -------------------------------------------------------


 [ERROR] Found 1 error
```

---

# The fix

```php
public function getFilename(): string
{
  return $this->course?->getCode()?->getFileName();
}
```

```php
public function getFilename(): ?string
{
  return $this->course?->getCode()?->getFileName();
}
```

<!--
- Updating the return type to allow for nullable values (:?string)
-->

---

# Run levels

- There are 10 run levels (0-9) that change the strictness of the checks.
- Level 0 is used by default.
- Running level 5 will run all the levels from 0-5

<!--
Level 0
- Basic checks
- unknown classes
- unknown functions
- unknown methods called on $this
- wrong number of arguments passed to those methods and functions
- always undefined variables

Level 1
- possibly undefined variables
- unknown magic methods and properties on classes with __call and __get

Level 2
- unknown methods checked on all expressions (not just $this)
- validating PHPDocs

Level 3
- Return types
- types assigned to properties

Level 4
- Dead code checking - always false instanceof and other type checks 
- Unreachable code after

Level 5
- Checking types of arguments passed to methods and functions

Level 6 
- Missing typehints

Level 7 
- Report partially wrong union types 

Level 8
- Report calling methods and accessing properties on nullable types

Level 9
- 9 be strict about the mixed type

-->

---

# How to run PHPStan at a given level

```bash
./vendor/bin/phpstan analyse -l 5 src
```

---

# How to ignore code

```php
private $firstName /** @phpstan-ignore-line */

/** @phpstan-ignore-next-line */
private $lastName 
```

<!--
- This would normally error as both $firstName and $lastName have no data types
- You can also use two forward slashes (//)
-->

---

# How to configure

- Neon format (phpstan.neon, phpstan.neon.dist)
- CLI

---
Neon format is similar to YAML

```yaml
parameters:
  level: 6
  paths:
    - src
    - tests
```

---

# Priority order

1. If a config file is supplied via CLI then it will be used
2. Otherwise, if `phpstan.neon` exists then it will be used
3. Otherwise, if `phpstan.neon.dist` exists that it will be used
4. If no config is supplied then defaults will be used

---

# Git

- Put `phpstan.neon.dist` in source control
- Let devs create their own `phpstan.neon`
- Add `phpstan.neon` to `.gitignore`

---

## Including files

```yaml
includes:
  - phpstan.neon.dist
  - phpstan_test.neon.dist
```

---

# Punning paths

```yaml
parameters:
  paths:
    - src
    - tests
```

```bash
./vendor/bin/phpstan analyse src tests
```

---

# Excluding files

```yaml
parameters:
  excludePaths:
    - tests/*/data/*

```

---

# Ignoring errors

```yaml
parameters:
  ignoreErrors:
    - '#Function pcntl_open not found\.#'
    - '#Call to an undefined method Traversable<mixed, mixed>::uasort\(\)#'
```

---

# Lots more config

See [https://phpstan.org/config-reference](https://phpstan.org/config-reference) for more

<!-- Includes config switches to turn certain checks on and off
-->

---

# Increase code confidence

--- 

# Recommendations for any project

PHPCs -> PHPStan -> PHPUnit

```bash
$ make tests
```
```bash
$ composer test
```

<!--
- Add PHPStan to your CI
- Add PHPStan before running any unit tests and after any code sniffing or linting
- Ensure that developers can run all the CI commands including PHPStan locally using one command
- Enforce that no code can be merged into the main branches unless the CI fully passes
- Don't analyse code that you haven't written.  Don't include the vendor
- Don't upgrade your framework or PHP version until you have fixed all PHPStan errors
- After upgrading your framework or PHP version run PHPStan as you may need to adjust your code
- After upgrading PHPStan run PHPStan to check if anything new has been picked up
-->
---

# Recommendations for new projects

<!--
- Start at the highest level
- Only reduce the level if you are 100% sure you cannot fix the issue
- Ignore one off errors in the config instead of opting to lower the level. You will end up missing other checks
- Try to not ignore or exclude any errors
-->

---

# Recommendations for legacy projects

<!--
- Run the highest level once to see what needs fixing 
- Start on the lowest level and fix any issues.  Then incrementally bump up the level
- Each time you bump up the level update the CI to use that level
- Fix errors in a separate branch which doesn't contain other features/fixes
- Make sure you have other tests that support your changes
- Ensure you have installed the correct support packages for your libraries and frameworks
-->

---

# 3 confidence levels for legacy projects

---

# 1) PHPStan is already in use at the highest run level and working well

- High confidence level

---

# 2) PHPStan is not installed

- Very low confidence level

How do you install PHPStan on a legacy project?

<!--
- Get the by in of the team
- Run at the highest level to see what needs fixing
- Run at each level and create a bug per run level
- Attempt to fix a level per sprint
- Use phpstan.neon as a means of testing beyond the baseline
- Put the fixes in a separate branch/pr
- In the PR update the run level in phpstan.neon.dist
-->
---

# 3) PHPStan is installed but using a low run level

- Low confidence level

How do you upgrade PHPStan on a legacy project?
<!--
- Run the next level base level via phpstan.neon
- Mention what needs fixing to the team. They might not be aware that PHPStan has other levels
- Attempt to fix a level per sprint
- Use phpstan.neon as a means of testing beyond the baseline
- Put the fixes in a separate branch/pr
- In the PR update the run level in phpstan.neon.dist
-->
---

# Thank you

[@pfwd](https://twitter.com/pfwd])


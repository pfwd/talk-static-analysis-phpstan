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
- Playing with PHP > 20 years
- Host of the How To Code Well 
  -- Podcast [howtocodewell.fm](https://howtocodewell.fm)
  -- YouTube channel [youtube.com/howtocodewell](https://youtube.com/howtocdewell)
  -- Twitch live coders team [howtocodewell.net/live](https://howtocodewell.net/live)
  -- Tutorials and courses [howtocodewell.net](https://howtocodewell.net)

---

# Get the slides

[https://github.com/pfwd/talk-static-analysis-phpstan](https://github.com/pfwd/talk-static-analysis-phpstan)

Feedback
[https://joind.in/talk/37f80](https://joind.in/talk/37f80)

---

# #1
## What does code confidence mean to me

---

## There are three types of projects that every programmer deals with during their career

---

#  1# New projects

<!--
- Start from scratch, no code (yet)
- No architectural decisions have been made (yet)
- No frameworks or library’s chosen (yet)
- No bugs (yet)
- No end users (yet)
- Shopping list of new requirements
-->

---

# 2# Legacy projects

<!--
- It could be a spaghetti code base
- It could have mixture of frameworks and library's
- The code could be out of date
- It could be using an older version of PHP
- It could have incoming change requests
- It could have poor technical documentation
- There might be 0 tests
- There could be lots of known bugs
- There could be lots of unknown bugs
- There could be many end users. Some might be complaining
- There could be performance, security, and data integrity issues
- You could have a low confidence that an upgrading or improving something will work
-->

---

# 3# Migrations/rebuilds

<!--
- Data integrity could be poor
- Existing user base
- Downtime issues
- There could be many reasons why a migration is needed
- Work arounds
-->

---

# The dream

---

# New projects

Start clean, continue clean whilst building up confidence with the code

<!--
- Be aware of known issues before 1st deployment
- Gain visual feedback on what parts of code needs to be fixed
- Spot potential gotchas in the new architecture
- Create a CI that reports errors quickly
- Ensure the team follows the same rules
- Move quickly whilst building stability
-->
---

# Legacy projects

Quickly identify issues whilst building up confidence with the code

<!--
- Be aware of known issues before deployment
- Clean up existing code smells
- Enforce coding standards and rules in the CI
- Standardize the codebase
- Make the devs happy
-->

---

# Migrated projects

Ensure the migration is smooth with as little disruption as possible

<!--
- End users should only experience positive changes
- Be aware of the risky areas
- Keep data secure
-->

---

# Code confidence

The dreaded 3am phone call/Slack message on Saturday after the Friday production deployment 
VS 
Knowing your code and everyone else's code will get checked before it goes anywhere near production

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

# #2
## What is Static Analysis

---

# From Wikipedia

"Static program analysis is the analysis of computer software performed without executing any programs, in contrast with dynamic analysis, which is performed on programs during their execution"

---

# What does that mean?

- Static analysis will search code for non coding compliance without the need for code execution.
- It compares the code against a given set of rules
- It tells you which file and line doesn't conform to which rule
- It prevents very bad things from happening

---

# Type checking

```php
$var = new StdClass() + 5;
echo $var;

// PHP Warning:  Uncaught TypeError: Unsupported operand types
```

---
# PHP type system is at runtime
- The code needs to run to see the errors
---

# #3
## PHPStan has entered the chat

- [phpstan.org](https://phpstan.org/)
- Is free and open source
- Has pro paid features

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

1. If a config file is supplied via CLI then it will be used (`-c`)
2. Otherwise, if `phpstan.neon` exists then it will be used
3. Otherwise, if `phpstan.neon.dist` exists that it will be used
4. If no config is supplied then defaults will be used

---

# Git

- Put `phpstan.neon.dist` in source control
- Let devs create their own `phpstan.neon`
- Add `phpstan.neon` to `.gitignore`

---

## Including config files

```yaml
includes:
  - phpstan.neon.dist
  - phpstan_test.neon.dist
```

---

# Checking paths

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
```

---

# Lots more config

See [https://phpstan.org/config-reference](https://phpstan.org/config-reference) for more

<!-- Includes config switches to turn certain checks on and off
-->

---

# #4
## How to increase code confidence using PHPStan

--- 

# Recommendations for any project

---
# Test order is important

PHPCs -> PHPStan -> PHPUnit

---

# One command to rule them all
```bash
$ make tests
```
```bash
$ composer test
```
---
# Use a CI
<!--
- Enforce that no code can be merged into the main branches unless the CI fully passes
-->
---
# Only test your code
<!--
- Don't analyse code that you haven't written.  
- Don't include the vendor
- Ignore any code that is automatically generated like migrations
-->
---

# Be careful with upgrades
<!--
- Don't upgrade your framework or PHP version until you have fixed all PHPStan errors
- After upgrading your framework or PHP version run PHPStan as you may need to adjust your code
- After upgrading PHPStan run PHPStan to check if anything new has been picked up
-->
---


# Use other extensions that match your setup
```bash
phpstan/phpstan-doctrine
```
```bash
phpstan/phpstan-symfony
```

---
# Recommendations for new projects

<!--
- Only reduce the level if you are 100% sure you cannot fix the issue
- Ignore one off errors via the config instead of opting to lower the level. You will end up missing other checks
- Try to not ignore or exclude any errors
-->
---
<!--
- Use the the max level. This will keep you at the highest possible level when PHPstan is upgraded
-->
# Run at max level
```bash
./vendor/bin/phpstan analyse -l max src
```

```bash
parameters:
  level: max
  paths:
    - src
```

---

# Get stricter
[https://github.com/phpstan/phpstan-strict-rules](https://github.com/phpstan/phpstan-strict-rules)
```bash
composer require --dev phpstan/phpstan-strict-rules
```
```bash
includes:
    - vendor/phpstan/phpstan-strict-rules/rules.neon
```
OR
[https://github.com/phpstan/extension-installer](https://github.com/phpstan/extension-installer)

---

# Recommendations for legacy projects

<!--
- Fix errors in a separate branch which doesn't contain other features/fixes
- Ensure you have installed the correct support packages for your libraries and frameworks
-->

---

# Run the highest level once

<!--
- This shows you what you are up against
- Don't be afraid of what you might find
- Keep note of how many errors you have
-->

---
# Start small and go gradually

<!--
- Do one level at a time
- Do it gradually
-->

---
# Make sure you have tests to back up your changes

<!--
- Unit tests
- Acceptance tests
- Sometimes if its broken it actually works
-->

---

# 3 Confidence levels for legacy projects

---

# 1) PHPStan is already in use and is running at the highest level and working well

- High confidence level

---
# 2) PHPStan is installed but using a low run level

- Low confidence level

How do you upgrade PHPStan on a legacy project?
<!--
- Run the next base level via phpstan.neon
- Mention what needs fixing to the team. They might not be aware that PHPStan has other levels
- Attempt to fix a level per sprint
- Rinse and repeat
-->
---

# 3) PHPStan is not installed

- Very low confidence level

How do you install PHPStan on a legacy project?

<!--
- Get the by in of the team
- Run at the highest level to see what needs fixing
- Run at each level and create a ticket per run level
- Attempt to fix a level per sprint
- Use phpstan.neon as a means of testing beyond the baseline
- Put the fixes in a separate branch/pr
- In the PR update the run level in phpstan.neon.dist
-->
---

# Generics are Awesome

Loop over an array of products getting the ID of each product

Sounds easy right?

---

# Oh no
```php
$products = [
    new PreOrder(),
    new Subscription(),
    new Product(),
    'SKUABCD',
];
```
---
# A work around
```php
foreach ($products as $product) {
    if (!$product instanceof Product || 
        !$product instanceof Subscription ||  
        !$product instanceof PreOrder || 
        ) 
    {
          continue;
    }

    $id = $product->getId();
    //..
}
```
---
```php
function getProductIds(array $products) {
    foreach ($products as $product) {
        // Is $product actually an instance of Product?
    }
}
```
<!--
- Data integrity
- What type is $product
-->
---
# Messy code
- Checks get out of hand
- Not very readable
- Prone to mistakes
---

```php
/**
 * @param array<int, Product|Subscription|PreOrder|string> $products
 * @return array<int, int>
 */
function getProductIds(array $products): array
{
    $ids = [];
    foreach($products as $product){
      if(is_string($product)){
          continue;
      }
      
      $ids[] = $product->getId()
    }
    return $ids;
}
```
<!-- 
- Documents the requirements of the array
- Autocompletion
- Ideally use an interface if this a common occurrence
-->
---

# When to use annotations or native type hints

- It's up to you!
- Don't double up
- Use native type hints where possible
- Use annotations when you can't use native type hints

---

```php
/**
 * @return array<string, int>
 */
function getItems(): array
{
  return [
    'hello' => 1,
    'world' => 2
  ];
}

```
<!--
Native type hint wont work for the array shape
-->
---

```php

function getName(): string
{
  return 'Peter Fisher'
}

```
<!--
No need to use an annotation
-->
---
# Static Analysis could save you money
If you're relying on Bugsnag or Sentry to catch errors that Static Analysis can catch then you're doing it wrong

<!--
- Bugsnag or Sentry have pricing plans based on the number of events triggered.
- The more compliant your code is the less events will be triggered
-->
---

# Thank you

[@pfwd](https://twitter.com/pfwd])

Please give feedback
![QR Code](../src/assets/images/phpsw_qr_code.png)


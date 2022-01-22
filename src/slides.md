---
theme: uncover
class:
  - lead
  - invert
size: 16:9
footer: "Peter Fisher BSc MBCS [howtocodewell.net](https://howtocodewell.net) [@howToCodeWell](https://twitter.com/howtocodewell) [@pfwd](https://twitter.com/pfwd)"
---

# TALK TITLE GOES HERE

---

# Get the slides
[https://github.com/pfwd/talk-static-analysis-phpstan](https://github.com/pfwd/talk-static-analysis-phpstan)

---

# $ whoami = Peter Fisher

- PHP Contractor from the UK
- Host of the How To Code Well
-- Podcast [howtocodewell.fm](https://howtocodewell.fm)
-- YouTube channel [youtube.com/howtocodewell](https://youtube.com/howtocdewell)
-- Twitch live coders team [howtocodewell.net/live](https://howtocodewell.net/live)
-- Tutorials and courses [howtocodewell.net](https://howtocodewell.net)

---

# There are two types of projects that every programmer deals with during their career

---

# New projects

<!--
- Start from scratch, no code (yet)
- No architectural decisions have been made (yet)
- No frameworks or library’s chosen (yet)
- No bugs (yet)
- Shopping list of new requirements
- No end users (yet)
-->

---

# Legacy projects

<!--
- Massive entangled code base
- Mixture of frameworks and library's
- Out of date code
- Older version of PHP
- Incoming change requests
- No tests
- Lots of known bugs
- Lots of unknown bugs
- Lots of end users. Some are complaining
- Performance issues
- Security concerns
- Low confidence that an upgrade will work
-->

---

# The dream

---

# New projects

Start clean, continue clean and build up confidence with the code

<!--
- Be aware of known issues before deployment
- Gain visual feedback on what code needs to be fixed
- Spot potential gotchas in the new architecture
- Create a CI that reports errors
- Ensure the team follows the same rules
-->
---

# Legacy projects

Quickly identify issues and gain confidence with the code

<!--
- Be aware of known issues before deployment
- Clean up code smells
- Enforce coding standards and rules in the CI
- Have confidence with the codebase
- Standardize the codebase
-->
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

# Why you should care

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
0 basic checks, unknown classes, unknown functions, unknown methods called on $this, wrong number of arguments passed to those methods and functions, always undefined variables

1 possibly undefined variables, unknown magic methods and properties on classes with __call and __get

2 unknown methods checked on all expressions (not just $this), validating PHPDocs

3 return types, types assigned to properties

4 basic dead code checking - always false instanceof and other type checks, dead else branches, unreachable code after return; etc.

5 checking types of arguments passed to methods and functions

6 report missing typehints

7 report partially wrong union types - if you call a method that only exists on some types in a union type, level 7 starts to report that; other possibly incorrect situations

8 report calling methods and accessing properties on nullable types

9 be strict about the mixed type - the only allowed operation you can do with it is to pass it to another mixed
-->

---
# How to run PHPStan at a given level

```bash
./vendor/bin/phpstan analyse -l 5 src
```

---
# How to configure

---


# Recommended usage

--- 

# Recommendations for new projects

<!--
- Start at the highest level
- Only reduce the level if you are 100% sure you cannot fix the issue
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

# Recommendations for any projects
<!--
- Add PHPStan to your CI
- Add PHPStan before running any unit tests and after any code sniffing or linting
- Ensure that developers can run all the CI commands including PHPStan locally using one command
- Enforce that no code can be merged into the main branches unless the CI fully passes
- Don't analyse code that you haven't written.  Don't include the vendor
- Don't upgrade your framework or PHP version until you have fixed all PHPStan errors
- After upgrading your framework or PHP version run PHPStan as you may need to adjust your code
- After upgrading PHPstan run PHPstan to check if anything new has been picked up
-->
---

# Other Static Analysis tools


# TYPO3 with composer tricks

## Disable TYPO3 composer plugin temporarily

So you have a TYPO3 Extension project. The project (naturally) depends on the composer package
`typo3/cms` in some version. If you now use `composer install` or `composer update`, the
TYPO3 composer plugin kicks in and creates a nice full-blown project for you. - Because
that's what you want, right?

```
├── Classes
├── composer.json
├── composer.lock
├── Configuration
├── ext_...
├── index.php -> typo3_src/index.php
├── phpunit.xml.dist
├── Resources
├── Tests
# WTF!?
├── typo3 -> typo3_src/typo3
├── typo3conf
├── typo3_src
└── vendor
    └── typo3
```

**No.**

TYPO3 CMS (or better the composer plugin that moves composer packages around; to typo3conf/ext 
instead of `vendor` for example), for some obscure reason doesn't distinguish between libraries
and distributions. That's why you always get a full blown cms project. Neat.

You can disable the behaviour by using the `no-plugins` flag.

```
composer install --no-plugins
```

### Why using `typo3/cms-core` as dependency is no solution to the underlying problem

The [composer wiki page](https://wiki.typo3.org/Composer) suggests using `typo3/cms-core`
as dependency to declare what version of the TYPO3 core to use.

This doesnt solve the underlying problem: Using `typo3/cms-core` as a dependency will fail
in any case where `typo3/cms` is not part of the current composer dependency workspace.
This is because `typo3/cms-core` is provided by `typo3/cms` via the composer `replace`
mechanism.

You are still required to depend on `typo3/cms` and therefore to use the `--no-plugins` flag
if you need to install or update composer dependencies.

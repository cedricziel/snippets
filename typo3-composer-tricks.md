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

TYPO3 CMS (or better the composer plugins that moves composer packages around; to typo3conf/ext 
instead of `vendor` for example), for some obscure reason doesn't distinguish between libraries
and distributions. That's why you always get a full blown cms project. Neat.

You can disable the behaviour by using the `no-plugins` flag.

```
composer install --no-plugins
```

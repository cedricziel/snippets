# composer

## Setting config at runtime

Synopsis: `composer config $key value` (https://getcomposer.org/doc/03-cli.md#modifying-extra-values)

So you may want to configure a local cache directory because `.gitlab-ci.yml` can only cache build-dir local directories?

Easy: `composer config cache-files-dir .composercache`

Add the following to your `.gitlab-ci.yml`:

```
image: cedricziel/docker-php-tools:7.0

before_script:
  - composer config cache-files-dir .composercache

cache:
  paths:
    - .composercache
```

PS: Don't forget to exclude the given directory in your `.gitignore` file

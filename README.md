# composer-p3-combined-probe

## Feature exercised

Three P3 Composer feature-coverage patterns combined in a single `composer.json`: `metapackage`, `multiple-version-constraints`, and `stability-flag`.

## Patterns

### 1. metapackage (`sentry/sdk: ^4.0`)

`sentry/sdk` 4.0.0 is a genuine `type: metapackage` on Packagist. It has no installable code — its only content is `require: {sentry/sentry: ^4.0}`. Mend must include the `sentry/sdk` node itself in the dependency tree AND resolve its child `sentry/sentry` as a transitive dependency.

### 2. multiple-version-constraints (`monolog/monolog: "^2.0 || ^3.0"`)

The OR constraint tells Composer to accept either the 2.x or 3.x line. With PHP 8.3 (bundled in `composer:2.8`), Composer resolves to a single version: `3.10.0`. Mend must report exactly one entry for `monolog/monolog`, not two separate entries for the two constraint ranges.

### 3. stability-flag (`nesbot/carbon: "^2.0@beta"`)

`minimum-stability` is `stable` and `prefer-stable` is `true` at root level. The `@beta` per-package flag (recorded as `stability-flags: {"nesbot/carbon": 10}` in the lockfile) allows Composer to install a beta release of `nesbot/carbon` if no stable release satisfies `^2.0`. In practice, Composer resolved to the stable `2.73.0`. Mend must not drop or duplicate this package due to the `@beta` suffix in the constraint string.

## Expected dependency tree

### Direct dependencies (3)

| Package | Version | Type | Source |
|---|---|---|---|
| `sentry/sdk` | 4.0.0 | metapackage | registry |
| `monolog/monolog` | 3.10.0 | library | registry |
| `nesbot/carbon` | 2.73.0 | library | registry |

### All packages (18 total, all group=main, no require-dev)

| Package | Version | Direct | Parent(s) |
|---|---|---|---|
| `sentry/sdk` | 4.0.0 | yes | root |
| `sentry/sentry` | 4.25.0 | no | sentry/sdk |
| `guzzlehttp/psr7` | 2.9.0 | no | sentry/sentry |
| `psr/http-factory` | 1.1.0 | no | guzzlehttp/psr7 |
| `psr/http-message` | 2.0 | no | guzzlehttp/psr7 |
| `ralouphie/getallheaders` | 3.0.3 | no | guzzlehttp/psr7 |
| `jean85/pretty-package-versions` | 2.1.1 | no | sentry/sentry |
| `psr/log` | 3.0.2 | no | sentry/sentry, monolog/monolog |
| `symfony/options-resolver` | v8.0.8 | no | sentry/sentry |
| `symfony/deprecation-contracts` | v3.6.0 | no | symfony/options-resolver, symfony/translation |
| `monolog/monolog` | 3.10.0 | yes | root |
| `nesbot/carbon` | 2.73.0 | yes | root |
| `carbonphp/carbon-doctrine-types` | 3.2.0 | no | nesbot/carbon |
| `psr/clock` | 1.0.0 | no | nesbot/carbon |
| `symfony/polyfill-mbstring` | v1.37.0 | no | nesbot/carbon, symfony/translation |
| `symfony/polyfill-php80` | v1.37.0 | no | nesbot/carbon |
| `symfony/translation` | v6.4.34 | no | nesbot/carbon |
| `symfony/translation-contracts` | v3.6.1 | no | symfony/translation |

Platform packages (`php`, `ext-json`, `ext-curl`, `ext-mbstring`, `ext-iconv`) must NOT appear in the tree.

## Probe metadata

| Field | Value |
|---|---|
| probe name | composer-p3-combined-probe |
| patterns | metapackage, multiple-version-constraints, stability-flag |
| composer version | 2.8 |
| php version | 8.3 (bundled in composer:2.8) |
| lockfile generated | yes (via `docker run --rm composer:2.8 composer install`) |
| total packages | 18 |
| direct packages | 3 |
| transitive packages | 15 |
| require-dev packages | 0 |
# Reusable Workflows

Reusable GitHub Actions workflows for Laravel application CI. Five workflows: Pest tests, Pest type coverage, Pint styling, Larastan static analysis, and Dependabot auto-merge. Callers are tiny one-file wrappers, so every application you maintain updates in one place.

## Table of Contents

- [Getting Started](#getting-started)
- [Self-Hosted Runners](#self-hosted-runners)
- [Tests](#tests)
- [Type Coverage](#type-coverage)
- [Code Styling](#code-styling)
- [Code Analysis](#code-analysis)
- [Auto Merge](#auto-merge)
- [Support This Project](#support-this-project)
- [Contributing & Security](#contributing--security)
- [License](#license)

## Getting Started

Each workflow lives in this repo. You call it from your own repo with a small wrapper file under `.github/workflows/`. The wrapper passes inputs and secrets; the reusable workflow runs the actual job.

If your `composer.json` pulls private Git packages, create a `KEYMASTER_TOKEN` secret on the consuming repo with a PAT that has `repo` scope. You can rename the secret in your wrapper; the examples below use `KEYMASTER_TOKEN` to match my own setup.

### Shared defaults

The test, type coverage, code analysis, and code styling workflows all share a few opinionated defaults worth knowing about:

- **WIP commits skip CI.** If the commit message contains `wip`, the job exits before doing any work. Good for checkpoint commits you don't want burning CI minutes.
- **New commits cancel old runs.** A `concurrency` group keyed on workflow name and ref cancels any in-progress run when a new push arrives. This applies to pushes and pull requests equally, so rapid-fire commits only execute the final run. Branches are isolated from each other (different ref, different group).
- **Least-privilege permissions.** Tests, type coverage, and code analysis request `contents: read`. Code styling and auto merge request `contents: write` because they push commits or merge PRs.

## Self-Hosted Runners

Every workflow accepts a `runner` input. The default is `ubuntu-latest`; pass your own label to run on your own hardware:

```yml
with:
  runner: 'my-runner-label'
```

### One runner per machine: nothing to do

A single runner instance behaves like a GitHub-hosted machine, minus the teardown. The workflows already handle the usual self-hosted quirks: `COMPOSER_ALLOW_SUPERUSER` is set for runners that run as root, and the database bootstrap is idempotent, so leftover databases from earlier runs don't break the next one.

### Several runner instances on one machine: isolate them

`shivammathur/setup-php` downloads Composer onto `/usr/local/bin/composer` on every job, installs global tools like Pint into `$HOME/.composer`, and symlinks into the tool cache. Those are all machine-wide paths. Run two or more runner instances on the same host and concurrent jobs overwrite each other mid-run. The failures look random and never point at the real cause:

- `PHP Fatal error: Class "..." not found in phar:///usr/local/bin/composer/...` (one job rewrote the phar while another was executing it)
- `/usr/bin/env: bad interpreter: Text file busy`
- `Unable to find Composer at '.../setup-php/tools/composer'`

The fix is three environment variables in each runner instance's `.env` file (the file next to `run.sh` in the runner directory, injected into every job). setup-php honors `SETUP_PHP_TOOLS_DIR` and `SETUP_PHP_TOOL_CACHE_DIR`, and a per-instance `HOME` separates the Composer home, npm cache, and Playwright browsers:

```
HOME=/opt/runner-homes/runner-1
SETUP_PHP_TOOLS_DIR=/opt/runner-homes/runner-1/tools
SETUP_PHP_TOOL_CACHE_DIR=/opt/runner-homes/runner-1/tool-cache
```

Give each instance its own directory, create the directories up front, and restart the runner services. Disk cost is small. Composer and Pint are a few megabytes per instance, and the per-instance caches warm up after one run each.

Two shared paths remain: the system PHP install (setup-php's per-job ini tweaks are identical and idempotent) and the Node tool cache, which is read-only once a version is cached. When you bump the PHP or Node version, let one job run alone first so the install isn't racing its siblings.

## Tests

Runs [Pest](https://pestphp.com) with Node install + Vite build baked in, plus optional parallel sharding, Playwright browsers, a separate Python test job, FFmpeg, extra apt packages, Redis, and MySQL or PostgreSQL with one or more databases.

### Path

```
.github/workflows/tests.yml
```

### Example caller

```yml
name: Tests

on:
  push:
    paths:
      - '**.php'
      - 'composer.json'
      - 'composer.lock'
      - 'phpunit.xml'
      - 'phpunit.ci.xml'
      - '.github/workflows/tests.yml'
  workflow_dispatch:

jobs:
  tests:
    name: Tests
    uses: mozex/reusable-workflows/.github/workflows/tests.yml@main
    secrets:
      TOKEN: ${{ secrets.KEYMASTER_TOKEN }}
    with:
      php_version: '8.3'
      phpunit_config_file: 'phpunit.xml'
      node_package_manager: 'yarn'
```

### Inputs

| Input | Type | Default | Purpose |
|---|---|---|---|
| `runner` | string | `'ubuntu-latest'` | Runner label: a GitHub-hosted label or your own. See [Self-Hosted Runners](#self-hosted-runners). |
| `php_version` | string | `'8.3'` | PHP version for `shivammathur/setup-php`. |
| `php_extensions` | string | See below | Comma-separated extensions. Trim the list to cut setup time (drop `imagick` and `soap` if you don't use them). |
| `phpunit_config_file` | string | `'phpunit.xml'` | Path to the PHPUnit config. |
| `custom_commands` | string | `''` | Shell commands that run before Composer install. |
| `before_test_commands` | string | `''` | Shell commands that run right before Pest starts, after all other setup. Good spot for `php artisan migrate --force` or seeding. |
| `database_driver` | string | `'mysql'` | `mysql` or `pgsql` starts that service and creates databases. `sqlite` skips DB setup entirely (point your `.env` at sqlite). `none` also skips DB setup. |
| `databases` | string | `'testing'` | Comma-separated database names. Only used when `database_driver` is `mysql` or `pgsql`. |
| `reset_postgres_password` | boolean | `true` | Reset the postgres superuser password to `root` before creating databases. Needed on ephemeral GitHub-hosted runners; set `false` on a persistent self-hosted box whose postgres auth is already configured. |
| `enable_redis` | boolean | `false` | Start the runner's pre-installed Redis server. |
| `enable_sharding` | boolean | `false` | Run Pest across a matrix of parallel shards. |
| `shard_count` | number | `4` | Number of shards when sharding is on. Any positive integer works. |
| `install_node_dependencies` | boolean | `true` | Run `yarn install` or `npm ci`. Set to `false` for rare apps without a `package.json`. |
| `build_node_assets` | boolean | `true` | Run `yarn build` or `npm run build`. Set to `false` for apps without a frontend build step. |
| `node_version` | string | `'24.15'` | Node.js version. |
| `node_package_manager` | string | `'yarn'` | `yarn` or `npm`. |
| `apt_packages` | string | `''` | Space-separated apt packages installed via `awalsh128/cache-apt-pkgs-action`. |
| `install_ffmpeg` | boolean | `false` | Install FFmpeg and FFprobe. |
| `install_playwright` | boolean | `false` | Install Playwright Chromium plus OS deps, with cache. |
| `python_tests` | boolean | `false` | Run a separate pytest job via `php artisan python:test`. |
| `python_version` | string | `'3.12'` | Python version for the pytest job. |
| `timeout_minutes` | number | `30` | Job timeout in minutes. |

Default `php_extensions`: `dom, curl, libxml, mbstring, zip, pcntl, pdo, sqlite, pdo_sqlite, bcmath, soap, intl, gd, exif, iconv, imagick, fileinfo`.

Required secret: `TOKEN` if `composer.json` pulls private packages.

### Behavior notes

- **Node install and Vite build run by default.** The workflow is designed for Laravel applications, which all have a `package.json` and a frontend build. If you have an edge-case app that doesn't need one of them, set `install_node_dependencies: false` or `build_node_assets: false`.
- **Playwright browsers** are cached by version at `~/.cache/ms-playwright`. Cache hit: only OS deps install (apt packages aren't cacheable). Cache miss: `playwright install chromium --with-deps`.
- **Sharding** uses Pest's `--shard=X/Y` flag. A small `setup` job generates the shard list before the test jobs fan out, so you can pick any count: 2, 3, 8, 17, whatever.

### More examples

Parallel run across 8 shards:

```yml
with:
  enable_sharding: true
  shard_count: 8
```

Two MySQL databases (primary and secondary):

```yml
with:
  databases: 'testing,secondary'
```

PostgreSQL with Redis:

```yml
with:
  database_driver: 'pgsql'
  enable_redis: true
```

SQLite in memory instead of MySQL. Set `DB_CONNECTION=sqlite` and `DB_DATABASE=:memory:` in your `.env.example`:

```yml
with:
  database_driver: 'sqlite'
```

Leaner PHP extensions for faster setup:

```yml
with:
  php_extensions: 'dom, curl, libxml, mbstring, zip, pcntl, bcmath, intl, fileinfo, pdo_sqlite'
```

Skip the frontend build (e.g. API-only Laravel app):

```yml
with:
  build_node_assets: false
```

Skip Node entirely (rare; app without a `package.json`):

```yml
with:
  install_node_dependencies: false
  build_node_assets: false
```

Playwright browser tests:

```yml
with:
  install_playwright: true
```

Run migrations and seeders right before tests:

```yml
with:
  before_test_commands: |
    php artisan migrate --force
    php artisan db:seed --class=TestSeeder --force
```

Enable the Python test job alongside Pest:

```yml
with:
  python_tests: true
  python_version: '3.12'
```

## Type Coverage

Runs [Pest's type coverage](https://pestphp.com/docs/type-coverage) check with an adjustable minimum threshold.

### Path

```
.github/workflows/type-coverage.yml
```

### Example caller

```yml
name: Type Coverage

on:
  push:
    paths:
      - '**.php'
      - 'composer.json'
      - 'composer.lock'
      - 'phpunit.xml'
      - 'phpunit.ci.xml'
      - '.github/workflows/type-coverage.yml'
  workflow_dispatch:

jobs:
  type-coverage:
    name: Type Coverage
    uses: mozex/reusable-workflows/.github/workflows/type-coverage.yml@main
    secrets:
      TOKEN: ${{ secrets.KEYMASTER_TOKEN }}
    with:
      php_version: '8.3'
      phpunit_config_file: 'phpunit.xml'
      min: 100
```

### Inputs

| Input | Type | Default | Purpose |
|---|---|---|---|
| `runner` | string | `'ubuntu-latest'` | Runner label: a GitHub-hosted label or your own. See [Self-Hosted Runners](#self-hosted-runners). |
| `php_version` | string | `'8.3'` | PHP version. |
| `phpunit_config_file` | string | `'phpunit.xml'` | PHPUnit config. If it's not `phpunit.xml`, it's copied to that name before the run. |
| `min` | number | `100` | Minimum type coverage percentage. The job fails below this. |
| `custom_commands` | string | `''` | Shell commands that run before Composer install. |
| `enable_sharding` | boolean | `false` | Split the coverage run across parallel shards. |
| `shard_count` | number | `4` | Shard count when sharding is on. |
| `timeout_minutes` | number | `10` | Job timeout in minutes. |

### More examples

Lower the threshold to 90%:

```yml
with:
  min: 90
```

Split across 4 shards:

```yml
with:
  enable_sharding: true
  shard_count: 4
```

## Code Styling

Runs [Laravel Pint](https://github.com/laravel/pint) and commits style fixes back to the branch. With `test_mode: true`, it only reports violations without committing, which is what you want on PRs from forks.

### Path

```
.github/workflows/code-styling.yml
```

### Example caller

```yml
name: Code Styling

on:
  push:
    paths:
      - '**.php'
      - 'pint.json'
      - '.github/workflows/code-styling.yml'
  workflow_dispatch:

jobs:
  code-styling:
    name: Code Styling
    uses: mozex/reusable-workflows/.github/workflows/code-styling.yml@main
    with:
      php_version: '8.3'
      message: 'Styling'
      test_mode: false
```

### Inputs

| Input | Type | Default | Purpose |
|---|---|---|---|
| `runner` | string | `'ubuntu-latest'` | Runner label: a GitHub-hosted label or your own. See [Self-Hosted Runners](#self-hosted-runners). |
| `php_version` | string | `'8.3'` | PHP version. |
| `message` | string | `'Styling'` | Commit message used when pushing auto-fixed changes. |
| `test_mode` | boolean | `false` | Runs `pint --test` without committing. Use for read-only checks on fork PRs. |
| `timeout_minutes` | number | `5` | Job timeout in minutes. |

## Code Analysis

Runs [Larastan](https://github.com/larastan/larastan) (PHPStan with Laravel rules) with `--memory-limit=-1` and the GitHub error format, so issues show up inline in the PR diff.

### Path

```
.github/workflows/code-analysis.yml
```

### Example caller

```yml
name: Code Analysis

on:
  push:
    paths:
      - '**.php'
      - 'phpstan.neon.dist'
      - '.github/workflows/code-analysis.yml'
  workflow_dispatch:

jobs:
  code-analysis:
    name: Code Analysis
    uses: mozex/reusable-workflows/.github/workflows/code-analysis.yml@main
    secrets:
      TOKEN: ${{ secrets.KEYMASTER_TOKEN }}
    with:
      php_version: '8.3'
      larastan_config_file: 'phpstan.neon.dist'
```

### Inputs

| Input | Type | Default | Purpose |
|---|---|---|---|
| `runner` | string | `'ubuntu-latest'` | Runner label: a GitHub-hosted label or your own. See [Self-Hosted Runners](#self-hosted-runners). |
| `php_version` | string | `'8.3'` | PHP version. |
| `larastan_config_file` | string | `'phpstan.neon.dist'` | Path to the PHPStan config. |
| `custom_commands` | string | `''` | Shell commands that run before Composer install. |
| `timeout_minutes` | number | `5` | Job timeout in minutes. |

Required secret: `TOKEN` if `composer.json` pulls private packages.

## Auto Merge

Automatically merges Dependabot pull requests for minor and patch version bumps once all required checks pass. Major updates stay manual so you can review breaking changes.

### Path

```
.github/workflows/auto-merge.yml
```

### Example caller

```yml
name: Auto Merge

on: pull_request_target

jobs:
  auto-merge:
    name: Auto Merge
    uses: mozex/reusable-workflows/.github/workflows/auto-merge.yml@main
    secrets:
      TOKEN: ${{ secrets.KEYMASTER_TOKEN }}
```

### Requirements

- A `TOKEN` secret holding a PAT (or GitHub App token) with `repo` and `pull-request` write access. The default `GITHUB_TOKEN` can't enable auto-merge on a PR opened by Dependabot.
- Auto-merge switched on under Settings → General → Pull Requests for the repo's default branch.

### Behavior notes

- The job only runs when the PR actor is `dependabot[bot]`.
- It uses `gh pr merge --auto --merge`, so the PR waits for required checks before the merge fires.
- Major version bumps from Dependabot are intentionally skipped. They need human review.

## Support This Project

I maintain this package along with [several other open-source PHP packages](https://mozex.dev/docs) used by thousands of developers every day.

If my packages save you time or help your business, consider [**sponsoring my work on GitHub Sponsors**](https://github.com/sponsors/mozex). Your support lets me keep these packages updated, respond to issues quickly, and ship new features.

Business sponsors get logo placement in package READMEs. [**See sponsorship tiers →**](https://github.com/sponsors/mozex)

## Contributing & Security

- Development setup and PR guidelines: [CONTRIBUTING.md](CONTRIBUTING.md)
- Reporting security issues: [security policy](../../security/policy)

## License

The MIT License (MIT). See [LICENSE.md](LICENSE.md).

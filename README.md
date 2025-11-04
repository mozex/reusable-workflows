# Reusable Workflows
A collection of reusable GitHub Actions workflows I use in my public and private projects.

- [Tests](#tests)
- [Type Coverage](#type-coverage)
- [Code Styling](#code-styling)
- [Code Analysis](#code-analysis)
- [Auto Merge](#auto-merge)
- [Support Us](#support-us)
- [Contributing](#contributing)
- [Security Vulnerabilities](#security-vulnerabilities)
- [Credits](#credits)
- [License](#license)

## Tests

Workflow to run Laravel tests using [Pest](https://pestphp.com).

### Path

```
.github/workflows/tests.yml
```

### Code

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
- php_version (string, default: '8.3') – PHP version to use
- phpunit_config_file (string, default: 'phpunit.xml') – PHPUnit config file path
- custom_commands (string, default: '') – Optional shell commands to run before installing deps (e.g., set up services)
- databases (string, default: 'testing') – Comma-separated DB names to create in MySQL (e.g., "testing,secondary")
- enable_sharding (boolean, default: false) – Run tests across a matrix of shards for speed
- shard_count (number, default: 4) – Number of shards to split the suite into (used only when sharding is enabled)
- node_package_manager (string, default: 'yarn') – 'yarn' or 'npm' for JS install/build steps
- secrets.TOKEN – Optional GitHub token for Composer installs (private packages)

### Examples

Enable sharding (8 shards)
```yml
with:
  enable_sharding: true
  shard_count: 8
```

Multiple databases
```yml
with:
  databases: 'testing,secondary'
```

Use npm instead of Yarn
```yml
with:
  node_package_manager: 'npm'
```

Run custom commands before dependencies
```yml
with:
  custom_commands: |
    php -v
    node -v
```

## Type Coverage

Workflow to run Type Coverage using [Pest](https://pestphp.com/docs/type-coverage).

### Path

```
.github/workflows/type-coverage.yml
```

### Code

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
- php_version (string, default: '8.3')
- phpunit_config_file (string, default: 'phpunit.xml') – PHPUnit config to use
- min (number, default: 100) – Minimum type coverage threshold
- custom_commands (string, default: '') – Optional pre-steps
- enable_sharding (boolean, default: false) – Enable sharded runs
- shard_count (number, default: 4) – Shard count when sharding is enabled
- secrets.TOKEN – Optional GitHub token for Composer installs

### Examples

Enable sharding (4 shards)
```yml
with:
  enable_sharding: true
  shard_count: 4
```

Lower threshold to 90
```yml
with:
  min: 90
```

## Code Styling

Workflow to run [Laravel Pint](https://github.com/laravel/pint) and automatically fix code style violations. Changes are
pushed back to the GitHub repository.

### Path
```
.github/workflows/code-styling.yml
```

### Code
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
      test_mode: true
```

### Inputs
- php_version (string, default: '8.3')
- message (string, default: 'Styling') – Commit message for auto-commit
- test_mode (boolean, default: false) – If true, runs pint in test mode without committing changes

## Code Analysis

Workflow to run [Larastan](https://github.com/larastan/larastan) in projects.

### Path
```
.github/workflows/code-analysis.yml
```

### Code
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
      larastan_config_file: 'phpstan.php'
```

### Inputs
- php_version (string, default: '8.3')
- larastan_config_file (string, default: 'phpstan.neon.dist') – Larastan/PHPStan config file
- custom_commands (string, default: '') – Optional shell commands before install/analysis
- secrets.TOKEN – Optional GitHub token for Composer installs

### Example
Use a custom config and a pre-step
```yml
with:
  larastan_config_file: 'phpstan.php'
  custom_commands: 'echo Preparing analysis'
```

## Auto Merge

Workflow that can automatically merge Dependabot pull requests after the CI builds has run successfully.

### Path

```
.github/workflows/auto-merge.yml
```

### Code

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

### Notes
- Requires a token with repo permissions in secrets.KEYMASTER_TOKEN
- Automatically merges Dependabot PRs that are minor or patch updates after checks pass

## Support Us

Creating and maintaining open-source projects requires significant time and effort. Your support will help enhance the
project and enable further contributions to the Laravel community.

Sponsorship can be made through the [GitHub Sponsors](https://github.com/sponsors/mozex) program. Just click the "**[Sponsor](https://github.com/sponsors/mozex)**" button at the top of this repository. Any amount is greatly appreciated, even a contribution as small as $1 can make a big difference and will go directly towards developing and improving this package.

Thank you for considering sponsoring. Your support truly makes a difference!

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## Security Vulnerabilities

Please review [our security policy](../../security/policy) on how to report security vulnerabilities.

## Credits

- [Mozex](https://github.com/mozex)
- [All Contributors](../../contributors)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
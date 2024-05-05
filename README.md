# Reusable Workflows
A collection of reusable GitHub Actions workflows I use in my public and private projects.

- [Tests](#tests)
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
      test_mode: true
```

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
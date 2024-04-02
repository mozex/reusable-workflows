# Reusable Workflows
A collection of reusable GitHub Actions workflows I use in my public and private projects.

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
  coce-styling:
    name: Code Styling
    uses: mozex/reusable-workflows/.github/workflows/code-styling.yml@main
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
# Reusable Workflows
A collection of reusable GitHub Actions workflows I use in my public and private projects.

## `.github/workflows/code-styling.yml`

Workflow to run [Laravel Pint](https://github.com/laravel/pint) and automatically fix code style violations. Changes are pushed back to the GitHub repository.

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

## `.github/workflows/code-analysis.yml`

Workflow to run [Larastan](https://github.com/larastan/larastan) in projects.

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
      GITHUB_TOKEN: ${{ secrets.KEYMASTER_TOKEN }}
    with:
      php_version: '8.3'
      larastan_config_file: 'phpstan.php'
```
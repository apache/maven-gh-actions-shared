# Apache Maven shared GitHub Actions


# Usage

Create GitHub workflow in project with content:

```yaml
name: Verify

on:
  push:
    branches-ignore:
      - dependabot/**
  pull_request:

jobs:
  build:
    name: Verify
    uses: apache/maven-gh-actions-shared/.github/workflows/maven-verify-with-its.yml@main

```

Excludes from build matrix:

```yaml
...
    uses: apache/maven-gh-actions-shared/.github/workflows/maven-verify-with-its.yml@main
    with:
      matrix-exclude: >
        [ 
          {"jdk": "8"},   # exclude jdk 8 from all builds
          {"os": "windows-latest"}, # exclude windows from all builds
          {"jdk": "8", "os": "windows-latest"} # exclude jkd 8 on windows
        ]
```

# Resources

- [Workflow syntax](https://docs.github.com/en/actions/learn-github-actions/workflow-syntax-for-github-actions)
- [Reusing workflows](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows)

<!---
 Licensed to the Apache Software Foundation (ASF) under one or more
 contributor license agreements.  See the NOTICE file distributed with
 this work for additional information regarding copyright ownership.
 The ASF licenses this file to You under the Apache License, Version 2.0
 (the "License"); you may not use this file except in compliance with
 the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
-->
# Apache Maven Shared GitHub Actions

# Usage

https://github.com/apache/maven-gh-actions-shared/

## Actionlint

Lint a repository's own GitHub Actions workflows with
[actionlint](https://github.com/rhysd/actionlint) — workflow syntax, expression
contexts, and embedded shell (via shellcheck). It runs through
`github/super-linter` restricted to the GitHub Actions validator, because the ASF
allowed-actions policy permits GitHub-owned actions but not the stand-alone
actionlint action/image.

Create `.github/workflows/actionlint.yml` in the consuming repository:

```yaml
name: Actionlint

on:
  push:
    paths:
      - '.github/workflows/**'
  pull_request:
    paths:
      - '.github/workflows/**'

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  actionlint:
    uses: apache/maven-gh-actions-shared/.github/workflows/actionlint.yml@v5
```

Optional input `validate-all-codebase` (default `true`) lints every workflow file;
set it to `false` to lint only files changed in the event's commit range.

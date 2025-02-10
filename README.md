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

Create GitHub workflow in project file:

```
.github/workflows/maven-verify.yml
```

 with content:

```yaml
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

name: Verify

on:
  push:
  pull_request:

jobs:
  build:
    name: Verify
    uses: apache/maven-gh-actions-shared/.github/workflows/maven-verify.yml@v4

```
## Events that can trigger workflows

In such configuration workflow can be executed in the same time twice for two separate events.

It can occurs when we create PR for branch from the same repository, we have two events:
- `push` on branch
- `pull_request` on PR for the same branch

In order to minimize resource consumptions shared workflow from version `v2` 
detect such situation and skips the execution for `pull_request` event 
if `PR` is created for local branch.

Only workflow for PR from external forks will be executed.

## Breaking changes in V3 

 - replace `maven_version` by `maven-matrix` and `ff-maven`
 - rename `maven_args` to `maven-args`
 - remove not used `verify-site-goal`

# Additional configurations

## Attach logs on failure

We can store some logs of execution in case of failure as workflow attachments:

```yaml
...
    uses: apache/maven-gh-actions-shared/.github/workflows/maven-verify.yml@v4
    with:
      failure-upload-path: |
        **/target/surefire-reports/*
```
## Excludes from build matrix:

```yaml
...
    uses: apache/maven-gh-actions-shared/.github/workflows/maven-verify.yml@v4
    with:
      matrix-exclude: >
        [ 
          {"jdk": "8"},   # exclude jdk 8 from all builds
          {"os": "windows-latest"}, # exclude windows from all builds
          {"jdk": "8", "os": "windows-latest"} # exclude jkd 8 on windows
        ]
```

## Change default goals

```yaml
...
    uses: apache/maven-gh-actions-shared/.github/workflows/maven-verify.yml@v4
    with:
      ff-goal: 'install'
      verify-goal: 'install -P run-its'
```

## Testing against different Maven versions

```yaml
...
    uses: apache/maven-gh-actions-shared/.github/workflows/maven-verify.yml@v4
    with:
      ff-maven: "3.8.6"                     # Maven version for fail-fast-build
      maven-matrix: '[ "3.2.5", "3.8.6" ]'  # Maven versions matrix for verify builds
```

## Testing with Maven 4

```yaml
...
    uses: apache/maven-gh-actions-shared/.github/workflows/maven-verify.yml@v4
    with:
      maven4-enabled: true
      maven4-version: 'xxx'    # only needed if you want to override default
      verify-goal: 'verify'    # only needed if project doesn't have a run-its profile
```

## Build with Maven 4 only

```yaml
...
    uses: apache/maven-gh-actions-shared/.github/workflows/maven-verify.yml@v4
    with:
      maven4-build: true
```

## Disable matrix build - only fail-fast build 

```yaml
...
    uses: apache/maven-gh-actions-shared/.github/workflows/maven-verify.yml@v4
    with:
      matrix-enabled: false
```
 
## More options

More options with default values can be found in workflow source in `inputs` section:

https://github.com/apache/maven-gh-actions-shared/blob/v4/.github/workflows/maven-verify.yml

# Release drafter configuration

## Only default branch

We need to add an action:

```
.github/workflows/release-drafter.yml
```

with content:

```yml
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.
   
name: Release Drafter
on:
  push:
    branches:
      - master
  workflow_dispatch:

jobs:
  update_release_draft:
    uses: apache/maven-gh-actions-shared/.github/workflows/release-drafter.yml@v4
```

and configuration:

```
.github/release-drafter.yml
```

with content:

```yml
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

_extends: maven-gh-actions-shared

tag-template: <project-tag-prefix>-$RESOLVED_VERSION
```

## Multiple versions from multiple branches

We need change a branch name in `.github/workflows/release-drafter.yml` in each branch:

```yml
name: Release Drafter
on:
  push:
    branches:
      - branch-name # <---
  workflow_dispatch:
```

## First release from new branch

When we create new branch for old version, release drafter will, can not find the previous release.

We need a change a `target_commitish` for the latest release.

Example for `maven-clean-plugin`

Release `3.4.0` was done form master branch, next we create new branch for 3.x - `maven-clean-plugin-3.x`

Release `3.4.0` has old `target_commitish`

```
gh api /repos/apache/maven-clean-plugin/releases/tags/maven-clean-plugin-3.4.0

...
"id": 161340667, <-- needed for update
"tag_name": "maven-clean-plugin-3.4.0",
"target_commitish": "master", <-- shold be new branch
...
```

We need update it, by:

```
gh api --method PATCH /repos/apache/maven-clean-plugin/releases/161340667 -f "target_commitish=refs/heads/maven-clean-plugin-3.x"
```

## pre-release, beta versions

To have a `pre-release`, beta versions we need add to `.github/release-drafter.yml`

```yml
include-pre-releases: true
prerelease: true
```

## Specific configuration per branch

When we need to have different configuration for each branch we need create a new `release-drafter` configuration in **default branch** ❗❗❗️

When configuration name is different then default, we need specified configuration name from shared repository:

```yml
_extends: maven-gh-actions-shared:.github/release-drafter.yml
...
```

Next in each branch we need provide new configuration name in action:

```
  update_release_draft:
    uses: apache/maven-gh-actions-shared/.github/workflows/release-drafter.yml@v4
    with:
      config-name: '<config-name-per-branch>' 
```

# Pull Request Automation

Create GitHub workflow in project file:

```
.github/workflows/pr-automation.yml
```

with content:

```yaml
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

name: PR Automation
on:
  pull_request_target:
    types:
      - closed
      - unlabeled
      - review_requested

jobs:
  pr-automation:
    name: PR Automation
    uses: apache/maven-gh-actions-shared/.github/workflows/pr-automation.yml@v4

```

We need a use `pull_request_target` event because we need `GITHUB_TOKEN` with write permission
to update labels, milestones of PR from forked repositories.

After approval or merged:
- default label - `maintenance` will be added
- current milestone will be set

## milestone configuration

We need exactly **one** open milestone in order to detect which is current.

If we need more than one open milestone, we have to add a branch to the milestone description
```
branch: <branch name>
```

This identifies the default for that branch.

# Labels synchronization

We can synchronize labels for all Maven repositories by action: [Labels sync](https://github.com/apache/maven-gh-actions-shared/actions/workflows/labels-sync.yml)

Labels list are in file: [./.github/labels.js](https://github.com/apache/maven-gh-actions-shared/blob/main/.github/labels.js)

Action require GitHub token which will be used for performing updates.
Please create new [Personal access tokens (classic)](https://github.com/settings/tokens) with `repo` scope.

# Management of stale issues and PRs

We need an action in project:

```yml
.github/workflows/stale.yml
```

with body:

```yml
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

name: Stale

on:
  schedule:
    - cron: '15 3 * * *'
  issue_comment:
    types: [ 'created' ]

jobs:
  stale:
    uses: 'apache/maven-gh-actions-shared/.github/workflows/stale.yml@v4'

```

The cron time definition can be randomly changed to avoid executing at the same time in all project.

## Issues and PRs with waiting-for-feedback label

- will be marked as stale after 60 days
- will be closed after next 30 days
- items with milestone or labels `priority:blocker`, `priority:critical` will not be checked
- `waiting-for-feedback` label will be removed after comments

# Resources

- [Workflow syntax](https://docs.github.com/en/actions/learn-github-actions/workflow-syntax-for-github-actions)
- [Reusing workflows](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows)


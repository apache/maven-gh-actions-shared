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

## Disable matrix build - only fail-fast build 

```yaml
...
    uses: apache/maven-gh-actions-shared/.github/workflows/maven-verify.yml@v4
    with:
      matrix-enabled: false
```

## Pull Request Automation

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
  pull_request:
    types:
      - closed
      - unlabeled
      - demilestoned
  pull_request_review:
    types:
      - submitted

jobs:
  pr-automation:
    name: PR Automation
    uses: apache/maven-gh-actions-shared/.github/workflows/pr-automation.yml@v4

```

After approval or merged:
- default label - `maintenance` will be added
- current milestone will be set
 
## More options

More options with default values can be found in workflow source in `inputs` section:

https://github.com/apache/maven-gh-actions-shared/blob/v4/.github/workflows/maven-verify.yml

# Resources

- [Workflow syntax](https://docs.github.com/en/actions/learn-github-actions/workflow-syntax-for-github-actions)
- [Reusing workflows](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows)


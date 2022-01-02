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
    uses: apache/maven-gh-actions-shared/.github/workflows/maven-verify.yml@v2

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

# Additional configurations

## Attach logs on failure

We can store some logs of execution in case of failure as workflow attachments:

```yaml
...
    uses: apache/maven-gh-actions-shared/.github/workflows/maven-verify.yml@v2
    with:
      failure-upload-path: |
        **/target/surefire-reports/*
```
## Excludes from build matrix:

```yaml
...
    uses: apache/maven-gh-actions-shared/.github/workflows/maven-verify.yml@v2
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
    uses: apache/maven-gh-actions-shared/.github/workflows/maven-verify.yml@v2
    with:
      ff-goal: 'install'
      verify-goal: 'install -P run-its'
```

## More options

More options with default values can be found in workflow source in `inputs` section:

https://github.com/apache/maven-gh-actions-shared/blob/v2/.github/workflows/maven-verify.yml

# Resources

- [Workflow syntax](https://docs.github.com/en/actions/learn-github-actions/workflow-syntax-for-github-actions)
- [Reusing workflows](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows)


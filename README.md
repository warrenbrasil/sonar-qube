# Warren - Run Tests and SonarQube Analysis :test_tube:

This action is responsible for running tests and publish the test resport to SonarQube.

## Prerequisites

Before using this action, please be sure that all tests projects have the following dependencies installed and updated to at least the latest stable version

- `xunit`
- `xunit.runner.visualstudio`
- `Microsoft.NET.Test.Sdk`
- `coverlet.collector`
- `coverlet.msbuild`
- `GitHubActionsTestLogger`

## Inputs

### `sonar-token`

**Required**: SonarQube token

### `sonar-host-url`

**Required**: SonarQube URL

### `sonar-project-key`

**Required**: SonarQube project key

To get the SonarQube project key go to `SonarQube`, find or setup your project, then click on `Project Information` at the top right corner. The `Project Key` will be found at drawer window that should be opened on you right.

### `sonar-organization`

**Required**: SonarQube organization name

### `pull-request`

**Required**: Boolean to change behavior of dotnet-sonarscanner for pull requests

### `branch-name`

**Required**: when `pull-request` input parameter is set to **false**
**Optional**: when `pull-request` input parameter is set to **true**

Branch name to be displayed on SonarQube.

### `pull-request-number`

**Required**: when `pull-request` input parameter is set to **true**
**Optional**: when `pull-request` input parameter is set to **false**

The number of the pull request

### `pull-request-base`

**Required**: when `pull-request` input parameter is set to **true**
**Optional**: when `pull-request` input parameter is set to **false**

The target branch name from the pull request

### `pull-request-head`

**Required**: when `pull-request` input parameter is set to **true**
**Optional**: when `pull-request` input parameter is set to **false**

The target source name from the pull request

### `solution`

**Optional**: Solution file name **without** the .sln extension **(do not use with project input parameter)**

### `project`

**Optional**: The path of test project to be tested **(do not use with solution input parameter)**

| :warning: the parameters `solution` and `project` are optional for you to choose one to use, but you **MUST** choose one :warning: |
| ---------------------------------------------------------------------------------------------------------------------------------- |

### `tests-dir`

**Optional**: The directory name which contains all the tests projects **(default is tests)**

### `code-exclusions`

**Optional**: The matching path for code that should be excluded from coverage **(absolute, relative and pattern paths are acceptable)**

### `assembly-exclusions`

**Optional**: The matching pattern for assemblies that should be excluded from coverage **(absolute, relative and pattern paths are acceptable)**

## Usage

Using for push on long-lived branches:

```yml
- name: Warren - Run Tests and SonarQube Analysis
  uses: warrenbrasil/sonar-qube@v1
  with:
    sonar-token: ${{ secrets.SONAR_TOKEN }}
    sonar-host-url: ${{ secrets.SONAR_HOST_URL }}
    sonar-project-key: warrenbrasil_my-repo-sonar-key
    sonar-organization: myorg
    solution: Warren.Core.MyRepo
    pull-request: false
    branch-name: ${{ github.head_ref || github.ref_name }}
```

Using for pull requests:

```yml
- name: Warren - Run Tests and SonarQube Analysis
  uses: warrenbrasil/sonar-qube@v1
  with:
    sonar-token: ${{ secrets.SONAR_TOKEN }}
    sonar-host-url: ${{ secrets.SONAR_HOST_URL }}
    sonar-project-key: warrenbrasil_my-repo-sonar-key
    sonar-organization: myorg
    solution: Warren.Core.MyRepo
    pull-request: true
    pull-request-number: ${{ github.event.pull_request.number }}
    pull-request-base: ${{ github.event.pull_request.base.ref }}
    pull-request-head: ${{ github.event.pull_request.head.ref }}
```

Using with file code exclusions:

```yml
- name: Warren - Run Tests and SonarQube Analysis
  uses: warrenbrasil/sonar-qube@v1
  with:
    sonar-token: ${{ secrets.SONAR_TOKEN }}
    sonar-host-url: ${{ secrets.SONAR_HOST_URL }}
    sonar-project-key: myorg_my-project-key
    sonar-organization: myorg
    solution: Warren.Core.MyRepo
    code-exclusions: "tests/Warren.Core.MyRepo.Fixtures/**/*.cs"
    pull-request: false
    branch-name: ${{ github.head_ref || github.ref_name }}
```

Using with multiline file code exclusions:

```yml
- name: Warren - Run Tests and SonarQube Analysis
  uses: warrenbrasil/sonar-qube@v1
  with:
    sonar-token: ${{ secrets.SONAR_TOKEN }}
    sonar-host-url: ${{ secrets.SONAR_HOST_URL }}
    sonar-project-key: myorg_my-project-key
    sonar-organization: myorg
    solution: Warren.Core.MyRepo
    code-exclusions: |
      "tests/Warren.Core.MyRepo.Fixtures/**/*.cs,
      src/Warren.Core.MyRepo.Services/**/*.cs"
    pull-request: false
    branch-name: ${{ github.head_ref || github.ref_name }}
```

Using with assembly exclusions:

```yml
- name: Warren - Run Tests and SonarQube Analysis
  uses: warrenbrasil/sonar-qube@v1
  with:
    sonar-token: ${{ secrets.SONAR_TOKEN }}
    sonar-host-url: ${{ secrets.SONAR_HOST_URL }}
    sonar-project-key: myorg_my-project-key
    sonar-organization: myorg
    solution: Warren.Core.MyRepo
    assembly-exclusions: "[*.Tests.*]*"
    pull-request: false
    branch-name: ${{ github.head_ref || github.ref_name }}
```

Using with multiline assembly exclusions:

```yml
- name: Warren - Run Tests and SonarQube Analysis
  uses: warrenbrasil/sonar-qube@v1
  with:
    sonar-token: ${{ secrets.SONAR_TOKEN }}
    sonar-host-url: ${{ secrets.SONAR_HOST_URL }}
    sonar-project-key: myorg_my-project-key
    sonar-organization: myorg
    solution: Warren.Core.MyRepo
    assembly-exclusions: |
      "[*.Tests.*]*%2c
      "[Warren.Core.MyRepo.Services]*"
    pull-request: false
    branch-name: ${{ github.head_ref || github.ref_name }}
```

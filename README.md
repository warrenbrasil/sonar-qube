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

### `code-exclusions`

**Optional**: The matching path for code that should be excluded from coverage **(absolute, relative and pattern paths are acceptable)**

### `quality-gate-wait`

**Optional**: Poll to SonarQube instance until the Quality Gate status is available **(default is `true`)**

### `verbosity`

**Optional**: Verbosity level for `dotnet test` command **(default is `m`)**

Allowed values are `q[uiet]`, `m[inimal]`, `n[ormal]`, `d[etailed]`, and `diag[nostic]`

### `enable-dotnet-restore`

**Optional**: Runs `dotnet restore` before runs `dotnet test`. **(default is `false`)**

### `enable-dotnet-build`

**Optional**: Runs `dotnet build --no-restore` before runs `dotnet test`. **(default is `false`)**

### `dotnet-test-flags`

**Optional**: Add flags for `dotnet test` command. **(default is no flags)**.

Full flags list is available [here](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-test#synopsis).

## Usage

Using for push on long-lived branches:

```yml
- name: Warren - Run Tests and SonarQube Analysis
  uses: warrenbrasil/sonar-qube@v2
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
  uses: warrenbrasil/sonar-qube@v3
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
  uses: warrenbrasil/sonar-qube@v3
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
  uses: warrenbrasil/sonar-qube@v3
  with:
    sonar-token: ${{ secrets.SONAR_TOKEN }}
    sonar-host-url: ${{ secrets.SONAR_HOST_URL }}
    sonar-project-key: myorg_my-project-key
    sonar-organization: myorg
    solution: Warren.Core.MyRepo
    code-exclusions: >-
      tests/Warren.Core.MyRepo.Fixtures/**/*.cs,
      src/Warren.Core.MyRepo.Services/**/*.cs
    pull-request: false
    branch-name: ${{ github.head_ref || github.ref_name }}
```

Using with dotnet restore, dotnet build and dotnet test flags:

```yml
- name: Warren - Run Tests and SonarQube Analysis
  uses: warrenbrasil/sonar-qube@v3
  with:
    sonar-token: ${{ secrets.SONAR_TOKEN }}
    sonar-host-url: ${{ secrets.SONAR_HOST_URL }}
    sonar-project-key: myorg_my-project-key
    sonar-organization: myorg
    solution: Warren.Core.MyRepo
    pull-request: false
    branch-name: ${{ github.head_ref || github.ref_name }}
    enable-dotnet-restore: true
    enable-dotnet-build: true
    dotnet-test-flags: --no-build --no-restore
```

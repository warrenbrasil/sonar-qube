# Warren - Run Tests and SonarQube Analysis :test_tube:

Composite GitHub Action that runs your .NET tests with code coverage and publishes the analysis (coverage + Quality Gate) to SonarQube.

**Current version:** `v7` → `v7.2` (SonarScanner for .NET pinned to `11.3.*`)

## Quick start

Store the SonarQube token and server URL as repository or organization secrets (`SONAR_TOKEN`, `SONAR_HOST_URL`), then add the action after checking out the code and setting up the .NET SDK.

Pull requests:

```yml
on:
  pull_request:

jobs:
  sonar:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: "8.0.x" # your SDK version

      - name: Warren - Run Tests and SonarQube Analysis
        uses: warrenbrasil/sonar-qube@v7
        with:
          sonar-token: ${{ secrets.SONAR_TOKEN }}
          sonar-host-url: ${{ secrets.SONAR_HOST_URL }}
          sonar-project-key: my-project-key
          sonar-organization: my-org
          solution: MySolution
          pull-request: true
          pull-request-number: ${{ github.event.pull_request.number }}
          pull-request-base: ${{ github.event.pull_request.base.ref }}
          pull-request-head: ${{ github.event.pull_request.head.ref }}
```

Push on a long-lived branch:

```yml
on:
  push:
    branches: [main]

jobs:
  sonar:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: "8.0.x" # your SDK version

      - name: Warren - Run Tests and SonarQube Analysis
        uses: warrenbrasil/sonar-qube@v7
        with:
          sonar-token: ${{ secrets.SONAR_TOKEN }}
          sonar-host-url: ${{ secrets.SONAR_HOST_URL }}
          sonar-project-key: my-project-key
          sonar-organization: my-org
          solution: MySolution
          pull-request: false
          branch-name: ${{ github.head_ref || github.ref_name }}
```

## How it works

The action runs these steps, in this order, inside your job:

1. **Install JDK 17** (`actions/setup-java@v4`, `microsoft` distribution). The scanner needs Java; you do not have to install it yourself.
2. **Install SonarScanner for .NET** (`dotnet-sonarscanner`, version `11.3.*`) as a local tool in `$HOME/sonar-scanner`. If the installation fails, the step fails.
3. **`dotnet-sonarscanner begin`**, authenticated with `sonar.token`, in one of two modes:
   - **Pull request mode** (`pull-request: true`): sets `sonar.pullrequest.key`, `sonar.pullrequest.branch` and `sonar.pullrequest.base`.
   - **Branch mode** (`pull-request: false`): sets `sonar.branch.name`.

   In both modes it sets `sonar.qualitygate.wait` from `quality-gate-wait`, `sonar.cs.opencover.reportsPaths=$GITHUB_WORKSPACE/reports/**/*.opencover.xml`, and `sonar.coverage.exclusions` when `code-exclusions` is set.
4. **`dotnet restore`** (when `enable-dotnet-restore` is `true`) with `dotnet-restore-flags`.
5. **`dotnet build`** (when `enable-dotnet-build` is `true`) with `dotnet-build-flags`.
6. **`dotnet test`** on `<solution>.sln` **or** on `<project>`:

   ```bash
   dotnet test <solution>.sln | <project> \
     --collect:"XPlat Code Coverage" \
     --results-directory:"$GITHUB_WORKSPACE/reports/" \
     <dotnet-test-flags> \
     --logger "GitHubActions;report-warnings=false" \
     --verbosity <verbosity> -- \
     DataCollectionRunSettings.DataCollectors.DataCollector.Configuration.Format=opencover \
     DataCollectionRunSettings.DataCollectors.DataCollector.Configuration.ExcludeByAttribute=CompilerGeneratedAttribute
   ```

   Coverage is collected by coverlet in OpenCover format into `$GITHUB_WORKSPACE/reports/`, which is where the `begin` step tells the scanner to look.
7. **`dotnet-sonarscanner end`**: uploads the analysis. With `quality-gate-wait: true` (the default) it waits for the Quality Gate and **the step fails if the Quality Gate fails**.

> [!IMPORTANT]
> The tests run **inside this action**, between `begin` and `end`. If the SonarQube `begin` step fails (for example, the server is unreachable or incompatible with the scanner), the tests **do not run** at all. Before trusting the check, open the action's steps in the job log and confirm that `Run tests` actually ran.

> [!NOTE]
> Set exactly one of `solution` or `project`. If both or neither are set, the `Run tests` step is skipped and no coverage is produced.
>
> `pull-request` must be exactly `true` or `false`. With any other value no `begin` step runs.

## Versioning

| Tag | Type | Points to | Receives fixes |
|---|---|---|---|
| `@v7` | Floating | Latest stable `7.x` (today `v7.2`) | Yes |
| `@v7.2`, `@v7.1` | Fixed | That exact release | No (move to a newer tag) |
| `@v1` … `@v6`, `@v7-beta*` | Legacy | — | No |

- Use **`@v7`** to receive fixes automatically (recommended).
- Use **`@v7.2`** to pin an exact version.

## Migrating

From `v6`, `v7-beta*` or `v7.1` to the current version:

1. Change the tag to `warrenbrasil/sonar-qube@v7` (or `@v7.2` to pin).
2. No input changes since `v6`. The only exception: if you come from a `v7-beta*` tag, remove the `export-path` and `environment-cleanup` inputs (they no longer exist).
3. You do **not** need to install a JDK or the scanner yourself: the action does both.

From `v5` or older, also remove `tests-dir` and `target_frameworks` (removed in `v6`); use the restore/build/test inputs instead.

## Prerequisites

Before using this action, make sure that all test projects have the following dependencies installed and updated to at least the latest stable version:

- `xunit`
- `xunit.runner.visualstudio`
- `Microsoft.NET.Test.Sdk`
- `coverlet.collector`
- `coverlet.msbuild`
- `GitHubActionsTestLogger`

The .NET SDK must be available on the runner (for example, via `actions/setup-dotnet`). The action runs `dotnet` from the workspace root, so `solution` and `project` are resolved relative to it.

## Requirements

- A SonarQube Server compatible with **SonarScanner for .NET 11.3**.
- SonarScanner for .NET **12** requires **SonarQube Server 2025.1 or later**. This action pins the scanner to `11.3.*` until that requirement is met.

## Inputs

| Name | Required | Default | Description |
|---|---|---|---|
| `sonar-token` | Yes | — | SonarQube token. Pass it from a secret (`${{ secrets.SONAR_TOKEN }}`). |
| `sonar-host-url` | Yes | — | SonarQube URL. Pass it from a secret (`${{ secrets.SONAR_HOST_URL }}`). |
| `sonar-project-key` | Yes | — | SonarQube project key. In SonarQube, open your project and click **Project Information** (top right); the **Project Key** is shown in the drawer. |
| `sonar-organization` | Yes | — | SonarQube organization name. |
| `pull-request` | Yes | — | `true` for pull request analysis, `false` for branch analysis. |
| `branch-name` | When `pull-request: false` | — | Branch name to be displayed on SonarQube. |
| `pull-request-number` | When `pull-request: true` | — | The number of the pull request. |
| `pull-request-base` | When `pull-request: true` | — | The target (base) branch of the pull request. |
| `pull-request-head` | When `pull-request: true` | — | The source (head) branch of the pull request. |
| `solution` | One of `solution`/`project` | `""` | Solution file name **without** the `.sln` extension. Do not use with `project`. |
| `project` | One of `solution`/`project` | `""` | Path of the test project to be tested. Do not use with `solution`. |
| `code-exclusions` | No | `""` | Paths excluded from coverage (absolute, relative and pattern paths are accepted; comma-separated). |
| `quality-gate-wait` | No | `true` | Wait for the Quality Gate result. When `true`, the step fails if the Quality Gate fails. |
| `verbosity` | No | `m` | Verbosity of `dotnet test`: `q[uiet]`, `m[inimal]`, `n[ormal]`, `d[etailed]` or `diag[nostic]`. |
| `enable-dotnet-restore` | No | `true` | Run `dotnet restore` before `dotnet test`. |
| `dotnet-restore-flags` | No | `""` | Flags for `dotnet restore` ([reference](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-restore#synopsis)). |
| `enable-dotnet-build` | No | `true` | Run `dotnet build` before `dotnet test`. |
| `dotnet-build-flags` | No | `--no-restore` | Flags for `dotnet build` ([reference](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-build#synopsis)). |
| `dotnet-test-flags` | No | `--no-build --no-restore` | Flags for `dotnet test` ([reference](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-test#synopsis)). |

`branch-name` and the `pull-request-*` inputs are declared optional in `action.yml`, but the mode selected by `pull-request` needs them.

## Usage examples

Push on long-lived branches:

```yml
- name: Warren - Run Tests and SonarQube Analysis
  uses: warrenbrasil/sonar-qube@v7
  with:
    sonar-token: ${{ secrets.SONAR_TOKEN }}
    sonar-host-url: ${{ secrets.SONAR_HOST_URL }}
    sonar-project-key: my-project-key
    sonar-organization: my-org
    solution: MySolution
    pull-request: false
    branch-name: ${{ github.head_ref || github.ref_name }}
```

Pull requests:

```yml
- name: Warren - Run Tests and SonarQube Analysis
  uses: warrenbrasil/sonar-qube@v7
  with:
    sonar-token: ${{ secrets.SONAR_TOKEN }}
    sonar-host-url: ${{ secrets.SONAR_HOST_URL }}
    sonar-project-key: my-project-key
    sonar-organization: my-org
    solution: MySolution
    pull-request: true
    pull-request-number: ${{ github.event.pull_request.number }}
    pull-request-base: ${{ github.event.pull_request.base.ref }}
    pull-request-head: ${{ github.event.pull_request.head.ref }}
```

With code exclusions:

```yml
- name: Warren - Run Tests and SonarQube Analysis
  uses: warrenbrasil/sonar-qube@v7
  with:
    sonar-token: ${{ secrets.SONAR_TOKEN }}
    sonar-host-url: ${{ secrets.SONAR_HOST_URL }}
    sonar-project-key: my-project-key
    sonar-organization: my-org
    solution: MySolution
    code-exclusions: "tests/MyProject.Fixtures/**/*.cs"
    pull-request: false
    branch-name: ${{ github.head_ref || github.ref_name }}
```

With multiline code exclusions:

```yml
- name: Warren - Run Tests and SonarQube Analysis
  uses: warrenbrasil/sonar-qube@v7
  with:
    sonar-token: ${{ secrets.SONAR_TOKEN }}
    sonar-host-url: ${{ secrets.SONAR_HOST_URL }}
    sonar-project-key: my-project-key
    sonar-organization: my-org
    solution: MySolution
    code-exclusions: >-
      tests/MyProject.Fixtures/**/*.cs,
      src/MyProject.Services/**/*.cs
    pull-request: false
    branch-name: ${{ github.head_ref || github.ref_name }}
```

With `dotnet restore`, `dotnet build` and `dotnet test` flags:

```yml
- name: Warren - Run Tests and SonarQube Analysis
  uses: warrenbrasil/sonar-qube@v7
  with:
    sonar-token: ${{ secrets.SONAR_TOKEN }}
    sonar-host-url: ${{ secrets.SONAR_HOST_URL }}
    sonar-project-key: my-project-key
    sonar-organization: my-org
    solution: MySolution
    pull-request: false
    branch-name: ${{ github.head_ref || github.ref_name }}
    dotnet-restore-flags: --no-cache
    dotnet-build-flags: --no-restore --configuration Release
    dotnet-test-flags: --no-build --no-restore --configuration Release
```

> [!TIP]
> Setting a `dotnet-*-flags` input **replaces** its default. Keep `--no-restore` / `--no-build` in your flags if you still want to avoid restoring and building twice.

## Where to see the results

- **In the job log:** with `quality-gate-wait: true`, the `End Sonar Scan` step prints a line like:

  ```text
  QUALITY GATE STATUS: PASSED - View details on <sonar-host-url>/dashboard?id=<project-key>
  QUALITY GATE STATUS: FAILED - View details on <sonar-host-url>/dashboard?id=<project-key>&pullRequest=<number>
  ```

  The `&pullRequest=<number>` part appears for pull request analyses.
- **In SonarQube:** open the project by its key. Pull request analyses are under the **Pull Requests** tab.
- **Badges:** available in the project's **Project Information** in SonarQube.

## Troubleshooting

**`api/v2/analysis/version` returns 404 / "Pre-processing failed" in the `begin` step**
A newer SonarScanner for .NET (12.x) was used against a SonarQube Server older than 2025.1. Use `@v7` or `@v7.2`, which pin the scanner to `11.3.*`. In this situation **the tests did not run**.

**Warning that `sonar.login` is deprecated**
`v7.1` and earlier authenticate with `sonar.login`, which is deprecated (and removed from SonarQube Cloud). `v7.2` uses `sonar.token`. Move to `@v7` or `@v7.2`; no input changes.

**The step is red but all tests passed**
With `quality-gate-wait: true` (the default), a failed Quality Gate fails the step. Check the `QUALITY GATE STATUS` line and the project in SonarQube. To only report the result without failing, set `quality-gate-wait: false`.

**Coverage is 0% or missing**
Check that `Run tests` actually ran (exactly one of `solution`/`project` set), that the test projects reference `coverlet.collector`, and that coverage files were produced under `$GITHUB_WORKSPACE/reports/`.

## Changelog

- **v7.2** — Pin SonarScanner for .NET to `11.3.*` (12.x requires SonarQube Server 2025.1+); authenticate with `sonar.token` instead of the deprecated `sonar.login`; scanner installation failures are no longer ignored. No input changes. (#9)
- **v7.1** — Install the scanner in a local tool path (`$HOME/sonar-scanner`) and invoke it from there.
- **v7** — Install JDK 17 (required by newer scanner versions). The `v7` tag is floating and follows the latest `7.x`.
- **v6** — Coverage reports generated under `$GITHUB_WORKSPACE/reports`; new restore/build/test inputs (`enable-dotnet-restore`, `dotnet-restore-flags`, `enable-dotnet-build`, `dotnet-build-flags`, `dotnet-test-flags`); `tests-dir` and `target_frameworks` removed.
- **v5** — Target frameworks support (`target_frameworks` input).
- **v4** — Java distribution changed to `microsoft`, version 17.
- **v3** — `verbosity` input for `dotnet test`.
- **v2** — Code exclusions only in `begin`; `quality-gate-wait` input.
- **v1** — Initial release.

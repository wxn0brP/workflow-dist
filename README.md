# Reusable Workflow: Build and Deploy TypeScript Project

This workflow builds a TypeScript project, optionally handles `package.json` scripts, and deploys the specified files to a clean branch. It can also publish the package to NPM on tag pushes.

## Usage

To use this reusable workflow, call it from another workflow file like this:

```yaml
name: Main CI

on:
  push:
    branches:
      - main
    tags:
      - 'v*'

jobs:
  build-and-deploy:
    uses: ./.github/workflows/build-ts.yml
    with:
      # See inputs section for all available options
      files: "dist package.json README.md"
      branch: "dist"
      publishToNpm: true
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## Inputs

The workflow accepts the following inputs:

| Name | Description | Type | Required | Default |
|---|---|---|---|---|
| `files` | A space-separated list of files and directories to be copied to the deployment branch. | `string` | No | `"dist package.json"` |
| `branch` | The name of the target branch for deployment. | `string` | No | `"dist"` |
| `customCommands` | Custom shell commands to be executed after the build process. | `string` | No | `""` |
| `preBuildCustomCommands` | Custom shell commands to be executed before the build process. | `string` | No | `""` |
| `publishToNpm` | Set to `true` to publish the package to NPM. This only runs on tag pushes (`refs/tags/*`). | `boolean` | No | `false` |
| `scriptsHandling` | Defines how to process the `scripts` section in `package.json`.<br>- `retain-all`: Keep all original scripts.<br>- `remove-all`: Remove the entire `scripts` section.<br>- `keep-start`: Keep only the `start` script.<br>- `custom-list`: Keep a specific list of scripts defined in `customScripts`. | `string` | No | `"keep-start"` |
| `customScripts` | A comma-separated list of script names to retain when `scriptsHandling` is set to `custom-list`. | `string` | No | `""` |
| `createVersionedBranch` | If `true` and the workflow is triggered by a tag, it creates an additional versioned branch (e.g., `dist-v1.2.3`). | `boolean` | No | `true` |
| `path` | The working directory where commands will be executed. | `string` | No | `"."` |
| `out` | The output directory within the deployment branch where files will be placed. | `string` | No | `"."` |

## Secrets

The workflow may require the following secrets:

| Name | Description | Required |
|---|---|---|
| `NPM_TOKEN` | An NPM authentication token required for publishing the package. This is only needed if `publishToNpm` is `true`. | Yes, if `publishToNpm` is `true`. |

## Workflow Steps

1.  **Checkout Repo**: Checks out the source repository.
2.  **Setup Node.js**: Installs Node.js (version 22).
3.  **Install Dependencies**: Runs `npm i`.
4.  **Install TypeScript & tsc-alias**: Installs `typescript` and `tsc-alias` globally.
5.  **Run Pre Build Custom Commands**: Executes commands from `preBuildCustomCommands` if provided.
6.  **Build Project**: Runs `npm run build`.
7.  **Run Custom Commands**: Executes commands from `customCommands` if provided.
8.  **Modify scripts in package.json**: Adjusts the `scripts` in `package.json` based on the `scriptsHandling` input.
9.  **Prepare and Push to Target Branch**:
    *   Copies the specified `files` to a temporary location.
    *   Checks out an orphan branch specified by the `branch` input.
    *   Moves the files into the `out` directory.
    *   Commits and force-pushes the files to the target branch.
    *   If `createVersionedBranch` is true and on a tag, it creates and pushes a versioned branch as well.
10. **Publish to npm**: If `publishToNpm` is `true` and the trigger was a tag, it publishes the package to the NPM registry.

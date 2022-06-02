# GitHub Actions Reusable Workflows for Node.js projects
> This module offers reusable workflows to ease javascript development.
>
> - Setup node (node version & auth)
> - Install dependencies
> - Build
> - Test
> - Publish new version

## Workflow Setup
[`.github/workflows/setup.yml`](.github/workflows/setup.yml)

### Scenarios

- [Setup node without authentication](#Setup-node-without-authentication)
- [Setup node with authentication](#Setup-node-with-authentication)
- [Setup node with Jfrog Artifactory authentication](#Setup-node-with-Jfrog-Artifactory)

#### Setup node without authentication

```yaml
  - uses: absolunet/github-node-reusable-workflows/.github/workflows/setup.yml@production
```

#### Setup node with authentication

```yaml
  - uses: absolunet/github-node-reusable-workflows/.github/workflows/setup.yml@production
    secrets:
      scope: ${{ secrets.<SCOPE> }}
      registry-url: ${{ secrets.<REGISTRY_URL> }}
      auth-token: ${{ secrets.<VARIABLE_AUTH_TOKEN> }}
```

#### Setup node with Jfrog Artifactory authentication
Issue with Jfrog Artifactory - it uses _auth instead of standard _authToken

```yaml
  - uses: absolunet/github-node-reusable-workflows/.github/workflows/setup.yml@production
    with:
      jfrog-artifactory: true
    secrets:
      scope: ${{ secrets.<SCOPE> }}
      registry-url: ${{ secrets.<REGISTRY_URL> }}
      auth-token: ${{ secrets.<VARIABLE_AUTH_TOKEN> }}
```

## Workflow Build
[`.github/workflows/build.yml`](.github/workflows/build.yml)

### Scenarios

- [Build packages](#Build-packages)


#### Build packages

```yaml
  - uses: absolunet/github-node-reusable-workflows/.github/workflows/build.yml@production
```

## Workflow Version bump
[`.github/workflows/version_bump.yml`](.github/workflows/version_bump.yml)

### Scenarios

- [Version bump and commit](#Version-bump-and-commit)

#### Version bump and commit
```yaml
  - uses: absolunet/github-node-reusable-workflows/.github/workflows/version_bump.yml@production
```


## Workflow Publish
[`.github/workflows/publish.yml`](.github/workflows/publish.yml)

### Scenarios

- [Publish and Tag](#Publish-and-Tag)

#### Publish and Tag

```yaml
  - uses: absolunet/github-node-reusable-workflows/.github/workflows/publish.yml@production
```


## Usage
How to create a github workflow.

- Create a directory `.github/workflows` in the root of your directory.
- Then, create 3 workflows
  - `test.yml` : Run Test
  - `version-bump.yml` : Version bump and git commit
  - `publish.yml` : Publish to registry and git tag

### Scenarios:
- [A public package to be published](#A-public-package-to-be-published)

#### A public package to be published

#### Workflow Run tests - `test.yml`

```yaml
name: Publish to registry and git tag

on:
  push:
    branches:
      - production

jobs:
  call_workflow_setup:
    name: Setup environment
    uses: absolunet/github-node-reusable-workflows/.github/workflows/setup.yml@production
    with:
      branch-ref: ${{ github.head_ref }}
#      jfrog-artifactory: true # Jfrog Artifactory
    secrets:
#      scope: ${{ secrets.<VARIABLE_SCOPE> }} # private
#      registry-url: ${{ secrets.<VARIABLE_REGISTRY_URL> }} # private
      auth-token: ${{ secrets.<VARIABLE_AUTH_TOKEN> }}
  call_workflow_build:
    name: Build project
    needs: [ call_workflow_setup ]
    uses: absolunet/github-node-reusable-workflows/.github/workflows/build.yml@production
  call_workflow_publish:
    name: Publish packages
    needs: [ call_workflow_build ]
    uses: absolunet/github-node-reusable-workflows/.github/workflows/publish.yml@production
```

#### Workflow Version bump and git commit - `version-bump.yml`

```yaml
name: Version bump and git commit

on:
  pull_request:
    types:
      - opened
    branches:
      - production

jobs:
  call_workflow_setup:
    name: Setup environment
    if: startsWith(github.head_ref, 'release')
    uses: absolunet/github-node-reusable-workflows/.github/workflows/setup.yml@production
    with:
      branch-ref: ${{ github.head_ref }}
#      jfrog-artifactory: true # Jfrog Artifactory
    secrets:
      github-token: ${{ secrets.<GITHUB_ACCESS_TOKEN> }}
      auth-token: ${{ secrets.<VARIABLE_AUTH_TOKEN> }}
#      scope: ${{ secrets.<VARIABLE_SCOPE> }} # private
#      registry-url: ${{ secrets.<VARIABLE_REGISTRY_URL> }} # private
  call_workflow_version_bump:
    name: Bump version
    needs: [ call_workflow_setup ]
    uses: absolunet/github-node-reusable-workflows/.github/workflows/version_bump.yml@production
```

#### Workflow Publish to registry and git tag - `publish.yml`
```yaml
name: Publish to registry and git tag

on:
  push:
    branches:
      - production

jobs:
  call_workflow_setup:
    name: Setup environment
    uses: absolunet/github-node-reusable-workflows/.github/workflows/setup.yml@production
    with:
      branch-ref: ${{ github.head_ref }}
#      jfrog-artifactory: true # Jfrog Artifactory
    secrets:
      github-token: ${{ secrets.<GITHUB_ACCESS_TOKEN> }}
      auth-token: ${{ secrets.<VARIABLE_AUTH_TOKEN> }}
#      scope: ${{ secrets.<VARIABLE_SCOPE> }} # private
#      registry-url: ${{ secrets.<VARIABLE_REGISTRY_URL> }} # private
  call_workflow_build:
    name: Build project
    needs: [ call_workflow_setup ]
    uses: absolunet/github-node-reusable-workflows/.github/workflows/build.yml@production
  call_workflow_publish:
    name: Publish packages
    needs: [ call_workflow_build ]
    uses: absolunet/github-node-reusable-workflows/.github/workflows/publish.yml@production

```

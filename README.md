# SW-Workflows

Reusable GitHub Actions workflows and composite actions for the [Simplify9](https://github.com/simplify9) open source project ecosystem (Simply Works catalog).

## Overview

This repository contains a collection of reusable GitHub Actions designed to streamline CI/CD processes for .NET projects within the Simplify9 ecosystem. These actions provide standardized workflows for building, testing, packaging, and releasing .NET applications and libraries.

## Available Actions

### 🔢 determine-semver

**Purpose:** Automatically computes the next semantic version based on major/minor inputs and existing git tags.

**Location:** `actions/determine-semver.yml`

**Inputs:**
- `major` (required): Major version number
- `minor` (required): Minor version number

**Outputs:**
- `version`: Computed semantic version (e.g., `1.2.3`)

**Usage Example:**
```yaml
- name: Determine Version
  id: version
  uses: simplify9/SW-Workflows/actions/determine-semver@main
  with:
    major: '1'
    minor: '2'

- name: Use Version
  run: echo "Next version will be ${{ steps.version.outputs.version }}"
```

### 🏗️ dotnet-restore-build

**Purpose:** Restores dependencies and builds .NET projects with configurable options.

**Location:** `actions/dotnet-restore-build.yml`

**Inputs:**
- `projects` (optional): Project(s) to restore and build (wildcards allowed, default: `**/*.csproj`)
- `configuration` (optional): Build configuration (default: `Release`)
- `dotnet-version` (optional): Version of .NET SDK to use (default: `8.0.x`)

**Usage Example:**
```yaml
- name: Build Projects
  uses: simplify9/SW-Workflows/actions/dotnet-restore-build@main
  with:
    projects: 'src/**/*.csproj'
    configuration: 'Release'
    dotnet-version: '8.0.x'
```

### 🧪 dotnet-restore-build-test

**Purpose:** Restores, builds, and tests .NET projects in a single workflow step.

**Location:** `actions/dotnet-restore-build-test.yml`

**Inputs:**
- `projects` (optional): Project(s) to restore and build (wildcards allowed, default: `**/*.csproj`)
- `test-projects` (required): Test project(s) to run (wildcards allowed, default: `**/*UnitTests/*.csproj`)
- `configuration` (optional): Build configuration (default: `Release`)
- `dotnet-version` (optional): Version of .NET SDK to use (default: `8.0.x`)

**Usage Example:**
```yaml
- name: Build and Test
  uses: simplify9/SW-Workflows/actions/dotnet-restore-build-test@main
  with:
    projects: 'src/**/*.csproj'
    test-projects: 'tests/**/*Tests.csproj'
    configuration: 'Release'
```

### 📦 dotnet-pack-push

**Purpose:** Packs a .NET project and pushes the resulting NuGet package to a specified feed.

**Location:** `actions/dotnet-pack-push.yml`

**Inputs:**
- `project` (required): Path to the .csproj file to pack
- `configuration` (optional): Build configuration (default: `Release`)
- `version` (optional): Semantic version for the NuGet package
- `api-key` (required): API key for the NuGet feed
- `nuget-source` (required): NuGet feed/source URL

**Usage Example:**
```yaml
- name: Pack and Push NuGet
  uses: simplify9/SW-Workflows/actions/dotnet-pack-push@main
  with:
    project: 'src/MyLibrary/MyLibrary.csproj'
    version: ${{ steps.version.outputs.version }}
    api-key: ${{ secrets.NUGET_API_KEY }}
    nuget-source: 'https://api.nuget.org/v3/index.json'
```

### 🏷️ tag-github-origin

**Purpose:** Creates a git tag on the GitHub repository using the REST API.

**Location:** `actions/tag-github-origin.yml`

**Inputs:**
- `github-token` (required): GitHub token for authentication
- `repository` (required): GitHub repository in `owner/repo` format
- `tag` (required): Tag name to create
- `sha` (required): Commit SHA to tag

**Usage Example:**
```yaml
- name: Create Git Tag
  uses: simplify9/SW-Workflows/actions/tag-github-origin@main
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    repository: ${{ github.repository }}
    tag: 'v${{ steps.version.outputs.version }}'
    sha: ${{ github.sha }}
```

## Complete Workflow Examples

### Basic CI/CD Pipeline

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
    
    - name: Build and Test
      uses: simplify9/SW-Workflows/actions/dotnet-restore-build-test@main
      with:
        projects: 'src/**/*.csproj'
        test-projects: 'tests/**/*Tests.csproj'

  release:
    if: github.ref == 'refs/heads/main'
    needs: build-test
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
    
    - name: Determine Version
      id: version
      uses: simplify9/SW-Workflows/actions/determine-semver@main
      with:
        major: '1'
        minor: '0'
    
    - name: Build
      uses: simplify9/SW-Workflows/actions/dotnet-restore-build@main
      with:
        projects: 'src/**/*.csproj'
    
    - name: Pack and Push
      uses: simplify9/SW-Workflows/actions/dotnet-pack-push@main
      with:
        project: 'src/MyLibrary/MyLibrary.csproj'
        version: ${{ steps.version.outputs.version }}
        api-key: ${{ secrets.NUGET_API_KEY }}
        nuget-source: 'https://api.nuget.org/v3/index.json'
    
    - name: Create Release Tag
      uses: simplify9/SW-Workflows/actions/tag-github-origin@main
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}
        repository: ${{ github.repository }}
        tag: 'v${{ steps.version.outputs.version }}'
        sha: ${{ github.sha }}
```

## Contributing

This repository is part of the Simplify9 open source ecosystem. Contributions are welcome! Please ensure that:

1. Actions follow the composite action format
2. All inputs and outputs are properly documented
3. Actions are tested before submission
4. Breaking changes are clearly documented

## License

This project is part of the Simplify9 open source initiative. Please refer to the individual project licenses within the Simplify9 ecosystem.

## Support

For issues, questions, or contributions related to these workflows, please visit the [Simplify9 organization](https://github.com/simplify9) or create an issue in this repository.
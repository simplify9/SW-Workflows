# SW-Workflows

Reusable GitHub Actions workflows and composite actions for the [Simplify9](https://github.com/simplify9) open source project ecosystem (Simply Works catalog).

## Overview

This repository contains a collection of reusable GitHub Actions designed to streamline CI/CD processes for .NET projects within the Simplify9 ecosystem. These actions provide standardized workflows for building, testing, packaging, and releasing .NET applications and libraries.

## Available Actions

### 🔢 determine-semver

**Purpose:** Automatically computes the next semantic version based on major/minor inputs and existing git tags.

**Location:** `actions/determine-semver`

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

### 🏗️ dotnet-build

**Purpose:** Restores, builds, and optionally tests .NET projects. This is a unified action that can handle both build-only and build+test scenarios.

**Location:** `actions/dotnet-build`

**Inputs:**
- `projects` (optional): Project(s) to restore and build (wildcards allowed, default: `**/*.csproj`)
- `test-projects` (optional): Test project(s) to run (wildcards allowed, default: `**/*UnitTests/*.csproj`)
- `configuration` (optional): Build configuration (default: `Release`)
- `dotnet-version` (optional): Version of .NET SDK to use (default: `8.0.x`)
- `run-tests` (optional): Whether to run tests after building (default: `true`)

**Usage Examples:**

Build only:
```yaml
- name: Build Projects
  uses: simplify9/SW-Workflows/actions/dotnet-build@main
  with:
    projects: 'src/**/*.csproj'
    run-tests: 'false'
```

Build and Test:
```yaml
- name: Build and Test
  uses: simplify9/SW-Workflows/actions/dotnet-build@main
  with:
    projects: 'src/**/*.csproj'
    test-projects: 'tests/**/*Tests.csproj'
    run-tests: 'true'
```

> **Note:** This action is self-contained and doesn't depend on other actions in this repository, making it safe to use from external repositories.

### 📦 dotnet-pack-push

**Purpose:** Packs .NET project(s) and pushes the resulting NuGet package(s) to a specified feed. Supports both single and multiple projects.

**Location:** `actions/dotnet-pack-push`

**Inputs:**
- `projects` (required): Path(s) to .csproj file(s) to pack (supports wildcards and multiple projects)
- `configuration` (optional): Build configuration (default: `Release`)
- `version` (optional): Semantic version for the NuGet package(s)
- `api-key` (required): API key for the NuGet feed
- `nuget-source` (required): NuGet feed/source URL

**Common NuGet Sources:**
- **NuGet.org (public)**: `https://api.nuget.org/v3/index.json`
- **GitHub Packages**: `https://nuget.pkg.github.com/OWNER/index.json`
- **Azure DevOps**: `https://pkgs.dev.azure.com/ORG/_packaging/FEED/nuget/v3/index.json`

**Usage Examples:**

Single project:
```yaml
- name: Pack and Push Single Project
  uses: simplify9/SW-Workflows/actions/dotnet-pack-push@main
  with:
    projects: 'src/MyLibrary/MyLibrary.csproj'
    version: ${{ steps.version.outputs.version }}
    api-key: ${{ secrets.NUGET_API_KEY }}
    nuget-source: 'https://api.nuget.org/v3/index.json'
```

Multiple projects with wildcards:
```yaml
- name: Pack and Push Multiple Projects
  uses: simplify9/SW-Workflows/actions/dotnet-pack-push@main
  with:
    projects: 'src/*/*.csproj'
    version: ${{ steps.version.outputs.version }}
    api-key: ${{ secrets.NUGET_API_KEY }}
    nuget-source: 'https://api.nuget.org/v3/index.json'
```

All packable projects:
```yaml
- name: Pack and Push All Projects
  uses: simplify9/SW-Workflows/actions/dotnet-pack-push@main
  with:
    projects: '**/*.csproj'
    version: ${{ steps.version.outputs.version }}
    api-key: ${{ secrets.NUGET_API_KEY }}
    nuget-source: 'https://api.nuget.org/v3/index.json'
```

Push to GitHub Packages:
```yaml
- name: Pack and Push to GitHub Packages
  uses: simplify9/SW-Workflows/actions/dotnet-pack-push@main
  with:
    projects: 'src/MyLibrary/MyLibrary.csproj'
    version: ${{ steps.version.outputs.version }}
    api-key: ${{ secrets.GITHUB_TOKEN }}
    nuget-source: 'https://nuget.pkg.github.com/${{ github.repository_owner }}/index.json'
```

### 🏷️ tag-github-origin

**Purpose:** Creates a git tag on the GitHub repository using the REST API.

**Location:** `actions/tag-github-origin`

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
      uses: simplify9/SW-Workflows/actions/dotnet-build@main
      with:
        projects: 'src/**/*.csproj'
        test-projects: 'tests/**/*Tests.csproj'
        run-tests: 'true'

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
      uses: simplify9/SW-Workflows/actions/dotnet-build@main
      with:
        projects: 'src/**/*.csproj'
        run-tests: 'false'
    
    - name: Pack and Push
      uses: simplify9/SW-Workflows/actions/dotnet-pack-push@main
      with:
        projects: 'src/*/*.csproj'
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
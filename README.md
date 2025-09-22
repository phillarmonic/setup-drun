# Setup drun GitHub Action

A GitHub Action to install and setup [drun](https://github.com/phillarmonic/drun) - a declarative task runner for DevOps workflows.

## Features

- ✅ **Cross-platform support**: Linux, macOS, and Windows
- ✅ **Multi-architecture**: AMD64 and ARM64
- ✅ **Version flexibility**: Install latest or specific versions
- ✅ **Caching support**: Cache downloaded binaries for faster builds
- ✅ **Zero dependencies**: No additional tools required
- ✅ **GitHub token support**: Avoid API rate limiting

## Usage

### Basic Usage

```yaml
- name: Setup drun
  uses: phillarmonic/drun@v1
```

### Specify Version

```yaml
- name: Setup drun
  uses: phillarmonic/drun@v1
  with:
    version: 'v1.0.0'
```

### With Caching Disabled

```yaml
- name: Setup drun
  uses: phillarmonic/drun@v1
  with:
    version: 'latest'
    cache: 'false'
```

### With Custom GitHub Token

```yaml
- name: Setup drun
  uses: phillarmonic/drun@v1
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `version` | Version of drun to install (e.g., "v1.0.0", "latest") | No | `latest` |
| `token` | GitHub token for API requests (to avoid rate limiting) | No | `${{ github.token }}` |
| `cache` | Enable caching of downloaded binaries | No | `true` |

## Outputs

| Output | Description |
|--------|-------------|
| `version` | The version of drun that was installed |
| `path` | Path to the installed drun binary |
| `cache-hit` | Whether the binary was restored from cache |

## Example Workflows

### Simple CI/CD Pipeline

```yaml
name: CI/CD with drun

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup drun
        uses: phillarmonic/drun@v1
        with:
          version: 'latest'
      
      - name: Run tests
        run: drun test
      
      - name: Build application
        run: drun build
```

### Multi-platform Testing

```yaml
name: Multi-platform Test

on: [push, pull_request]

jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
    
    runs-on: ${{ matrix.os }}
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup drun
        uses: phillarmonic/drun@v1
        id: drun
      
      - name: Show drun info
        run: |
          echo "Installed version: ${{ steps.drun.outputs.version }}"
          echo "Binary path: ${{ steps.drun.outputs.path }}"
          echo "Cache hit: ${{ steps.drun.outputs.cache-hit }}"
      
      - name: Run drun tasks
        run: |
          drun --version
          drun test
```

### Version Matrix Testing

```yaml
name: Version Matrix Test

on: [push, pull_request]

jobs:
  test:
    strategy:
      matrix:
        drun-version: ['v1.0.0', 'v1.1.0', 'latest']
        os: [ubuntu-latest, macos-latest]
    
    runs-on: ${{ matrix.os }}
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup drun ${{ matrix.drun-version }}
        uses: phillarmonic/drun@v1
        with:
          version: ${{ matrix.drun-version }}
      
      - name: Test with drun
        run: drun test
```

### Docker Build Pipeline

```yaml
name: Docker Build with drun

on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup drun
        uses: phillarmonic/drun@v1
      
      - name: Build Docker image
        run: drun docker:build
      
      - name: Run tests in container
        run: drun docker:test
      
      - name: Push to registry
        if: github.event_name == 'push' && startsWith(github.ref, 'refs/tags/')
        run: drun docker:push
```

### Kubernetes Deployment

```yaml
name: Deploy to Kubernetes

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup drun
        uses: phillarmonic/drun@v1
      
      - name: Configure kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Deploy to staging
        run: drun k8s:deploy --env=staging
      
      - name: Run health checks
        run: drun k8s:health-check --env=staging
      
      - name: Deploy to production
        if: success()
        run: drun k8s:deploy --env=production
```

## Supported Platforms

| OS | Architecture | Status |
|----|--------------|--------|
| Linux | AMD64 | ✅ |
| Linux | ARM64 | ✅ |
| macOS | AMD64 | ✅ |
| macOS | ARM64 | ✅ |
| Windows | AMD64 | ✅ |
| Windows | ARM64 | ✅ |

## Caching

The action automatically caches downloaded binaries to speed up subsequent runs. The cache key includes:
- drun version
- Platform (OS + architecture)

To disable caching, set the `cache` input to `'false'`.

## Rate Limiting

The action uses the GitHub API to fetch release information. To avoid rate limiting:

1. The action uses `${{ github.token }}` by default
2. For private repositories or higher rate limits, provide a custom token via the `token` input

## Troubleshooting

### Binary Not Found

If you see "drun: command not found":

1. Check that the action completed successfully
2. Verify the platform is supported
3. Check the GitHub Actions logs for download errors

### Rate Limiting

If you encounter GitHub API rate limiting:

1. Provide a GitHub token via the `token` input
2. Use a specific version instead of "latest" to reduce API calls

### Version Not Found

If a specific version is not found:

1. Check available releases: https://github.com/phillarmonic/drun/releases
2. Ensure the version format is correct (e.g., "v1.0.0")
3. Use "latest" to get the most recent release

## Contributing

Issues and pull requests are welcome! Please see the [main repository](https://github.com/phillarmonic/drun) for contribution guidelines.

## License

This action is distributed under the same license as drun. See the [LICENSE](LICENSE) file for details.

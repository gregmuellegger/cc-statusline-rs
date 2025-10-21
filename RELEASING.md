# Release Process

This document describes how to create a new release of cc-statusline-rs.

## Creating a Release

The release process is automated using GitHub Actions. To create a new release:

1. **Update the version** (if needed)

   Update the version in `Cargo.toml`:
   ```toml
   [package]
   version = "0.2.0"  # Update this
   ```

2. **Commit any changes**

   ```bash
   git add Cargo.toml
   git commit -m "Bump version to 0.2.0"
   git push
   ```

3. **Create and push a tag**

   ```bash
   # Create a tag with the version number (must start with 'v')
   git tag v0.2.0

   # Push the tag to GitHub
   git push origin v0.2.0
   ```

4. **GitHub Actions will automatically:**
   - Create a GitHub release
   - Build binaries for multiple platforms:
     - Linux x86_64 (glibc)
     - Linux x86_64 (musl, static)
     - macOS x86_64 (Intel)
     - macOS aarch64 (Apple Silicon)
   - Upload the binaries as release assets
   - Generate and upload SHA256 checksums

## Release Artifacts

Each release includes the following artifacts:

- `statusline-linux-x86_64-v{version}.tar.gz` - Linux (glibc)
- `statusline-linux-x86_64-musl-v{version}.tar.gz` - Linux (static, musl)
- `statusline-macos-x86_64-v{version}.tar.gz` - macOS Intel
- `statusline-macos-aarch64-v{version}.tar.gz` - macOS Apple Silicon
- Corresponding `.sha256` checksum files for each binary

## Installing from GitHub Releases

Users can download pre-built binaries from the releases page:

```bash
# Example: Download and install for Linux x86_64
VERSION="0.2.0"
curl -L -O https://github.com/gregmuellegger/cc-statusline-rs/releases/download/v${VERSION}/statusline-linux-x86_64-v${VERSION}.tar.gz

# Verify checksum (optional)
curl -L -O https://github.com/gregmuellegger/cc-statusline-rs/releases/download/v${VERSION}/statusline-linux-x86_64-v${VERSION}.tar.gz.sha256
shasum -a 256 -c statusline-linux-x86_64-v${VERSION}.tar.gz.sha256

# Extract and install
tar xzf statusline-linux-x86_64-v${VERSION}.tar.gz
mkdir -p ~/.claude
mv statusline ~/.claude/cc-statusline-rs
chmod +x ~/.claude/cc-statusline-rs
```

## Version Numbering

This project follows [Semantic Versioning](https://semver.org/):

- **MAJOR** version for incompatible API changes
- **MINOR** version for added functionality in a backwards compatible manner
- **PATCH** version for backwards compatible bug fixes

## Troubleshooting

### Release Failed to Build

If the GitHub Actions workflow fails:

1. Check the Actions tab on GitHub for error details
2. Common issues:
   - Compilation errors on specific platforms
   - Missing dependencies in the CI environment
   - Invalid tag format (must be `v*.*.*`)

### Tag Already Exists

If you need to re-create a release with the same version:

```bash
# Delete the tag locally
git tag -d v0.2.0

# Delete the tag remotely
git push origin :refs/tags/v0.2.0

# Delete the release on GitHub (via web UI)

# Re-create and push the tag
git tag v0.2.0
git push origin v0.2.0
```

## CI/CD Workflow

The release workflow (`.github/workflows/release.yml`) is triggered automatically when a tag matching `v*.*.*` is pushed to the repository.

The workflow consists of two jobs:

1. **create-release**: Creates the GitHub release
2. **build-release**: Builds binaries for all platforms in parallel and uploads them

Build time per platform: ~3-5 minutes (first build)
Total release time: ~5-7 minutes (jobs run in parallel)

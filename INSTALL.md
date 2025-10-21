# Installation Guide for cc-statusline-rs

This guide provides two installation methods: using pre-built binaries or building from source.

## Option 1: Install from GitHub Releases (Recommended)

The easiest way to install cc-statusline-rs is to download a pre-built binary from the [GitHub Releases](https://github.com/gregmuellegger/cc-statusline-rs/releases) page.

### Quick Install

```bash
# Set the version you want to install
VERSION="0.1.0"  # Check releases page for latest version

# Detect your platform
OS=$(uname -s | tr '[:upper:]' '[:lower:]')
ARCH=$(uname -m)

# Download the appropriate binary
if [ "$OS" = "linux" ]; then
    PLATFORM="linux-x86_64"
elif [ "$OS" = "darwin" ]; then
    if [ "$ARCH" = "arm64" ]; then
        PLATFORM="macos-aarch64"
    else
        PLATFORM="macos-x86_64"
    fi
fi

# Download and install
curl -L -O https://github.com/gregmuellegger/cc-statusline-rs/releases/download/v${VERSION}/statusline-${PLATFORM}-v${VERSION}.tar.gz
tar xzf statusline-${PLATFORM}-v${VERSION}.tar.gz
mkdir -p ~/.claude
mv statusline ~/.claude/cc-statusline-rs
chmod +x ~/.claude/cc-statusline-rs

# Update settings.json (requires Python 3)
python3 -c "import json; \
data = json.load(open('~/.claude/settings.json')) if os.path.exists('~/.claude/settings.json') else {}; \
data['statusLine'] = {'type': 'command', 'command': '~/.claude/cc-statusline-rs'}; \
json.dump(data, open('~/.claude/settings.json', 'w'), indent=2)"

echo "Installation complete!"
```

### Available Platforms

- `statusline-linux-x86_64` - Linux (glibc)
- `statusline-linux-x86_64-musl` - Linux (static, no dependencies)
- `statusline-macos-x86_64` - macOS Intel
- `statusline-macos-aarch64` - macOS Apple Silicon

## Option 2: Build from Source

If you prefer to build from source or if a pre-built binary is not available for your platform, follow the instructions below.

### System Requirements

### Required Dependencies

1. **Rust Toolchain** (cargo and rustc)
   - Version: 1.90.0 or newer recommended
   - Install via: https://rustup.rs/

2. **Python 3**
   - Version: 3.11+ recommended
   - Used by Makefile to update settings.json

3. **Build Tools**
   - gcc/clang compiler
   - make
   - pkg-config

4. **OpenSSL Development Libraries**
   - Required for the `openssl-sys` crate
   - Provides SSL/TLS support for `reqwest`

5. **Git**
   - Required for cargo to fetch dependencies using git protocol

## Platform-Specific Installation

### Ubuntu/Debian

```bash
sudo apt-get update
sudo apt-get install -y \
  build-essential \
  libssl-dev \
  pkg-config \
  git \
  python3
```

### Fedora/RHEL/CentOS

```bash
sudo dnf install -y \
  gcc \
  openssl-devel \
  pkg-config \
  git \
  python3 \
  make
```

### macOS

```bash
# Install Xcode Command Line Tools
xcode-select --install

# Install Homebrew if not already installed
# /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# OpenSSL should be available via Xcode tools
# Python3 is included in recent macOS versions
```

### Arch Linux

```bash
sudo pacman -S --needed \
  base-devel \
  openssl \
  pkg-config \
  git \
  python
```

## Cargo Configuration (Network Issues)

If you encounter network errors when accessing crates.io (e.g., "Access denied" from sparse index), configure Cargo to use the git protocol instead:

```bash
mkdir -p ~/.cargo
cat > ~/.cargo/config.toml << 'EOF'
[registries.crates-io]
protocol = "git"

[net]
git-fetch-with-cli = true

[http]
check-revoke = false
EOF
```

This configuration:
- Uses git protocol instead of the sparse registry index
- Enables git CLI for fetching (more reliable in some network environments)
- Disables certificate revocation checking (may help with some network configurations)

## Build and Install

Once all dependencies are installed:

```bash
# Clone the repository (if not already done)
# git clone <repository-url>
# cd cc-statusline-rs

# Build and install
make install
```

The `make install` command will:
1. Build the release binary with `cargo build --release`
2. Copy the binary to `~/.claude/cc-statusline-rs`
3. Update `~/.claude/settings.json` with the statusline configuration

## Verification

After installation, verify the binary was created:

```bash
ls -lh ~/.claude/cc-statusline-rs
```

The binary should be approximately 685 KB in size.

## Troubleshooting

### Network Errors During Build

**Problem**: `error: failed to get successful HTTP response from https://index.crates.io/config.json`

**Solution**: Apply the Cargo configuration above to use git protocol instead of sparse index.

### OpenSSL Linking Errors

**Problem**: `error: linking with cc failed` or `cannot find -lssl`

**Solution**: Install OpenSSL development headers:
- Ubuntu/Debian: `sudo apt-get install libssl-dev`
- Fedora/RHEL: `sudo dnf install openssl-devel`
- macOS: Usually included with Xcode tools

### Missing pkg-config

**Problem**: `error: failed to run custom build command for openssl-sys`

**Solution**: Install pkg-config:
- Ubuntu/Debian: `sudo apt-get install pkg-config`
- Fedora/RHEL: `sudo dnf install pkg-config`
- macOS: `brew install pkg-config`

### Python Not Found

**Problem**: Makefile fails when updating settings.json

**Solution**: Install Python 3:
- Ubuntu/Debian: `sudo apt-get install python3`
- Fedora/RHEL: `sudo dnf install python3`
- macOS: `brew install python3` (or use system python3)

## Build Time

Expected build time on first compilation: 3-4 minutes
(Subsequent builds are much faster due to caching)

## Dependencies Installed by Cargo

The following Rust crates will be automatically downloaded and compiled:
- `serde` and `serde_json` - JSON parsing
- `chrono` - Date/time handling
- `reqwest` - HTTP client (with OpenSSL backend)
- Plus approximately 95 transitive dependencies

Total download size: ~15 MB
Compiled artifact size: ~685 KB (release build)

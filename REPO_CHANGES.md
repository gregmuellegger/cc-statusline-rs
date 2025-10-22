# Repository Changes Log

This file documents temporary changes made to the repository during debugging and development.

## 2025-10-22: Fix GitHub Actions Release Build Failure

**Problem**: The GitHub Actions release workflow was failing when building for `x86_64-unknown-linux-musl` target. The build failed with OpenSSL-related errors because the `reqwest` crate was trying to use OpenSSL which is not easily available for musl targets.

**Root Cause**:
- The `reqwest` dependency uses OpenSSL by default on Linux
- When cross-compiling to musl targets, OpenSSL libraries are not available
- Error: `failed to run custom build command for openssl-sys v0.9.109`

**Solution Applied**:
- Modified `Cargo.toml` to use `rustls-tls` instead of OpenSSL for the `reqwest` crate
- Changed from: `reqwest = { version = "0.11", features = ["blocking", "json"] }`
- Changed to: `reqwest = { version = "0.11", features = ["blocking", "json", "rustls-tls"], default-features = false }`
- This uses a pure Rust TLS implementation that works with musl targets

**Files Modified**:
- `Cargo.toml` - Updated reqwest dependency to use rustls-tls
- `Cargo.lock` - Updated lockfile with new dependencies

**Testing Status**:
- Local compilation verified with `cargo check` ✓
- GitHub Actions workflow run completed successfully ✓
- All targets built successfully:
  - ubuntu-latest, x86_64-unknown-linux-gnu ✓ (53s)
  - ubuntu-latest, x86_64-unknown-linux-musl ✓ (2m38s) - **Previously failing, now fixed!**
  - macos-latest, x86_64-apple-darwin ✓ (1m7s)
  - macos-latest, aarch64-apple-darwin ✓ (51s)

**Test Tag**: v0.2.1-test
**Workflow Run**: https://github.com/gregmuellegger/cc-statusline-rs/actions/runs/18715248692

**Next Steps**:
- The fix is confirmed working for all targets
- To clean up: Delete the test tag `v0.2.1-test` and test release when ready
- The actual v0.2.0 tag can be re-created to trigger a proper release, or bump to v0.2.1 for the next release

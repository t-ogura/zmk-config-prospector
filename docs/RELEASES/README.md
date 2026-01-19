# Release Notes

This directory contains release notes for all versions of Prospector Scanner.

## Release History

| Version | Release Date | Status | Highlights |
|---------|-------------|--------|------------|
| [v2.1.0](v2.1.0/release_notes.md) | 2026-01 | **Latest Stable** | Zephyr 4.x compatibility, latency optimization, layer limit fix, sleep/wake recovery |
| [v2.0.0](v2.0.0/release_notes.md) | 2025-11-20 | Stable | Touch panel support, USB display fix, thread safety, timeout brightness |
| [v1.1.1](v1.1.1/release_notes.md) | 2025-08-29 | Stable | Universal compatibility, 10-layer support, Device Tree fallback |
| [v1.1.0](v1.1.0/release_notes.md) | 2025-08-15 | Stable | 15x power optimization, battery support, ambient light sensor |
| [v1.0.0](v1.0.0/release_notes.md) | 2025-08-03 | Stable | Initial stable release with YADS-style UI |

## Version Numbering

We follow semantic versioning (MAJOR.MINOR.PATCH):
- **MAJOR**: Breaking changes that require migration
- **MINOR**: New features, backward compatible
- **PATCH**: Bug fixes and minor improvements

## Release Process

1. **Development**: Features developed on feature branches
2. **Testing**: Community testing and validation
3. **Documentation**: Release notes and migration guides
4. **Tagging**: Git tag creation (e.g., `v1.1.0`)
5. **Release**: GitHub Release with pre-built firmware

## Quick Links

- **Latest Release**: [Download v2.1.0](https://github.com/t-ogura/zmk-config-prospector/releases/latest)
- **All Releases**: [GitHub Releases](https://github.com/t-ogura/zmk-config-prospector/releases)
- **Migration Guide**: See each version's release notes for upgrade instructions
# Talos Core Utils Extension - Build Instructions

A Talos Linux system overlay extension that adds the `cat` command from GNU coreutils to the Talos host OS.

## Overview

This extension provides the `cat` command from GNU coreutils as a system extension for Talos Linux. It follows the official Siderolabs extension format and can be built and integrated into custom Talos images.

## Extension Structure

The extension consists of three main files located in `tools/coreutils/`:

- **pkg.yaml** - Build instructions (download, configure, compile, install)
- **manifest.yaml.tmpl** - Extension metadata and compatibility information
- **vars.yaml** - Version variables for the build process

## Prerequisites

To build this extension, you need:

1. **Docker** (19.03+) with experimental features enabled
2. **docker buildx** - Create a builder instance:
   ```bash
   docker buildx create --name local --use
   ```

3. **bldr** (Siderolabs build tool) - Download the appropriate binary:
   ```bash
   # For Linux x86_64
   curl -sSL https://github.com/siderolabs/bldr/releases/download/v0.5.6/bldr-Linux-amd64 -o bldr
   chmod +x bldr
   
   # For Linux ARM64
   curl -sSL https://github.com/siderolabs/bldr/releases/download/v0.5.6/bldr-Linux-arm64 -o bldr
   chmod +x bldr
   
   # For macOS x86_64
   curl -sSL https://github.com/siderolabs/bldr/releases/download/v0.5.6/bldr-Darwin-amd64 -o bldr
   chmod +x bldr
   
   # For macOS ARM64
   curl -sSL https://github.com/siderolabs/bldr/releases/download/v0.5.6/bldr-Darwin-arm64 -o bldr
   chmod +x bldr
   ```

## Building the Extension

### Build locally (output to directory)

This builds the extension and outputs the contents locally:

```bash
./bldr build --target coreutils --output=type=local,dest=_out
```

The extension files will be in `_out/rootfs/`.

### Build and push to registry

To build and push to a container registry:

```bash
./bldr build --target coreutils \
  --tag ghcr.io/Wu-Wu/coreutils:latest \
  --push
```

### Build for multiple architectures

Build for both amd64 and arm64:

```bash
./bldr build --target coreutils \
  --tag ghcr.io/Wu-Wu/coreutils:v1.0.0 \
  --platform linux/amd64,linux/arm64 \
  --push
```

### Using environment variables

You can also set registry and username via environment variables:

```bash
export REGISTRY=ghcr.io
export USERNAME=Wu-Wu

./bldr build --target coreutils \
  --tag $REGISTRY/$USERNAME/coreutils:v1.0.0 \
  --push
```

## Verifying the Build

To verify the extension was built correctly:

```bash
# Check the local output
ls -la _out/rootfs/usr/local/bin/

# Should show: cat binary exists
```

## Using in Custom Talos Image

Once the extension is built and pushed to a registry, you can reference it in your Talos machine configuration:

```yaml
machine:
  install:
    extensions:
      - image: ghcr.io/Wu-Wu/coreutils:v1.0.0
```

Then use `talosctl gen config` and `talosctl upgrade` to apply the custom Talos image with the extension.

## Build Process

The build process follows these steps (defined in `tools/coreutils/pkg.yaml`):

1. **Download** - Fetches GNU coreutils source from `ftp.gnu.org`
   - Downloads the specific version specified in `vars.yaml`
   - Verifies SHA256 and SHA512 checksums

2. **Prepare** - Extracts the source tarball
   - Extracts GNU coreutils source archive

3. **Configure** - Configures coreutils with minimal dependencies:
   - Sets prefix to `/usr/local`
   - Disables NLS (National Language Support)
   - Disables GMP support
   - Disables libattr and libacl
   - This keeps the build lightweight

4. **Build** - Compiles only the `cat` binary
   - Uses parallel make (`-j $(nproc)`) for faster compilation
   - Only builds the cat utility, not entire coreutils suite

5. **Install** - Installs the stripped binary
   - Installs compiled binary to `/rootfs/usr/local/bin/cat`
   - Strips the binary to reduce size

6. **Test** - Validates the extension
   - Uses `extensions-validator` to ensure compliance
   - Verifies rootfs structure and manifest

## Extension Metadata

The extension is marked as **contrib tier** (community-supported) and is compatible with Talos v1.0.0 and later.

**Extension Details:**
- **Name:** coreutils
- **Tier:** contrib
- **Author:** Wu-Wu
- **Description:** This system extension provides GNU coreutils cat command.
- **Talos Compatibility:** >= v1.0.0

## File Structure in Extension Image

After building, the extension contains:

```
extension-image/
├── manifest.yaml
└── rootfs/
    └── usr/
        └── local/
            └── bin/
                └── cat
```

## File Locations

After installation in Talos, the `cat` command will be available at:

```
/usr/local/bin/cat
```

You can verify it's available:

```bash
talosctl -n <node> run "which cat"
# Output: /usr/local/bin/cat

talosctl -n <node> run "cat --version"
# Output: cat (GNU coreutils) <version>
```

## Troubleshooting

### Build fails with "command not found: bldr"

Make sure `bldr` is in your PATH or use `./bldr` to run it from the current directory.

### Docker buildx not found

Create a builder instance:
```bash
docker buildx create --name local --use
```

### Registry authentication issues

If pushing to a private registry, authenticate first:
```bash
docker login ghcr.io
```

### Build takes too long

The build caches dependencies. Subsequent builds will be faster. You can also enable more aggressive caching in your buildx configuration.

## Configuration Variables

The build can be customized by modifying `tools/coreutils/vars.yaml`:

```yaml
VERSION: "{{ .COREUTILS_VERSION }}"
TIER: "contrib"
```

And `tools/coreutils/pkg.yaml` for build flags and configuration options.

## Notes

- This extension uses **bldr**, the official Siderolabs build tool
- The extension follows the exact same pattern as official Siderolabs extensions like `util-linux-tools`
- Only the `cat` binary is included (minimal footprint)
- The extension is optimized for the Talos host OS only
- Binary is stripped to reduce size
- Build is reproducible and deterministic
- Cross-compilation is supported (build for different architectures)

## Related Documentation

- [Talos Linux System Extensions](https://www.talos.dev/latest/talos-guides/configuration/system-extensions/)
- [Siderolabs Extensions Repository](https://github.com/siderolabs/extensions)
- [bldr Build Tool](https://github.com/siderolabs/bldr)
- [GNU Coreutils](https://www.gnu.org/software/coreutils/)

## Contributing

If you want to extend this extension to include other GNU coreutils commands, modify the `build` step in `tools/coreutils/pkg.yaml` to build additional utilities.

For example, to include both `cat` and `ls`:
```bash
cd build
make -j $(nproc) cat ls
```

## License

This extension repository is provided as-is for use with Talos Linux.

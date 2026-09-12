# Image to Normal Map CLI

Version: **0.10.0**. Prebuilt executables; no Rust installation is required.

## Layout

```text
cli/
  macos/image-to-normal-map
  linux/image-to-normal-map
  windows/image-to-normal-map.exe
LICENSE.md
SUPPORT.md
THIRD_PARTY_NOTICES.md
Readme.md
```

Run the following commands from this directory. Replace input paths with your own images.

## macOS

Universal binary for Apple Silicon and Intel Macs.

```bash
chmod +x ./cli/macos/image-to-normal-map
./cli/macos/image-to-normal-map --version
./cli/macos/image-to-normal-map --help
./cli/macos/image-to-normal-map input.png --alpha-map
./cli/macos/image-to-normal-map input.png --strength 2.0 --blur-radius 1.0 --normal-z 0.75 -vv
```

This distribution is not Apple Developer ID signed or notarized. macOS may display a security warning. Only allow execution after verifying the source.

## Linux: Rocky / Debian / Ubuntu

Cross-compiled for x86_64 with a glibc 2.17 baseline. Intended for Rocky Linux 8+, Debian 10+, Ubuntu 18.04+, and compatible systems. Not an ARM Linux binary. Distribution-specific runtime testing has not been performed.

```bash
chmod +x ./cli/linux/image-to-normal-map
./cli/linux/image-to-normal-map --version
./cli/linux/image-to-normal-map --help
./cli/linux/image-to-normal-map input.png --alpha-map
./cli/linux/image-to-normal-map input.png --strength 2.0 --blur-radius 1.0 --normal-z 0.75 -vv
```

## Windows 10 and Later

x64 executable. Windows runtime testing has not been performed. In PowerShell:

```powershell
.\cli\windows\image-to-normal-map.exe --version
.\cli\windows\image-to-normal-map.exe --help
.\cli\windows\image-to-normal-map.exe "C:\images\input.png" --alpha-map
.\cli\windows\image-to-normal-map.exe "C:\images\input.png" --strength 2.0 --blur-radius 1.0 --normal-z 0.75 -vv
```

## Output and Options

By default, output is saved next to the input as `[name]-normal.[extension]`. With `--alpha-map`, `[name]-alpha.[extension]` is also generated. Existing output files are overwritten; use a different output path to preserve earlier results.

| Option | Purpose | Default |
|---|---|---|
| `--output PATH` | Normal or binary output path | Automatic |
| `--alpha-map` | Generate alpha map | Disabled |
| `--alpha-output PATH` | Separate alpha output; not available in binary mode | Automatic |
| `--exclude-colors '#ffffff,#ffff00'` | Exclude multiple colors | None |
| `--color-tolerance 8` | Per-channel color tolerance (0..255) | 0 |
| `--strength 2.0` | X/Y slope strength; positive finite value | 2.0 |
| `--blur-radius 1.0` | Blur slopes after strength (0..32) | 0 |
| `--normal-z 0.75` | Z before normalization; positive finite value | 1.0 |
| `--invert-y` | Flip green/Y direction | Disabled |
| `--detect-shadow` | Estimate low-chroma shadows and assign partial alpha | Disabled |
| `--shadow-chroma-threshold 18` | Maximum RGB channel difference (0..255) | 18 |
| `--shadow-min-luma 140` | Minimum shadow luminance (0..255) | 140 |
| `--shadow-max-alpha 160` | Maximum shadow alpha (0..255) | 160 |
| `--output-format image` | Image output | image |
| `--output-format uint32` | C header output | Optional |
| `--output-format uint32-binary` | Binary container output | Optional |
| `--original-map` | Include original RGBA in binary output | Disabled |
| `--stdout` | Send binary output to standard output | Disabled |
| `-v`, `-vv` | Settings, timings, and additional diagnostics | Disabled |

Fully transparent pixels are automatically excluded. Shadow detection is a color-based heuristic and may misclassify gray weapons, clothing, or grid lines. JPEG cannot preserve transparency; use PNG or WebP instead. Original pixel output is not changed by excluded colors or shadow processing.

Pipeline: luminance -> Sobel -> strength -> optional slope blur -> normal-z -> normalization -> RGB.

## C Headers

```bash
./cli/macos/image-to-normal-map input.png --output-format uint32 --alpha-map
```

On Linux, substitute `./cli/linux/image-to-normal-map`. On Windows:

```powershell
.\cli\windows\image-to-normal-map.exe input.png --output-format uint32 --alpha-map
```

Creates `input-normal.h` and `input-alpha.h`, including dimensions and pixel arrays. Original/normal pixels use `0xAABBGGRR`; alpha uses opaque grayscale `0xFFVVVVVV`.

## Binary Output

For file output, `--output maps.bin` works on all supported platforms:

```bash
./cli/linux/image-to-normal-map input.png --output-format uint32-binary --original-map --alpha-map --output maps.bin
```

```powershell
.\cli\windows\image-to-normal-map.exe input.png --output-format uint32-binary --original-map --alpha-map --output maps.bin
```

macOS/Linux streaming:

```bash
./cli/macos/image-to-normal-map input.png --output-format uint32-binary --original-map --alpha-map --stdout -vv > maps.bin
```

Avoid binary redirection through older PowerShell versions, which may perform text conversion. Prefer `--output`, or execute the following in **cmd.exe**:

```bat
cli\windows\image-to-normal-map.exe input.png --output-format uint32-binary --original-map --alpha-map --stdout > maps.bin
```

### Stream v2 Layout

All integers and pixels are little-endian. Pixels are row-major, starting at the top-left.

```text
ITNMAP\0\x02 (8 bytes)
width (u32)
height (u32)
flags (u32: bit 0 = original, bit 1 = alpha)
original pixel count (u32), then original pixels
alpha pixel count (u32), then alpha pixels
normal pixel count (u32), then normal pixels
```

Absent optional sections have count 0. Normal is always included. Logs go to stderr in stdout mode. This layout is not compatible with v1 parsers.

## Project License

See [LICENSE.md](LICENSE.md) for the License & Terms of Use. Commercial inquiries: **dev@hobbyino.com**.

[SUPPORT.md](SUPPORT.md) preserves the previous support and commercial inquiry text, including its original CC BY-NC reference. That reference differs from the new terms in LICENSE.md. The existing executables also retain the previous license help text and have not been rebuilt for this documentation change.

## Third-Party Notices

See [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) for the recorded dependency inventory. Third-party licenses remain unchanged. Verify copyright and license-text redistribution requirements before redistributing these binaries.

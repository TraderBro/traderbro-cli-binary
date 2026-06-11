# TraderBro CLI — Pre-built Binaries

> **Auto-updated on every push to main.**
> Last build: commit `7812cdd` at `2026-06-11T09:29:07Z`

This repository hosts pre-built binaries of the [TraderBro CLI](https://github.com/TraderBro/traderbro-cli).
Versioned releases (tagged `v*`) are also published here by GoReleaser.

---

## Download Links (Latest Build)

| OS      | Architecture          | File | Format |
|---------|-----------------------|------|--------|
| macOS   | Apple Silicon (arm64) | [traderbro_darwin_arm64.tar.gz](https://github.com/TraderBro/traderbro-cli-binary/releases/download/latest/traderbro_darwin_arm64.tar.gz) | tar.gz |
| macOS   | Intel (amd64)         | [traderbro_darwin_amd64.tar.gz](https://github.com/TraderBro/traderbro-cli-binary/releases/download/latest/traderbro_darwin_amd64.tar.gz) | tar.gz |
| Linux   | x86_64 (amd64)        | [traderbro_linux_amd64.tar.gz](https://github.com/TraderBro/traderbro-cli-binary/releases/download/latest/traderbro_linux_amd64.tar.gz) | tar.gz |
| Linux   | ARM64                 | [traderbro_linux_arm64.tar.gz](https://github.com/TraderBro/traderbro-cli-binary/releases/download/latest/traderbro_linux_arm64.tar.gz) | tar.gz |
| Windows | x86_64 (amd64)        | [traderbro_windows_amd64.zip](https://github.com/TraderBro/traderbro-cli-binary/releases/download/latest/traderbro_windows_amd64.zip) | zip |

Checksums: [traderbro_checksums.txt](https://github.com/TraderBro/traderbro-cli-binary/releases/download/latest/traderbro_checksums.txt)

---

## Quick Install

### macOS — Apple Silicon (arm64)
```bash
curl -fsSL https://github.com/TraderBro/traderbro-cli-binary/releases/download/latest/traderbro_darwin_arm64.tar.gz | tar -xz && sudo mv traderbro /usr/local/bin/
```

### macOS — Intel (amd64)
```bash
curl -fsSL https://github.com/TraderBro/traderbro-cli-binary/releases/download/latest/traderbro_darwin_amd64.tar.gz | tar -xz && sudo mv traderbro /usr/local/bin/
```

### Linux — x86_64 (amd64)
```bash
curl -fsSL https://github.com/TraderBro/traderbro-cli-binary/releases/download/latest/traderbro_linux_amd64.tar.gz | tar -xz && sudo mv traderbro /usr/local/bin/
```

### Linux — ARM64
```bash
curl -fsSL https://github.com/TraderBro/traderbro-cli-binary/releases/download/latest/traderbro_linux_arm64.tar.gz | tar -xz && sudo mv traderbro /usr/local/bin/
```

### Windows — x86_64 (amd64)
Download [traderbro_windows_amd64.zip](https://github.com/TraderBro/traderbro-cli-binary/releases/download/latest/traderbro_windows_amd64.zip), extract, and add `traderbro.exe` to your `PATH`.

---

## For AI Agents

If you are an AI agent or automated script that needs to install the TraderBro CLI, follow these steps:

1. **Detect OS**: run `uname -s`
   - `Darwin` → macOS
   - `Linux` → Linux
   - `MINGW*` / `MSYS*` / Windows → Windows

2. **Detect architecture**: run `uname -m`
   - `arm64` or `aarch64` → arm64
   - `x86_64` or `amd64` → amd64

3. **Select the download URL** from the table above and download + extract:
   - macOS arm64  → `https://github.com/TraderBro/traderbro-cli-binary/releases/download/latest/traderbro_darwin_arm64.tar.gz`
   - macOS amd64  → `https://github.com/TraderBro/traderbro-cli-binary/releases/download/latest/traderbro_darwin_amd64.tar.gz`
   - Linux arm64  → `https://github.com/TraderBro/traderbro-cli-binary/releases/download/latest/traderbro_linux_arm64.tar.gz`
   - Linux amd64  → `https://github.com/TraderBro/traderbro-cli-binary/releases/download/latest/traderbro_linux_amd64.tar.gz`
   - Windows amd64 → `https://github.com/TraderBro/traderbro-cli-binary/releases/download/latest/traderbro_windows_amd64.zip`

4. **Install**:
   ```bash
   curl -fsSL <url> | tar -xz && sudo mv traderbro /usr/local/bin/
   ```

5. **Verify**:
   ```bash
   traderbro --version
   ```

---

## About

- Source: [TraderBro/traderbro-cli](https://github.com/TraderBro/traderbro-cli)
- Docs: [docs.traderbro.ai](https://docs.traderbro.ai)
- Site: [traderbro.ai](https://traderbro.ai)

**Deploy**

This document covers building and releasing Linux binaries for `amd64` and `arm64`, plus install and rollback steps on a target host.

**Prereqs**

1. Docker installed and running.
2. `gh` installed and authenticated.
3. Repo remote set to `k161196/systemctl-tui`.

Set the default repo once:

```sh
gh repo set-default k161196/systemctl-tui
```

**Release Version**

Update `Cargo.toml` and tag/publish with the version you are releasing. Examples below use `v0.5.2`.

**AMD64 Build (Linux x86_64)**

**Build**

```sh
docker run --rm -t --platform=linux/amd64 \
  -v "$PWD":/work -w /work \
  -v cargo-registry:/root/.cargo/registry \
  -v cargo-git:/root/.cargo/git \
  ubuntu:24.04 \
  bash -lc 'apt update && apt install -y curl build-essential pkg-config libssl-dev git ca-certificates && \
            curl --proto "=https" --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --profile minimal && \
            . /root/.cargo/env && \
            cargo build --release'
```

Note: Do not mount `cargo-bin` or `rustup` volumes when building `amd64` from an ARM host. Those toolchains will be the wrong architecture.

**Rename Artifact**

```sh
cp target/release/systemctl-tui systemctl-tui-linux-amd64
```

**Release**

Create a new release:

```sh
gh release create v0.5.2 systemctl-tui-linux-amd64 --generate-notes
```

Upload to an existing release:

```sh
gh release upload v0.5.2 systemctl-tui-linux-amd64 --clobber
```

**Install On Target (amd64 host)**

```sh
sudo install -m 0755 /opt/kiran/systemctl-tui-linux-amd64 /usr/local/bin/systemctl-tui
sudo ln -sf /usr/local/bin/systemctl-tui /usr/local/bin/sm
```

**Remove/Undo**

```sh
sudo rm -f /usr/local/bin/sm
sudo rm -f /usr/local/bin/systemctl-tui
```

**ARM64 Build (Linux aarch64)**

**Build**

```sh
docker run --rm -t --platform=linux/arm64 \
  -v "$PWD":/work -w /work \
  -v cargo-registry:/root/.cargo/registry \
  -v cargo-git:/root/.cargo/git \
  ubuntu:24.04 \
  bash -lc 'apt update && apt install -y curl build-essential pkg-config libssl-dev git ca-certificates && \
            curl --proto "=https" --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --profile minimal && \
            . /root/.cargo/env && \
            cargo build --release'
```

**Rename Artifact**

```sh
cp target/release/systemctl-tui systemctl-tui-linux-arm64
```

**Release**

Upload to an existing release:

```sh
gh release upload v0.5.2 systemctl-tui-linux-arm64 --clobber
```

**Install On Target (arm64 host)**

```sh
sudo install -m 0755 /opt/kiran/systemctl-tui-linux-arm64 /usr/local/bin/systemctl-tui
sudo ln -sf /usr/local/bin/systemctl-tui /usr/local/bin/sm
```

**Remove/Undo**

```sh
sudo rm -f /usr/local/bin/sm
sudo rm -f /usr/local/bin/systemctl-tui
```

**Verify Host Architecture**

```sh
uname -m
```

`x86_64` means amd64, `aarch64` means arm64.

**Verify Binary Architecture**

```sh
file /path/to/systemctl-tui
```

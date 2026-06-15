# jmart-sandbox-images

Public, pure-OSS build sources for the **JMart Studio sandbox** images — the standardized
environment JMart serves into *users'* projects. These are **not** the container that builds JMart
Studio itself, and **not** plugin images.

Two principles drive every choice:

1. **Uniform floor** — the agent stands on an identical base in every sandbox, so its behavior doesn't
   vary by language.
2. **Pure images, mounted IP** — everything published here is pure OSS. JMart's proprietary
   **agent-core** is *mounted in read-only at runtime* by the app, **never baked** into these images.
   Agent updates ship as a new mounted bundle with zero image rebuilds, and no JMart IP ever lands in a
   public image.

## Layer model

```
L3   PROJECT      user's devcontainer.json + deps + bind-mount      most volatile, per-project
L2   TOOLCHAIN    language Feature + version manager                per-image (here)
L1½  DESKTOP      on-demand, opt-in desktop (Xvfb/VNC + kvm.mjs)     ships WITH the app (not here)
L1   AGENT CORE   static musl bundle, mounted read-only             ships WITH the app (not here)
L0   OS BASE      mcr.microsoft.com/devcontainers/base:ubuntu-24.04 pulled, never built
```

## Images

| Source | Publishes | Toolchain |
|---|---|---|
| `sandbox-base/`   | `ghcr.io/jmdisuanco/jmart-sandbox-images/sandbox-base`       | none (general fallback / polyglot) |
| `sandbox-node/`   | `ghcr.io/jmdisuanco/jmart-sandbox-images/sandbox-node`       | node Feature (Node LTS + nvm) — see note |
| `sandbox-python/` | `ghcr.io/jmdisuanco/jmart-sandbox-images/sandbox-python`     | python Feature + `uv` |
| `sandbox-go/`     | `ghcr.io/jmdisuanco/jmart-sandbox-images/sandbox-go`         | go Feature (GOTOOLCHAIN auto-fetch) |
| `sandbox-rust/`   | `ghcr.io/jmdisuanco/jmart-sandbox-images/sandbox-rust`       | rust Feature (rustup + stable) |
| `sandbox-cpp/`    | `ghcr.io/jmdisuanco/jmart-sandbox-images/sandbox-cpp`        | C/C++: build-essential, clang, cmake, ninja, gdb/lldb (Containerfile) |
| `sandbox-java/`   | `ghcr.io/jmdisuanco/jmart-sandbox-images/sandbox-java`       | java Feature (LTS JDK + Maven + Gradle) |
| `sandbox-zig/`    | `ghcr.io/jmdisuanco/jmart-sandbox-images/sandbox-zig`        | zig Feature (incl. `zig cc`) |
| `sandbox-arduino/`| `ghcr.io/jmdisuanco/jmart-sandbox-images/sandbox-arduino`    | `arduino-cli` + `socat` (Containerfile) — MCU |
| `sandbox-platformio/` | `ghcr.io/jmdisuanco/jmart-sandbox-images/sandbox-platformio` | PlatformIO core (venv) + `socat` (Containerfile) — MCU |
| `sandbox-esp/`    | `ghcr.io/jmdisuanco/jmart-sandbox-images/sandbox-esp`        | esptool/espefuse/espsecure (venv) + `socat` (Containerfile) — MCU |
| `sandbox-micropython/` | `ghcr.io/jmdisuanco/jmart-sandbox-images/sandbox-micropython` | mpremote + esptool (venv) + `socat` (Containerfile) — MCU |

Feature-based images are `FROM …/base:ubuntu-24.04` + the official/community Feature; toolchains without
a maintained Feature (C/C++, arduino-cli, PlatformIO, esptools) bake via a `Containerfile`. All pinned,
built with `devcontainer build --push` (see `.github/workflows/build-images.yml`), multi-arch
`linux/amd64,linux/arm64` (arm64 covers the Apple-Silicon sandbox VM).

**MCU images & serial:** `sandbox-{arduino,platformio,esp}` bake `socat` so the **host** serial service
(the existing Serial Monitor's Rust layer, sole owner of the physical port) can be **bridged** to an
in-container PTY (`/dev/ttyJMART`) over the sandbox exec channel — the build/flash tools open that path.
No USB-into-VM passthrough; serial is passed through our existing infra (host serial service + the same
`podman exec` stdio pipe the port-forwarder uses). The wiring + the serial footer UI live in the app.

> **Node version-manager note (spec deviation, flagged):** the spec names **fnm**, but there is no
> maintained `fnm` Dev Container Feature, so `sandbox-node` uses the official node Feature, which bakes
> **nvm**. `match-toolchain node` uses nvm. If fnm's speed matters we can bake it via a small Containerfile
> layer later — it changes nothing for consumers.

## What the app injects at launch (NOT baked here)

The sandbox manager (in the private app) hydrates each image per session:

- mounts **agent-core** read-only at `/opt/jmart-agent` (`source=${localEnv:JMART_AGENT_CORE}`) and
  prepends `/opt/jmart-agent/bin` to `PATH`;
- runs `postCreate: /opt/jmart-agent/bin/match-toolchain <lang>` to match the project's declared version
  (`.python-version`/`pyproject`, `.nvmrc`/`engines`, `go.mod`, `rust-toolchain.toml`);
- bind-mounts the project (rw) + attaches the disposable env volume (deps);
- adds the on-demand **desktop** overlay (which ships with the app, not here) only when the session
  requests the `desktop` capability.

A project's own `.devcontainer/devcontainer.json` is honored as-is — agent-core is mounted, not injected,
so the user's build is untouched.

## License & provenance

Repo sources: MIT (see `LICENSE`). The published images **redistribute** upstream OSS (the base image,
Dev Container Features); their licenses + an SBOM are tracked in `NOTICE.md` (work in progress).

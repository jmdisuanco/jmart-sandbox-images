# NOTICE — redistributed OSS (work in progress)

The `sandbox-*` images published from this repo **redistribute** third-party open-source software. This
file (plus a generated SBOM) will enumerate those components, their licenses, and a source offer. Tracked
as a follow-up per the spec ("SBOM + license bundle + source offer for the redistributed OSS images — one
set covers images *and* the Podman runtime").

Known components to cover:

- **Base image:** `mcr.microsoft.com/devcontainers/base:ubuntu-24.04` (Ubuntu 24.04 + devcontainers base
  tooling) and everything it transitively ships.
- **Dev Container Features:** `ghcr.io/devcontainers/features/{python,node,go,rust}`,
  `ghcr.io/devcontainers-extra/features/uv`.
- **Desktop Feature packages:** xvfb, fluxbox, x11-utils, scrot, xdotool, x11vnc, novnc, websockify.
- **Version managers:** uv, nvm, rustup, Go toolchain.

TODO:
- [ ] Generate an SBOM per image in CI (e.g. `syft` / `trivy`) and attach it to the GHCR artifact.
- [ ] Collect upstream LICENSE texts into `licenses/`.
- [ ] Add a written source offer.

Repo sources themselves are MIT (see `LICENSE`).

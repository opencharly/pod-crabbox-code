# AGENTS.md — pod-crabbox-code

The code-server lease image for the Crabbox `containerd` provider: the `pod-crabbox-code` image
(`charly.yml`) and the variant layer it composes. The lease contract itself belongs
to the base image's repo; this repo owns only the variant delta.

Canonical files:

- `charly.yml` — the image: its pinned base image, user/entrypoint, and candy list.
- `candy/<layer>/charly.yml` — the variant's packages, plan steps and `check:` steps.
- `.github/workflows/image.yml` — the image build; it triggers on
  `tag-on-merge` **completion** (`workflow_run`), never on `push: tags`
  (`opencharly/.github#277`).
- `.github/workflows/tag-on-merge.yml` — the org's CalVer tag + `CHANGELOG/`
  dispatcher, installed by the org bootstrap.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-image:image` — the `candy:` image entry (base, user, entrypoint).
- `/charly-image:layer` — the candy body: `distro:` packages, plan step verbs,
  and the `check:` surface.
- `/charly-distros:cachyos` — the CachyOS base and Arch package semantics.
- `/charly-check:check` — the check verbs, the ADE requirement, and the R10 gate.
- `/charly-internals:git-workflow` — before any git/PR action.

## What is unique here

- **The code-server version is the URL's version.** `candy/code-server/charly.yml`
  carries `CODE_SERVER_VERSION`; the `download:` step fetches that exact release, and
  the check asserts the binary reports it. Bump the var and the check together.
- **The starter binds loopback only.** `--bind-addr 127.0.0.1:8080` is the boundary;
  the lease's SSH session is the only way in. Do not add `--auth password` with a
  baked password or bind `0.0.0.0`.
- The base image's direct-mode entrypoint is inherited: sshd stays PID 1 and the
  readiness hook runs before the consumer starts the editor.

## Build / validate / test

- `charly box validate` at the repo root — the structural check on the image and
  its candy (including the pulled base image at its pinned tag).
- `charly box generate pod-crabbox-code` — inspect the emitted Containerfile and the baked
  `ai.opencharly.description` plan.
- `charly check box pod-crabbox-code` — the build-scope `check:` steps.
- `charly check run check-pod-crabbox-code-serves` — the disposable R10 bed: boots the
  image as a pod, drives the `/crabbox` contract, starts `crabbox-start-code` over SSH as
  `crabbox`, and asserts the installed release ANSWERS over HTTP on its loopback-only
  8080 (plus SSH as `crabbox`, `sudo -n true`, and the readiness hook exiting 0). A
  build-scope check cannot prove a running service answers; this bed can.
- The PR image build (`.github/workflows/image.yml`, `pull_request` path) builds
  the image and publishes nothing; the merge-time `workflow_run` path publishes
  `ghcr.io/opencharly/pod-crabbox-code:<git-tag>`.

## Modify this repo

- Keep the base tag in step: a base-contract fix lands in `opencharly/pod-crabbox` first,
  then this repo's `base:` tag advances (producer-first). The base is a pinned OCI
  artifact, not an `import:`; `charly box reconcile` does not apply to it.
- A package may be added only from a named source (the base candy, an existing
  `layer-*`/`pod-*` repo's Arch section, the provider's docs, or the Arch/AUR
  package index). Record the source in a comment.
- Every claim in the plan needs a deterministic `check:`. A package that merely
  installs without a probe is not proof.
- Never weaken a sandbox boundary: no `seccomp=unconfined`, no `--no-sandbox`.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly`. Do not restate its rules here.

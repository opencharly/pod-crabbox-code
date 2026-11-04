# pod-crabbox-code

The **code-server lease image** for the Crabbox `containerd` provider — the
base [`pod-crabbox`](https://github.com/opencharly/pod-crabbox) lease contract with a
pinned `code-server` release and a loopback-only starter.

## What it provides

| Property | Value |
|---|---|
| Base | CachyOS base image `ghcr.io/opencharly/pod-crabbox:2026.307.1828` |
| SSH | `sshd` on **2222**, public-key only, user `crabbox` |
| code-server | pinned release under `/usr/local/lib/code-server` |
| Starter | `/usr/local/bin/crabbox-start-code` on `127.0.0.1:8080` |

## How to use it

```sh
ssh crabbox@<lease> crabbox-start-code
# then forward 127.0.0.1:8080 over the SSH session
```

## Layout

- `charly.yml` — the `pod-crabbox-code` image (pinned base image + entrypoint).
- `candy/` — the variant's own layer: packages, files and `check:` steps.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `.github/workflows/image.yml` — builds on `pull_request` and publishes at the merge
  tag, so the image tag equals the git tag.
- `CHANGELOG/` — per-CalVer history.

## Related

- [`opencharly/pod-crabbox`](https://github.com/opencharly/pod-crabbox) — the lease
  base image this family builds on.
- [`CONTAINERD-IMAGES.md`](https://github.com/opencharly/opencharly/blob/main/CONTAINERD-IMAGES.md)
  §2 — the lease-image contract.

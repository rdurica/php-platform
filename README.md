# PHP Platform

Shared **FrankenPHP / PHP 8.5** Docker base images, GitHub **Reusable Workflows**, and a shared **Renovate preset** for all applications created from **[rdurica/php-starter-kit](https://github.com/rdurica/php-starter-kit)**.

[![PHP](https://img.shields.io/badge/PHP-8.5-blue.svg)](https://www.php.net/)
[![FrankenPHP](https://img.shields.io/badge/FrankenPHP-1.12-blue.svg)](https://frankenphp.dev/)
[![Platform CI](https://github.com/rdurica/php-platform/actions/workflows/ci.yaml/badge.svg)](https://github.com/rdurica/php-platform/actions/workflows/ci.yaml)
[![Publish Images](https://github.com/rdurica/php-platform/actions/workflows/publish-images.yaml/badge.svg)](https://github.com/rdurica/php-platform/actions/workflows/publish-images.yaml)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

> **Looking to start a new PHP project?**
> Use the **[rdurica/php-starter-kit](https://github.com/rdurica/php-starter-kit)** template repository — it consumes these base images and workflows out of the box.

---

## Architecture

Instead of copying Dockerfiles, Caddy configs, PHP INI settings, and CI pipelines into every application repository, shared infrastructure lives here and is consumed by versioned tags:

```text
  ┌─────────────────────────────────────────────────────────────────────┐
  │                       rdurica/php-platform                          │
  │                                                                     │
  │  • GHCR Base Images     ghcr.io/rdurica/php-platform/{prod,ci,dev}  │
  │  • Reusable Workflows   reusable-{code-quality,ci-checks,build}.yaml│
  │  • Renovate Preset      renovate-preset.json                        │
  └───────────────────┬─────────────────────────────┬───────────────────┘
                      │                             │
      git tag vX.Y.Z  │ (base images + workflows)   │ Renovate PRs
                      ▼                             ▼
  ┌─────────────────────────────────────────────────────────────────────┐
  │                   rdurica/php-starter-kit (template)                │
  │                     └── your application repos                      │
  │                                                                     │
  │  • Thin build/prod/Dockerfile (~12 lines)                           │
  │  • compose.yaml / compose.ci.yaml pulling prebuilt GHCR images      │
  │  • Thin caller workflows + customizable .github/workflows/ci.yaml   │
  │  • renovate.json extending github>rdurica/php-platform//renovate-preset
  └─────────────────────────────────────────────────────────────────────┘
```

---

## Base Docker Images (GHCR)

Published on every `v*.*.*` tag via [publish-images.yaml](.github/workflows/publish-images.yaml):

| Image | Base / Contents | Intended Use |
| --- | --- | --- |
| `ghcr.io/rdurica/php-platform/prod:<ver>` | FrankenPHP 1.12 (`php8.5-alpine`), PHP production INI (`expose_php = Off`), OPcache + JIT, hardened `Caddyfile` (`admin off`), `docker-entrypoint.sh` (`APP_NAME` aware), `robbyte` user, `HEALTHCHECK` (no Composer) | Final runtime stage in app `build/prod/Dockerfile` |
| `ghcr.io/rdurica/php-platform/ci:<ver>` | FrankenPHP 1.12 (`php8.5-alpine`), PHP extensions (`pdo_mysql`, `pdo_pgsql`, `zip`, `intl`, `bcmath`, `gd`, `sockets`), `git`, Composer 2 | `vendor` build stage in prod Dockerfiles & fast CI tests |
| `ghcr.io/rdurica/php-platform/dev:<ver>` | Everything in `ci` + Xdebug, Node.js/npm, OpenSSH, Symfony CLI, Laravel installer, baked dev `Caddyfile` (auto HTTPS + Vite proxy) | Local development via `compose.yaml` |

### Tagging Policy

| Tag format | Example | Behavior |
| --- | --- | --- |
| Exact SemVer | `1.0.0` | Immutable pin (recommended in apps for reproducible builds + Renovate PRs) |
| Minor | `1.0` | Moves with patch releases (`1.0.x`) |
| Major | `1` | Moves with minor & patch releases (`1.x.x`) |
| Latest | `latest` | Latest published build |

---

## Reusable Workflows

All workflows support exact version pinning (`@v1.0.0`, recommended with Renovate) or rolling major pinning (`@v1`).

### 1. Code Quality (`reusable-code-quality.yaml`)

Validates Compose files, runs frontend checks (`typecheck`, `lint`, `format:check` when `src/package.json` exists), and runs backend checks (`composer validate --strict`, PHPStan, PHP CS Fixer, `composer audit` when `src/composer.json` exists).

```yaml
jobs:
  quality:
    uses: rdurica/php-platform/.github/workflows/reusable-code-quality.yaml@v1.0.0
```

| Input | Default | Description |
| --- | --- | --- |
| `node-version` | `'22'` | Node.js version for frontend checks |
| `compose-ci-file` | `'compose.ci.yaml'` | CI Compose file path |
| `run-compose-config` | `true` | Validate `compose.yaml` and CI Compose file |
| `run-frontend` | `true` | Run npm checks if `src/package.json` exists |
| `run-backend` | `true` | Run Composer/PHPStan/CS Fixer checks |

### 2. CI Checks (`reusable-ci-checks.yaml`)

Handles boilerplate CI validation (`docker compose config` and Hadolint on `build/prod/Dockerfile`) while leaving your app's `tests` job directly inside `.github/workflows/ci.yaml` so you can freely customize database services, fixtures, or test commands per app.

```yaml
jobs:
  checks:
    uses: rdurica/php-platform/.github/workflows/reusable-ci-checks.yaml@v1.0.0
```

| Input | Default | Description |
| --- | --- | --- |
| `dockerfile` | `'build/prod/Dockerfile'` | Dockerfile path to lint with Hadolint |
| `compose-ci-file` | `'compose.ci.yaml'` | CI Compose file path |
| `hadolint-ignore` | `'DL3018,DL4006,SC2028'` | Comma-separated Hadolint rules to ignore |

### 3. Production Image Build (`reusable-build.yaml`)

Multi-stage production build, GHCR push, and Trivy vulnerability scan with SARIF upload.

```yaml
jobs:
  build:
    permissions:
      contents: read
      packages: write
      security-events: write
    uses: rdurica/php-platform/.github/workflows/reusable-build.yaml@v1.0.0
```

| Input | Default | Description |
| --- | --- | --- |
| `dockerfile` | `'build/prod/Dockerfile'` | Production Dockerfile path |
| `context` | `'.'` | Docker build context |
| `registry` | `'ghcr.io'` | Target container registry |
| `push-latest` | `true` | Also push `:latest` tag |
| `trivy-scan` | `true` | Scan built digest with Trivy and upload SARIF |

---

## Renovate Preset (`renovate-preset.json`)

Applications inherit the shared preset via `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>rdurica/php-platform//renovate-preset"]
}
```

Behavior:
- Groups `ghcr.io/rdurica/php-platform/*` Docker image updates and `rdurica/php-platform` reusable workflow tag updates into a single **`php-platform`** PR per release.
- Enables `automerge` (when CI passes) for **`minor`**, **`patch`**, and **`digest`** updates.
- Keeps **`major`** updates (e.g., `v1` → `v2` / new PHP major versions) manual for explicit review.

---

## Releasing a Platform Update

1. Make changes in `docker/` or `.github/workflows/`.
2. Open a PR or push to `main` — `Platform CI` lints and smoke-tests all 3 images (`prod`, `ci`, `dev`).
3. Create and push a SemVer tag (e.g., `git tag -a v1.0.1 -m "v1.0.1" && git push origin v1.0.1`).
4. `Publish Base Images` builds and pushes `1.0.1`, `1.0`, `1`, and `latest` for `prod`, `ci`, and `dev` to GHCR, scans each image with Trivy, and updates the floating `v1` git tag.
5. Renovate propagates `1.0.1` across all connected application repositories.

---

## Related Repositories

- **[rdurica/php-starter-kit](https://github.com/rdurica/php-starter-kit)** — Template repository for starting new Laravel, Symfony, or Nette applications on top of `php-platform`.
- **[rdurica/php_starter_kit](https://github.com/rdurica/php_starter_kit)** — Legacy monolithic starter kit (unsupported).

## License

This project is licensed under the terms of the [MIT license](LICENSE).

# AGENTS.md

## About the project
docker-debian-ansible is a Debian (11–12) image with Ansible pre-installed,
designed for testing playbooks and roles. All versions live in a single repository
and are published under different tags to Docker Hub and GHCR.

## Repository structure
- `debian<version>/` — separate build context: `Dockerfile`, `initctl_faker`,
  `.dockerignore`.
- `.github/workflows/`:
  - `ci.yml` — checks on PR/push to master; calls reusable `test-image.yml`.
  - `build.yml` — manual build check across all platforms (`workflow_dispatch`);
    calls reusable `build-image.yml` with `push: false`.
  - `release.yml` — publishing on tags: `validate-tag` → `config` → `test` → matrix
    `release` (reusable `build-image.yml`, `secrets: inherit`).
  - `config.yml` — reusable workflow, single source of constants: outputs
    `versions` (JSON array), `platforms` (JSON array), `latest_version` (string);
    callers drive their matrices from `needs.config.outputs.*`.
  - `test-image.yml`, `build-image.yml` — reusable workflows with shared steps.
- `Makefile` + `scripts/test.mk` — `make test-build` smoke test mirroring
  `test-image.yml`; the version is set via the `version_branch` variable (default:
  the latest supported version).
- `.context/optimize-ci.md` — CI/CD architecture plan and rationale (read before
  changing the workflows).
- `.gitignore` — files kept out of git (`.context`, `.omo`).

## Build and test
- Local build: `cd debian12 && docker build -t docker-debian-ansible .`
- Smoke test, as in CI (`test-image.yml`):
  1. `podman build -t docker-debian-ansible:<version> ./debian<version>/.`
  2. `podman run --name test-container --detach --privileged \
     --volume /sys/fs/cgroup:/sys/fs/cgroup:ro docker-debian-ansible:<version>`
  3. `podman exec --tty test-container env TERM=xterm ansible --version`
- Or via the Makefile: `make test-build` (override the version with
  `make test-build version_branch=11`).

## CI/CD conventions
- Version matrix: `11, 12`.
- README lists version tags in descending order (newest first) — both in `Tags`
  and in `Archive tags`.
- Versions, platforms, and the `latest` tag mapping live in `config.yml`
  (outputs `versions`, `platforms`, `latest_version`) — edit it to change the
  build/test matrix; no other workflow needs editing for a version change.
- `10` is archived — no longer built or published (see README, "Archive tags").
- Platforms are defined per version in `config.yml` (limited by the architectures
  the `debian` base image publishes): 11 → `linux/amd64`, `linux/arm/v7`,
  `linux/arm64/v8`; 12 → the same plus `linux/ppc64le`.
- Release tag: `x.x` or `x.x.x`, optional `v` prefix
  (regex `^v?[0-9]{1,2}\.[0-9]{1,2}(\.[0-9]{1,2})?$`), no suffixes — enforced by
  the `validate-tag` job.
- `GITHUB_TOKEN` permissions are minimal: workflow-level `contents: read`.
  Registry-capable callers grant `packages: write` on their calling job
  (`release` in `release.yml`, `test-build` in `build.yml`), and `build-image.yml`
  requests it on the `build` job. Note: a called workflow's token is the
  *intersection* of the caller job's and the called job's permissions.
- Images are published to Docker Hub (`kirillzak/docker-debian-ansible`) and GHCR
  (`ghcr.io/<owner>/<repo>`).

## Commit message conventions
Use Conventional Commits:

- New feature: `feat: <description>`
- Bug fix: `fix: <description>`

Guidelines:

- The subject (first line) should be as short as possible; aim for ≤72 characters
  (GitHub truncates longer subjects with an ellipsis).
- Hard limit: the entire commit message (subject + optional body) must not exceed
  140 characters.
- If more context is needed, add a body separated by a blank line; keep the total
  message within the 140-character limit.
- Other Conventional Commits types may be used when `feat`/`fix` do not fit:
  `docs:`, `refactor:`, `chore:`, `ci:`, `test:`.

## Branch workflow
- After finishing the work on a branch, creating the PR, and passing all checks,
  merge the PR with the branch-deletion option so the working branch is removed
  from the GitHub repository.

## How to add a new Debian version
1. Create `debian<version>/` with `Dockerfile`, `initctl_faker`, `.dockerignore`.
2. Add `"<version>"` to the `versions` output in `.github/workflows/config.yml`
   (and update `latest_version` there if the new version becomes `latest`); add the
   tag to README.
3. If the new version is the latest, update the `version_branch` default in
   `scripts/test.mk`.
4. Run `workflow_dispatch` on `build.yml` and run the local smoke test
   (`make test-build`).
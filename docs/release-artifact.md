# Release artifacts

This repo now ships three bounded release artifacts.

Each artifact has a different job:
- source template
- runnable bootstrap-download package
- runnable preloaded package

Runtime and preloaded artifacts are platform-specific because the `pkg` wrapper executable is built for the current runner platform.

## Source template

Artifact:
- `service-lasso-app-packager-pkg-<version>-source.tar.gz`

Purpose:
- give downstream teams a downloadable starter repo shape
- keep tracked `services/` metadata in the template repo itself

What ships:
- `.gitignore`
- `.npmrc`
- `.github/`
- `README.md`
- `package.json`
- `package-lock.json`
- `src/`
- `services/`
- `docs/`
- `scripts/`
- `tests/`
- generated `release-artifact.json`

Honest label:
- **starter-template source artifact**

## Runnable bootstrap-download package

Artifact:
- `service-lasso-app-packager-pkg-<version>-runtime-<platform>.tar.gz`

Purpose:
- provide a ready-to-run Node host with the core runtime already installed
- include bundled Service Admin assets for the host shell
- include a `pkg` launcher executable at the artifact root
- keep the canonical repo-owned `services/` inventory inside the artifact
- prove that Echo Service is acquired from manifest-owned release metadata before use

What ships:
- `.npmrc`
- `README.md`
- `package.json`
- `package-lock.json`
- `src/`
- `services/`
- `docs/`
- installed `node_modules/`
- bundled admin assets under `.payload/admin/`
- bundled Node runtime under `node-runtime/`
- packaged wrapper executable:
  - `service-lasso-app-packager-pkg.exe` on Windows
  - `service-lasso-app-packager-pkg` on POSIX
- generated `release-artifact.json`

How it works:
- the app repo owns `services/echo-service/service.json`
- that manifest carries the bounded `artifact` block pointing at the Echo Service release assets
- on `install`, Service Lasso downloads and unpacks the matching archive from the manifest metadata
- the app artifact itself does not ship the Echo Service archive preloaded
- the `pkg` launcher loads the colocated app payload and boots the same host/runtime flow as `service-lasso-app-node`

Honest label:
- **runnable bootstrap-download package**

## Runnable preloaded package

Artifact:
- `service-lasso-app-packager-pkg-<version>-preloaded-<platform>.tar.gz`

Purpose:
- provide a ready-to-run Node host with the core runtime already installed
- include bundled Service Admin assets for the host shell
- include a `pkg` launcher executable at the artifact root
- keep the canonical repo-owned `services/` inventory inside the artifact
- prove that the Echo Service archive is already present before first install/use

What ships:
- everything in the runnable bootstrap-download package
- preseeded Echo Service archive under:
  - `.workspace/services/echo-service/.state/artifacts/<releaseTag>/<assetName>`

How it works:
- the app repo still owns the same canonical `services/echo-service/service.json`
- the preloaded artifact seeds the matching archive into the runtime-owned service state before first use
- on `install`, Service Lasso reuses that archive and skips the network fetch

Honest label:
- **runnable preloaded package**

## What the release proves

The release now proves:
- the repo owns explicit tracked service metadata under `services/`
- the `pkg` wrapper can launch the packaged Node host payload repeatably
- the runnable artifact can boot Service Lasso and Service Admin without sibling-repo checkout tricks
- Echo Service acquisition depends on manifest-owned archive metadata instead of a generated local wrapper
- bootstrap-download mode installs the service payload before first use
- preloaded mode installs from an already-shipped archive without a first-run download

## Private package note

This starter depends on the published private core package:
- `@service-lasso/service-lasso`

For local or CI installs from GitHub Packages, provide a token with package read access through:
- `NODE_AUTH_TOKEN`

The repo `.npmrc` is included in the staged artifact so the starter knows which registry and scope to use.

## Commands

Build the local `pkg` wrapper:

```bash
npm run package:pkg
```

Stage the release artifacts:

```bash
npm run release:artifact
```

Stage and verify the release artifacts:

```bash
npm run release:verify
```

## Current expectation

Any application using Service Lasso should keep a tracked `services/` folder in its repo with the service metadata it intends to manage.

This app-packager-pkg starter uses that tracked inventory directly and adds a bounded `pkg` launcher on top of the canonical app-node payload model.

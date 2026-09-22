This repo contains a Github Action workflow for auditing Helix editor source code and another workflow for both auditing the source and reproducing attested Helix builds.
The workflows were written with the help of Github Copilot and were manually reviewed and heavily modified to suit my needs and discard scraps.

# Motivation
I initially decided to make this effort due to the absence of security audit of the source code of many open-source projects and the lack of provenance for the build artifacts. So, I wanted to ascertain people of both the security and the integrity of the OSS build they intend to use, and Helix editor was the starting point I chose.


# Rationale
The official v25.07.1 release of Helix editor has verified tag and commit (were signed with the committer [@the-mikedavis](https://github.com/the-mikedavis)'s signature), and the release is also published by Github Action according to the defined workflows. Here is the evidence:
- The workflow present at the `25.07.1` tag explicitly generates the source archive named `helix-25.07.1-source.tar.xz`contained in the release:
	```bash
	tar cJf dist/helix-$tag-source.tar.xz -C $source .
    ```
    This is in the `publish` job of [`release.yml`](https://github.com/helix-editor/helix/blob/a05c151bb6e8e9c65ec390b0ae2afe7a5efd619b/.github/workflows/release.yml#L234-L285).
- The same workflow then uploads all files in `dist/` to the GitHub release using:
	```yml
	- name: Upload binaries to release
	  uses: svenstaro/upload-release-action@v2
	  ...
	  file: dist/*
	  file_glob: true
	  tag: ${{ github.ref_name }}
	  overwrite: true
	```
 This is in the `publish` job of [`release.yml`](https://github.com/helix-editor/helix/blob/a05c151bb6e8e9c65ec390b0ae2afe7a5efd619b/.github/workflows/release.yml#L288-L296)
- The release itself lists `github-actions` as its author, which is consistent with the automated release workflow, see [Release 25.07.1 · helix-editor/helix](https://github.com/helix-editor/helix/releases/tag/25.07.1)
- The `25.07.1` tag points to commit [`a05c151`](https://github.com/helix-editor/helix/commit/a05c151bb6e8e9c65ec390b0ae2afe7a5efd619b), whose release changes and workflow are consistent with the published patch release.
- The timestamps of the assets coincide with the workflow/release publication time on July 18, 2025 at 22:13 (GMT+7) which proves the assets were not manually replaced or modified later.

Those things serve as build provenance by themselves (that the binaries were built from the source and point to the signed tag and commit, despite of no Github Release Attestations or Artifact Attestations) and there's no need to reproduce the binaries for SHA256 digest matching. So now, that we're certain about the integrity of the release's tag, commit, source, and builds, we are left with the need to audit the source files; using GitHub's CodeQL Rust security-extended scan against the codebase, GitHub's CodeQL Action scan against the Github Action workflow, and using RustSec's `cargo audit` against the dependencies. However, `cargo audit` checks for vulnerabilities in the current RustSec advisory database, so we'll just treat the alerts as informational since it is completely legitimate/reasonable if the scan found historical vulnerabilities in the dependencies that was not discovered before the release.

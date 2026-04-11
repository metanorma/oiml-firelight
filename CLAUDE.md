# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Deployment repository for the Firelight Metanorma interactive viewer. Contains presentation XML collections and a GitHub Actions workflow that builds and deploys them to GitHub Pages using the Firelight (anafero-cli) build tool.

## Deployment Workflow

`.github/workflows/deploy.yml` triggers on push to `main`, PRs, and `workflow_dispatch`. For each collection in the matrix:

1. Creates a fresh git repo with the collection's presentation XML
2. Creates `anafero-config.json` pointing to Firelight packages on GitHub
3. Runs `npx @riboseinc/anafero-cli@0.0.70 build-site` with `--path-prefix /oiml-firelight/<collection-name>`
4. Uploads artifact preserving subdirectory structure
5. Deploy job merges artifacts, creates landing page, deploys to GitHub Pages

## Critical: Path Prefix

Firelight `--path-prefix` must have a leading `/` and NO trailing `/`. Each collection needs its own prefix including the collection name. The prefix is set via `data-path-prefix` on the `<html>` element.

## Critical: Artifact Structure

`actions/upload-artifact@v4` strips the `path` prefix from uploaded files. Use `path: dist` (not `path: dist/r060`) to preserve subdirectory structure. Combine with `merge-multiple: true` on download.

## Collections

Collections use either `collection.presentation.xml` (multi-document) or `document.presentation.xml` (single document). The matrix parameterizes `xml` for this. New collections must have their presentation XML committed to git.

Landing page: `https://metanorma.github.io/oiml-firelight/`

## Notes

- Anafero CLI requires a git repo with the XML committed (uses git for content addressing)
- Use `rm -rf .git && git init .` to create a fresh repo in CI (avoid conflicts with checkout)
- `--experimental-vm-modules` node flag is required for anafero-cli
- The Firelight source is at https://github.com/metanorma/firelight
- Firelight loads content adapters from `git+https://github.com/metanorma/firelight#main` at build time, so fixes to that repo take effect immediately
- Known Firelight console warning: `Non-slash-prepended path after getSiteRootRelativePath` — cosmetic, the viewer still works

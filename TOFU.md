# Tofu gallery copy

This repository is Tofu's copy of **storytold/photocraft** (licence: Apache-2.0), listed in Tofu's app gallery as **PhotoCraft**.

- `main` mirrors the upstream repository and is never committed to by Tofu.
- `tofu` carries Tofu's own changes; tags named `pin/<date>` mark the versions the gallery points to.
- A visitor who chooses this app gets a private snapshot of the pinned version in their own Tofu account. Their copy does not follow upstream updates.

How Tofu runs it: upstream is a Rust project. Tofu cannot build it, so this copy ships PhotoCraft's published web build rather than the Rust sources.

## What Tofu changes

Nothing yet.

## Licence

Apache-2.0. Licence files and copyright notices stay as they are; every file Tofu changes carries a note saying so.

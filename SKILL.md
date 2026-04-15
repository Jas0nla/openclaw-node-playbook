# OpenClaw Node Playbook

Use this skill when working on Jason's OpenClaw gateway/node topology, especially:

- Raspberry Pi gateway in Docker
- macOS node attachment
- SSH tunnel-backed node routing
- OpenClaw `exec` failures after pairing appears healthy

## Scope

This skill is for:

- checking gateway/node deployment state
- aligning `tools.exec.*` and approvals
- diagnosing why node `exec` fails even when the node is connected
- steering browser work away from unsafe shell-wrapper forms

## Core facts from this environment

- Pi gateway is Docker-based
- UI update is not the primary upgrade path when image tags are pinned in host `.env`
- `jason-mac` can pair and connect successfully while node `exec` still fails
- OpenClaw runtime may deny interpreter/runtime wrapper commands even after approvals are opened
- Node browser routing also depends on the node-side `browser` plugin being allowed locally

## High-value workflow

1. Confirm node pairing and connection first.
2. Confirm gateway `tools.exec.host/security/ask/node`.
3. Confirm gateway and node approvals files.
4. If failures persist, check whether the command is an interpreter/runtime wrapper.
5. Avoid `sh -lc` / `bash -lc` / similar wrappers on node exec.
6. Prefer direct executable forms first.
7. For browser issues, confirm the node local config still allows the `browser` plugin.
8. Prefer node-native capabilities such as `system.which` and `browser.proxy`.

## Practical constraints

- Do not treat every `approval required` as an approvals-file problem.
- `approval cannot safely bind this interpreter/runtime command` is a runtime safe-binding failure.
- A minimal direct command like `id` may work while a shell-wrapped command fails.
- Complex browser/system workflows should move toward `browser.proxy` rather than larger shell blobs.
- `browser control disabled` can come from the node's `plugins.allow` filtering out `browser`, even if the gateway/browser config looks correct.

## Preferred next step for browser automation

If the goal is browser work on the node:

- use `browser.proxy` before trying to compose shell-based `exec`
- if gateway browser routing fails, inspect the node's local `plugins.allow` and `plugins.entries.browser.enabled` first
